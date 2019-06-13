---
layout:     post
title:      Docker快速部署v2ray翻墙
subtitle:   翻墙
date:       2019-06-13
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Linux
    - VPS
    - Docker
    - v2ray
---

>翻墙

# 前言

最近SS和SSR被封的比较厉害，好VPS的IP被封了，于是转向V2ray + WebSocket + TLS + Web，目前已知最安全的翻墙方案，本方案采用Nginx作代理，Caddy没测试成功

# 安装配置步骤

## 前提条件

- 一个VPS服务器（KVM），安装Debian、Ubuntu、CentOS
- 一个域名(非动态域名)
- 申请免费域名证书
- 服务器安装Docker，请上官网看[安装Docker教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/#extra-steps-for-aufs)
- 安装docker-compose

## 准备脚本和证书

### 创建一个文件夹把所有文件放到一个文件夹下

文件夹和文件结构：
- v2rayn/
  - conf.d/
    - v2ray.conf
  - keys/
    - v2ray.crt
    - v2ray.key
  - config.json
  - docker-compose.yml
  - nginx.conf

其中v2ray.crt和v2ray.key两个证书文件是事先申请好的nginx专用域名证书，证书文件名改成这样就可以。

### config.json

```json

{
  "log" : {
     "access": "/var/log/v2ray/access.log",
     "error": "/var/log/v2ray/error.log",
     "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 8002, # v2ray监听端口号，与nginx配置保持一致
      "listen":"0.0.0.0", #v2ray监听地址，不需要改动
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "UUID", #此处请用自己的UUID替换www.uuidgenerator.net
            "level": 1,
            "alterId": 1  #客户端需保持一致
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/UUID" #websocket路径，可任意输入与nginx配置保持一致，建议也用UUID
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}

```

### nginx.conf，标准配置不需要修改

```nginx

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

### v2ray.conf，nginx站点配置文件

```nginx

server {
  listen  443 ssl; #监听地址
  ssl_certificate       /etc/v2ray/v2ray.crt; #证书文件，路径不需要修改
  ssl_certificate_key   /etc/v2ray/v2ray.key; #证书文件，路径不需要修改
  ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers           HIGH:!aNULL:!MD5;
  server_name           xxxxx.xxx; #此处用自己的域名替换
        location /UUID { #websocket路径和v2ray配置保持一致
        proxy_redirect off;
        proxy_pass http://v2ray:8002; #websocket地址和端口号，此处地址为docker在links中映射的主机名，端口号为v2ray监听地址
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        }
}

```

### docker-compose.yml

```docker

version: '2'

services:
  v2ray:
    image: v2ray/official  #v2ray镜像名称
    container_name: v2ray  #v2ray容器名称
    volumes:
    - ./config.json:/etc/v2ray/config.json #映射v2ray配置文件路径
    - ./log/v2ray:/var/log/v2ray #映射v2ray日志文件路径
    expose:
    - "8002" #开放端口给容器，不对外开放
    restart: unless-stopped
  nginx:
    image: nginx:stable-alpine #nginx镜像名称
    container_name: v2ray_nginx #nginx容器名称
    volumes:
      - ./keys:/etc/v2ray  #映射证书文件路径
      - ./nginx.conf:/etc/nginx/nginx.conf  #映射nginx主配置文件
      - ./conf.d:/etc/nginx/conf.d #映射nginx其他配置文件路径
      - ./html:/usr/share/nginx/html #映射nginx页面路径
      - ./log/nginx:/var/log/nginx #映射nginx日志路径
    ports:
    - "443:443" #开放443端口
    links:
    - v2ray:v2ray #连接v2ray容器，映射主机名，此处非常重要，如果没有这个映射nginx就无法连接v2ray的websocket服务
    restart: unless-stopped
networks:
  default:
    external:
      name: v2ray #设置容器使用的默认网络

```

## 文件准备好后整个目录一起上传到服务器任意目录

* 此方案使用的路径都是相对路径，文件夹放到任意地方都可以

## 进入目录运行容器

```shel

docker-compose up -d

```

## 安装BBR内核，可选步骤，可以提高速度

```shell

wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh

```
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：`lsmod | grep bbr`。如果出现tcp_bbr字样表示bbr已安装并启动成功。

## 配置客户端

* windows客户端：https://github.com/2dust/v2rayN

* Android客户端：https://github.com/2dust/v2rayNG/releases

## 客户端配置文件样例

```json

{
  "inbounds": [
    {
      "port": 1080,  #客户端监听端口号，可根据实际情况修改
      "listen": "127.0.0.1",
      "protocol": "socks",
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "xxxxx.xxx", #此处用自己的域名替换
            "port": 443, #服务器端nginx监听端口
            "users": [
              {
                "id": "UUID", #和服务器设置保持一致
                "alterId": 1 #和服务器设置保存一致
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/UUID" #websockert路径，和服务器保持一致
        }
      }
    }
  ]
}

```
