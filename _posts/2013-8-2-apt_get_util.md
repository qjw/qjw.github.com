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
	apt-get install build-essential gdb
	apt-get install vim ctags diffutils patch file
	apt-get install subversion git
	apt-get install python2.6 lua5.1 sun-java6-jre
	apt-get install wget curl tcpdump dnsutils nmap telnet
	apt-get install netcat-traditional lftp iputils-ping
	apt-get install smbfs #@support windows share file system
	apt-get install zip p7zip p7zip-rar
	apt-get install ethtool #ethtool
	apt-get install xmlstarlet #xmlstarlet
	apt-get install tree
	apt-get install realpath #realpath
	apt-get install lrzsz #lz sz
	apt-get install lsof procps psmisc #top pstree
	apt-get install dos2unix
	apt-get install aptitude
	apt-get install sudo
	
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
