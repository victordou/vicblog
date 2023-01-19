---
title: "TruenNAS 设置经验分享"
date: 2023-01-18T15:18:38+08:00
draft: false
toc: false
images:
tags: 
  - truenas
  - zfs
slug: truenas-setup-guide
---

白嫖王Linus 大骂 TrueNAS 设置不人性化（因为他从 UNRAID 切换到 TruenNAS 了），设置复杂且提示少。在它还叫作 FreeNAS 时，我是这么评价的：“反人类的UI，操作逻辑混乱”。 所以那时我在用 NAS4Free(后来改名叫 XigmaNAS)，自从FreeNAS完善UI之后，并改名 TrueNAS，我也看到了很多 UI逻辑 的改进，我就开始转 TrueNAS 了，XigmaNAS 建立的 ZFS 存储池直接导入，无任何问题（我没问题，不代表你不会出现问题，越菜的人问题越多）。

目前 TrueNAS 还是有不少 BUG，但是几乎不影响你存储文件的安全，WEBUI 也改善了很多（虽然 TrueNAS 最近对存储池UI进行了大改，分开管理存储池和数据集，很多老外UP夸这个设计，但我觉得还需要继续改进，当前页面依然很复杂，应该默认隐藏一下当前没有使用的功能），但仍不适合纯小白。我对 WEBUI 好不好用评断标准就是 “以最少的鼠标点击次数，完成我的更改请求”。

所以我就写一个简单的配置向导，想深入，可以看我文章底部推荐的一些视频和文章。

## 安装 TrueNAS
+ 下载ISO，Rufus DD模式写入U盘

  + 安装界面同时选择两个硬盘，会自动组 ZFS Mirror/镜像，坏一个启动盘也不影响NAS开机，小工作室推荐这么搞，个人就算了吧，一个新盘+备份文件+1小时就能恢复挂掉之前的状态。

  + 现在 TrueNAS 推 root 账户去密码化，推荐选择admin继续安装，admin只能用来登录 WEBUI，root 是后台账号，所以登陆后新增一个自己名字的账号来 SMB 访问共享。

  + 让自己的账号也能登录 WEBUI -> 把自己的账号加入`builtin_administrators`组，顺便加入 `root apps docker` 组

## 登录网页UI

+ System Setting -> General
  + 调时区语言（就那么几个字，还调语言？当前22.12版本好像简体中文不生效），备份设置在右上角。

+ Network
  + 从 DHCP 调到固定 IP，记得 手动加一下 DNS + 网关，Global Configuration Settings - Nameserver + IPv4 Default Gateway

+ Credentials -> Local Users -> Add
  + 添加新用户，只填带 * 用户名和密码部分 + 上面说到的几个组名字，其他不懂不要动，不懂就问。

+ Storage -> Create新建池 / Import导入池
  + 只要是ZFS都能跨设备导入，没有限制。你不懂怎么规划存储池结构，就点那个 `Suggest Layout`，你真的要研究怎么组存储池才安全，请你先去研究再来新建，因为存储池建好了，就不能资料无损的移除硬盘了。

+ Datasets(数据集，类似磁盘分区)
  + 可新建多个dataset，每个dataset用不同的 Record Size，提升某些程序响应。但大家都是片子，直接1M起步吧。dataset下面可以继续建dataset，一直套娃下去，虽然 SMB 分享之后 dataset 显示为文件夹，但是他是相当于系统分区，每个分区配置都可以不一样，分区之前移动文件也是完整复制+删除，不会瞬间移动文件完成。

  + LZ4 压缩保持默认开启，假如都是片子 没啥压缩效果，但是假如你保存了一下压缩率低的文件，系统会以极低CPU消耗帮你节省你的磁盘文件体积占用，除非特殊高大上应用有要求，不要关LZ4压缩。

  + Atime，默认推荐关闭，文件访问时间记录，用极大的磁盘带宽消耗来记录你打开片子的访问时间，你不想关，随便你，SMB速度减半不要哭。

  + Record Size -> 片子直接1M起，你存储文件体积越大，就选越大。小文件就往小了选，随便改，改了之后写入的文件才会以这个大小存储，之前的文件的Record Size还是不变。

  + ACL 推荐 SMB/NFSv4，权限修改和Windows NTFS权限一样，一条一条的加权限。

  + 以上除了我写到的，其他的设置就别乱改了，改了你的 NAS 就会爆炸。

