---
title: "traefik反向代理 https加密访问任何家用设备"
date: 2022-04-25T22:56:43+08:00
draft: false
toc: false
images:
tags: 
  - network
  - traefik
  - docker
slug: use-traefik-https-everything
---

好久没更新了，首发入手PS5，也没啥游戏值得玩的了，期待一下下个月的PS PLUS EXTRA，加钱升级，大方还是不如微软，但是毕竟微软是财大气粗且弱势。

话说最近看到一个[Techno Tim的视频](https://www.youtube.com/watch?v=liV3c9m_OX8)，讲到了traefik ssl everything, 我突然来劲了，家里网络已经很久没变化了，手痒了都，开搞。

众所周知，中国宽带网络是限制80,443端口的，导致视频中一些配置不符合我们的习惯，我会删除一些对我们多余的东西，比如https跳转，原教程 config.yml 中大量https跳转，但https跳转的前提是你得开http啊。

traefik 会自动更新 let's Encryt https证书，家用反向代理，自定义端口 custom port: 8443，这难道不诱人吗？

> 如何有任何不明白可访问[traefik官方Doc](https://doc.traefik.io/traefik/getting-started/quick-start/)，我自己也是看了官方资料才明白的很多的。

## 准备已安装 Docker, docker-compose 的服务器
偷懒下，安装教程引用[Techno Tim](https://docs.technotim.live/posts/docker-compose-install) + [官方教程](https://docs.docker.com/engine/install/)，脚本复制粘贴完事。

## traefik 安装开始

准备项
- 你的Cloudflare注册邮箱
- 去Cloudflare获取你的API Token，要求最低权限：All zones-Zone:Read, DNS:Edit，或者最高权限的API Key.
- 购买域名一个，并添加到Cloudflare里

> 还有其他域名提供商能够使用的[acme](https://doc.traefik.io/traefik/https/acme/)，但不适合我们这类小白

> traefik 管理页面理论上可以不加密码，但是你要发布到公网上，请一定要加密码（加密码请看配置中traefik-auth两项）

按下面操作新建文件夹

```bash
mkdir traefik
cd traefik
mkdir data
touch data/acme.json
chmod 600 data/acme.json
touch data/traefik.yml
touch data/config.yml
touch docker-compose.yml
docker network create proxy
```
（可选）生成网页密码，填入到配置中
```bash
sudo apt update
sudo apt install apache2-utils
echo $(htpasswd -nb admin 12345678) | sed -e s/\\$/\\$\\$/g
```

接下来我会使用nano编辑器，你喜欢啥用啥。

#### docker-compose.yml配置

```bash
nano docker-compose.yml
```

```yml
version: '3'

services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - 8443:8443
    environment:
      - CF_API_EMAIL=XXX@XXX.com
      - CF_DNS_API_TOKEN=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
      # - CF_API_KEY=YOU_API_KEY
      # be sure to use the correct one depending on if you are using a token or key
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /home/XXX/traefik/data/traefik.yml:/traefik.yml:ro
      - /home/XXX/traefik/data/acme.json:/acme.json
      - /home/XXX/traefik/data/config.yml:/config.yml:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=XXX:XXXXXXXXXXXXXX"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik.XXXX.com`)"
      - "traefik.http.routers.traefik-secure.middlewares=traefik-auth"
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=XXXX.com"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.XXXX.com"
      - "traefik.http.routers.traefik-secure.service=api@internal"

networks:
  proxy:
    external: true
```

#### traefik.yml配置

```bash
nano data/traefik.yml
```

```yml
api:
  dashboard: true
  debug: true
entryPoints:
  https:
    address: ":8443"
serversTransport:
  insecureSkipVerify: true
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: /config.yml
certificatesResolvers:
  cloudflare:
    acme:
      email: XXX@XXX.com
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "223.6.6.6:53"
          - "119.29.29.29:53"
```

#### config.yml其他设备配置

```bash
nano data/config.yml
```

```yml
http:
 #region routers 
  routers:
    jellyfin:
      entryPoints:
        - "https"
      rule: "Host(`jellyfin.XXXX.com`)"
      middlewares:
        - default-headers
      tls: {}
      service: jellyfin
    nextcloud:
      entryPoints:
        - "https"
      rule: "Host(`nextcloud.XXXX.com`)"
      middlewares:
        - default-headers
      tls: {}
      service: nextcloud

#endregion
#region services
  services:
    jellyfin:
      loadBalancer:
        servers:
          - url: "http://192.168.xx.xx:8096"
        passHostHeader: true

    nextcloud:
      loadBalancer:
        servers:
          - url: "https://192.168.xx.xx:9443"
        passHostHeader: true

    default-headers:
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 15552000
        customFrameOptionsValue: SAMEORIGIN
        customRequestHeaders:
          X-Forwarded-Proto: https

    default-whitelist:
      ipWhiteList:
        sourceRange:
        - "10.0.0.0/8"
        - "192.168.0.0/16"
        - "172.16.0.0/12"

    secured:
      chain:
        middlewares:
        - default-whitelist
        - default-headers
```
## 启动traefik
> Warning!!! 由于本次使用的是dns来获取证书，请大家关闭openwrt防火墙中的自定义规则（劫持DNS至openwrt），因为有冲突。
```bash
docker-compose up -d
```

> 如何你之前配置配错了，修改配置后重新启动docker命令
```bash
docker-compose up -d --force-recreate
```

感兴趣的可以去看看原po，和我的区别，我已经帮大家把https改成8443，支持cloudflare隐藏你的IP(但速度慢)。
推荐大家准备两台 traefik，一台https://xxx.local.xxxx.com, 专门给内部网络访问 不开外部权限；另一台就是教程中https://xxx.xxx.com:8443.

### 内部域名批量解析
且你是openwrt，使用下面配置

```bash
nano /etc/dnsmasq.conf
```
文件末尾填入
```
address=/.local.xxx.com/192.168.xx.xx
```

## 外部部域名批量解析
> 记得开启路由器端口8443到你的docker服务器ip地址上
Cloudflare 新增域名解析到你的路由器DDNS上

### 安装Portainer
看[原po吧](https://docs.technotim.live/posts/traefik-portainer-ssl/)

### 新装的docker怎么加反代
理论上新docker只要加上下面这段，都会自动加入到traefik清单里

```
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.portainer-secure.entrypoints=https"
      - "traefik.http.routers.portainer-secure.rule=Host(`portainer.local.example.com`)"
      - "traefik.http.routers.portainer-secure.tls=true"
      - "traefik.http.routers.portainer-secure.service=portainer"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
      - "traefik.docker.network=proxy"
```