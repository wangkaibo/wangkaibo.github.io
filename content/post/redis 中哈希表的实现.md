---
title: "redis 中哈希表的实现"
date: 2018-03-15T23:49:03+08:00
tags: ["redis", "算法"]
draft: true
url: '/redis-hashtable'
---

一直对哈希表如何实现

字典所使用的哈希表由dict.h（[dict.c源码](https://github.com/antirez/redis/blob/unstable/src/dict.c)）和 dict.h（[dict.h源码](https://github.com/antirez/redis/blob/unstable/src/dict.h)） 实现。

下面分析

```c
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
```





