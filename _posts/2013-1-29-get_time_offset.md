---
layout: post
title:  获得开机时间
category: bash
---
         
##脚本
我们可以通过读取文件**/proc/uptime**来获取开机时间，单位秒。

/proc/uptime

2个数字的意义，第一个数值代表系统总的启动时间，第二个数值则代表系统空闲的时间，都是用秒来表示的。


##Windows
使用**GetTickCount**函数可以获得开机以来的毫秒数

在游戏里面，通常使用函数**QueryPerformanceCounter/QueryPerformanceFrequency**来计算高精度时间差

##Linux
使用**clock_gettime**函数，并配合参数**CLOCK_MONOTONIC**可以获得开机以来的时间，通过一个**struct timespec**结构返回时间

##参考
1. <http://hi.baidu.com/cnh4wk/item/6dbf9562ecb34c93c5d249fa>
2. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms724408\(v=vs.85\).aspx>
2. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms644904\(v=vs.85\).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms644905\(v=vs.85\).aspx>
3. <http://linux.die.net/man/3/clock_gettime>