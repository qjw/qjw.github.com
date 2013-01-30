---
layout: post
title:  fd,FILE*,HANDLE相互转换
category: bash
---
         
##fd和FILE*相互转换
使用**fileno**函数从FILE指针获得fd。

使用**fdopen**函数打开一个fd并返回FILE指针

###参考
1. <http://msdn.microsoft.com/en-us/library/zs6wbdhx\(v=vs.80\).aspx>
1. <http://linux.die.net/man/3/fileno>
1. <http://pubs.opengroup.org/onlinepubs/009604499/functions/fdopen.html>

##fd和HANDLE相互转换
使用**_get_osfhandle**函数从fd获得HANDLE

使用**_open_osfhandle**函数从HANDLE获得fd


###参考
1. <http://msdn.microsoft.com/en-us/library/ks2530z6.aspx>
1. <http://msdn.microsoft.com/en-us/library/bdts1c9x.aspx>
1. <http://blog.csdn.net/zjl_wzw/article/details/6162846>

##重新打开
使用**freopen**函数重新打开FILE*

使用**DuplicateHandle**函数重新打开HANDLE

使用**dup/dup2/dup3**函数重新打开fd，但是不能变更权限，可以使用**fcntl配合F_SETFL**变更权限。
 

###参考
1. <http://www.cplusplus.com/reference/cstdio/freopen/>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms724251\(v=vs.85\).aspx>
1. <http://www.kernel.org/doc/man-pages/online/pages/man2/fcntl.2.html>
1. <http://www.kernel.org/doc/man-pages/online/pages/man2/dup.2.html>
