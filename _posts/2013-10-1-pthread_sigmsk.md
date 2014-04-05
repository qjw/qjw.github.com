---
layout: post
title: 线程信号屏蔽
category: cpp
---

下面的实例强制线程2接收信号.

	#include <stdlib.h>
	#include <pthread.h>
	#include <signal.h>
	#include <stdio.h>

	void sig(int sig)
	{
		printf("get sig is %d %lx\n", sig,pthread_self());
	}

	void *f1(void *arg) 
	{
		printf("my %s id is %lx\n",__FUNCTION__,pthread_self());
		while(1) sleep(1);
	}
	void *f2(void *arg) 
	{
		printf("my %s id is %lx\n",__FUNCTION__,pthread_self());

		sigset_t set;
		sigemptyset(&set);
		sigaddset(&set, SIGINT);
		pthread_sigmask(SIG_UNBLOCK, &set, NULL);
		
		while(1) sleep(1);
	}

	int main(int argc, char** argv) 
	{
		printf("my %s id is %lx\n",__FUNCTION__,pthread_self());
		pthread_t thread1;
		pthread_t thread2;

		// set signal handle
		struct sigaction action;
		action.sa_handler = sig;
		action.sa_flags = 0;
		sigemptyset(&action.sa_mask);
		sigaction(SIGINT, &action, NULL);

		// block signal whole process
		sigset_t set;
		sigemptyset(&set);
		sigaddset(&set, SIGINT);
		sigprocmask(SIG_BLOCK,&set,NULL);

		// create two thread
		pthread_create(&thread1, NULL, f1, NULL);
		pthread_create(&thread2, NULL, f2, NULL);

		while(1) sleep(1);
		return 0;
	}
	
##注意
1. 默认情况下，不能保证信号被主线程接收。
2. [sigprocmask](http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_sigmask.html)给当前**进程**设置信号掩码，[pthread_sigmask](http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_sigmask.html)给特定**线程**设置信号掩码
	
##参考
1. <http://blog.csdn.net/sctq8888/article/details/7427227>