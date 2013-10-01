---
layout: post
title: gettid获得线程ID
category: cpp
---

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <pthread.h>
	#include <unistd.h>

	#include <sys/syscall.h>  
	#define gettid() syscall(__NR_gettid)

	void *hello(void *ptr)
	{
		while(1)
		{   
			printf("%lx %ld\n",pthread_self(),gettid());
			sleep(1);   
		}   
	}

	int main()
	{
		pthread_t thread_;
		pthread_create(&thread_, NULL, hello, NULL);  
		while(1)
		{   
			printf("%lx %ld\n",pthread_self(),gettid());
			sleep(1);                                                                                                                          
		}   
		return 0;
	}
	
	
---

	(gdb) r
	Starting program: /root/test/a.out 
	[Thread debugging using libthread_db enabled]
	[New Thread 0xb7e74b70 (LWP 1188)]
	b7e756c0 1185
	b7e74b70 1188
	b7e756c0 1185
	b7e74b70 1188
	^C
	Program received signal SIGINT, Interrupt.
	0xb7fe2424 in __kernel_vsyscall ()
	(gdb) info threads 
	  2 Thread 0xb7e74b70 (LWP 1188)  0xb7fe2424 in __kernel_vsyscall ()
	* 1 Thread 0xb7e756c0 (LWP 1185)  0xb7fe2424 in __kernel_vsyscall ()	
	
##参考
1. <http://blog.csdn.net/zhuliting/article/details/6012466>
1. <http://man7.org/linux/man-pages/man2/gettid.2.html>