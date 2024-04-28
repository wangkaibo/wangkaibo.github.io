---
title: "MySQL 分区遇到的小问题"
date: 2016-12-08T22:23:15+08:00
tags: ["MySQL"]
draft: false
url: '/mysql-fen-qu-yu-dao-de-xiao-wen-ti'
---

近期公司有个业务数据量已经很大了，我们考虑对数据库进行分区。本文做个小纪录

<!--more-->

### MySQL 分区功能简介
在 MySQL 5.1 之前，当一张表数据过多时，数据表的文件过大，严重影响到查询速度时，程序员们会将一张大表根据业务逻辑横向分割为多张表来达到数据在硬盘分开存放的目的。

MySQL 5.1 后支持的分区功能可以算是 MySQL 官方「分表」的一种解决方案。它的主要优点是业务代码改动极小。

判断 MySQL 是否开启了分区功能可以使用 `show variables like "%part%";` 命令查看：
```
mysql> show variables like "%part%";  
+-------------------+-------+  
| Variable_name     | Value |  
+-------------------+-------+  
| have_partitioning | YES   |  
+-------------------+-------+  
1 row in set (0.00 sec)  
```

### InnoDB 分区问题

Range 分区是最常见的分区方式
```
CREATE TABLE `xxxxx` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `uid` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '用户Id',
  `create_time` int(11) unsigned DEFAULT '0' COMMENT '创建时间',
  PRIMARY KEY (`id`,`uid`),
  KEY `uid` (`uid`)
) ENGINE=InnoDB AUTO_INCREMENT=69210 DEFAULT CHARSET=utf8 COMMENT='支持每日赠送鲜花表'
/*!50100 PARTITION BY RANGE (mod(uid,10))
(PARTITION sfg0 VALUES LESS THAN (1) ENGINE = InnoDB,
 PARTITION sfg1 VALUES LESS THAN (2) ENGINE = InnoDB,
 PARTITION sfg2 VALUES LESS THAN (3) ENGINE = InnoDB,
 PARTITION sfg3 VALUES LESS THAN (4) ENGINE = InnoDB,
 PARTITION sfg4 VALUES LESS THAN (5) ENGINE = InnoDB,
 PARTITION sfg5 VALUES LESS THAN (6) ENGINE = InnoDB,
 PARTITION sfg6 VALUES LESS THAN (7) ENGINE = InnoDB,
 PARTITION sfg7 VALUES LESS THAN (8) ENGINE = InnoDB,
 PARTITION sfg8 VALUES LESS THAN (9) ENGINE = InnoDB) */
```
测试环境使用以上 SQL 建表后，插入数据测试时发现，数据在硬盘上的并不是按照分区来存放的数据文件的，还是存了单个文件。但是将数据表引擎改为 MyISAM 后，是可以正常按照是个文件的分区方式存放数据。这应该和两种引擎的数据结构有关。

我查看了测试环境 MySQL 的配置文件 `innodb_file_per_table` 字段发现，测试环境数据库 InnoDB 引擎使用的默认的共享表空间的配置。同一个数据库下所有表数据都存储在 `data` 目录下 `ibdata1` 文件中，所以无法对它进行分区。久而久之 `ibdata1` 这个文件也会越来越大，也不利于数据表中的数据的转移和备份。

修改数据库配置文件，在 `my.cnf` 中 `[mysqld]` 下配置：
```
innodb_file_per_table=1
```

更改配置后创建一个 名为test的数据库测试分区后的数据是否存在了不同的文件中，如图：

![分区后存储](http://7xu5j5.com1.z0.glb.clouddn.com/mysql-fenqu.png)

10 个分区的数据文件的命名方式为表名#分区名.idb。

通过测试环境测试，没有改任何业务代码，数据库性能得到了一定的提升。可见 MySQL 分区是个低成本的解决方案。