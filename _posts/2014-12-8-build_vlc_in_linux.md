---
layout: post
title: Linux编译原生vlc
category: linux
---

##下载源码

	#!/bin/bash
	#安装依赖工具
	apt-get install git libtool build-essential pkg-config autoconf
	apt-get install subversion yasm cvs cmake
	#下载源码
	git clone git://git.videolan.org/vlc.git
	cd vlc
	./bootstrap

也可以直接下载源码包，经测试，会快很多。

##安装依赖库

	# 注意不是apt-get install build-dep,另外这个命令会去墙外的地址下东西，代理是必须的
	apt-get build-dep vlc
	cd contrib
	mkdir native
	cd native
	../bootstrap
	make

##编译

运行命令**./configure --help**，根据你的功能需要，配置选项。例如

	./configure --disable-sout \
		--enable-static=false \
		--disable-nls \
		--disable-debug \
		--disable-gprof \
		--disable-cprof \
		--disable-coverage \
		--disable-lua \
		--disable-httpd \
		--disable-dc1394 \
		--disable-dv1394 \
		--disable-dvdread \
		--disable-dvdnav \
		--disable-bluray \
		--disable-smbclient \
		--disable-sftp \
		--disable-vcd \
		--disable-vcdx \
		--disable-screen \
		--disable-vnc \
		--disable-freerdp \
		--disable-chromaprint \
		--disable-chromecast
	make

编译结束，会生成vlc、vlcstatic等可执行文件

##参考
1. <https://wiki.videolan.org/UnixCompile/>
