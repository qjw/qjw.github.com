---
layout: post
title: 多线程程序fork的陷阱
category: cpp
---

在多线程程序中，若fork之后，理解exec，那没啥问题，若仅仅fork，那就难说了

首先明确几点：

1. fork之后，子进程继承了mutex的属性（比如lock或者unlock）
2. fork之后，子进程是单线程

假设在fork时，一个多线程共享的mutex被另一个线程锁住了。此时子进程继承了这个属性，但是子进程并不一定知道被锁了，这很容易导致**死锁**。

虽然这种情况，在谨慎地设计后可以避免，然后，很多库函数内部都使用了锁，这就很难无视了。

所以最好是不要在多线程程序中直接fork，除非fork之后用作exec。

##参考
1. <http://blog.csdn.net/hanchaoman/article/details/5685582>