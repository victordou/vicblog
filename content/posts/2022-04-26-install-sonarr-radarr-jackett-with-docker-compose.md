---
title: "一键安装 Sonarr Radarr Jackett 追剧一条龙"
date: 2022-04-26T00:18:41+08:00
draft: false
toc: false
images:
tags: 
  - docker
  - movie
slug: install-sonarr-radarr-jackett-with-docker-compose
---

[感谢原po](https://nubcakes.net/index.php/2019/04/03/how-to-install-sonarr-radarr-and-jackett-with-docker-compose/), 我就基本照着抄就是了。

## 准备已安装 Docker, docker-compose 的服务器
偷懒下，安装教程引用[Techno Tim](https://docs.technotim.live/posts/docker-compose-install) + [官方教程](https://docs.docker.com/engine/install/)，脚本复制粘贴完事。


## 开搞

按下面操作新建文件夹
> 接下来我会使用nano编辑器，你喜欢啥用啥。

```bash
mkdir docker
mkdir docker/radarr
mkdir docker/sonarr
mkdir docker/jackett
mkdir docker/portainer
mkdir downloads
mkdir downloads/movies
mkdir downloads/tv
cd docker
touch docker-compose.yml
nano docker-compose.yml
```

#### docker-compose.yml配置

```yml
version: '3'

services:
  radarr:
    container_name: radarr
    restart: unless-stopped
    ports:
      - 7878:7878
    volumes:
      - /home/xxx/docker/radarr:/config
      - /home/xxx/downloads:/downloads
      - /home/xxx/downloads/movies:/movies
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    image: linuxserver/radarr
 
  sonarr:
    container_name: sonarr
    restart: unless-stopped
    ports:
      - 8989:8989
    volumes:
      - /home/xxx/docker/sonarr:/config
      - /home/xxx/downloads:/downloads
      - /home/xxx/downloads/tv:/tv
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    image: linuxserver/sonarr
 
  jackett:
    container_name: jackett
    restart: unless-stopped
    ports:
      - 9117:9117
    volumes:
      - /home/xxx/docker/jackett:/config
      - /home/xxx/downloads:/downloads
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    image: linuxserver/jackett
 
  portainer:
    container_name: portainer
    restart: unless-stopped
    ports:
      - 9000:9000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /home/xxx/docker/portainer:/data
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    image: portainer/portainer-ce
```


## 映射NAS SMB存储

因为我是虚拟机安装的，所以要映射下载盘

```bash
apt update
apt install cifs-utils
```
使用下面命令挂载到 /home/xxx/downloads
```
sudo mount -t cifs -o uid=1000,gid=1000,username=xxx //192.168.xxx.xxx/xxx/downloads /home/xxx/downloads
```
如果你想开机自动挂载

```
nano /etc/fstab
```
粘贴以下内容
```
//192.168.xx.xx/xxx/downloads /home/xxx/downloads cifs username=xxx,password=xxx,uid=1000,gid=1000,noauto,x-systemd.automount,_netdev,vers=2.1 0 0
```

```
sudo mount -a
```

测试一下，一切正常就可以重启试试了

## 软件如何使用？

太多了 自己看吧 https://wiki.servarr.com/