---
layout:     post
title:      PVE安装后要做的
subtitle:   虚拟化
date:       2019-05-31
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Linux
    - PVE
    - Debian
---

>Linux 知识库

# 修改bash设置

# 去掉企业订阅，注释掉下面配置文件的内容

```sh

vi /etc/apt/sources.list.d/pve-enterprise.list

```

# 增加PVE源并更新，如有内核更新需重启

```sh

echo "deb http://download.proxmox.com/debian/pve stretch pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
wget http://download.proxmox.com/debian/proxmox-ve-release-5.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-5.x.gpg

apt update -y
apt dist-upgrade -y

```

# 打开Pci passthrough，[官方说明网址](https://pve.proxmox.com/wiki/Pci_passthrough)

## edit:

```sh

vi /etc/default/grub

```

## change

GRUB_CMDLINE_LINUX_DEFAULT="quiet"

## to

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"

## then

```sh
 
 update-grub

```

## add to /etc/modules

```

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

```