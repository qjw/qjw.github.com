---
layout: post
title:  LD_PRELOAD妙用
category: linux
---
	
在Unix操作系统的动态链接库的世界中，LD_PRELOAD就是这样一个环境变量，它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。

通过修改这个值，我们可以将默认自定义的动态库放到前面，替换系统或者其他函数，实现hook的功能，最典型的就是tcmalloc,见[这里](http://www.s135.com/post/349/)。

当然打开这个口子，也意味着极大的安全隐患，一个恶意的入侵者可以任意攻击你的系统，见[这里](http://blog.csdn.net/haoel/article/details/1602108)