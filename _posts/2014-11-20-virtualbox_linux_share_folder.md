---
layout: post
title: Virtualbox使用共享文件夹实现Linux文件共享
category: other
---

##安装附加工具

点击菜单设备-> 安装增强工具  来挂在cdrom

    mount /dev/cdrom /mnt/cdrom
    cd /mnt/cdrom
    ./VBoxLinuxAdditions.run

---

    king@king:/mnt/king/tmp/git$ ll /mnt/cdrom/
    \total 50772
    dr-xr-xr-x 6 root root     2048 Oct 11 02:37 ./
    drwxr-xr-x 4 root root     4096 Nov 20 14:40 ../
    dr-xr-xr-x 2 root root     2048 Oct 11 02:37 32Bit/
    dr-xr-xr-x 2 root root     2048 Oct 11 02:37 64Bit/
    -r-xr-xr-x 1 root root      647 Sep 12 21:10 AUTORUN.INF*
    -r-xr-xr-x 1 root root     6966 Oct 11 03:34 autorun.sh*
    dr-xr-xr-x 2 root root     2048 Oct 11 02:37 cert/
    dr-xr-xr-x 2 root root     2048 Oct 11 02:37 OS2/
    -r-xr-xr-x 1 root root     5523 Oct 11 03:34 runasroot.sh*
    -r-xr-xr-x 1 root root  7126478 Oct 11 03:34 VBoxLinuxAdditions.run*
    -r-xr-xr-x 1 root root 16415744 Oct 11 04:35 VBoxSolarisAdditions.pkg*
    -r-xr-xr-x 1 root root 17355176 Oct 11 03:36 VBoxWindowsAdditions-amd64.exe*
    -r-xr-xr-x 1 root root   312384 Oct 11 03:33 VBoxWindowsAdditions.exe*
    -r-xr-xr-x 1 root root 10751528 Oct 11 03:33 VBoxWindowsAdditions-x86.exe*

##设置共享文件夹

设置为可读写,记住共享文件夹名称。不要设置自动挂载，在我的机器测试发现自动挂载的磁盘非root用户不可用

##手动挂载

    sudo mount -t vboxsf -o umask=000,rw,gid=100,uid=1000 kingqiu /mnt/king/

其中的gid，uid，umask等不能省，否则非root用户将无法使用

kingqiu是共享文件夹的名称

也可以在/etc/fstab中加入一项实现完美的自动挂载，不过我测试不行，求解


##参考
1. <http://www.cnblogs.com/linjiqin/p/3615477.html>
