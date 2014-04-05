---
layout: post
title: 异步Winhttp 使用及其注意点
category: network
---

#要点
参考[借助 C++ 进行 Windows 开发：异步 WinHTTP。](http://msdn.microsoft.com/zh-cn/magazine/cc716528.aspx)

        if (WINHTTP_INVALID_STATUS_CALLBACK == ::WinHttpSetStatusCallback(
            request,
            Callback, 
            WINHTTP_CALLBACK_FLAG_ALL_NOTIFICATIONS,
            0)) // reserved
        {
            // Call GetLastError for error information
        }

        typedef void ( CALLBACK *WINHTTP_STATUS_CALLBACK)(
          _In_  HINTERNET hInternet,
          _In_  DWORD_PTR dwContext,
          _In_  DWORD dwInternetStatus,
          _In_  LPVOID lpvStatusInformation,
          _In_  DWORD dwStatusInformationLength
        );
        
#问题
参考<http://support.microsoft.com/kb/839872/zh-cn>

实际开发中会发现，部分机器有小概率会出现 ERROR_WINHTTP_OPERATION_CANCELLED，ERROR_WINHTTP_INVALID_SERVER_RESPONSE，ERROR_WINHTTP_TIMEOUT之类的错误码，但实际却是正常的。

**尽量用同步winhttp实现**
        
#实时关闭下载
对于下载，往往希望能够实时终止下载线程，但是winhttp并不能使用微软的[Wait Functions](http://msdn.microsoft.com/en-us/library/windows/desktop/ms687069\(v=vs.85\).aspx)，也不能用[select](http://linux.die.net/man/2/select)，或者[epoll](http://www.kernel.org/doc/man-pages/online/pages/man4/epoll.4.html)来多路监听，是不是只能杀进程呢？

其实可以很简单的[WinHttpCloseHandle](http://msdn.microsoft.com/en-us/library/windows/desktop/aa384090\(v=vs.85\).aspx)。此时GetLastError可能会返回ERROR_WINHTTP_OPERATION_CANCELLED或者ERROR_INVALID_HANDLE。不过这作为客户手动中断的证据显然不是很合适，所以做更高层次的判断很有必要。
        
#参考
1. <http://msdn.microsoft.com/zh-cn/magazine/cc716528.aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/aa384115(v=vs.85).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/aa383917(v=vs.85).aspx>
1. <http://support.microsoft.com/kb/839872/zh-cn>
1. <http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms681381(v=vs.85).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/aa383770(v=vs.85).aspx>