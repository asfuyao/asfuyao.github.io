---
layout:     post
title:      搬瓦工VPS使用Docker部署Shadowsocks和Kcptun
subtitle:   翻墙
date:       2019-01-01
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - linux
    - Shadowsocks
    - Kcptun
---

>快速部署翻墙服务

# 服务器安装Ubuntu16.04_x64

## 1. 添加用户

```shell
adduser 用户名
```

## 2. 给用户管理员权限

```shell
chmod +w /etc/sudoers
vi /etc/sudoers
```

- 添加新行：用户名 ALL=(ALL:ALL) ALL
- 重启系统用新用户登录

## 3. 更新系统

```shell
sudo apt update
sudo apt upgrade
sudo reboot
```

## 4. 使用Docker官方脚本安装docker ce

```shell
sudo wget -qO- get.docker.com | bash
```

## 5. 把当前用户添加到docker组

```shell
sudo usermod -aG docker 用户名
```

## 6. 设置docker开机自动运行

```shell
sudo systemctl enable docker
```

## 7. 注销重新登录，获取shadowsocks和kcptun官方镜像

```shell
docker pull shadowsocks/shadowsocks-libev
docker pull xtaci/kcptun
```

## 8. 启动shadowsocks（默认加密方法是aes-256-gcm）

```shell
docker run --name shadowsocks -e PASSWORD=密码 -p shadowsocks端口:8388 -p 18548:8388/udp -d --restart=always shadowsocks/shadowsocks-libev
```

## 9. 启动kcptun

```shell
docker run --name kcptun -p 29900:29900/udp -d --restart=always xtaci/kcptun server -t "服务器IP:shadowsocks端口" -l ":29900" -key "kcptun密码" -crypt aes -datashard 10 -parityshard 3 -mtu 1350 -sndwnd 512 -rcvwnd 512 -dscp 0 -mode fast2
```

[kcptun的其他参数可参考](https://hub.docker.com/r/xtaci/kcptun)

## 10. 客户端下载地址

[shadowsocksWindows 客户端](https://github.com/shadowsocks/shadowsocks-windows/releases)

[kcptun图形化客户端](https://github.com/dfdragon/kcptun_gclient/releases)

[kcptun官方发布下载](https://github.com/xtaci/kcptun/releases)