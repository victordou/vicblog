---
title: "为什么用OpenWrt"
date: 2020-08-23T16:15:38+08:00
draft: false
toc: false
images:
tags: 
  - openwrt
slug: why-openwrt
---

在谈OpenWrt之前，需要介绍下什么是[OpenWrt](https://zh.wikipedia.org/wiki/OpenWrt)，用人话来说就是：可替换路由器出厂系统的第三方系统，是用[Linux](https://zh.wikipedia.org/wiki/Linux)做出来的，开源。

为什么不用路由器出厂系统，你会问。那是因为出厂系统功能少，速度慢。网件Netgear是大厂中系统最慢的，修改任何设置都需要1-2分钟，而且还会断网。华硕Asus的快一点，但是依然达不到速度快的等级。TP-Link则是一朵奇葩，速度很快，但是功能少，大多数都是VxWork的系统，且都是低配，国外的TP-Link要好一点（毕竟国内还是低价跑量，不识货的居多）。

说说我的网络设备吧：

1. 目前使用Esxi架设了一个OpenWrt主路由，x86架构，方便系统升级和刷不死。因为Esxi可运行多个虚拟OpenWrt，可随意切换多个版本的OpenWrt。
2. 两个刷了OpenWrt的无线路由，分别是AC867+AC1300，房间全覆盖AC 5GHz，两个路由都是支持802.11 kvr，可无限漫游，但是我只启用了802.11 kv， 802.11r 未启用。
3. 所有无线路由都是使用LAN连接，无线路由使用了[AP模式](https://openwrt.org/docs/guide-user/network/wifi/dumbap)，因为我每个房间都有2-4根网线。
4. 入户光猫接2层管理型交换机，对WAN和LAN划分不同VLAN，可单线传输WAN+LAN，节省网线，且交换机每根网线都可以变成WAN，直连拨号。

那么怎么刷机OpenWrt呢，这是一个很复杂的问题，欢迎去[恩山论坛](https://www.right.com.cn/forum/)讨论此问题，或者去[OpenWrt官方](https://openwrt.org/toh/start)。

官方OpenWrt缺少很多国人专用软件，推荐使用中国特供OpenWrt: https://github.com/coolsnowwolf/lede ，你问为什么这个叫LEDE，这个是历史遗留问题：`OpenWrt另有一个复刻分支项目，名为LEDE，两者于2018年1月合并，合并后的项目使用OpenWrt的名字、LEDE的源代码。`

上面的OpenWrt编译简单，且成功率高，软件齐全，编译前记得把 lede/feeds.conf.default 里的#src-git helloworld https://github.com/fw876/helloworld 前的 `#` 去除。