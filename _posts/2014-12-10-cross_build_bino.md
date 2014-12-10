---
layout: post
title: Bino播放器编译
category: linux
---

bino是一个开源的3D播放器，支持红蓝和左右的3D视频。

bino for windows使用mingw编译，并且官方文档使用一个**[mxe](http://mxe.cc/)**的框架来进行编译。而后者只能运行*nix下。所以需要准备一个Linux（或者其他*nix）。笔者使用debian进行编译。

##MXE

(M cross environment) is a Makefile that compiles a cross compiler and cross compiles many free libraries such as SDL and Qt.

简单的说，可以理解为mxe是一对自动化脚本，用它可以比较轻松的（选择性地）安装mingw，以及第三方库。其支持的库见<http://mxe.cc/#packages>

在安装mxe之前，先参考系统要求，见<http://mxe.cc/#requirements>，例如[Debian](http://mxe.cc/#requirements-debian)。

	apt-get install autoconf automake bash bison bzip2 \
						cmake flex gettext git g++ intltool \
						libffi-dev libtool libltdl-dev libssl-dev \
						libxml-parser-perl make openssl patch perl \
						pkg-config scons sed unzip wget xz-utils
						
	# On 64-bit Debian, install also:
	apt-get install g++-multilib libc6-dev-i386

##搭建bino编译环境

这个过程中，会网络下载必须的包。
	
	$ cd /path/to
	$ git clone -b master https://github.com/mxe/mxe.git
	$ cd mxe
	$ make gettext pthreads ffmpeg libass openal glew qt nsis
	
##编译bion

在编译之前，先确定mxe的体系名称，例如**i686-w64-mingw32.static**

	root@kingqiu:/home/king/bino# ls -l /home/king/mxe/usr/
	total 28
	drwxr-xr-x  2 root root 4096 Dec  9 14:32 bin
	drwxr-xr-x 11 root root 4096 Dec  9 13:11 i686-w64-mingw32.static

---

	$ cd /path/to/bino
	$ autoreconf -i
	$ ./configure --host=i686-w64-mingw32.static --build=`build-aux/config.guess` --target=i686-w64-mingw32.static
	$ make
	
笔者在编译过程中，还碰到一些代码编译错误，以及若干静态库(.a)符号重复的问题。代码错误只能老实改掉。静态库重复，笔者的做法如下

将.a中重复的.o/.obj其中一方删掉，然后把删掉的.o/.obj中未重复的代码放到bino源码中一起编译（google搜对应库的源码）。

修改.a方法和ar是一样的。

1. mxe/usr/bin/i686-w64-mingw32.static-ar -t .a 查看内容
1. mxe/usr/bin/i686-w64-mingw32.static-ar -d .a .o/.obj 删除其中的.o
	
##参考
1. <http://bino3d.org/>
1. <http://mxe.cc/>