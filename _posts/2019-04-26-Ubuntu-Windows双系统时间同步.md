---
layout: post
title:  Ubuntu-Windows双系统时间同步
subtitle:   Linux相关
date:   2019-04-26
author: Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
- Linux
- Ubuntu
- Npt
---

# 前言：

- 安装windows10和Ubuntu双系统后会出现时间不同步的情况，往往windows系统的时间会有错误，一般会有8个小时的误差。

# 原因：

- 主要因为本地时间与硬件时间的时差
- 本地时间是操作系统上的时间
- 硬件时间是计算机的BIOS时间
- 因为双系统装在同一个计算机上，所以windows与ubuntu的硬件时间是一定相同的，因此出现上述问题的原因是两个系统的本地时间不同。
 
# 双系统本地时间不同的原因：

- windows中本地时间与硬件时间相同，当修改windows系统时间（本地时间）时，实际上计算机硬件时间也随之变为本地时间。
- ubuntu等linux发行版的本地时间与硬件时间不同，硬件时间使用UTC时间，即协调世界时(Coordinate Universal Time)，中国与UTC的时差为+8，即UTC+8，因此本地时间与硬件时间有8小时的时差。
- 所以，当winows与ubuntu的本地时间--硬件时间转换关系不同时，一定会出现时间不同步问题。

# ubuntu18.04解决办法：

## 设置硬件时间为本地时间

```shell
sudo hwclock --localtime --systohc
```

## 安装ntpdate：

```shell
sudo apt-get install ntpdate
```

## 设置校正服务器

```shell
sudo ntpdate 0.cn.pool.ntp.org
```

npt服务器：China — cn.pool.ntp.org

```url
0.cn.pool.ntp.org
1.cn.pool.ntp.org
2.cn.pool.ntp.org
3.cn.pool.ntp.org
```

# ubuntu16.04解决办法：

```shell
timedatectl set-local-rtc 1 --adjust-system-clock
```

# ubuntu14.04解决办法：
- 编辑/etc/default/rcS 将UTC=yes改成UTC=no ，这是以前的方法，新版本的Ubuntu使用systemd启动之后，时间也改成了由timedatectl来管理，此方法就不适用了