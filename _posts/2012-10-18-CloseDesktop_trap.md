---
layout: post
title: CloseDesktop及其陷阱
category: windows
---

若需要线程能够在另一个DeskTop里面工作（比如发送UI消息），就需要使用SetThreadDesktop 设置其Desktop属性。

但是若动态的使用OpenDesktop 来获取desktop，那么就有可能导致desktop对象泄漏，因为在线程终止前该HDESK无法释放，

就算调用CloseDesktop也会返回False，GetLastError返回"正在使用"。除非这个线程退出

##参考
* <http://msdn.microsoft.com/en-us/library/windows/desktop/ms686250(v=vs.85).aspx>
* <http://msdn.microsoft.com/en-us/library/windows/desktop/ms682024(v=vs.85).aspx>

 The CloseDesktop function will fail if any thread in the calling process is using the specified desktop handle or if the handle refers to the initial desktop of the calling process.

