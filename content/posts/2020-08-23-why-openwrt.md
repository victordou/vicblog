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

在谈OpenWrt之前，需要介绍下什么是[OpenWrt](https://zh.wikipedia.org/wiki/OpenWrt)，用人话来说就是：可替换路由器出厂系统的第三方系统，是用[Linux](https://zh.wikipedia.org/wiki/Linux)做出来的开源系统。

为什么不用路由器出厂系统，你会问。那是因为出厂系统功能少，速度慢。网件Netgear是大厂中系统最慢的，修改任何设置都需要1-2分钟，而且还会断网。华硕Asus的快一点，但是依然达不到速度快的等级。TP-Link则是一朵奇葩，速度很快，但是功能少，大多数都是VxWork的系统，且都是低配，国外的TP-Link要好一点（毕竟国内还是低价跑量，不识货的居多）。

### 我的网络设备：

1. 目前使用Esxi架设了一个OpenWrt主路由，x86架构，方便系统升级和刷不死。因为Esxi可运行多个虚拟OpenWrt，可随意切换多个版本的OpenWrt。
2. 两个刷了OpenWrt的无线路由，分别是AC867+AC1300，房间全覆盖AC 5GHz，两个路由都是支持802.11 kvr，可无限漫游，但是我只启用了802.11 kv， 802.11r 未启用。
3. 所有无线路由都是使用LAN连接，无线路由使用了[AP模式](https://openwrt.org/docs/guide-user/network/wifi/dumbap)，因为我每个房间都有2-4根网线。
4. 入户光猫接2层管理型交换机，对WAN和LAN划分不同VLAN，可单线传输WAN+LAN，节省网线，且交换机每根网线都可以变成WAN，直连拨号。

### 如何刷机
那么怎么刷机OpenWrt呢，这是一个很复杂的问题，欢迎去[恩山论坛](https://www.right.com.cn/forum/)讨论此问题，或者去[OpenWrt官方](https://openwrt.org/toh/start)。

官方 OpenWrt 缺少很多国人专用软件，推荐使用中国特供 OpenWrt: <https://github.com/immortalwrt/immortalwrt>

类似官方[OpenWrt Firmware Selector](https://firmware-selector.openwrt.org/), 你也可以通过网页自己去选择想要 ImmortalWrt 插件 <https://firmware-selector.immortalwrt.org/>

[Deprecated](https://github.com/coolsnowwolf/lede) LEDE。你问为什么这个叫LEDE，这个是历史遗留问题：OpenWrt另有一个复刻分支项目，名为LEDE，两者于2018年1月合并，合并后的项目使用OpenWrt的名字、LEDE的源代码。

### 路由器历史
[知乎网友denglj](https://www.zhihu.com/people/denglj)关于路由历史的总结：<https://www.zhihu.com/question/20822589/answer/1001410286>

> 2002年10月，Linksys公司（由一对移民美国的台湾夫妇曹英伟和吴健创建）发布了名为WRT54G的无线路由器的第1个版本，该机型基于32-bit MIPS芯片，搭载了基于Linux内核的固件，并且可以刷机；WRT的含义，Linksys原意可能是指 Wirless Receiver/Transmitter，现在大家都解读为Wirless RouTer；

> 2003年3月，Cisco公司以5亿美元的价格收购了Linksys，成为其子公司，此后Linksys推出的产品都标记上 Linksys by Cisco；

> 2003年6月，Linux Kernel 开发组听闻WRT54G搭载了包含GPL开源协议的Linux代码，要求Linksys开源相关部分的代码，为此FSF(自由软件基金会)还起诉Cisco；

> 2003年7月，尽管各方对WRT54G固件是否应该开源有所争议，不过Cisco和Linksys迫于外界压力还是开源了WRT54G固件，至此，各种定制固件和路由器刷机开始流行了起来（Lintel在其2012年的一份文档中说是2003年3月思科被迫开源，其实有误，3月份思科和领势还在忙收购的事吧）；

> 2004年1月，OpenWrt 项目启动（据其官网自我介绍，未找到源码或论坛讨论等其他佐证），基于WRT54G固件开发，后来（不知道是多久以后，根据2004和2005年的论坛信息显示，Alchemy如日中天的时候OpenWrt还是个弟弟）发布名为“stable release”的版本；

> 2004年6月，Sevasoft公司基于Linksys固件放出了Alchemy v1.0，他们做的固件在当时是最流行的，在v1.0之前的pre版本就很流行了，开发也很活跃，Sevasoft也是WRT54G系列最早的开源开发者之一；

> 2004年11月， Timothy Jans(又叫 Avenger 2.0)，基于Linksys开源的WRT54G系列的固件发布了HyperWRT，主要是开放一些官版受限的功能，但又尽可能保持原汁原味；

> 2005年1-2月，Linksys为了支持这些第三方固件把WRT54G v4机型拿来重新上市，并重命名为WRT54GL其中L是指Linux，据Linksys相关负责人在2018年称，WRT54GL是迄今为止最畅销的路由器；Sveasoft公司鉴于自己在Alchemy固件上取得的巨大成功，有了转商业运营赚钱的想法，但限于GPL协议，又不得不开源，想出的变通做法就是，让用户每年交20美元的订阅费，付费用户可以进入论坛享用商用版固件，而免费用户只有使用更新较慢的版本，Alchemy社区内也有人看不惯这种收费行为，于是自行修改Alchemy然后对外发布；

> 2005年1月22日，Sebastian Gottschall(又叫BrainSlayer)基于Alchemy v16固件开发了第一版DD-WRT v16固件，版本号沿用Alchemy的，DD是德国东部Dresden城市的汽车牌照的缩写，该城市是DD-WRT开发组生活的地方；

> 2005年2月，HyperWRT原版停更， tofu和 Thibor 两名开发者基于它继续开发，项目名为 HyperWRT +tofu 和 HyperWRT Thibor；

> 2005年中，OpenWrt 发布了名为 “experimental”的版本；

> 2005年12月，DD-WRT v23发布，鉴于Alchemy社区的内斗和商业付费的风险，DD-WRT自v23开始将固件核心替换为了OpenWrt；

> 2006年2月，HyperWRT+tofu停止开发，合并入HyperWRT Thibor，后者开发至2008年2月停更；

> 2006年12月， Jonathan Zarate发布了 Tomato(番茄，也简称TT) 的第一个版本，基于HyperWRT为Linksys WRT54G系列和Buffalo WHR-G54S系列机型定制更易用功能更强的固件，主要支持博通Broadcom的系列芯片；

> 2007年1月，OpenWrt发布了代号为White Russian的固件，这个版本之后的OpenWrt才变得越发流行；

> 2008年7月14日，Eric Bishop基于OpenWrt Kamikaze(v7.x和v8.x)发布了Gargoyle(石像鬼)固件的第一个稳定版v1.0，加入了自己的包管理器，格式为 gpkg；

> 2008年7月26日，DD-WRT v24 SP1 发布，然后至今停更，从2010年起DD-WRT社区诸多开发者在不断发布各种变体版本的固件；

> 2010年，华硕ASUS发布了RT-N56U路由器（具体发布日期没找到，华硕官网说RT-N56U获得了2010年的iF设计奖），该机型搭载的是联发科MTK的芯片；

> 2010年6月28日，Tomato(番茄) 官方更新了v1.28稳定版，然后至今还未更新；

> 2011年1月，华硕在CES上发布了RT-N66U路由器，该机型最早使用Asuswrt固件的（华硕更早机器搭载的固件还没统一成型），Asuswrt是基于Tomato-RT/Tomato-USB开发，主要支持博通Broadcom芯片和部分高通Atheros芯片；同月，高通Qualcomm收购创锐讯Atheros为全资子公司，后者主要研发无线通讯芯片；

> 2012年5月3日，俄罗斯人Andy Padavan(老毛子)创建了rt-n56u项目并提交初始化代码，基于 Asuswrt-Merlin 固件开发，由于RT-N56U搭载的是联发科芯片，后来Padavan被移植到多款基于联发科芯片的路由；

> 2012年6月19日，加拿大人Eric Sauvageau创建了Asuswrt-Merlin(梅林) 项目并提交了初始化代码，基于 Asuswrt 3.0.0.3.144；

> 2013年10月，Cisco公司将Linksys卖给了Belkin公司，Linksys至此与思科无关，新东家Belkin保留了Linksys原品牌；

> 2013年-2016年，国内各大神开始在论坛活跃最频繁的时间段，发布相关教程和他们基于OpenWrt/LEDE、Tomato、DD等定制的固件，Lean、Lintel、佐须之男都出名在这个时段；

> 2016年3月1日，佐须之男在Tomato基础上(应该是基于Tomato v1.28)，发布了Tomato Phoenix(不死鸟)的第一个公开测试版，主要增加了Tomato对联发科芯片的支持；

> 2016年5月，OpenWrt 的部分核心成员基于OpenWrt另起炉灶开了LEDE项目，主要因看不惯既有社区的乌烟瘴气和旧源码的质量；

> 2017年1月24日，Lintel宣布因团队接手Newifi系列路由的固件维护，而PandoraBox(潘多拉)停止更新，该固件是为了照顾部分英文水平较菜和动手能力较差的玩家，对OpenWRT/LEDE做了本地化，并预编译或安装了大陆用户常用的某些功能，最早发布在 openwrt.org.cn上；

> 2017年2月，LEDE的第一个稳定版发布，版本号为v17.01.0，LEDE的主要改进是重构了OpenWrt代码，替换了文件系统改为JFFS2，更友好的Web界面LuCI，更多的opkg包支持等；

> 2018年1月，OpenWrt 老项目和离家出走的LEDE决定复合，名为OpenWrt/LEDE，在原LEDE团队的规矩和主导下运作，但名字仍叫OpenWrt；

> 2018-2019年，OpenWrt和LEDE主要在版本号为v18.x的代码上完成合并工作，DD-WRT、Asuswrt-Merlin、Padavan等活跃项目也在持续更新。

> 现在，随着路由固件定制、开发技术的普及，各路论坛里基于OpenWrt/LEDE、DD-WRT、Merlin、Padavan、Tomato等知名固件的各种私人订制层出不穷……

~~上面的OpenWrt编译简单，且成功率高，软件齐全，编译前记得把 lede/feeds.conf.default 里的#src-git helloworld https://github.com/fw876/helloworld 前的 `#` 去除。~~

Updated in 2022/12/12