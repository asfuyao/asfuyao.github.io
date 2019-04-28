---
layout: post
title:  Linux下用git连接github
subtitle:   Linux相关
date:   2019-04-26
author: Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
- Linux
- Github
- Git
---

# 生产密匙

```shell
ssh-keygen -t rsa -C "your_email@example.com"
```

# 查看公共密匙文件，会要求输入密码

```shell
cat ~/.ssh/id_rsa.pub
```

# 把密匙内容复制到Github

- 进入 Github的Settings
- 点击 SSH and GPG keys
- 点击 New SSH key，将密匙内容粘贴进来

# 配置git属性

```shell
git config --global user.name "your_name"
git config --global user.email "your_email@example.com"
```

# 然后就可以正常使用github了，首次使用会要求输入秘钥的密码

```shell
git clone git@github.com:your_name/project_url
```