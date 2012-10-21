---
layout: post
title: PostMessage权限问题
category: windows
---

跨进程发送消息,使用PostMessage非常方便,但是若从一个低权限的程序向一个高权限的程序发送消息,可能会返回5(权限受限)。

解决办法就是在高权限程序中，对接受消息的句柄用[ChangeWindowMessageFilterEx](http://msdn.microsoft.com/en-us/library/windows/desktop/dd388202\(v=vs.85\).aspx
)修改权限

        hWnd [in]
        Type: HWND
        A handle to the window whose UIPI message filter is to be modified.
        message [in]
        Type: UINT
        The message that the message filter allows through or blocks.
        action [in]
        Type: DWORD
        The action to be performed, and can take one of the following values:
        Value	Meaning
        MSGFLT_ALLOW
        1
        Allows the message through the filter. This enables the message to be received by hWnd,
        regardless of the source of the message, even it comes from a lower privileged process.
        MSGFLT_DISALLOW
        2
        Blocks the message to be delivered to hWnd if it comes from a lower privileged process, 
        unless the message is allowed process-wide by using the ChangeWindowMessageFilter function or globally.
        MSGFLT_RESET
        0
        Resets the window message filter for hWnd to the default. Any message allowed globally
        or process-wide will get through, but any message not included in those two categories, 
        and which comes from a lower privileged process, will be blocked.
         
        pChangeFilterStruct [in, out, optional]
        Type: PCHANGEFILTERSTRUCT
        Optional pointer to a CHANGEFILTERSTRUCT structure.
