---
layout: post
title: 使用freesshd实现Windows远程控制
category: other
---

##安装

下载[freesshd](http://www.freesshd.com/?ctt=download)

安装之后，打开配置页，切换到TAB **Users**

1. 点击Add按钮
2. 输入Login/Password，中间选择"Password stored as SHA1 hash"
3. User can use 选择所有

**新建帐号，最好重启freesshd**

##使用

	root@qjw-VirtualBox:~# ssh test@192.168.1.7 "cmd /c dir c:\\"
	test@192.168.1.7's password: 
	 驱动器 C 中的卷没有标签。
	 卷的序列号是 887F-A0D8

	 c:\ 的目录

	2013/11/24  09:37    <DIR>          DevKit
	2013/10/15  22:10    <DIR>          Intel
	2009/07/14  11:20    <DIR>          PerfLogs
	2013/12/20  22:45    <DIR>          Program Files
	2014/01/10  20:36    <DIR>          Program Files (x86)
	2013/11/24  09:33    <DIR>          Ruby193
	2013/10/15  00:36    <DIR>          Users
	1980/01/01  00:01    <DIR>          Windows
				   0 个文件              0 字节
				   8 个目录  2,704,166,912 可用字节
				   
##陷阱
对于在使用管道的while循环中使用ssh的情形，需要特别注意。例如下面的代码枚举根目录所有文件（有删减）
	
	root@qjw-VirtualBox:~# find / -mindepth 1 -maxdepth 1 | while read line;do echo $line;done
	/srv
	/boot
	/sbin
	/etc
	
然后，若在while循环中加入ssh语句，例如

	find / -mindepth 1 -maxdepth 1 | while read line;do ssh test@192.168.1.7 "cmd /c dir c:\\"; done
	
那么在第一次循环结束后，就退出了，为什么呢？这是因为ssh将管道中stdin所有的内容吃掉了。解决办法如下，注意**<&-**

	find / -mindepth 1 -maxdepth 1 | while read line;do ssh test@192.168.1.7 "cmd /c dir c:\\" <&-; done


##参考
1. <http://stackoverflow.com/questions/8376166/execute-a-command-on-remote-hosts-via-ssh-from-inside-a-bash-script>