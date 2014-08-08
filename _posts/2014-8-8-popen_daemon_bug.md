---
layout: post
title: popen调用后台进程导致的bug
category: linux
---

在Linux程序中，popen可以执行一个脚本，并且获取它的输出。

	#include <sys/types.h> 
	#include <unistd.h> 
	#include <stdlib.h> 
	#include <stdio.h> 
	#include <string.h>

	int main() 
	{ 
	    FILE   *fp_; 
	    char   buf_[10]; 
	    fp_ = popen( "./test.sh", "r" ); 

	    int ret_ = -1;
	    do{
		memset(buf_,0,sizeof(buf_));
		int ret_ = fread( buf_, sizeof(char), sizeof(buf_), fp_);
		if(ret_ < 0)
		{
		    break;
		}
		else if(ret_ == 0)
		{
		    ret_ = 0;
		    break;
		}
		else
		{
		    printf("%s",buf_);
		}
	    }while(1);
	    pclose(fp_ ); 
	    return ret_;
	}  
	
	
test.sh 如下

	#!/bin/bash

	loop()
	{
	    while true
	    do
		sleep 1
	    done
	}

	ls -l
	(
	loop
	) &
	
在test.sh中，运行ls -l之后，立即运行一个后台脚本就退出了，但实际情况如下

	king      9121 4340   368 pts/1    S+   22:51   0:00          |           |   \_ ./a.out
	king      9122    0     0 pts/1    Z+   22:51   0:00          |           |       \_ [sh] <defunct>

	king      9125   0.0  0.0  14064   724 pts/1    S+   22:51   0:00 /bin/bash ./test.sh
	king      9128 8884   372 pts/1    S+   22:51   0:00  \_ sleep 1

可以看到a.out一直卡在那里不返回，它执行的test.sh变成了僵尸。而test.sh运行的后台进程仍然在运行。若此时杀掉那个后台的test.sh，那么a.out立即返回。

原因是因为popen会重定向stdout，并且main函数中一个read该句柄直到close，而一旦该进程创建子进程后，这个句柄就被子进程继承，然后不到这个后台子进程结束，这个句柄就不会close。

一种简单的改法就是在调用test.sh调用后台脚本时先关闭stdout

	#!/bin/bash

	loop()
	{
	    while true
	    do
		sleep 1
	    done
	}

	ls -l

	(
	loop
	) >&- &
	
不过，若后台进程有stdout写入会出现错误

	./test.sh: 第 8 行: echo: 写错误: 错误的文件描述符
	./test.sh: 第 8 行: echo: 写错误: 错误的文件描述符
	./test.sh: 第 8 行: echo: 写错误: 错误的文件描述符


