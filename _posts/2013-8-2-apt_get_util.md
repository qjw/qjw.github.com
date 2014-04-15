---
layout: post
title: shell常用软件 和 配置
category: other
---
	
	#!/bin/bash
	# 查看命令属于哪个包 dpkg -S  $COMMAND
	apt-get install coreutils grep sed mawk
	apt-get install findutils #find xargs du
	apt-get install openssh-client openssh-server
	# libc-bin  ldd
	# http://www.ibm.com/developerworks/cn/linux/l-tsl/
	apt-get install build-essential gdb strace ltrace 
	apt-get install psmisc #top vmstat pstree
	apt-get install sysstat #iostat mpstat sar pidstat
	apt-get install libmudflap0 libmudflap0-4.4-dev
	apt-get install vim ctags diffutils patch file
	apt-get install subversion git
	apt-get install python2.6 lua5.1 sun-java6-jre nodejs
	apt-get install wget curl tcpdump dnsutils nmap telnet
	apt-get install netcat-traditional lftp iputils-ping
	apt-get install smbfs #@support windows share file system
	apt-get install zip p7zip p7zip-rar
	apt-get install ethtool #ethtool
	apt-get install xmlstarlet #xmlstarlet
	apt-get install tree doxygen
	apt-get install realpath #realpath
	apt-get install lrzsz #lz sz
	apt-get install lsof procps
	apt-get install dos2unix dialog libfuse-dev fuse-utils
	apt-get install aptitude
	apt-get install sudo
	apt-get install libevent-1.4-2 libevent-dev
	apt-get install libprotobuf-dev libprotobuf-c0 libprotobuf-c0-dev 
	apt-get install libprotobuf6 protobuf-compiler protobuf-c-compiler
	apt-get install libtcmalloc-minimal0
	apt-get install netcat
	apt-get install rrdtool
	apt-get install rdesktop grdesktop #mstsc
	apt-get install gtkvncviewer gvncviewer #vnc client
	apt-get install qemu-system-x86 qemu-system-x86-64
	apt-get install mupdf evince poppler-data #pdf reader
	apt-get install gnome-mplayer vlc #video player
	apt-get install eclipse eclipse-cdt
	apt-get install chromium-browser
	apt-get install chmsee #chm reader
	apt-get install g2ipmsg # 飞鸽传书
	apt-get install filezilla
	apt-get install meld
	
---
	
	# .bashrc
	alias ll='ls -l --color=auto'
	alias ls='ls --color=auto'
	alias s='cd ..'
	alias vir='vi -R'
	alias ps='ps aufx'
	alias grep='grep -i --color'

	c_1="\[\e[0m\]"
	c0="\[\e[30m\]"
	c1="\[\e[31m\]"
	c2="\[\e[32m\]"
	c3="\[\e[33m\]"
	c4="\[\e[34m\]"
	c5="\[\e[35m\]"
	c6="\[\e[36m\]"
	c7="\[\e[37m\]"
	PS1="$c0$c1\w $c2$c3<\u@\h> $c4$c5$c6$c7\T $c1$c2\$ $c_1";
	export PS1

## Git Diff配置

	apt-get install meld
	git config --global diff.external meld
	git config --global diff.external diff.py

---

	#!/usr/bin/python

	import sys
	import os

	os.system('meld "%s" "%s"' % (sys.argv[2], sys.argv[5]))

## Flash配置

使用Linux Mint发现百度云无法在线看视频，但是youku确可以。

默认安装的flash 插件是**mint-flashplugin** ，换成**adobe-flashplugin**即可。

