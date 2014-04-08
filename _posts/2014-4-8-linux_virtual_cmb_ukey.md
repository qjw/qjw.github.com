---
layout: post
title: Linux下使用Virtualbox安装虚拟机支持招行Ukey
category: other
---

## 安装

运行命令**aptitude install virtualbox-4.3** 安装virtualbox。新建虚拟机，安装Windows XP。

挂载ISO VBoxGuestAdditions，通常路径是**/usr/share/virtualbox/VBoxGuestAdditions.iso**，并在Windows XP中安装光盘中的程序。

下载[ VirtualBox Extension Pack](http://download.virtualbox.org/virtualbox/4.3.10/Oracle_VM_VirtualBox_Extension_Pack-4.3.10-93012.vbox-extpack)。见<https://www.virtualbox.org/wiki/Downloads>

打开Oracle VirtualBox，在菜单栏中找到“管理”–>“全局设定”,“Extension Packages”中添加下载的VirtualBox Extension Pack，根据提示安装即可（*若不安装此packet，那么将无法支持usb2.0*）。

启动VirtualBox并打开设置（settings)对话框，选择USB菜单，启用USB和**USB2.0**的控制器。

##运行

将**当前用户**加入**用户组vboxusers**

确保启动virtualbox之前，先插入Ukey。

运行时，为了避免权限问题干扰，加上**sudo**运行。


##参考
1. <http://www.xiaojian.org/article/310.html>
1. <http://my.oschina.net/wufa/blog/86440>
1. <http://qyiyunso.blog.163.com/blog/static/35077686201092653624956/>
1. <http://blog.csdn.net/csfreebird/article/details/7835233>

