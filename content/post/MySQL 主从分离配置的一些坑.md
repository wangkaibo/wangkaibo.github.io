---
title: "MySQL 主从分离配置的一些坑"
date: 2015-11-18T21:01:47+08:00
tags: ["mysql"]
draft: false
url: '/mysql-zhu-cong-fen-chi-pei-zhi-de-yi-xie-keng'
---

最近在多个虚拟机上做主从分离的实验，遇到的小问题，记录一下。

<!--more-->

### 开启 binlog 日志

修改 `my.cnf` 后 MySQL 无法启动，修改内容如下：
```
log-bin=mysql-bin
```

后来通过搜索相关资料发现 MySQL 5.7 以后开启binlog 日志需要配置 `server-id`。 

参见 [官方文档](https://dev.mysql.com/doc/refman/5.7/en/replication-options.html)：
>In MySQL 5.7.2 and earlier, if you start a master server without using --server-id to set its ID, the default ID is 0. In this case, the master refuses connections from all slaves, slaves refuse to connect to the master, and the server sets the server_id system variable to 1. In MySQL 5.7.3 and later, the --server-id must be used if binary logging is enabled, and a value of 0 is not changed by the server. If you specify --server-id without an argument, the effect is the same as using 0. In either case, if the server_id is 0, binary logging takes place, but slaves cannot connect to the master, nor can any other servers connect to it as slaves. (Bug #11763963, Bug #56718)
### Slave_IO_Running: Connecting 问题
主服务器配置：
```
grant replication slave on *.* to 'slave'@'192.168.206.191' identifie by 'XXX';

mysql> show master status;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| master-bin.000003 |      882 |              |                  |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
从服务器配置：
```
change master to
master_host='192.168.10.130',
master_user='xxx',
master_password='password',
master_log_file='mysql-bin.000005',
master_log_pos=261;
```


运行 `show slave status` 发现 Slave_IO_Running 一栏显示 Connecting。这种问题一般考虑以下几个问题：

* master、slave 一些配置信息，尤其注意 master_log_file、master_log_pos 


* 网络、防火墙、selinux 问题
* master、slave 服务器 server-id 需要不一致

### MySQL 的 server UUID

这个问题比较少见，多见于主从服务器使用的同一个系统快照时。

`show slave status` 会提示报错信息：

>Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.

MySQL 的 `datadir` 下 有个`auto.cnf` 文件，这个文件记录了 MySQL 服务器的UUID。

MySQL 的 data 目录可以使用 `show variables like "%dir%";` 查看。

MySQL 的可以把其中一台服务器的`auto.cnf` 删除，重启 MySQL 服务器后会再次生成新的不重复的 UUID 文件。