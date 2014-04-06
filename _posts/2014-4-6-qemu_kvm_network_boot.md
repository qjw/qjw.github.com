---
layout: post
title: 使用Qemu-kvm网络启动安装PE
category: other
---

##安装必要软件

下载grub4dos（<http://download.gna.org/grub4dos/grub4dos-0.4.4-2009-06-20.zip>）。解压**grldr**和**menu.lst**到/home/fuck目录下。

下载**pe.iso**（<http://pan.baidu.com/share/link?shareid=753138449&uk=2500090602&fid=35094960>）到/home/fuck目录下

##更新menu.lst

	color blue/green yellow/red white/magenta white/magenta
	timeout 1
	default 0

	title Install
	map --mem (pd)/pe.iso (hd32) 
	map --hook 
	chainloader (hd32)
	boot 

##安装

运行下列命令(**内存必须够大**)

	qemu-system-i386 -cpu core2duo -m 512 -boot n\
		-net nic -net user,net=10.2.0.0/8,host=10.2.0.6,dhcpstart=10.2.0.20,hostname=tux_kvm_guest \
		-net user,tftp=/home/fuck,bootfile=/grldr

这样和**-cdrom**的好处时，不占用ide通道，kvm只能最多指定4个通道（磁盘镜像）


##参考
1. <http://chenhm.com/2013/02/pxe-winpe/>
1. <http://www.bitscn.com/os/windows/201005/186780.html>
1. <http://hi.baidu.com/wwfmarcpjkbdiod/item/5a6793eb46cc70f72a09a494>
1. <http://doc.opensuse.org/products/draft/SLES/SLES-kvm_sd_draft/cha.qemu.running.html#cha.qemu.running.networking>
1. <https://www.suse.com/documentation/sles11/book_kvm/data/cha_qemu_running_networking.html>
