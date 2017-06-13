---
title: 跳跃表原理C语言实现
date: 2017-06-13 15:41:53
tags: [C语言,算法]
---
## 跳跃表和传统链表图示
* 上图是传统链表，下图是跳跃表图示
* 可以简单理解跳跃表是在传统顺序链表上通过给节点增加额外（随机个数）指针来做索引，
其思想类似于折半查询的思想，其效率可以媲美红黑树。
![](/images/skip_list.png)

## 跳跃表C语言实现
``` C
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

#define MAX_L 10
typedef int typekey;
typedef int typevalue;

typedef struct Node
{
    typekey key;
    typevalue val;
    int level;
    struct Node * next[1];
} Node;

typedef struct Skiplist
{
    int level;
    Node * header;
} Skiplist;


int randomLevel()
{
    int level=1;
    while (rand()%2) {
        level++;
    }
    level= (MAX_L > level) ? level : MAX_L;
    return level;
}

Node * create_node(int level)
{
    return (Node *)malloc(sizeof(Node) + (level -1)*sizeof(Node*));
}

Skiplist * create_skiplist()
{
    int i;
    Skiplist *p = (Skiplist *)malloc(sizeof(Skiplist));
    p->level = 0;
    p->header = create_node(MAX_L);
    for (i = 0; i < MAX_L; i++) {
        p->header->next[i] = NULL;
    }

    return p;
}


typevalue search(Skiplist *list, typekey key)
{
    Node *q, *p = list->header;
    for (int i = list->level - 1; i >= 0 ; --i) {
        while ((q = p->next[i]) && q->key < key) {
            p = q;
        }
        if (q && q->key == key) {
            return q->val;
        }
    }
    return NULL;
}

void insert(Skiplist * list, typekey key, typevalue val)
{
    int level = randomLevel();
    Node *pnode = NULL;

    list->level = list->level < level ? level : list->level;
    Node *q, *p = list->header;
    for (int i = list->level - 1; i >= 0 ; --i) {
        while ((q = p->next[i]) && q->key < key) {
            p = q;
        }
        ;注意可插入相同key的节点
        if (i <= level -1) {
            if (pnode == NULL) {
                pnode = create_node(level);
                pnode->key = key;
                pnode->val = val;
                pnode->level = level;
            }
            p->next[i] = pnode;
            pnode->next[i] = q;
        }
    }
}

void delete_node(Skiplist *list, typekey key)
{
    Node *q, *p = list->header;
    Node *pnode = NULL;
    ;若有相同key一次只删除一个
    for (int i = list->level - 1; i >= 0 ; --i) {
        while ((q = p->next[i]) && q->key < key) {
            p = q;
        }
        if (q == NULL && p == list->header) {
            list->level--;
        }
        if (q && q->key == key) {
            pnode = q;
            p->next[i] = q->next[i];
        }
    }
    if (pnode) {
        free(pnode);
    }
}

void delete_list(Skiplist *list)
{
    Node *p = list->header;
    while ((p = p->next[0])) {
        free(p);
    }
    free(list->header);
    free(list);
}

int main()
{
    Skiplist *list = create_skiplist();
    int i;
    int key;
    for (i = 0; i < 100; i++) {
        key = rand() % 100 + 1;
        insert(list, key, key);
        printf("%d ", key);
    }
    printf("\n");

    typevalue val;
    char *s;
    Node *p = list->header;
    while ((p = p->next[0])) {
        val = search(list, p->key);
        s = val == p->val ? "OK" : "FAILED";
        printf("search %s 0x%d val: %d level: %d", s, (int) p, p->val, p->level);
        printf(" (");
        for (int j = 0; j < p->level; ++j) {
            printf("0x%d ", (int) (p->next[j]));
        }
        printf(")\n");
    }

    delete_list(list);

    return 0;
}
```
## 检查内存溢出
``` shell
gcc -Wall main.c -g -o test
valgrind --tool=memcheck --leak-check=full ./test
```