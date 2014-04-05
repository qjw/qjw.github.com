---
layout: post
title:  僵尸进程及其预防
category: linux
---

##参考
1. <http://blog.csdn.net/duchuanying/article/details/1496087>


##什么是僵尸进程?

In UNIX System terminology, a process that has terminated,but whose parent has not yet waited for it, is called a zombie.

在UNIX 系统中,一个进程结束了,但是他的父进程没有等待(调用wait / waitpid)他,那么他将变成一个僵尸进程.但是如果**该进程的父进程已经先结束了,那么该进程就不会变成僵尸进程**,因为每个进程结束的时候,系统都会扫描当前系统中所运行的所有进程, 看有没有哪个进程是刚刚结束的这个进程的子进程,如果是的话,就由Init来接管他,成为他的父进程,从而保证每个进程都会有一个父进程.  而Init进程会自动wait 其子进程,因此被Init接管的所有进程都不会变成僵尸进程.

##僵尸进程的危害

由于子进程的结束和父进程的运行是一个异步过程,即父进程永远无法预测子进程到底什么时候结束. 那么不会因为父进程太忙来不及waid子进程,或者说不知道子进程什么时候结束,而丢失子进程结束时的状态信息呢?不会.因为UNIX提供了一种机制可以保证 只要父进程想知道子进程结束时的状态信息,就可以得到. 这种机制就是: 在每个进程退出的时候,内核释放该进程所有的资源,包括打开的文件,占用的内存等.但是仍然为其保留一定的信息(**包括进程号the process ID,退出状态the termination status of the process,运行时间the amount of CPU time taken by the process等**),直到父进程通过wait / waitpid来取时才释放.但这样就导致了问题,如果你进程不调用wait / waitpid的话, 那么保留的那段信息就不会释放,其进程号就会一定被占用,但是系统所能使用的进程号是有限的,如果大量的产生僵死进程,将因为没有可用的进程号而导致系统不能产生新的进程.此即为僵尸进程的危害,应当避免.

##僵尸进程的避免

1. 父进程通过wait和waitpid等函数等待子进程结束，这会导致父进程挂起 
2. 如果父进程很忙，那么可以用**signal函数为SIGCHLD安装handler**，因为子进程结束后，父进程会收到该信号，可以在handler中调用wait回收
3. 如果父进程不关心子进程什么时候结束，那么可以用**signal（SIGCHLD, SIG_IGN）** 通知内核，自己对子进程的结束不感兴趣，那么子进程结束后，内核会回收，并不再给父进程发送信号 
4. 还有一些技巧，就是**fork两次**，父进程fork一个子进程，然后继续工作，子进程fork一 个孙进程后退出，那么孙进程被init接管，孙进程结束后，init会回收。不过子进程的回收还要自己做。
