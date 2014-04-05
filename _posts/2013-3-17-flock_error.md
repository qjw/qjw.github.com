---
layout: post
title:  flock的陷阱
category: linux
---

Locks created by flock() are associated with an open file table entry. This means that duplicate file descriptors (created by, for example, fork(2) or dup(2)) refer to the same lock, and this lock may be modified or released using any of these descriptors. Furthermore, the lock is released either by an explicit LOCK_UN operation on any of these duplicate descriptors, or when all such descriptors have been closed.

flock打开的句柄，会被继承到子进程。在当前进程退出后，会自动关闭句柄，然而若该句柄还被其他进程引用，那么flock不会解锁。这就导致这个文件锁被继承。

解决办法:**使用fcntl设置FD_CLOEXEC标志位取消继承，或者使用fcntl记录锁**。

##参考
1. <http://linux.die.net/man/2/flock>
1. <http://blog.chinaunix.net/uid-24774106-id-3488489.html>
1. <http://blog.chinaunix.net/uid-24774106-id-3488649.html>
1. <http://bbs.csdn.net/topics/330147328>
