---
layout: post
title:  Ubuntu16.04-Server-安装后的基本设置
subtitle:   Linux相关
date:   2019-04-26
author: Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
- Linux
- Ubuntu
- Server
---

>Linux知识整理

# 一、修改网卡名

## 1. 编辑接口配置文件，添加参数

```shell
sudo vi /etc/default/grub
```

### 修改前

```shell
GRUB_CMDLINE_LINUX=""
```

### 修改后

```shell
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```

## 2. 更新grub

```shell
sudo update-grub
```

*若提示没有此命令，请先输入安装命令*

```shell
sudo apt-get install grub2-common
```

## 3. 修改网卡名称并设置固定IP地址

### 修改网卡接口配置文件

```shell
sudo vi /etc/network/interfaces
```

### 修改前（网卡名不同机器会不一样，请根据实际情况修改）

```shell
# The primary network interface
auto eno13
iface eno13 inet dhcp
```

### 修改后

```shell
# The primary network interface
auto eth0
iface eth0 inet static
address 192.168.91.11
netmask 255.255.255.0
gateway 192.168.91.1
dns-nameservers 192.168.91.1
```

## 4.更换源为阿里源

### 备份源文件

```shell
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 新建配置文件

```shell
 sudo vi /etc/apt/sources.list
```

### 添加如下内容

```shell
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

## 5. 搞定，重启系统后生效