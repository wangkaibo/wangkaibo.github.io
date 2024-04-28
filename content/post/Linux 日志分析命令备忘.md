---
title: "Linux 日志分析命令备忘"
tags: ["Linux"]
date: 2017-02-08T10:49:55+08:00
url: "/linux-log-file-statistic-commands"
draft: false
---

[参考 ](http://xuqq999.blog.51cto.com/3357083/774714)

#### 查看日志中访问次数最多的前10个IP
```
cat access.log | cut -d ' ' -f 1 | awk '{print $0}' | uniq -c | sort -nr | head -n 100
```
<!--more-->

* awk：太强大，一两句说不清楚
* uniq -c: 数据去重，-c 的意思是计数（--count）
* sort -nr： 排序，-n 按照数字排序(--numeric-sort)，-r 倒序排序(--reverse)
* head -n 100： -n, --lines=[-]K
#### 当前WEB服务器中联接次数最多的ip地址
```
netstat -ntu |awk '{print $5}' |sort | uniq -c| sort -nr
```
#### 查看日志中出现100次以上的IP，并倒序显示
``` 
cat access.log | cut -d ' ' -f 1 | awk '{print $0}' | uniq -c | sort -nr | awk '{if ($1 > 10) print $
```