+ Shares -> SMB -> Add
  + Path -> /mnt(挂载点) / pool(存储池) / dataset(数据分区)。推荐只分享 dataset，不分享存储池，因为不这么做，TrueNAS报错会多到自己看不懂，其他人也不一定看得懂。

  + 分享名随便设置，其他全部默认设置，保存后会启动SMB服务，勾选SMB自启。

+ Data Protection
  + Scrub Tasks 会自动把你的存储池加进去，每个月校验修复你的存储池文件损坏，别乱动默认设置。

  + Periodic Snapshot Tasks 定期快照，需要对全部 dataset 进行快照，节省时间的方法可以选 存储池+Recursive递归，把你存储池下面所有dataset进行快照。快照保存时间，快照可选每天每小时每分钟进行。

  + TrueNAS 的快照功能集成到了 Windows VSS/卷影复制服务功能上，你的每个快照都可以在 Windows Explorer 的`右键-属性-以前的版本`中看到，而且能够即时的浏览文件历史版本，没有恢复等待时间。

  + Replication Tasks，dataset数据集可以备份到另一个存储池或者另一台TrueNAS Core/Scale，备份使用的是快照发送，所以请使用高级模式进行配置，不然会重复新建备份快照（重复快照任务不影响文件安全，但是我就是看着不舒服）

  + S.M.A.R.T. Tests，新建所有硬盘 每天 short 每周 long 的SMART检查，防患于未然。

+ Apps
  + 选择存储 Docker 文件的存储池，会自动新建一个  ix-applications 的 dataset。存储可以选一个单盘SSD，然后每分钟或每小时备份到机械硬盘RAID阵列上。

  + Setting -> Advanced Setting -> 取消勾选 Enable Host Path Safety Checks，企业级隔离功能，我们全都是片子，不怕不安全。

  + 不要使用 Truecharts 软件库，手动使用 Launch Docker Image 新增 docker, 这样能保证对你的数据最大的控制权。使用自带 TrueNAS软件库 或者 TrueChart软件库，轻则更新不及时，重则完全不启动。TrueNAS K3S 复杂程度超出你的想象。

  + 顺便说一下 Docker 版 BT 下载工具端口映射问题，Docker BT 软件是无法将端口映射到你的路由器 upnp上的，比如 qBit 开放的 Peer TCP/UDP 端口是 6881，TrueNAS 软件库将 Docker 端口6881 转发到了 TrueNAS 端口20988，这时候你需要手动将路由器 WAN口 端口6881 转发到 TrueNAS LAN 端口20988 上。这样做是为了保证 qBit 网页设置的端口和你实际 WAN 口开放的一致，如果不一致会导致 BT 下载速度慢，也就是端口映射失败的情况。所以我的做法是，不使用官方软件库，手动设置 qBit 端口为 46881，端口转发全部填46881。

## NFSv4 ACL
* 跟着UI自己设置吧，一条一条加，灵活性高，和 NTFS 的逻辑一模一样，不需要理解UGO/0755这种逻辑。

