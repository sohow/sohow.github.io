---
title: redis源码阅读之双端双向链表adlist
date: 2017-06-05 14:19:11
tags: redis
---
## 双端双向链表
* 双链表优点是便于逆向查找
* 双端优点是便于插入尾部

### adlist结构图
![](/images/redis_adlist.png)

### adlist结构体
``` shell
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```
