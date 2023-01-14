---
title: "利用 Github Action 自动编译OpenWrt"
date: 2022-05-17T15:21:34+08:00
draft: false
toc: false
images:
tags: 
  - openwrt
  - Github
slug: github-action-auto-compile-openwrt
---

> 想自己用 Linux 编译 请看这个视频 <https://www.bilibili.com/video/BV1854y1W7fu/>
### 本次教程基于下列前车之鉴
<https://github.com/KFERMercer/OpenWrt-CI>


## 注册 Github
本步骤跳过，网上一大堆，请参考。

## 1. Fork Openwrt
打开 <https://github.com/immortalwrt/immortalwrt/fork>

## 2. 修改 ▶️Actions 的 Workflows yml文件
1. 进入你 `Fork` 后的 `OpenWrt` 页面
2. 检查当前的 `OpenWrt` 有没有 `.github/workflows` 文件夹
    > 有 -> 进入`.github/workflows` 修改里面的 `***.yml` 文件

    > 没有 -> 进入 `▶️Actions` 会自动新建 `.github/workflows\main.yml`
3. 复制我在下文提供的 yml模板，粘贴。
4. （可选）参考下文进行自定义修改，比如 `加软件` `改平台`
5. 修改两处 branch 名，从模板的 `openwrt-21.02` 修改为你当前正在编辑 branch，比如 `master` 或 `main`
6. 修改完成后 `Start commit` -> `Commit changes`

## 3. 此时你的 OpenWrt 会自动开始编译
时长约两小时，完成后，去`▶️Actions`页面下载 `OpenWrt_firmware` 即可

