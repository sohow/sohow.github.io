---
title: redis源码阅读之字典dict
date: 2017-06-06 13:49:44
tags: redis
---
## 数据结构
![](/images/redis_dict.png)

``` c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;

typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;

typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;

/* If safe is set to 1 this is a safe iterator, that means, you can call
 * dictAdd, dictFind, and other functions against the dictionary even while
 * iterating. Otherwise it is a non safe iterator, and only dictNext()
 * should be called while iterating. */
typedef struct dictIterator {
    dict *d;
    long index;
    int table, safe;
    dictEntry *entry, *nextEntry;
    /* unsafe iterator fingerprint for misuse detection. */
    long long fingerprint;
} dictIterator;
```

## hash函数
``` c
/* The default hashing function uses SipHash implementation
 * in siphash.c. */

uint64_t siphash(const uint8_t *in, const size_t inlen, const uint8_t *k);
uint64_t siphash_nocase(const uint8_t *in, const size_t inlen, const uint8_t *k);
```

## 解决hash冲突
* 拉链法解决冲突

## 解决拉链法解决冲突的弊端
redis使用的是渐进式重散列(incremental rehashing)
* 重散列
创建一个更大的hash表，把原来旧的数据一次性全部重新hash到新的表上。
* 渐进式重散列
创建一个更大的hash表，把原来旧的数据分步(如get或者set操作时，每次都只重新hash1个)重新hash到新的表上。
如上图dict中有两个hash tabe, 其中ht[1]就是在重散列时用到的。

## 渐进式重散列的时机
* get
* set
* delete
* ...

## 反向递增二进制扫描器(dictScan)
* 正常的游标递增是从1,2,3,4,5 ... 自然数加1递增的, 用二进制描述是从最低位加1, 若溢出则向高位进位。
``` shell 
000 001 010 011 100 101 110 111
```
* 而dictScan的游标递增方式比较特别， 用二进制描述是从最高位加1, 若溢出则向低位进位。
``` shell 
000 100 010 110 001 101 011 111
```
* dictScan采用这样特殊的游标递增方式是为了避免在扩容重散列时导致数据读取重复问题和缩容重散列时导致数据读取遗漏。
* dictScan也不是完美的，其在扩容重散列时没有问题，但在缩容重散列时有可能会导致读取到少量重复数据。

### 扩容和缩容前后游标对应关系图
![](/images/redis_dict_scan.png)
* 上图是扩容，通俗点理解原来一个变成了两个，由于原来的到新的没有重合的，所以没有重复读取的情况
* 下图是缩容，通俗点理解原来两个变成了一个，由于原来到新的是合二为一有重合，所以有可能会出现重复读取的情况，但这也是为了不遗漏数据所必要的，
没有重复的情况是当下一个应读取下一组时就不会读取到重复数据。 另外，重复bucket的数量=(bigger_table_size / smaller_table_size) - 1

