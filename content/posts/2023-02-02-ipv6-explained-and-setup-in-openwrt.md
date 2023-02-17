---
title: "IPv6 原理及如何设置 OpenWrt"
date: 2023-02-02T13:35:25+08:00
draft: false
toc: false
images:
tags: 
  - openwrt
  - network
slug: ipv6-explained-and-setup-in-openwrt
---
如果你觉得看文字很无聊 可以看 onemarcfifty 老师的视频，和B站老湿基
1. [IPv6 from scratch - the very basics of IPv6 explained](https://youtu.be/oItwDXraK1M)
2. [IPv6 explained - SLAAC and DHCPv6 (IPv6 from scratch part 2)](https://youtu.be/jlG_nrCOmJc)
3. [IPv6 with OpenWrt](https://youtu.be/LJPXz8eA3b8)
4. <https://www.bilibili.com/video/BV1N14y1j7Ku>

此文章大部分内容都是引用自 <https://github.com/onemarcfifty/cheat-sheets/blob/main/networking/ipv6.md>

## 1. IPv6 地址含义
- IPv6 采用 128位 的地址 (IPv4使用的是32位)，约 340,282,366,920,938,463,463,374,607,431,768,211,456个 IPv6地址。

- IPv6 因为掩码太长了，手写不方便，就用用`/`加数字，比如 `/64`，IPv4也有 `/` 比如 `192.168.0.0/24` = `192.168.0.0 255.255.255.0`

- IPv6 不使用 IPv4 的十进制进行书写，而使用 16进制，可以使用Windows计算器的程序员模式来计算 HEX(16进制) 和 BIN(二进制) 换算。F=1111 E=1110 9=1001。

        FE80:DEAD:BEEF:BABE:0000:0000:0000:0010
        FE80:DEAD:BEEF:BABE:0:0:0:10 (首位的 0 可以省略，中间和末尾不能省略)

- 地址中很多重复的 0 可使用 `::` 进行缩写，方便人类查看。

        FE80:DEAD:BEEF:BABE::10

- 但是，同一个地址内只能存在一个 `::` 缩写，多个 `::` 会导致地址翻译长地址出错，根据 RFC 5952 而且只缩写最长的0，缩写短的0不符合规则。

        FE80::1:0:0:1   (正确写法)
        FE80::1::1      (错误写法)
        FE80:0:0:0:1::1（错误写法）


## 2. IPv6 Scope/地址范围

| 范围 | 含义 |
| -- | -- |
| ::1/128 | Loopback/本机回环地址（就是localhost，类似 127.0.0.1） |
| fe80::/10 | Link-Local/链路本地地址 （同交换机下的地址，不能转发到其他局域网，类似 169.254.0.0/16) |
| fc00::/7 | Unique-Local/唯一区域地址 (大局域网地址，不能转发到公网，类似 192.168.0.0/16) |
| 2000::/3 | Global-Unicast/全球单播 (互联网地址，就是公网 IP) |

> IANA维护官方的 IPv6 地址列表 <https://www.iana.org/assignments/ipv6-address-space/ipv6-address-space.xhtml> 或者 <https://en.wikipedia.org/wiki/Reserved_IP_addresses>

> 现实中 Link-Local/链路本地地址 `fe80::/10` 被使用的只有 `fe80::/64` 这一段，只要你启用了 IPv6，网口会自动根据 MAC地址 自动生成 IPv6地址，和 IPv4 的 `169.254.0.0/16` 只在没有 DHCP 时才会生成的情况不一样。

- 每个 IPv6 地址都有 Scope，OpenWrt 可以通过 ssh 内输入 `ip a` 来查看对应 IPv6 的 Scope.
    + scope host
    + scope link
    + scope global

- 可以用 [subnet子网计算器](https://www.tunnelsup.com/subnet-calculator/)来算出 IP 地址范围

## 3. IPv6 Subnet/子网
| 掩码位数 (MSB) | 含义 |
| -- | --  |
| 最前 48 位: | **ISP 电信网络** 地址 |
| 接着 16 位: | **可划分子网** 地址 |
| 最后 64 位: | **子网内设备** 地址 |

+ 这个表的意思就是，ISP 比如电信，购买一个 `2048:DEAD:BEEF::/48` 网段，这个网段可以划分 2^16=65536 个子网，可以分配给 `65536` 个电信用户使用。每个用户 都能获取到一个 `/64` 的地址。大部分ISP 都会给你 /56 /60，这样你就可以自己划分子网，多个LAN等。

+ [微软说](https://learn.microsoft.com/zh-cn/azure/virtual-network/ip-services/ipv6-overview) IPv6 的子网大小只能是 /64。 将来当你决定将子网路由到本地网络时，这种大小可以确保兼容性，因为某些路由器只能接受 /64 IPv6 路由。


## 4. DHCPv6 vs SLAAC

+ `Static` 固定 IPv6 地址，纯手动输入 IPv6 地址，你吃的很饱可以这么搞。

+ `SLAAC`(stateless address autoconfiguration)，你的电脑读取你的路由器在局域网公告的 Prefix Delegated/PD，来自动设置 IPv6 地址，同时你的电脑也会想局域网公告，我要用自己生成的这个IP，大家没意见我就用了啊。

+ `DHCPv6` 你的电脑，向路由器申请 IPv6 地址，和 IPv4 一样。

> 同一个网口能同时拥有多个 IPv4 地址，IPv6 也一样，大部分设备都是同时打开 SLAAC + DHCPv6，所以你的电脑可以获取很多个 IPv6地址。

DHCPv6 和 SLAAC 是IPv6中两种不同的地址自动配置方法。它们的区别主要有以下几点：

- `SLAAC` 是无状态的，即路由器不需要记录或管理分配给终端设备的地址，而是通过邻居发现协议（NDP）向终端设备发送路由通告（RA）消息，包含网络前缀、前缀长度和默认网关地址等信息，让终端设备自己生成全局唯一的IPv6地址。

- `DHCPv6` 可以是有状态的或无状态的，即路由器可以记录或管理分配给终端设备的地址，也可以不记录或管理。

    - 有状态的 DHCPv6 意味着路由器通过 DHCPv6 协议向终端设备分配 IPv6 地址，并提供其他信息，如 DNS 服务器、域名等。

    - 无状态的 DHCPv6 意味着路由器不分配IPv6地址，而是让终端设备使用SLAAC生成地址，并通过DHCPv6协议提供其他信息。

- SLAAC 和 DHCPv6 之间还有一些其他的区别，比如 SLAAC 支持隐私扩展（Privacy Extensions），可以让终端设备定期更换临时 IPv6 地址以保护隐私；而 DHCPv6 支持租约（Lease）机制，可以让路由器回收过期或未使用的 IPv6 地址以节省资源。


# OpenWrt 实践

> 默认 OpenWrt 会默认自带 IPv6 的设置，但是有设置还不够，需要安装 IPv6 组件，OpenWrt官方最低要求：DHCPv6 客户端 (**odhcp6c**), RA & DHCPv6 服务器 (**odhcpd**) 和 IPv6 防火墙 (**ip6tables**) 还有 Luci网页配置 的 (**luci-proto-ipv6**）

> 因为默认 IPv6 全部都是公网，没有 NAT地址 转换，所以要记住 IPv6 地址流动方向是从 `WAN` -> `LAN` -> `二级路由 WAN` -> `二级路由 LAN`，为了让下级端口获取 `Prefix Delegated/PD`，或 `DHCPv6 Relay`，那么上级的端口就需要开启 `Delegate IPv6 prefixes/委托 IPv6 前缀` 或 `Relay/中继`

+ 首先是 `Unique-Local` 地址，类似本地局域网地址 如 192.168.0.0/16，供内部局域网使用，适用于组建有很多分支机构的大型 IPv6 局域网，家用可以不用开。
    - 进入`OpenWrt` -> `Network/网络` -> `Interfaces/接口` -> `Global network options/全局网络选项`
    - `IPv6 ULA-Prefix/IPv6 ULA 前缀`，这里可以填写任意 fc00::/7 地址内 最大 FDFF 的地址。

### 情景1 - 从 PPPOE 获取 IPv6 Prefix 地址
- 获取 `IPv6-PD`, 类似 `2048:DEAD:BEEF::/60`
- `Network/网络` -> `Interfaces/接口`
    - `WAN` ->  `Edit/编辑` ->  `Advanced Settings/高级设置`
        - `Obtain IPv6 address/获取 IPv6 地址` ->  `Automatic/自动`。如果选择 `Manual/手动`，将会关闭 DHCPv6 和 SLAAC。
        - `Delegate IPv6 prefixes/委托 IPv6 前缀` ->  `打勾✓`，如果不打勾，路由器 LAN口 将不会自动分配到 公网IPv6 地址。

    - `LAN` ->  `Edit/编辑` ->  `Advanced Settings/高级设置`
        - `Delegate IPv6 prefixes/委托 IPv6 前缀` ->  `打勾✓`，如果不打勾，你的二级路由将不会获取 公网IPv6 分配规则。
        - `IPv6 assignment length/IPv6 分配长度` ->  选择或输入大于 WAN口 `IPv6-PD` 的数字，如果你有多 LAN，或者二级路由，请使用使用[subnet子网计算器](https://www.tunnelsup.com/subnet-calculator/)合理分配子网。

- 开启`DHCPv6` or `SLAAC`
    - `LAN` -> `Edit/编辑` -> `DHCP Server/服务器` -> `IPv6 Setting/设置`
        - `RA 服务`, 设置为 `server/服务器模式`。
            - Router Advertisement，路由通告，路由器主动在网络上告诉局域网设备，我是路由器。
        - `DHCPv6`, 设置为 `server/服务器模式`。
            - 字面意思，分配IP的。
        - `NDP Proxy/代理`, 选择`禁用`
            - 全名Neighbour Discovery Protocol Proxy，上下级路由之间转发 `Neighbor Solicitation + Neighbor Advertisement` 消息。
        - **relay/中继模式**，转发 上级路由/ISP IPv6 参数
        - **server/服务器模式**，路由器自己作为服务器，分发 IPv6 参数
        - **hybrid/混合模式**，路由器先尝试转发 上级路由/ISP 的参数，如果 ISP 没有，则自己作为服务器分发 IPv6 参数。
    - `LAN` -> `Edit/编辑` -> `DHCP Server/服务器` -> `IPv6 RA Settings/设置` 
        - `Enable SLAAC` -> `打勾✓`，很多设备不支持`DHCPv6` 只支持 `SLAAC`。

> 邻居发现协议代理（Neighbor Discovery Protocol Proxy，ND Proxy）是一种用于 IPv6 的服务，类似于 IPv4 的代理ARP（Proxy ARP），可以在不能在链路层进行桥接的情况下，实现在同一个子网前缀内桥接多个网络段。它的作用是让一个设备（如路由器）代替另一个设备（如主机）响应邻居请求（Neighbor Solicitation）和邻居通告（Neighbor Advertisement）消息，从而使两个不同网络段的设备能够互相通信。

> 例如，在一个家庭网络中，有两个不同的无线接入点（AP），分别连接到主路由器上。如果主路由器支持 ND Proxy 功能，它就可以在收到来自一个AP下的主机A的邻居请求消息时，代替另一个 AP 下的主机B发送邻居通告消息，从而让A和B能够建立连接。

### 情景2 - 从 DHCPv6 获取 IPv6 地址，无 Prefix。

- `Network/网络` -> `Interfaces/接口` -> 新增一个 DHCPv6 的 `WAN6` 接口（默认就有，没有就新增）
- `WAN6` -> `Edit/编辑`
    - `General Settings/常规设置`
        - `Request IPv6-address/请求 IPv6 地址` -> 选择`try`
        - `Request IPv6-prefix of length/请求指定长度的 IPv6 前缀` -> 理论上此情景下是没有`Prefix`，自然这个设置也就无效了。
    -  `Advanced Settings/高级设置`
        - `Delegate IPv6 prefixes/委托 IPv6 前缀` -> 理论上此情景下是没有`Prefix`，自然这个设置也就无效了。
    - `DHCP Server/服务器`
        - `General Setup/常规设置`
            - `Ignore interface/忽略此接口` -> 请`打勾✓`，因为 IPV6 只需要下面 DHCPv6 中继服务。
        - `IPv6 Setting/设置`
            - `Designated master/指定的主接口` -> 必须`打勾✓`
            - `RA 服务` -> 设置为 `relay/中继模式`。
            - `DHCPv6` -> 设置为 `relay/中继模式`。
            - `NDP Proxy/代理` -> 设置为 `relay/中继模式`
- `LAN` - `Edit/编辑`
    - `Advanced Settings/高级设置`
        - `Delegate IPv6 prefixes/委托 IPv6 前缀` -> 理论上此情景下是没有`Prefix`，自然这个设置也就无效了。
        - `IPv6 assignment length/IPv6 分配长度` -> 理论上此情景下是没有`Prefix`，自然这个设置也就无效了。
    - `DHCP Server/服务器` -> `IPv6 Setting/设置`
        - `RA 服务` -> 设置为 `relay/中继模式`。
        - `DHCPv6` -> 设置为 `relay/中继模式`。
        - `NDP Proxy/代理` -> 设置为 `relay/中继模式`

### 个性化路由器 IPv6 地址段

- 任意 `LAN` 接口 -> `Edit/编辑` -> `Advanced Settings/高级设置`
    - `IPv6 assignment hint/分配提示`
        - 比如 ISP 分配给你 `2048:DEAD:BEEF:bab0::/60`，`/60` 说明你可以分配 16个 `/64` 的 多LAN口/多子网，这里填 `e`，你的 LAN IPv6 地址 就会变成 `2048:DEAD:BEEF:babe::1/64` 与其他 LAN 做区分。
    - `IPv6 suffix/后缀`
        - 比如 ISP 分配给你 `2048:DEAD:BEEF:bab0::/60`，填 `99` 你的 LAN IPv6 地址 就会变成 `2048:DEAD:BEEF:bab0::99/60`
### 个性化电脑 IPv6 地址
- 进入`OpenWrt` -> `Network/网络` -> `DHCP/DNS`
    - `Static Lease/静态地址分配` -> `新增/修改静态分配条目` -> `IPv6-Suffix/后缀`
    - 填写任意4位以下16进制数 -> 如`1234`或者`DEAD`。

## 防火墙

> 全世界都可以 Ping 通你的公网 IPv6地址，那是因为OpenWrt防火墙通信规则开通了Ping `Allow-ICMPv6-Input`，你不想被 Ping，可以关闭。

- 举个[官方文档](https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_ipv6_examples)里的例子
```
uci add firewall rule
uci set firewall.@rule[-1].name="Forward-IPv6"
uci set firewall.@rule[-1].src="wan"
uci set firewall.@rule[-1].dest="lan"
uci set firewall.@rule[-1].dest_ip="::DEAD/-64"
uci set firewall.@rule[-1].family="ipv6"
uci set firewall.@rule[-1].proto="tcp udp"
uci set firewall.@rule[-1].target="ACCEPT"
uci commit firewall
/etc/init.d/firewall restart
```
## IPv6 的使用

+ 使用IPv6地址访问 Web网页需要加 `[]`，`http://[2048:DEAD:BEEF:bab0::99]` 或者 `http://[2048:DEAD:BEEF:bab0::99]:80`

+ 还是弄域名吧，谁要记这么长的地址。

+ IPv6 组播地址，[完整清单见 IANA](https://www.iana.org/assignments/ipv6-multicast-addresses/ipv6-multicast-addresses.xhtml)

| 地址 | 含义 |
| -- | -- |
| ff02::1 | 内网所有IPv6设备 |
| ff02::2 | 内网所有路由器（主动通过RA通告自己是路由器）|
| ff02::fb | mDNSv6（HomeKit, AirPlay 会用到） |
| ff02::1:2 | 内网所有DHCP服务器（主动通过RA通告自己DHCP服务器）|
| ff02::101 | 内网所有NTP服务器（同上）|

### 单播/Unicast 组播/Multicast 任播/Anycast 广播/Broadcast

+ `IPV6` 没有 广播/Broadcast
+ `单播/Unicast`: 一对一，俗称私聊
+ `组播/Multicast`: 群聊，加群的才能收到信息，可随时加群退群。
+ `任播/Anycast`: 多个服务器拥有相同的 IP地址，客户端会选择最近的进行链接，比如打电话给 119，会自动路由到你家附近的消防站。
+ `广播/Broadcast`: 类似广场舞的喇叭，除非断网，否则你会一直听到广场舞的声音。