## 4. 自定义固件
1. 找到你需要安装的包名，官方软件包大全 <https://openwrt.org/packages/index/start>
2. 软件包可以从编译环境下 `make defconfig` 生成 `.config` 中找到
> 比如 # CONFIG_PACKAGE_luci-app-dockerman is not set
3. 添加包名到`openwrt-ci.yml` 中 `cat >> .config <<EOF`后面，对着我的例子进行修改。
> 修改为 CONFIG_PACKAGE_luci-app-dockerman=y
4. 编译其他固件，需要修改 `Target`-> `子Target` -> `设备名` 修改下面三行做例子。
> 注意要和其他行 `首字母对齐`，错位必编译错误！
```yml
          CONFIG_TARGET_ramips=y
          CONFIG_TARGET_ramips_mt7621=y
          CONFIG_TARGET_ramips_mt7621_DEVICE_youhua_wr1200js=y
```
## 5. 我目前使用的模板
> 我x86默认编译vmdk给ESXi，你用不到可以删除此行。
```yml
#
# This is free software, lisence use MIT.
# 
# Copyright (C) 2019 P3TERX <https://p3terx.com>
# Copyright (C) 2020 KFERMercer <KFER.Mercer@gmail.com>
# 
# <https://github.com/KFERMercer/OpenWrt-CI>
#

name: OpenWrt-CI

on:
  push:
    branches: 
      - openwrt-21.02
  # schedule:
  #   - cron: 0 20 * * *
  workflow_dispatch:
  release:
    types: [published]

jobs:

  build_openwrt:

    name: Build OpenWrt firmware

    runs-on: ubuntu-22.04

    if: github.event.repository.owner.id == github.event.sender.id

    steps:

      - name: Checkout
        uses: actions/checkout@v3
        with:
          ref: openwrt-21.02

      - name: Space cleanup
        env:
          DEBIAN_FRONTEND: noninteractive
        run: |
          docker rmi `docker images -q`
          sudo rm -rf /usr/share/dotnet /etc/mysql /etc/php /etc/apt/sources.list.d /usr/local/lib/android
          sudo -E apt-get -y purge azure-cli ghc* zulu* hhvm llvm* firefox google* dotnet* powershell openjdk* adoptopenjdk* mysql* php* mongodb* dotnet* moby* snapd* || true
          sudo -E apt-get update
          sudo -E apt-get -y install build-essential clang flex g++ gawk gcc-multilib gettext git libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev file wget asciidoc binutils bzip2 util-linux help2man intltool libelf-dev make patch perl-modules python3-dev xsltproc qemu-utils
          sudo -E apt-get -y autoremove --purge
          sudo -E apt-get clean
          df -h

      - name: Update feeds
        run: |
          # sed -i 's/#src-git helloworld/src-git helloworld/g' ./feeds.conf.default
          # echo "src-git helloworld https://github.com/fw876/helloworld.git" >> "feeds.conf.default"
          ./scripts/feeds update -a
          ./scripts/feeds install -a

      - name: Generate configuration file
        run: |
          rm -f ./.config*
          touch ./.config

          #
          # 在 cat >> .config <<EOF 到 EOF 之间粘贴你的编译配置, 需注意缩进关系
          # 例如:

          cat >> .config <<EOF
          CONFIG_TARGET_x86=y
          CONFIG_TARGET_x86_64=y
          CONFIG_TARGET_x86_64_DEVICE_generic=y
          CONFIG_VMDK_IMAGES=y
          CONFIG_LUCI_LANG_en=y
          CONFIG_LUCI_LANG_zh_Hans=y
          CONFIG_PACKAGE_luci=y
          CONFIG_PACKAGE_zabbix-extra-network=y
          CONFIG_PACKAGE_luci-app-attendedsysupgrade=y
          CONFIG_PACKAGE_luci-app-ddns=y
          CONFIG_PACKAGE_luci-app-firewall=y
          CONFIG_PACKAGE_luci-app-https-dns-proxy=y
          CONFIG_PACKAGE_luci-app-nlbwmon=y
          CONFIG_PACKAGE_luci-app-sqm=y
          CONFIG_PACKAGE_luci-app-ssr-plus=y
          CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Hysteria=y
          CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Trojan=y
          CONFIG_PACKAGE_luci-app-turboacc=y
          CONFIG_PACKAGE_TURBOACC_INCLUDE_BBR_CCA=y
          CONFIG_PACKAGE_luci-app-upnp=y
          CONFIG_PACKAGE_luci-app-vlmcsd=y
          CONFIG_PACKAGE_luci-app-wireguard=y
          CONFIG_PACKAGE_luci-app-wol=y
          CONFIG_PACKAGE_avahi-dbus-daemon=y
          CONFIG_PACKAGE_avahi-utils=y
          CONFIG_PACKAGE_bind-dig=y
          CONFIG_PACKAGE_ddns-scripts-cloudflare=y
          CONFIG_PACKAGE_ddns-scripts-dnspod=y
          CONFIG_PACKAGE_ddns-scripts_aliyun=y
          CONFIG_PACKAGE_nmap-ssl=y
          CONFIG_PACKAGE_adguardhome=y
          CONFIG_PACKAGE_curl=y
          CONFIG_PACKAGE_nano=y
          CONFIG_PACKAGE_open-vm-tools=y
          CONFIG_PACKAGE_qrencode=y
          CONFIG_PACKAGE_whois=y
          CONFIG_PACKAGE_luci-proto-ipv6=y
          CONFIG_PACKAGE_wget-ssl=y
          CONFIG_PACKAGE_libip6tc=y
          CONFIG_PACKAGE_ip6tables=y
          CONFIG_PACKAGE_ip6tables-mod-nat=y
          CONFIG_PACKAGE_kmod-ipt-nat6=y
          CONFIG_PACKAGE_kmod-nf-nat6=y
          CONFIG_PACKAGE_odhcpd-ipv6only=y
          CONFIG_PACKAGE_odhcp6c=y
          CONFIG_PACKAGE_6in4=y
          EOF

          #
          # ===============================================================
          # 

          sed -i 's/^[ \t]*//g' ./.config
          make defconfig

      - name: Make download
        run: |
          make download -j8 || make download -j1 V=s
          rm -rf $(find ./dl/ -size -1024c)
          df -h

      - name: Compile firmware
        run: |
          make -j$(nproc) || make -j1 V=s
          echo "======================="
          echo "Space usage:"
          echo "======================="
          df -h
          echo "======================="
          du -h ./ --max-depth=1
          du -h /mnt/openwrt/ --max-depth=1 || true

      - name: Prepare artifact
        run: |
          mkdir -p ./artifact/firmware
          mkdir -p ./artifact/package
          mkdir -p ./artifact/buildinfo
          rm -rf $(find ./bin/targets/ -type d -name "packages")
          cp -rf $(find ./bin/targets/ -type f) ./artifact/firmware/
          cp -rf $(find ./bin/packages/ -type f -name "*.ipk") ./artifact/package/
          cp -rf $(find ./bin/targets/ -type f -name "*.buildinfo" -o -name "*.manifest") ./artifact/buildinfo/

      - name: Deliver buildinfo
        uses: actions/upload-artifact@v3
        with:
          name: OpenWrt_buildinfo
          path: ./artifact/buildinfo/

      - name: Deliver package
        uses: actions/upload-artifact@v3
        with:
          name: OpenWrt_package
          path: ./artifact/package/

      - name: Deliver firmware
        uses: actions/upload-artifact@v3
        with:
          name: OpenWrt_firmware
          path: ./bin/targets/

      - name: Upload release asset
        if: github.event == 'release'
        uses: svenstaro/upload-release-action@v2
        with:
          repo_token: ${{ secrets.YOURTOKEN }}
          file: ./artifact/firmware/*
          tag: ${{ github.ref }}
          file_glob: true
```
