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

## 本次教程基于下列前车之鉴
<https://github.com/immortalwrt/immortalwrt>

<https://github.com/KFERMercer/OpenWrt-CI>

## 注册Github
本步骤跳过，网上一大堆

## Fork Openwrt
<https://github.com/immortalwrt/immortalwrt/fork>

## 开启Github Action
1. 进入你 `fork` 成功后的lede/openwrt
2. 进入 `Action`
3. 阅读后打勾，启用 `Action`

## 准备Workflows配置文件
1. 复制下面的下面的`OpenWrt-CI.yml`全部内容

## 编辑Workflows配置文件
1. 返回你的OpenWrt的 `Code` 页面
2. 打开Action页面，会自动新建 `.github/workflows/` 文件夹
3. 新增或修改 `openwrt-ci.yml`文件
4. 粘贴你之前复制的.yml内容
5. 自定义yml的内容，修改branch: `openwrt-21.02` 或 ``master`
6. 修改完成 `Start commit` -> `Commit changes`

## 此时你的 OpenWrt 会自动开始编译
时长越两小时，完成后，去Action页面下载OpenWrt_firmware即可

## 如何自定义固件
1. 找到你需要安装的包名，官方<https://openwrt.org/packages/index/start>
2. 软件包可以从 make defconfig 配置文件 `.config` 中找到
3. 添加包名到`openwrt-ci.yml` 中 `cat >> .config <<EOF`后面

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
  release:
    types: [published]

jobs:

  build_openwrt:

    name: Build OpenWrt firmware

    runs-on: ubuntu-20.04

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
          sudo -E apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs gcc-multilib g++-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler antlr3 gperf swig libtinfo5
          sudo -E apt-get -y autoremove --purge
          sudo -E apt-get clean
          df -h

      - name: Update feeds
        run: |
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
          # CONFIG_TARGET_ROOTFS_EXT4FS is not set
          CONFIG_VMDK_IMAGES=y
          # CONFIG_TARGET_IMAGES_GZIP is not set
          CONFIG_LUCI_LANG_en=y
          CONFIG_LUCI_LANG_zh_Hans=y
          CONFIG_PACKAGE_luci=y
          CONFIG_PACKAGE_ipv6helper=y
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
          CONFIG_PACKAGE_snmpd=y
          CONFIG_PACKAGE_nano=y
          CONFIG_PACKAGE_open-vm-tools=y
          CONFIG_PACKAGE_qrencode=y
          CONFIG_PACKAGE_whois=y
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

## 如何手动开始编译，不需要手动commit
1. 先更新你的OpenWrt源-> `Fetch upstream` -> `Fetch Merge`
2. 修改 `openwrt-ci.yml`
```yml
on:
  push:
    branches: 
      - master
```
改成
```yml
on:
  workflow_dispatch:
```
3. 手动开始编译 `Action` -> `OpenWrt-CI` -> `Run workflow`

Updated in 2022/12/12