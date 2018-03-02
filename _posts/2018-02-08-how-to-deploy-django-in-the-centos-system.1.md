---
title: Centos6.5升级Python2.7
date: 2018-03-02 23:26:00
categories: 
- CentOS
- Blog
tags: [Centos]
author: wangll
description: 将系统Python2.6版本升级至2.7
---
# Centos配置生产环境Django部署

## 升级CentOS，安装开发者工具

``` bash
yum -y update
yum groupinstall -y 'development tools'
```

安装`SSL`, `bz2`, `zlib`等工具

``` bash
yum install -y zlib-devel bzip2-devel openssl-devel xz-libs wget
```

## 源码安装Python2.7

``` bash
wget https://www.python.org/ftp/python/2.7.14/Python-2.7.14.tgz
tar zxvf Python-2.7.14.tgz
cd Python-2.7.14
./configure
make
make altinstall

# 环境验证
/usr/local/bin/python2.7 -V
```

## 配置

``` bash
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
```

检查版本
`python -V`

解决系统 Python 软链接指向 Python2.7 版本后，因为yum是不兼容 Python 2.7的，所以yum不能正常工作，我们需要指定 yum 的Python版本

`vim /usr/bin/yum`
将文件头部的`#!/usr/bin/python`改成`#!/usr/bin/python2.6.6`