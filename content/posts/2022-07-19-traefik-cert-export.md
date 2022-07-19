---
title: "traefik acme 证书导出"
date: 2022-07-19T17:03:20+08:00
draft: false
toc: false
images:
tags: 
  - traefik
slug: traefik-cert-export
---

来自 <https://blog.cubieserver.de/2021/minimal-traefik-v2-certificate-export/>

目前有很多方式导出证书，docker可以自动化，自己选择吧。
* https://github.com/ldez/traefik-certs-dumper (Go)
* https://github.com/koshatul/traefik-acme (Go)
* https://gist.github.com/dmnt3d/a9696d1590df0a1410be8954df15f59c (Python)
* https://r4uch.com/export-traefik-certificates/ (Bash)

如何使用bash脚本手动导出证书？
## 要求
1. Linux 系统的虚拟机
2. traefik 生成的acme.json
3. Linux 安装了 jq + openssl

## 开始操作

1. 在acme.json目录复制粘贴新建以下脚本，命名随意，如tc.sh
```bash
#!/bin/bash

# Requirements: you will need to install jq and maybe openssl

# creates a directory for all of your certificates
mkdir -p certificates/

# reads the acme.json file, please put this file in the same directory as your script
json=$(cat acme.json)

export_cer_key () {
    echo $json | jq -r '.[].Certificates[] | select(.domain.main == "'$1'") | .certificate' | base64 -d > certificates/$1.cer
    echo $json | jq -r '.[].Certificates[] | select(.domain.main == "'$1'") | .key' | base64 -d > certificates/$1.key
}

export_pfx () {
        openssl pkcs12 -export -out certificates/$domain.pfx -inkey certificates/$domain.key -in certificates/$domain.cer -passout pass: 
}

read -p "Do you want to export as .pfx file as well [y]?" REPLY

# iterates through all of your domains
for domain in $(echo $json | jq -r '.[].Certificates[].domain.main')
do
    if [[ $REPLY =~ ^[Yy]$ ]]
    then
        export_cer_key "$domain"
        export_pfx "$domain"
    else
        export_cer_key "$domain"
    fi
done

```

2. 对刚新建的脚本添加执行权限
```bash
chmod +x tc.sh
```

3. 确认同目录下同时存在tc.sh 和 acme.json
```
traefik
├── tc.sh
└── acme.json
```

4. 执行脚本
```bash
./tc.sh
```

5. 可选导出 pfx 格式证书，证书会出现在 certificates 文件夹里面
```
traefik
├── certificates
│   ├── cert.cer
│   ├── cert.key
│   └── cert.pfx
├── tc.sh
└── acme.json
```

### 假如以上是半自动证书导出，下面还有全手动的方法。

1. 复制 acme.json 中 certificates 那一行
2. 粘贴到 [Base64解码](https://oktools.net/base64) 中解码栏
3. 文本编辑器保存解码结果为 cert.crt
4. key同理，保存为 cert.key