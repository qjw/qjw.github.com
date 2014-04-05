---
layout: post
title:  Linux正确的退出子进程
category: linux
---
 
        pid_t cpid = fork();
        if (cpid == -1) {
            perror("fork");
            exit(EXIT_FAILURE);
        }
        if (cpid == 0) { // in child
            execl("/bin/sh", "sh", "-c", cmdstring, (char *)0);
            perror("execvp);
            _exit(EXIT_FAILURE);
        }

在上面的代码里面有两个exit函数，分别是exit和_exit函数。那为什么在父进程里面用的是exit，子进程要用_exit呢？

参考<<UNIX高级环境编程>>

exit在内部调用_exit，不过在之前做了很多额外的工作，这些内容包括
1. 清理I/O缓冲
1. 调用atexit注册的函数


###以下内容摘录自<http://www.cppblog.com/mydriverc/articles/29106.html>

exit()’与‘_exit()’的基本区别在于前一个调用实施与调用库里用户状态结构 
(user-mode constructs)有关的清除工作(clean-up)，而且调用用户自定义的清除程序 
(译者注：自定义清除程序由atexit函数定义，可定义多次，并以倒序执行)，相对 
应，后一个函数只为进程实施内核清除工作。 

在由‘fork()’创建的子进程分支里，正常情况下使用‘exit()’是不正确的，这是 
因为使用它会导致标准输入输出(译者注：stdio: Standard Input Output)的缓冲区被 
清空两次，而且临时文件被出乎意料的删除(译者注：临时文件由tmpfile函数创建 
在系统临时目录下，文件名由系统随机生成)。在C++程序中情况会更糟，因为静 
态目标(static objects)的析构函数(destructors)可以被错误地执行。(还有一些特殊情 
况，比如守护程序，它们的*父进程*需要调用‘_exit()’而不是子进程；适用于绝 
大多数情况的基本规则是，‘exit()’在每一次进入‘main’函数后只调用一次。) 

在由‘vfork()’创建的子进程分支里，‘exit()’的使用将更加危险，因为它将影响 
*父*进程的状态