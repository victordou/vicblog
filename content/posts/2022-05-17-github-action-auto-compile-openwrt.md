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
<https://github.com/coolsnowwolf/lede>

<https://github.com/KFERMercer/OpenWrt-CI>

## 注册Github
本步骤跳过，网上一大堆

## Fork Openwrt
<https://github.com/coolsnowwolf/lede/fork>

## 开启Github Action
1. 进入你 `fork` 成功后的lede/openwrt
2. 进入 `Action`
3. 阅读后打勾，启用 `Action`

## 准备Workflows配置文件
1. 打开[KFERMercer OpenWrt-CI.yml](https://github.com/KFERMercer/OpenWrt-CI/blob/master/openwrt-ci.yml)
2. 复制全部内容

## 编辑Workflows配置文件
1. 返回你的OpenWrt的 `Code` 页面
2. 依次打开 `.github/workflows/` 文件夹
3. 打开 `openwrt-ci.yml`
4. 粘贴你之前复制的.yml内容
5. 根据你的要求自定义yml的内容
6. 修改完成 `Start commit` -> `Commit changes`

## 此时你的 OpenWrt 会自动开始编译
时长越两小时，下载OpenWrt_firmware即可

## 如何自定义固件
1. 找到你需要安装的包名，官方<https://openwrt.org/packages/index/start>
2. Lean/coolsnowwolf 的软件包可以从 make defconfig 配置文件 `.config` 中找到
3. 添加包到`openwrt-ci.yml` 中 `cat >> .config <<EOF`后面

以下删除网易云+迅雷快鸟，并添加一堆自用软件baidupcs, mwan3, ipv6, DOH, Cloudflare DDNS, SMB映射等。
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
      - master
  # schedule:
  #   - cron: 0 20 * * *
  release:
    types: [published]

jobs:

  build_openwrt:

    name: Build OpenWrt firmware

    runs-on: ubuntu-latest

    if: github.event.repository.owner.id == github.event.sender.id

    steps:

      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: master

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

          # sudo mkdir -p -m 777 /mnt/openwrt/bin /mnt/openwrt/build_dir/host /mnt/openwrt/build_dir/hostpkg /mnt/openwrt/dl /mnt/openwrt/feeds /mnt/openwrt/staging_dir
          # ln -s /mnt/openwrt/bin ./bin
          # mkdir -p ./build_dir
          # ln -s -f /mnt/openwrt/build_dir/host ./build_dir/host
          # ln -s -f /mnt/openwrt/build_dir/hostpkg ./build_dir/hostpkg
          # ln -s /mnt/openwrt/dl ./dl
          # ln -s /mnt/openwrt/feeds ./feeds
          # ln -s /mnt/openwrt/staging_dir ./staging_dir

          df -h

      - name: Update feeds
        run: |
          echo "src-git helloworld https://github.com/fw876/helloworld.git" >> "feeds.conf.default"
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
          CONFIG_LUCI_LANG_en=y
          CONFIG_LUCI_LANG_zh-cn=y
          CONFIG_PACKAGE_luci-app-baidupcs-web=y
          CONFIG_PACKAGE_luci-app-cifs-mount=y
          CONFIG_PACKAGE_luci-app-wireguard=y
          CONFIG_PACKAGE_luci-app-https-dns-proxy=y
          CONFIG_PACKAGE_luci-app-netdata=y
          # CONFIG_PACKAGE_luci-app-ttyd is not set
          CONFIG_PACKAGE_luci-app-ssr-plus=y
          CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_IPT2Socks=y
          CONFIG_PACKAGE_luci-app-ssr-plus_INCLUDE_Trojan=y
          # CONFIG_PACKAGE_luci-app-unblockmusic is not set
          CONFIG_PACKAGE_luci-app-uugamebooster=y
          # CONFIG_PACKAGE_luci-app-xlnetacc is not set
          CONFIG_PACKAGE_luci-app-mwan3helper=y
          CONFIG_PACKAGE_luci-theme-bootstrap=y
          CONFIG_PACKAGE_luci-theme-material=y
          CONFIG_PACKAGE_ddns-scripts_cloudflare.com-v4=y
          CONFIG_PACKAGE_ddns-scripts_no-ip_com=y
          CONFIG_PACKAGE_ddns-scripts_route53-v1=y
          CONFIG_PACKAGE_kmod-macvlan=y
          CONFIG_PACKAGE_bind-dig=y
          CONFIG_PACKAGE_nano=y
          CONFIG_PACKAGE_qrencode=y
          CONFIG_PACKAGE_open-vm-tools=y
          CONFIG_PACKAGE_whois=y
          CONFIG_PACKAGE_nmap-ssl=y
          CONFIG_PACKAGE_ipv6helper=y
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
        uses: actions/upload-artifact@v2
        with:
          name: OpenWrt_buildinfo
          path: ./artifact/buildinfo/

      - name: Deliver package
        uses: actions/upload-artifact@v2
        with:
          name: OpenWrt_package
          path: ./artifact/package/

      - name: Deliver firmware
        uses: actions/upload-artifact@v2
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

## 添加SSR PLUS
我上面的配置已经添加了[这一段命令](https://github.com/fw876/helloworld)
```bash
echo "src-git helloworld https://github.com/fw876/helloworld.git" >> "feeds.conf.default"
```