---
layout:     post
title:      配置VIM使用molokai配色
subtitle:   日常维护
date:       2019-05-30
author:     Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Linux
    - Shell
    - VIM
---

>Linux 知识库

# 进入主目录执行下面脚本

```shell

cd ~
mkdir .vim
cd .vim
git clone https://github.com/tomasr/molokai.git
cp -rf molokai/colors/ ./colors
echo colorscheme molokai >> vimrc
echo set t_Co=256 >> vimrc
echo set background=dark  >> vimrc

```