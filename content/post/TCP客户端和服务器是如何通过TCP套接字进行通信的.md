---
title: "TCP客户端和服务器是如何通过TCP套接字进行通信的"
date: 2017-11-16T00:09:43+08:00
tags: ["tcp"]
draft: false
url: '/tcp-client-server-connection'
---

最近在补充HTTP/TCP协议的一些知识。看到《HTTP权威指南》中TCP连接时觉得下图描述的比之前看其他人写的博文要清楚的多。

<!--more-->

![TCP客户端和服务器是如何通过TCP套接字进行通信的](http://static.wangkaibo.com/FiQBi2N9YOQgXvhO_t6KMZNGLtgn)

图：TCP客户端和服务器是如何通过TCP套接字进行通信的



在发送数据前，TCP要穿送两个分组来建立连接，也就是传说中的三次握手。



![image-20180529140036784](http://static.wangkaibo.com/FvWkHp29uMmz9icxpxxGbLpK8DqS)

1. 第一次握手：建立连接时，客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；



2. 第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；



3. 第三次握手：客户端收到服务器的SYN＋ACK包，验证ack=j+1，并向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。完成三次握手，客户端与服务器开始传送数据。

