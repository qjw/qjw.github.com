---
layout: post
title:  进程互斥时，拉起已知GUI进程
category: windows
---

对很多需要保证单实例的程序，当发现程序已经存在，往往需要将已知的程序显示到最前，最简单的办法就是[SetForegroundWindow](http://msdn.microsoft.com/en-us/library/windows/desktop/ms633539\(v=vs.85\).aspx)。

这种方案有一个缺陷，当已知程序处于最小化状态时，无法将其拉到最前。一个解决办法就是使用[ShowWindow](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms633548\(v=vs.85\).aspx) 并传入参数SW_SHOWNORMAL。

不过这种方案还是有一个小小的不足，若已知程序最大化了，那么会取消它的最大化，所以最好的办法就是判断是否最小化，若最小化恢复。判断是否最小化使用[IsIconic](http://msdn.microsoft.com/en-us/library/windows/desktop/ms633527\(v=vs.85\).aspx)。

        HWND hwnd = FindWindow(_T("xx"),_T("yy"));
        if(!!hwnd && IsIconic(hwnd))
        {
            ShowWindow(hwnd,SW_RESTORE);
        }
        SetForegroundWindow(hwnd);



