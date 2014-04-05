---
layout: post
title: Google-perftools  CPU Profiler
category: cpp
---

两年前研究过google-perftools 的Tcmalloc，其使用**线程本地存储**解决线程间锁开销，以及全局表来合并地址相连的碎内存方面，着实让人眼前一亮。

最近在工作中用了下CPU Profiler，这里做个笔记。

使用CPU Profiler分三个步骤

###链接库
若有源码，建议编译时，带**-lprofiler**选项，也可以用hack的办法，运行时**env LD_PRELOAD="/usr/lib/libprofiler.so" <binary>**。不过在我测试中，后者程序老是卡死。

###运行程序
CPU Profiler对**多线程**和**动态库**很友好，同时也支持动态开启关闭取样（似乎gprof没有发现类似的功能）。

为了开启取样，运行前可以设置环境变量CPUPROFILE。**env CPUPROFILE=/tmp/mybin.prof  <binary>**。CPUPROFILE的值表示取样结果存储的文件。

当然也可以动态设置，以及参数修改。这就需要借助CPU Profiler的API了。例如**ProfilerStart/ProfilerStop**

我个人很喜欢另外一种方式，用gdb下个断点到需要开启取样的地方，被断点后运行**p ProfilerStart("/tmp/mybin.prof")**来开启取样，ProfilerStop也用同样的办法。这种办法有几个好处：

1. 可以动态的变更位置，而不是写死在程序中
2. 由于不需要引入额外的头文件，所以避免一些修改编译环境（编辑脚本）的繁琐事情。（*在大多数公司，都有一套编译环境，修改还不算容易*）
	
###分析

	% pprof /bin/ls ls.prof
						   Enters "interactive" mode
	% pprof --text /bin/ls ls.prof
						   Outputs one line per procedure
	% pprof --gv /bin/ls ls.prof
						   Displays annotated call-graph via 'gv'
	% pprof --gv --focus=Mutex /bin/ls ls.prof
						   Restricts to code paths including a .*Mutex.* entry
	% pprof --gv --focus=Mutex --ignore=string /bin/ls ls.prof
						   Code paths including Mutex but not string
	% pprof --list=getdir /bin/ls ls.prof
						   (Per-line) annotated source listing for getdir()
	% pprof --disasm=getdir /bin/ls ls.prof
						   (Per-PC) annotated disassembly for getdir()
	% pprof --text localhost:1234
						   Outputs one line per procedure for localhost:1234
	% pprof --callgrind /bin/ls ls.prof
						   Outputs the call information in callgrind format
	
##参考
1. <http://google-perftools.googlecode.com/svn/trunk/doc/cpuprofile.html>
1. <http://www.ibm.com/developerworks/cn/linux/l-cn-googleperf/>