---
layout: post
title: 使用socketpair实现进程内全双工通信
category: linux
---

在Linux下，我们经常使用pipe创建个管道来实现进程内线程间（或者父子进程）通信。不过pipe创建的两个句柄，一个只能写，另一个只能读，属于单工通道。

为了实现双工通信，可以考虑**socketpipe**。

	#include <stdio.h>                                                              
	#include <sys/socket.h>                                                         
	#include <sys/types.h>                                                          
		                                                                        
	int main()
	{         
	    int fds_[1 + 1];
	    socketpair(AF_UNIX,SOCK_STREAM,0,fds_);
		                             
	    int type;
	    socklen_t length = sizeof( int );
	    getsockopt(fds_[0], SOL_SOCKET, SO_TYPE, &type, &length );
	    printf("sock type %d",type);
	    return 0;
	}
	

##参考
1. <http://liulixiaoyao.blog.51cto.com/1361095/533469/>

