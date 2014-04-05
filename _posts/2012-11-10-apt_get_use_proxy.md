---
layout: post
title:  apt-get 使用代理
category: other
---

##参考
1. <http://www.cnblogs.com/babykick/archive/2011/03/25/1996004.html>

##设置代理
在最新Ubuntu中，原来的老办法似乎失效（对于wget还可用）

        export http_proxy=http://127.0.0.1:8000
        sudo apt-get update

找了下，发现可以修改/etc/apt/apt.conf，不过似乎也不行，因为没那个文件

        Acquire::http::proxy "http://127.0.0.1:8000/";
        Acquire::ftp::proxy "ftp://127.0.0.1:8000/";
        Acquire::https::proxy "https://127.0.0.1:8000/";
        
最后可以直接用apt-get的参数指定代理

        sudo apt-get -o Acquire::http::proxy="http://127.0.0.1:8000/" update