* 什么你还想了解这个 ACL？ 放一个[B站-老湿基-视频BV1tG4y1C78v](https://www.bilibili.com/video/BV1tG4y1C78v/)，和[TruenNAS 官方手册](https://www.truenas.com/docs/scale/scaleuireference/storage/datasets/editaclscreens/#permissions-settings---advanced)自己去看吧。

## OpenZFS 其他相关资料

* ZFS 推荐你使用 ECC，并不是ZFS没有ECC就不能用了，我现在强烈推荐你把下面资料全部看一遍，你会听吗？

* ZFS 的特色是速度慢，不是速度快，他的优点是：提供了偏执狂般文件安全，干的活多，自然速度慢。

* 为什么没人用硬件Raid了，因为企业级机械硬盘都没有 520byte 扇区了，家用是 512byte 扇区，企业级多了 8byte 的校验，Raid卡会使用 8Byte 来校验文件正确性。所以说目前不管是软硬RAID，都没有任何应对硬盘比特位翻转的对策。

### 没有ECC内存时比特位翻转会发生什么
* 这种情况下位翻转不会有任何影响。
  + 位翻转发生在未使用的内存中。

* 位翻转导致 runtime 故障。

  + 当从磁盘读取的内容时发生位翻转。

  + 当程序更改代码时，通常会观察到故障。

  + 如果位翻转发生在系统内核或 /sbin/init 中的例程中，系统可能会崩溃。重新加载受影响的数据可以清除故障。也可以通过重新启动来解决。

* 位翻转发生在下列情况下可能会导致数据损坏。

  + 当文件使用到的内存比特位正准备写入到磁盘时发生位翻转。

  + 位翻转发生在 ZFS 的校验和计算之前，ZFS 将不会意识到数据已损坏。

  + 位翻转发生在 ZFS 的校验和计算之后，但在写入磁盘之前，ZFS 会检测到它，但它可能无法纠正它。

* 位翻转发生在下列情况下可能导致元数据损坏。

  + 当写入磁盘时盘面结构发生位翻转。

  + 位翻转发生在 ZFS 的校验和计算之前，ZFS 将不会意识到元数据已损坏。

  + 位翻转发生在 ZFS 的校验和计算之后，但在写入磁盘之前，ZFS 会检测到它，但它可能无法纠正它。

  + 以上此类事件中能否恢复将取决于损坏的内容。 在最坏的情况下，存储池可能变得不可导入。此时所有文件系统在绝对最坏的情况下，位翻转故障情况后的数据可靠性会很差。但这种情况应该被认为是非常罕见的。

以下是转-老湿基-
* 传统的RAID在恢复时，存在一些问题。假如4个10T的盘做了RAID5，其中某个盘坏了，系统重建RAID时会重建10T空间。哪怕实际仅用了2T，空闲的8T也会被傻瓜式校验一遍，因为这是个很低级的过程。在ZFS上，因为文件系统知道RAID模式，所以可以按需重建，只需要恢复2T的数据即可。

* 但需要注意，恢复阵列依旧是个脆弱的过程！ZFS也不例外。ZFS只是降低了任务难度，客观来看ZFS的恢复也挺严峻。

* 强烈不建议把问题硬盘直接拔下来再插入新盘，因为这种情况下新盘的数据就必须通过全盘读取其他好盘的方式计算出来，这个过程的代价比较高。而如果带着故障硬盘一起恢复，那么故障硬盘里的好数据就能直接导入新盘，而无需根据其他盘的校验值进行反向计算，这大大降低了其他盘在高强度读数据过程中发生损坏的风险。


ZFS 视频1 <https://www.bilibili.com/video/BV18V4y1K7o2/>

ZFS 文字版1 <https://www.wolai.com/littlenewton/gJvungs54zWgZ3YfoXAvKW>

ZFS 视频2 <https://www.bilibili.com/video/BV1Ad4y1Y7b5/>

ZFS 文字版2 <https://www.wolai.com/littlenewton/dT2737LXf412LKKZ973DsD>

OpenZFS 基础 <https://www.youtube.com/watch?v=MsY-BafQgj4>

如何扩展ZFS <https://www.youtube.com/watch?v=11bWnvCwTOU>

硬件Raid已死 <https://www.youtube.com/watch?v=11bWnvCwTOU>

OpenZFS 硬件推荐 <https://openzfs.github.io/openzfs-docs/erformance%20and%20Tuning/Hardware.html>

Debian ZFS 命令行使用教程 <https://pthree.org/2012/04/17/install-zfs-on-debian-gnulinux/>

