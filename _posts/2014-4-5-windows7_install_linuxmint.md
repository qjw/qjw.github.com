---
layout: post
title: Windows硬盘安装LinuxMint
category: other
---

##配置Grub4dos

在C盘新建文件boot.ini，写入以下内容

	[boot loader]
	[operating systems]
	c:\grldr.mbr="Grub4Dos"
	
下载grub4dos压缩包，解压**grldr、grldr.mbr、grub.exe、menu.lst**到C盘根目录

若boot.ini不生效，使用bcedit

	@echo off
	cls
	echo.
	echo   必须以管理员运行
	echo.
	pause
	set title=Grub4DOS
	set vid=
	bcdedit /create /d "%title%" /application bootsector >vid.ini
	for,/f,"tokens=2 delims={",%%i,In (vid.ini) Do (
		set vida=%%i
	)
	for,/f,"tokens=1 delims=}",%%i,In ("%vida%") Do (
		set vid={% raw %}{%%i}
	)
	echo %title% created as %vid%
	bcdedit /set %vid% device  partition=c: >nul
	bcdedit /set %vid% path \gsldr.mbr >nul
	bcdedit /displayorder %vid% /addlast >nul
	echo.
	pause
			bcdedit /enum
	pause
	:exit

##配置grub

下载linux mint，解压**linuxmint-16-cinnamon-dvd-64bit.iso\casper\initrd.lz、vmlinuz**到C盘根目录

修改menu.lst

	title linux mint debian Live CD 
	root (hd0,0) 
	kernel /vmlinuz boot=casper iso-scan/filename=/linuxmint-16-cinnamon-dvd-64bit.iso ro quiet splash 
	initrd /initrd.lz
	
重启电脑

安装过程中，使用**sudo umount -l /isodevice**先卸载ISO

	
##参考
1. <http://www.kisa747.com/windows7-grub4dos.html>
1. <http://my.oschina.net/wxwHome/blog/32171>
1. <http://download.gna.org/grub4dos/grub4dos-0.4.4-2009-06-20.zip>
1. <http://mirrors.hust.edu.cn/linuxmint-cd//stable/16/linuxmint-16-cinnamon-dvd-64bit.iso>
1. <http://www.codeif.com/topic/1513>