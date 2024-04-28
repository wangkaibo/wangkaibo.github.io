---
title: "VirtualBox CentOS（minimal）配置网络访问外网支持并 SSH"
date: 2015-10-08T00:24:17+08:00
tags: ["linux"]
draft: false
url: '/virtualbox-centos-minimal-network-config-and-ssh-login'
---

今天安装了一台 Centos 6.8 minimal 做测试使用，安装后，默认网卡是没有开机启动的。使用的是桥接模式，这种模式是通过宿主机的虚拟路由器将宿主机和虚拟机连接到一起。

<!--more-->

首先改下配置 eth-0 的配置

`vi /etc/sysconfig/network-scripts/ifcfg-eth0`：

```
...
ONBOOT=yes
...
```

手动重启网络 `services network restart`

现在还是 ping 不同外网，还需要改一下 DNS 的配置：

`vi /etc/resolv.conf`

```
...
nameserver 8.8.8.8
# 或者 114.114.114.114
```

保存后在重启下网络，现在发现可以 `PING` 通外网了。

如果需要手动配置静态IP。可以使用如下配置：

```
...
BOOTPROTO="static"
IPADDR=192.168.0.103
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
BROADCAST=192.168.0.255
DNS1=114.114.114.114
```

修改配置后重启后生效。

如果需要使用 ssh 登录，有时候还需要禁用下 IPV6，否则可能出现 ssh 无法连接的情况。