---
title: C语言实现Trie字典树支持中文
date: 2017-03-20 11:00:38
tags: [C语言,算法]
---
C语言实现Trie字典树，支持中文，支持删除节点
``` C
#include <stdio.h>
#include <wchar.h>
#include <locale.h>
#include <stdlib.h>
#include <memory.h>

#define DEFAULT_CHILD_SIZE 4
#define MAX_KEY_LENGTH 11
#define MAX_KEYS_LENGTH 100


typedef struct Node {
    wchar_t ch;
    _Bool is_end;
    struct Node **child;
    int child_cur_length;
    int child_total_length;
} Node;

typedef struct CHS {
    wchar_t chs[MAX_KEYS_LENGTH][MAX_KEY_LENGTH];
    int length;
} CHS;

Node* search_trie_child(Node *node, wchar_t ch)
{
    int i = 0;
    Node *pnode = NULL;
    Node *ptmp = NULL;
    for (i = 0; i < node->child_cur_length; i++) {
        ptmp = node->child[i];
        if (ptmp->ch == ch) {
            pnode = ptmp;
            break;
        }
    }
    return pnode;
}

Node* init_node(wchar_t ch)
{
    Node *pnode = (Node *)malloc(sizeof(Node));
    pnode->ch = ch;
    pnode->is_end = 0;
    pnode->child_cur_length = 0;
    pnode->child_total_length = DEFAULT_CHILD_SIZE;
    pnode->child = (Node**)malloc(pnode->child_total_length * sizeof(Node *));
    return pnode;
}

void free_node(Node *pnode)
{
    free(pnode->child);
    free(pnode);
}

Node* add_trie_child(Node *node, wchar_t ch)
{
    Node *pnode = init_node(ch);
    if (node->child_cur_length >= node->child_total_length) {
        node->child_total_length = node->child_total_length * 2;
        Node **tmp = (Node**)malloc(node->child_total_length * sizeof(Node *));
        memcpy(tmp, node->child, node->child_cur_length * sizeof(Node *));
        free(node->child);
        node->child = tmp;
    }
    node->child[node->child_cur_length++] = pnode;
    return pnode;
}

_Bool delete_trie_word(Node *trie, wchar_t *str, int cur)
{
    Node *pnode;

    if (str[cur] == '\0' && trie->is_end) {
        //匹配到了最后一个字符且是結束节点
        trie->is_end = 0;
        return 1;
    }

    if ( str[cur] == '\0' || ( pnode = search_trie_child(trie, str[cur]) ) == NULL ) {
        return 0;
    }

    if (delete_trie_word(pnode, str, cur + 1)) {
        if (pnode->child_cur_length == 0) {
            int i;
            //删除节点指针，并且把最后一个节点指针的位置调整到删除节点指针的位置
            for (i = 0; i < trie->child_cur_length; i++) {
                if (pnode == trie->child[i]) {
                    trie->child[i] = trie->child[trie->child_cur_length - 1];
                    trie->child[trie->child_cur_length - 1] = 0;
                    trie->child_cur_length--;
                    free_node(pnode);
                    break;
                }
            }
        }
        if (trie->child_cur_length == 0 && !trie->is_end) {
            return 1;
        }
    }

    return 0;
}

int add_trie(Node *trie, wchar_t *str)
{
    int len = (int)wcslen(str);
    int i = 0;
    Node *pnode = NULL;
    if (len >= MAX_KEY_LENGTH) {
        wprintf(L"too long str: %ls\n", str);
        return 0;
    }

    for (i = 0; i < len; i++) {
        if ( (pnode = search_trie_child(trie, str[i])) == NULL ) {
            pnode = add_trie_child(trie, str[i]);
        }
        trie = pnode;
    }
    trie->is_end = 1;

    return 0;
}

void delete_trie(Node *trie)
{
    int i;
    for (i = 0; i < trie->child_cur_length; i++) {
        delete_trie(trie->child[i]);
    }
    free_node(trie);
}

CHS* search_trie(Node *trie, wchar_t *str)
{
    int len = (int)wcslen(str);
    int i = 0;
    int j = 0;
    int k = 0;
    Node *pnode = NULL;
    Node *ptrie = NULL;
    CHS *pchs = (CHS *)malloc(sizeof(CHS));
    pchs->length = 0;
    memset(pchs, 0, sizeof(CHS));

    for (i = 0; i < len; i++) {
        ptrie = trie;
        for (j = i, k = 0; j < len; j++) {
            if ( (pnode = search_trie_child(ptrie, str[j])) == NULL ) {
                break;
            } else {
                ptrie = pnode;
                pchs->chs[pchs->length][k++] = str[j];
            }
            if (pnode->is_end) {
                pchs->chs[pchs->length][k] = '\0';
                pchs->length++;
                memcpy(&pchs->chs[pchs->length], pchs->chs[pchs->length-1], MAX_KEY_LENGTH);
            }
        }
    }

    return pchs;
}

void print_trie(CHS *pchs)
{
    int i;

    for (i = 0; i < pchs->length; i++) {
        wprintf(L"%ls\n", &pchs->chs[i]);
    }
}

int test()
{
    Node *trie = init_node(0);

    wchar_t *str1 = L"1你好";
    add_trie(trie, str1);

    wchar_t *str2 = L"2你好吗";
    add_trie(trie, str2);

    wchar_t *str3 = L"3你还好";
    add_trie(trie, str3);

    wchar_t *str4 = L"4你还好吗";
    add_trie(trie, str4);

    wchar_t *str5 = L"5你真的还好吗";
    add_trie(trie, str5);

    wchar_t *str6 = L"我好";
    add_trie(trie, str6);

    wchar_t *str7 = L"我好吗";
    add_trie(trie, str7);

    wchar_t *str8 = L"我还好";
    add_trie(trie, str8);

    wchar_t *str = L"cc你好我好你好吗";
    CHS *pchs = search_trie(trie, str);
    delete_trie_word(trie, str8, 0);
    delete_trie(trie);

    print_trie(pchs);
    free(pchs);

    return 0;
}

int main() 
{
    setlocale(LC_CTYPE, "");
    test();
    return 0;
}
```