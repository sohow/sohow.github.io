---
title: redis源码阅读之动态字符串sds
date: 2017-05-31 16:50:31
tags: redis
---
## sds优点
* 计算字符串长度时间复杂度由O(n)降低到O(1)
* 额外分配内存减少频繁的内存分配复制开销

## sds内存分布
图示为SDS_TYPE_5以上类型
![](/images/redis_sds_mem.png)
