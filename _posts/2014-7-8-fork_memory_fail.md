---
layout: post
title: fork时分配内存失败的陷阱
category: cpp
---

当某个进程占用很大内存，假设占用了整个物理内存的90%，此时若fork，由于fork出来的进程会共享内存，但是对于冲突的地方，需要作内存拷贝。若冲突的内存特别多，则会导致内存分配失败。

解决办法可以使用**vfork**替换fork。不过**system**，**popen**等函数都得重写。

另外还有改法是修改内核在分配内存时的策略，具体是修改文件**/proc/sys/vm/overcommit_memory**

overcommit_memory文件指定了内核针对内存分配的策略，其值可以是0、1、2。    
                           
	0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。 
	1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
	2， 表示内核允许分配超过所有物理内存和交换空间总和的内存

##参考
1. <http://stackoverflow.com/questions/18374399/fork-failures-cannot-allocate-memory>
1. <http://blog.csdn.net/anghlq/article/details/7087069>
1. <http://qjw.qiujinwu.com/blog/2013/09/28/fork_thread/>

