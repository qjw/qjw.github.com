---
layout: post
title:  System,Popen的陷阱
category: linux
---
 
为了更好的处理僵尸进程的问题，我们通常会考虑忽略SIGCHID信号，或者明确的Catch之并且在内部做wait处理。

system和popen函数也因为简单易用经常被用到，不过这两个函数有些细节必须要注意，否则可能导致一些严重但是却很难查的bug

system函数在内部会临时忽略(SIG_IGN)SIGINT和SIGQUIT信号，同时阻塞(SIG_BLOCK)SIGCHLD信号。具体见[这里](http://www.oschina.net/question/54100_30293)。

对于前者，若你的程序的退出以来SIGINT信号，而此时若它正在system执行一个耗时的程序，那么你的程序可能不会及时响应，甚至你的关闭指令丢失如果没有确认。

对于SIGCHLD信号，分两种情况处理。

##忽略SIGCHLD信号
我们知道，当忽略SIGCHID信号之后，所有的wait调用都会失败，errno=10(No child processes)，所以system，popen等任何隐式或显式地调用wait都会失败，具体案例见[这里](http://my.oschina.net/renhc/blog/54582)

解决办法就是在调用system等之前将SIGCHID改成默认或者明确的Catch。

##Catch SIGCHLD信号
对于system函数，无影响

但是对于popen函数，同样可能失败(10,No child processes)，这是因为当进程退出时，立即收到SIGCHLD，若接下来立即在信号处理回调里面wait，那么我们非信号回调的wait都会因为优先级的关系而等不到而失败。

解决办法参考system函数，临时阻塞信号，完了再解除阻塞

        sigset_t saveblock;
        sigset_t sa_mask
        sigemptyset(&sa_mask);
        sigaddset(&sa_mask, SIGCHLD);
        sigprocmask(SIG_BLOCK, &sa_mask, &saveblock);
        // do sth
        sigprocmask(SIG_SETMASK, &saveblock, (sigset_t *)0);

对于其他手动调用的wait函数，也必须在实现阻塞SIGCHLD信号。

**经测试，最新的popen也阻塞了SIGCHLD信号，在pclose中接触阻塞**

以下代码，若去掉pclose一行，那么Sigfunc不会出发。否则则会触发

	#include <unistd.h>
	#include <signal.h>
	#include <cstdio>
	#include <cstdlib>
	#include <cstring>
	#include <errno.h>
	#include <cassert>

	void SigFunc(int no)
	{
		printf("recv no '%d'",no);
	}

	int main()
	{
		signal(SIGCHLD, SigFunc);
		FILE* fp_ = popen("true","r");
		pclose(fp_);
		return 0;
	}

##Sample

	#include <unistd.h>
	#include <signal.h>
	#include <cstdio>
	#include <cstdlib>
	#include <cstring>
	#include <errno.h>
	#include <cassert>

	int Popen(const char* cmd_line)
	{
		FILE* fp_ = popen(cmd_line,"r");
		assert(fp_);
		char buf_[4096];
		fread(buf_,sizeof(buf_),1,fp_);
		return pclose(fp_);
	}

	int MyPopen(const char* cmd_line)
	{
		int ret = 0;
		__sighandler_t old_handler;
		old_handler = signal(SIGCHLD, SIG_DFL);
		ret = Popen(cmd_line);
		signal(SIGCHLD, old_handler);
		return ret;
	}

	int MySystem(const char *cmd_line)
	{
		int ret = 0;
		__sighandler_t old_handler;
		old_handler = signal(SIGCHLD, SIG_DFL);
		ret = system(cmd_line);
		signal(SIGCHLD, old_handler);
		return ret;
	}

	void SigFunc(int no)
	{

	}

	int main()
	{
		signal(SIGCHLD, SIG_IGN);

		if(system("true") != 0)
			printf("system error %d|%s\n",errno,strerror(errno));
		else
			printf("system succuss\n");

		if(MySystem("true") != 0)
			printf("MySystem error %d|%s\n",errno,strerror(errno));
		else
			printf("MySystem succuss\n");
	
		//--------------------------------------------------------
		if(Popen("echo -n test") != 0)
			printf("Popen error %d|%s\n",errno,strerror(errno));
		else
			printf("Popen succuss\n");

		if(MyPopen("echo -n test") != 0)
			printf("MyPopen error %d|%s\n",errno,strerror(errno));
		else
			printf("MyPopen succuss\n");

		//--------------------------------------------------------
		signal(SIGCHLD, SigFunc);
		if(system("true") != 0)
			printf("system error %d|%s\n",errno,strerror(errno));
		else
			printf("system succuss\n");

		if(MySystem("true") != 0)
			printf("MySystem error %d|%s\n",errno,strerror(errno));
		else
			printf("MySystem succuss\n");
	
		//--------------------------------------------------------
		if(Popen("echo -n test") != 0)
			printf("Popen error %d|%s\n",errno,strerror(errno));
		else
			printf("Popen succuss\n");

		if(MyPopen("echo -n test") != 0)
			printf("MyPopen error %d|%s\n",errno,strerror(errno));
		else
			printf("MyPopen succuss\n");

		return 0;
	}

---

	system error 10|No child processes
	MySystem succuss
	Popen error 10|No child processes
	MyPopen succuss
	system succuss
	MySystem succuss
	Popen succuss
	MyPopen succuss



