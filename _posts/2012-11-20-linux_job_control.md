---
layout: post
title:  Linux 进程组
category: linux
---
 
我们经常需要对某些进程集合进行统一控制，典型的场景如：
1. 当主进程退出之后，希望所有的子进程自动退出退出
1. 我们exec一个脚本之后，实际上它会启动多个进程，我们希望能够一并干掉

具体我们可以让这些进程放到一个进程组里面，然后向这个进程组发送消息。一种办法是使用[kill](http://linux.die.net/man/2/kill)函数。

        #include <sys/types.h>
        #include <signal.h>
        int kill(pid_t pid, int sig);
        //If pid is less than -1, then sig is sent to every process in the process group whose ID is -pid.

也可以使用[killpg](http://linux.die.net/man/2/killpg)函数

        #include <signal.h>
        int killpg(int pgrp, int sig);
        //killpg() sends the signal sig to the process group pgrp. See signal(7) for a list of signals.
        
为了获得pgid，可以使用[getpgrp](http://pubs.opengroup.org/onlinepubs/009695399/functions/getpgrp.html)函数，它返回当前进程的进程组ID。你也可以使用[getpgid](http://pubs.opengroup.org/onlinepubs/009695399/functions/getpgid.html)函数，它根据特定的PID来获取它的进程组ID。

默认情况下，子进程和父进程属于相同的进程组ID，为了修改子进程的进程组ID，必须**在[fork](http://linux.die.net/man/2/fork)之后，[exec](http://linux.die.net/man/3/exec)之前**重新重置进程组ID。可以使用[setpgrp](http://pubs.opengroup.org/onlinepubs/009695299/functions/setpgrp.html)将当前进程重置，它相当于[setpgid(0, 0)](http://pubs.opengroup.org/onlinepubs/009695399/functions/setpgid.html)。

If the calling process is not already a session leader, setpgrp() sets the process group ID of the calling process to the process ID of the calling process. If setpgrp() creates a new session, then the new session has no controlling terminal.
The setpgrp() function has no effect when the calling process is a session leader.

**为了让子进程自动退出，我们在创建任意子进程前先重置进程组id，然后捕获退出信号(SIGINT，SIGTERM等),收到信号后向当前整个进程组发送退出消息。**

**为了关闭一个shell脚本进程，我们在fork之后，exec之前重置进程组ID，当需要杀进程时，向fork的进程ID所在的进程组发送退出消息。**

