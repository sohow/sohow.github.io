---
title: redis源码阅读之跳跃表skiplist
date: 2017-06-15 15:19:11
tags: redis
---
## redis跳跃表插入
* span表示当前层到下一个节点的对应层的跨度，存储在当前节点
* rank[i]表示update[i]节点的排名
* update[i]表示i层的第一个要插入节点的前驱节点

``` shell
score: 2	 span: 1 1 1 1 		    @@@@
score: 26	 span: 1 1 1 1 1 7 	    @@@@@@
score: 27	 span: 1 1 1 1 6 	    @@@@@
score: 29	 span: 1 3 4 4 		    @@@@
score: 63	 span: 1 		    @
score: 67	 span: 1 		    @
score: 83	 span: 1 1 		    @@
score: 92	 span: 1 1 1 1 		    @@@@
score: 93	 span: 0 0 		    @@

rank: 2 update: 26
rank: 3 update: 27
rank: 4 update: 29
rank: 4 update: 29
rank: 4 update: 29
rank: 6 update: 67

score: 2	 span: 1 1 1 1 		    @@@@
score: 26	 span: 1 1 1 1 1 5 	    @@@@@@
score: 27	 span: 1 1 1 1 4 	    @@@@@
score: 29	 span: 1 3 3 3 		    @@@@
score: 63	 span: 1 		    @
score: 67	 span: 1 		    @
score: 69	 span: 1 1 2 2 3 3 3 3 	    @@@@@@@@
score: 83	 span: 1 1 		    @@
score: 92	 span: 1 1 1 1 		    @@@@
score: 93	 span: 0 0 		    @@

x->level[i].span = update[i]->level[i].span - (rank[0] - rank[i]);
3 = 7 - (6 - 2)
update[i]->level[i].span = (rank[0] - rank[i]) + 1;
5 = (6 - 2) + 1
```