---
layout: post
title:  脚本里捕获信号
category: bash
---
 
在程序里面，我们可以使用signal或者sigaction函数捕获信号，在脚本里面呢？

我们可以使用[trap](http://rcsg-gsir.imsb-dsgi.nrc-cnrc.gc.ca/documents/bourne/node42.html)命令。用法很简单【trap commands signals】，例如：

        #!/bin/sh
        TMPFILE=/usr/tmp/junk.$$
        trap 'rm -f $TMPFILE; exit 0' 1 2 15 




