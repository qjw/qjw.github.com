---
layout: post
title: Cancellation Points
category: cpp
---

##可中断函数

通常为了和谐地退出，希望让子线程合理的退出并等待，那么若子线程卡在某个诸如select，sleep等函数时，怎么办呢？

以下函数是可中断函数，直接pthread_cancel就可以让其理解返回

<table cellpadding="3">
<tbody><tr valign="top">
<td align="left">
<p class="tent"><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/accept.html"><i>accept</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/aio_suspend.html"><i>aio_suspend</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/clock_nanosleep.html"><i>clock_nanosleep</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/close.html"><i>close</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/connect.html"><i>connect</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/creat.html"><i>creat</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/fcntl.html"><i>fcntl</i>()</a><a href="#tag_foot_2"><sup><small>2</small></sup></a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/fdatasync.html"><i>fdatasync</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/fsync.html"><i>fsync</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/getmsg.html"><i>getmsg</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/getpmsg.html"><i>getpmsg</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/lockf.html"><i>lockf</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/mq_receive.html"><i>mq_receive</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/mq_send.html"><i>mq_send</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/mq_timedreceive.html"><i>mq_timedreceive</i>()</a><br>
&nbsp;</p>
</td>
<td align="left">
<p class="tent"><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/mq_timedsend.html"><i>mq_timedsend</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/msgrcv.html"><i>msgrcv</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/msgsnd.html"><i>msgsnd</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/msync.html"><i>msync</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/nanosleep.html"><i>nanosleep</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/open.html"><i>open</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pause.html"><i>pause</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/poll.html"><i>poll</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pread.html"><i>pread</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pselect.html"><i>pselect</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pthread_cond_timedwait.html"><i>pthread_cond_timedwait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pthread_cond_wait.html"><i>pthread_cond_wait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pthread_join.html"><i>pthread_join</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pthread_testcancel.html"><i>pthread_testcancel</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/putmsg.html"><i>putmsg</i>()</a><br>
&nbsp;</p>
</td>
<td align="left">
<p class="tent"><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/putpmsg.html"><i>putpmsg</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/pwrite.html"><i>pwrite</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/read.html"><i>read</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/readv.html"><i>readv</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/recv.html"><i>recv</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/recvfrom.html"><i>recvfrom</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/recvmsg.html"><i>recvmsg</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/select.html"><i>select</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sem_timedwait.html"><i>sem_timedwait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sem_wait.html"><i>sem_wait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/send.html"><i>send</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sendmsg.html"><i>sendmsg</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sendto.html"><i>sendto</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sigpause.html"><i>sigpause</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sigsuspend.html"><i>sigsuspend</i>()</a><br>
&nbsp;</p>
</td>
<td align="left">
<p class="tent"><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sigtimedwait.html"><i>sigtimedwait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sigwait.html"><i>sigwait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sigwaitinfo.html"><i>sigwaitinfo</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/sleep.html"><i>sleep</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/system.html"><i>system</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/tcdrain.html"><i>tcdrain</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/usleep.html"><i>usleep</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/wait.html"><i>wait</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/waitid.html"><i>waitid</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/waitpid.html"><i>waitpid</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/write.html"><i>write</i>()</a><br>
<a href="http://pubs.opengroup.org/onlinepubs/007904975/functions/writev.html"><i>writev</i>()</a><br>
&nbsp;</p>
</td>
</tr>
</tbody></table>


##参考
1. <http://pubs.opengroup.org/onlinepubs/007904975/functions/xsh_chap02_09.html>
1. <http://stackoverflow.com/questions/433989/posix-cancellation-points>
