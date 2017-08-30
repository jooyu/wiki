---
layout: post
title: VIP
date: 2017-05-16 14:30:00
tags:
- docker
categories: Java
description: docker
---


# VIP
虚IP。何为虚IP那，就是一个未分配给真实主机的IP，也就是说对外提供数据库服务器的主机除了有一个真实IP外还有一个虚IP，使用这两个IP中的 任意一个都可以连接到这台主机，所有项目中数据库链接一项配置的都是这个虚IP，当服务器发生故障无法对外提供服务时，动态将这个虚IP切换到备用主机。

 

开始我也不明白这是怎么实现的，以为是软件动态改IP地址，其实不是这样，其实现原理主要是靠TCP/IP的ARP协议。因为ip地址只是一个逻辑 地址，在以太网中MAC地址才是真正用来进行数据传输的物理地址，每台主机中都有一个ARP高速缓存，存储同一个网络内的IP地址与MAC地址的对应关 系，以太网中的主机发送数据时会先从这个缓存中查询目标IP对应的MAC地址，会向这个MAC地址发送数据。操作系统会自动维护这个缓存。这就是整个实现 的关键。

下边就是我电脑上的arp缓存的内容。

(192.168.1.219) at 00:21:5A:DB:68:E8 [ether] on bond0
(192.168.1.217) at 00:21:5A:DB:68:E8 [ether] on bond0
(192.168.1.218) at 00:21:5A:DB:7F:C2 [ether] on bond0

 

192.168.1.217、192.168.1.218是两台真实的电脑，

192.168.1.217为对外提供数据库服务的主机。

192.168.1.218为热备的机器。

192.168.1.219为虚IP。

大家注意红字部分，219、217的MAC地址是相同的。

再看看那217宕机后的arp缓存

(192.168.1.219) at 00:21:5A:DB:7F:C2 [ether] on bond0
(192.168.1.217) at 00:21:5A:DB:68:E8 [ether] on bond0
(192.168.1.218) at 00:21:5A:DB:7F:C2 [ether] on bond0 

这就是奥妙所在。当218 发现217宕机后会向网络发送一个ARP数据包，告诉所有主机192.168.1.219这个IP对应的MAC地址是00:21:5A:DB:7F:C2，这样所有发送到219的数据包都会发送到mac地址为00:21:5A:DB:7F:C2的机器，也就是218的机器。