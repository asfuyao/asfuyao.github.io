---
layout: post
title:  cmder下git-for-windows乱码解决方案
subtitle:   git相关
date:   2019-04-26
author: Winds
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
- Cmd
- Git
- Windows
---

>git知识整理

# 1. git status 时中文文件名乱码

## 现象：

```bat
\344\275\240\345\245\275
```

## 执行以下命令即可：

```bash
git config --global core.quotepath false
```

quotepath解释：

- The commands that output paths (e.g. ls-files, diff), when not given the -z option,will quote "unusual" characters in the pathname by enclosing the pathname in a double-quote pair and with backslashes the same way strings in C source code are quoted. If this variable is set to false, the bytes higher than 0x80 are not quoted but output as verbatim. Note that double quote, backslash and control characters are always quoted without -z regardless of the setting of this variable.

# 2. git log 查看提交中含中文乱码

## 现象：

```bat
<E4><BF><AE><E6>
```

## 修改git全局配置设置提交和查看日志编码都是utf-8

```bash
git config --global i18n.commitencoding utf-8 
git config --global i18n.logoutputencoding utf-8
```

## 修改git目录下etc\profile文件，设置less的字符集为utf-8

```bash
export LESSCHARSET=utf-8
```

## 修改cmder目录vendor\init.bat文件，添加以下代码,设定cmder编码为utf-8

```bash
@chcp 65001 > nul
```

chcp 65001的解释：

[Why is there no option to choose codepage 65001 (UTF-8) as a default codepage in console window](http://superuser.com/questions/692202/why-is-there-no-option-to-choose-codepage-65001-utf-8-as-a-default-codepage-in/692230#692230)

# 3. gitk 查看中文乱码

```bash
git config --global gui.encoding utf-8
```