---
title: Centos配置生产环境Django部署
date: 2018-02-08 03:18:09
categories: 
- Django
- Blog
tags: [Django]
author: wangll
description: 基于centos部署Django环境。
---
# Centos配置生产环境Django部署

*须知：*
*1. centos自带python2.7*
*2. 此教程适合大多数linux系统，本文以centos7.2为例*

## 安装基本环境

``` bash
yum groupinstall Development tools
yum install zlib-devel bzip2-devel pcre-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
```

## 升级Python3

``` bash
# 下载编译安装
wget http://python.org/ftp/python/3.5.2/Python-3.5.2.tgz
tar xf Python-3.5.2.tgz
cd Python-3.5.2
./configure --prefix=/data/SicentApp/python3 --with-dbmliborder=gdbm --with-ssl
make
make install
# 添加至环境变量
export PATH=/data/SicentApp/python3/bin/:$PATH
# 环境验证
which python3
pip3 -V
```

## 搭建虚拟环境

``` bash
pip3 install virtualenv
# 环境验证
pip3 list | grep virtualenv
```

## 安装 [uwsgi](http://uwsgi-docs.readthedocs.io/en/latest/Options.html)

``` bash
pip3 install uwsgi==2.0.13.1
# 环境验证
pip3 list | grep uwsgi

# 测试 uwsgi 是否正常：
#新建 test.py 文件，内容如下：

def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return "Hello World"
# 然后在终端运行：

uwsgi --http :8001 --wsgi-file test.py
#在浏览器内输入：http://127.0.0.1:8001，查看是否有"Hello World"输出，若没有输出，请检查你的安装过程。
```

## 安装 Django

``` bash
pip3 install django==2.0
# 测试 django 是否正常，运行：

django-admin.py startproject demosite
cd demosite
python3 manage.py runserver 0.0.0.0:8002
# 在浏览器内输入：http://{localhost}:8002，检查django是否运行正常。
# 如果提示Allow hosts相关提示，则说明远程服务器的IP需求被添加至白名单列表。移步至项目目录下修改setting.py文件
...
ALLOWED_HOSTS = ["192.168.56.101"]
...
```

## 安装[Nginx](http://www.runoob.com/linux/nginx-install-setup.html)

``` bash
cd ~
wget http://nginx.org/download/nginx-1.5.6.tar.gz
tar xf nginx-1.5.6.tar.gz
cd nginx-1.5.6
./configure --prefix=/etc/nginx --with-http_stub_status_module --with-http_gzip_static_module
make
make install
```

## uwsgi 配置

uwsgi支持ini、xml等多种配置方式，本文以 ini 为例， 在/data/SicentWebserver/目录下新建uwsgi9090.ini，添加如下配置：

``` bash
[uwsgi]
socket = 127.0.0.1:9090
master = true         //主进程
vhost = true          //多站模式
no-site = true        //多站模式时不设置入口模块和文件
workers = 2           //子进程数
reload-mercy = 10
vacuum = true         //退出、重启时清理文件
max-requests = 1000
limit-as = 512
buffer-size = 30000
pidfile = /var/run/uwsgi9090.pid    //pid文件，用于下面的脚本启动、停止该进程
daemonize = /data/SicentWebserver/uwsgi9090.log
```

## nginx配置

找到nginx的安装目录（如：/usr/local/nginx/），打开conf/nginx.conf文件，修改server配置：

``` bash
server {
        listen       80;
        server_name  localhost;

        location / {
            include  uwsgi_params;
            uwsgi_pass  192.168.56.101:9090;              //必须和uwsgi中的设置一致
            uwsgi_param UWSGI_SCRIPT demosite.wsgi;       //入口文件，即wsgi.py相对于项目根目录的位置，“.”相当于一层目录
            uwsgi_param UWSGI_CHDIR /demosite;       //项目根目录
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }
# 设置完成后，在终端运行：

uwsgi --ini /etc/uwsgi9090.ini &
/usr/local/nginx/sbin/nginx
# 在浏览器输入：http://192.168.56.101，你就可以看到 django 的 "It work" 了。
```