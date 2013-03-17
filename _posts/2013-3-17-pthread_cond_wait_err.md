---
layout: post
title:  Posix条件变量的一个陷阱
category: linux
---

##线程退出模式
Posix线程可以设定"退出模式"，通过pthread_setcancelstate函数开启(PTHREAD_CANCEL_ENABLE)/关闭(PTHREAD_CANCEL_DISABLE)"退出模式"，默认开启。而退出模式又可以细分为"PTHREAD_CANCEL_DEFERRED"和"PTHREAD_CANCEL_ASYNCHRONOUS",默认是"PTHREAD_CANCEL_DEFERRED"，它表示当收到退出"消息"后线程会在下一个退出点(cancellation point)终止。后者表示线程收到"消息"后立即退出，可以理解为杀线程。

##条件变量
条件变量总是需要和一个互斥量配合使用，例如

	pthread_mutex_lock(&m_mutex);   
	// do something
	pthread_cond_wait(&m_cond,&m_mutex);   
	// do something
	pthread_mutex_unlock(&m_mutex);  
	
大部分情况下，这种用法没有问题。前面提到，Posix线程在某些情况下会在下一个退出点终止，而pthread_cond_wait就是一个退出点。我们知道pthread_cond_wait在阻塞线程之前会自动释放互斥锁，而在返回之前自动锁上互斥锁。

现在问题来了，若pthread_cond_wait在等待时，其他线程调用pthread_cancel让它退出（*假若pthread_cond_wait所在的线程使用默认的配置*），那么pthread_cond_wait会立即返回，并且线程被终止，而后面的pthread_mutex_unlock就没被调用，若其他逻辑以来这个锁，**很显然，死锁了**。

##解决办法
Posix提供了一个函数pthread_cleanup_push，可以让线程自动执行一些清理操作，如下：

	void cleanup(void *arg)
	{
		pthread_mutex_unlock(&amp;mutex);
	}
	void * thread1(void * arg)
	{
		pthread_cleanup_push(cleanup, NULL); // thread cleanup handler 
		pthread_mutex_lock(&amp;mutex);
		pthread_cond_wait(&amp;cond, &amp;mutex);
		pthread_mutex_unlock(&amp;mutex);
		pthread_cleanup_pop(0 );
	}

##参考
1. <http://linux.die.net/man/3/pthread_setcancelstate>
1. <http://blog.csdn.net/digu/article/details/5842302>
