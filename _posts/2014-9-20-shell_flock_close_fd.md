---
layout: post
title: shell中使用flock作同步
category: bash
---

在shell中，通常我们要作互斥，最简单的办法就是**flock**命令

	#!/bin/bash 
		                                                                        
	for((i=0;i<100;i++)) 
	do                                                                              
	    {                                                                              
		flock -x 3
		date
		sleep 1
	    } 3<>/tmp/`basename $0`.lock 
	done 	
	
----

	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 08:38:05 $ ./test.sh 
	2014年 09月 21日 星期日 08:38:45 CST
	2014年 09月 21日 星期日 08:38:46 CST
	2014年 09月 21日 星期日 08:38:47 CST
	2014年 09月 21日 星期日 08:38:48 CST
	2014年 09月 21日 星期日 08:38:51 CST
	2014年 09月 21日 星期日 08:38:53 CST
	2014年 09月 21日 星期日 08:38:55 CST
	2014年 09月 21日 星期日 08:38:56 CST
	2014年 09月 21日 星期日 08:38:58 CST
	2014年 09月 21日 星期日 08:39:00 CST
	
	
如果在互斥逻辑中，直接或者间接的启动了后台进程，那么情况有不一样了。后台进程继承了flock的互斥句柄，默认情况下，除非后台进程退出，否则flock永远也无法获取锁

	#!/bin/bash 
		                                                                        
	for((i=0;i<100;i++)) 
	do                                                                              
	    {                                                                              
		flock -x 3
		date
		sleep 10 &
	    } 3<>/tmp/`basename $0`.lock 
	done 
	
---
	
	/tmp <king@king-HP-Compaq-6530b-FP587PA-AB2> 08:47:09 $ ./test.sh 
	2014年 09月 21日 星期日 08:47:14 CST
	2014年 09月 21日 星期日 08:47:24 CST
	2014年 09月 21日 星期日 08:47:34 CST
	
	
解决办法就是在创建后台进程之前，先关闭句柄

	#!/bin/bash 
		                                                                        
	for((i=0;i<100;i++)) 
	do                                                                              
	    {                                                                              
		flock -x 3
		date
		sleep 10 3>&- &
	    } 3<>/tmp/`basename $0`.lock 
	done 
	
	
##参考
1. <http://stackoverflow.com/questions/8866175/preventing-lock-propagation>
 
