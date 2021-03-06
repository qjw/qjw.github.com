---
layout: post
title:  Linux下等待所有进程结束
category: linux
---

为了避免在信号中做过多的逻辑，以及互斥，可以考虑建一个unix sock，然后将进程退出的必需信息作为报文发送出去。在主线程中用select等待即可。用pipe若同时写可能会有乱序问题。

        #include <unistd.h>
        #include <cstdio>
        #include <sys/types.h>
        #include <sys/wait.h>


        void selfpipe_sigh(int n)
        {
            printf("child exit\n");
            int st;
            int pid;
            while((pid = waitpid(-1, &st, WNOHANG)) > 0)
            {
                if(WIFEXITED(st))
                    printf("child %d exit normal,status %d\n",pid,WEXITSTATUS(st));
                else if(WIFSIGNALED(st))
                    printf("child %d exit by singal,sig %d\n",pid,WTERMSIG(st));
            }
        }

        int main(int argc,char** argv)
        {
            struct sigaction act;
            act.sa_handler = selfpipe_sigh;
            sigaction(SIGCHLD, &act, NULL);
            act.sa_handler = SIG_IGN;
            sigaction(SIGPIPE, &act, NULL);
            
            int pid_t = fork();
            if(pid_t < 0)
                return 1;
            if(0 == pid_t)
            {
                char* new_argv[]={
                    argv[1],
                    NULL
                };
                if(execvp(argv[1],new_argv) < 0)
                    return 1;
                return 0;
            }
            else
            {
                printf("pid %d\n",pid_t);
            }
            while(true)
            {
                printf("i am running\n");
                sleep(1);
            }
            return 0;
        }



#waitpid和SIGCHID关联讨论
见<http://bbs.chinaunix.net/thread-828942-2-1.html>

根本就不需要找回来！
好比有五个进程，
不妨分别称为 p1 p2 p3 p4 p5，
一开始 p1 结束了，发了一个 SIGCHLD(s1)，
这时父进程可能空闲了，于是开始处理这个信号，假设处理的过程中 p2 又结束了，又发了一个 SIGCHLD(s2)，
这时候已经有两个信号了（一个正在处理，一个待处理），这时如果 p3 又结束了，那么它发的那个 SIGCHLD(s3) 势必会丢失，
丢失了怎么办？
没关系，因为那个信号处理函数是个循环嘛，
所以 while(waitpid()) 的时候，会把 p1 p2 p3 都处理的。
即使是很不幸，因为十分凑巧的原因，p3 没有被回收，导致变成僵尸进程了，也没关系，
因为还有 p4 p5 嘛，等到 p4 或者 p5 结束的时候，
又会再一次调用 while(waitpid())，到时候虽说这个 while(waitpid()) 是由 p4/p5 引起的，但是它也会一并把 p3 也处理的，因为它是个循环嘛！

如果还搞不懂，你就再看看 waitpid 的 man。

记住一点：
waitpid 和 SIGCHLD 没关系，即使是某个子进程对应的 SIGCHLD 丢失了，只要父进程在任何一个时刻调用了 waitpid，那么这个进程还是可以被回收的。

哎呀呀，简直费劲死了，其实说白了，就是一个“生产者－消费者”问题。
子进程结束的时候，系统“生产”出一个僵尸进程，
同时用 SIGCHLD 通知父进程来“消费”这个僵尸进程，
即使是 SIGCHLD 丢失了，没有来得及消费，
但是只要有一次消费，就会把所有的僵尸进程都处理光光！
（我再说一遍：因为，while(waitpid()) 是个循环嘛！）

#忽略SIGCHLD信号避免僵尸进程
见<http://pubs.opengroup.org/onlinepubs/7908799/xsh/sigaction.html>

If a process sets the action for the SIGCHLD signal to SIG_IGN, the behaviour is unspecified, except as specified below. If the action for the SIGCHLD signal is set to SIG_IGN, child processes of the calling processes will not be transformed into zombie processes when they terminate. If the calling process subsequently waits for its children, and the process has no unwaited for children that were transformed into zombie processes, it will block until all of its children terminate, and wait(), wait3(), waitid() and waitpid() will fail and set errno to [ECHILD]. If the Realtime Signals Extension option is supported, any queued values pending will be discarded and the resources used to queue them will be released and made available to queue other signals.  
        
#参考
1. <http://stackoverflow.com/questions/282176/waitpid-equivalent-with-timeout>
1. <http://bbs.chinaunix.net/thread-828942-2-1.html>
1. <http://pubs.opengroup.org/onlinepubs/7908799/xsh/sigaction.html>