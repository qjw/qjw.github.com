---
layout: post
title: Posix Mutex
category: cpp
---

mutex常用语线程间互斥,简单的用法如下

	#include <pthread.h>

	int main()
	{
	#if 0
		pthread_mutex_t lock;  
		pthread_mutex_init(&lock,NULL); 
	#else
		pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;                                                                                      
	#endif
		pthread_mutex_lock(&lock);
		pthread_mutex_unlock(&lock);
		return 0;
	}
	
以上是mutex最简单的用法,属快速mutex（fastmutex）。不过这种方式不做错误判断，例如同一个线程加两次锁，或者不同的线程来释放锁也不会有问题，然后这通常隐藏着问题。

以下的error mutex锁可以避免这种情况，(*宏后面的NP表示not portable*)

	#define _GNU_SOURCE 
	#include <pthread.h>

	int main()
	{
	#if 1
		pthread_mutexattr_t attr;
		pthread_mutexattr_init(&attr);
		pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_ERRORCHECK_NP);

		pthread_mutex_t lock;                                                                                                                  
		pthread_mutex_init(&lock,&attr); 

		pthread_mutexattr_destroy(&attr);
	#else
		pthread_mutex_t lock = PTHREAD_ERRORCHECK_MUTEX_INITIALIZER_NP;
	#endif
		pthread_mutex_lock(&lock);
		pthread_mutex_unlock(&lock);
		return 0;
	}
	
此外还有recmutex锁，可以多次加锁，使用计数管理。以及跨进程锁(需要共享内存支持)。

另外一个问题,若某个加了锁的线程死了,那么其他线程是不是一直死锁了呢?

我们可以设置robust属性来初始化mutex以避免这个问题.

	pthread_mutexattr_setrobust_np(&attr,PTHREAD_MUTEX_ROBUST_NP);
	
当线程死了，那么另一个线程加锁时，会返回EOWNERDEAD错误码，此时你需要调用**pthread_mutex_consistent_np**来恢复一致性.
	
##参考
1. <http://hi.baidu.com/tim_bi/item/900b3d83b38911ded1f8cd9a>
1. <https://sourceware.org/pthreads-win32/manual/pthread_mutex_init.html>