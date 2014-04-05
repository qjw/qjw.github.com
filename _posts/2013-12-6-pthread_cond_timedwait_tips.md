---
layout: post
title: pthread_cond_timedwait 陷阱
category: linux
---

pthread_cond_timedwait 用于在同时等待条件变量时提供超时功能，不过该函数的超时时间是一个绝对时间。默认使用系统时间，这意味这，若修改系统时间，那么超时就不准确，有可能提前返回，也可能要几年才返回。

这在某些需求下会导致bug，我们可以做特殊的初始化以排除系统时间的影响。

	#include <stdio.h>                                                                 
	#include <stdlib.h>                                                                
	#include <string.h>                                                                
	#include <unistd.h>                                                                
	#include <pthread.h>
	
	int main()                                                                         
	{                                                                                  
		pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;                                 
		pthread_condattr_t attr;                                                       
		pthread_cond_t cond;                                                           
		pthread_condattr_init(&attr);                                                  
		pthread_condattr_setclock(&attr, CLOCK_MONOTONIC);                             
		pthread_cond_init(&cond, &attr);
		struct timespec tv;                                                            
		pthread_mutex_lock(&m);                                                        
		do{                                                                            
			clock_gettime(CLOCK_MONOTONIC, &tv);                                       
			tv.tv_sec += 1;                                                            
			pthread_cond_timedwait(&cond,&m,&tv);                                      
			printf("heart beat\n");                                                    
		}while(1);                                                                     
		pthread_mutex_unlock(&m);                                                      
		return 0;                                                                      
	}       
	
	
##参考
1. <http://linux.die.net/man/3/pthread_cond_timedwait>
1. <http://stackoverflow.com/questions/14248033/clock-monotonic-and-pthread-mutex-timedlock-pthread-cond-timedwait>
1. <http://www.qnx.com/developers/docs/6.5.0/index.jsp?topic=%2Fcom.qnx.doc.neutrino_lib_ref%2Fp%2Fpthread_cond_timedwait.html>