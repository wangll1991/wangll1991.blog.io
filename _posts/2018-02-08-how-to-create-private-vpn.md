---
title: 搭建自己的VPS服务
date: 2018-02-11 21:20:09
categories: 
- VPN
- Blog
tags: [VPN]
author: wangll
description: 基于centos搭建自己的VPS服务。
---
# 私有VPN搭建

## 租用私有服务器
略

## 部署Shadowsocks服务
### Github地址
Server：https://github.com/shadowsocks/shadowsocks/tree/master

Client for windows : https://github.com/shadowsocks/shadowsocks-windows 

IOS:    App Stores搜索Shadowrocket(需要支付12元)

### Deploy
CentOS:
``` bash
yum install python-setuptools && easy_install pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

### Run for config
Create a config file /etc/shadowsocks.json. Example:
``` json
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
To run in the foreground:

``` shell
ssserver -c /etc/shadowsocks.json
```

To run in the background:

``` shell
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```
## 运行Shadowsocks客户端
参考[中文文档指定](https://github.com/shadowsocks/shadowsocks-windows/wiki/Shadowsocks-Windows-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)


## 在IOS上进行访问
参考[伙伴的指定文档](https://fanach.github.io/post/ss-ios/)
