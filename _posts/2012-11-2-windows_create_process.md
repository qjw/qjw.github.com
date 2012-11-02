---
layout: post
title: Windows 创建进程经验小结
category: windows
---

#CreateProcess
Windows创建进程的API是[CreateProcess](http://msdn.microsoft.com/en-us/library/windows/desktop/ms682425\(v=vs.85\).aspx)。用法见[Sample](http://msdn.microsoft.com/en-us/library/windows/desktop/ms682512\(v=vs.85\).aspx)

        BOOL WINAPI CreateProcess(
          _In_opt_     LPCTSTR lpApplicationName,
          _Inout_opt_  LPTSTR lpCommandLine,
          _In_opt_     LPSECURITY_ATTRIBUTES lpProcessAttributes,
          _In_opt_     LPSECURITY_ATTRIBUTES lpThreadAttributes,
          _In_         BOOL bInheritHandles,
          _In_         DWORD dwCreationFlags,
          _In_opt_     LPVOID lpEnvironment,
          _In_opt_     LPCTSTR lpCurrentDirectory,
          _In_         LPSTARTUPINFO lpStartupInfo,
          _Out_        LPPROCESS_INFORMATION lpProcessInformation
        );

---

        #include <windows.h>
        #include <stdio.h>
        #include <tchar.h>

        void _tmain( int argc, TCHAR *argv[] )
        {
            STARTUPINFO si;
            PROCESS_INFORMATION pi;

            ZeroMemory( &si, sizeof(si) );
            si.cb = sizeof(si);
            ZeroMemory( &pi, sizeof(pi) );

            if( argc != 2 )
            {
                printf("Usage: %s [cmdline]\n", argv[0]);
                return;
            }

            // Start the child process. 
            if( !CreateProcess( NULL,   // No module name (use command line)
                argv[1],        // Command line
                NULL,           // Process handle not inheritable
                NULL,           // Thread handle not inheritable
                FALSE,          // Set handle inheritance to FALSE
                0,              // No creation flags
                NULL,           // Use parent's environment block
                NULL,           // Use parent's starting directory 
                &si,            // Pointer to STARTUPINFO structure
                &pi )           // Pointer to PROCESS_INFORMATION structure
            ) 
            {
                printf( "CreateProcess failed (%d).\n", GetLastError() );
                return;
            }

            // Wait until child process exits.
            WaitForSingleObject( pi.hProcess, INFINITE );

            // Close process and thread handles. 
            CloseHandle( pi.hProcess );
            CloseHandle( pi.hThread );
        }

##参数
若不带参数，我们可以将程序放到lpApplicationName或者lpCommandLine参数。

若带参数，可以将整个字符串放到lpCommandLine参数。

若程序和参数分开，那么程序路径放在lpApplicationName，参数放在lpCommandLine。**同时必须保证lpCommandLine前面有一个空格**
        
#GetExitCodeProcess 
当我们需要获取程序返回值时，使用[GetExitCodeProcess](http://msdn.microsoft.com/en-us/library/windows/desktop/ms683189\(v=vs.85\).aspx)

#ShellExecuteEx
对于一般的情况，[ShellExecuteEx](http://msdn.microsoft.com/en-us/library/windows/desktop/bb762154\(v=vs.85\).aspx)相比CreateProcess更加容易使用，**特别是依赖UAC时，就必须使用ShellExecuteEx了，此时CreateProcess会返回错误码ERROR_ELEVATION_REQUIRED**,具体细节见[这里](http://technet.microsoft.com/en-us/library/cc722422\(v=ws.10\).aspx)。它有个兄弟函数[ShellExecute](http://msdn.microsoft.com/en-us/library/windows/desktop/bb762153\(v=vs.85\).aspx)。Sample见[这里](http://msdn.microsoft.com/zh-cn/library/windows/desktop/bb776886\(v=vs.85\).aspx)

为了获得进程句柄，**需要附带标志位[SEE_MASK_NOCLOSEPROCESS](http://msdn.microsoft.com/en-us/library/windows/desktop/bb759784\(v=vs.85\).aspx)**

ShellExecuteEx不支持CreateProcess那种将程序和参数放到同一字符串中直接运行的用法，**你必须将程序名称传给SHELLEXECUTEINFO结构的lpFile，参数传给lpParameters。参数无前面空格的要求**

##UAC提权
用过Vista或者更高版本的Windows系统的人都知道，某些程序起来的时候会弹出一个UAC的对话框，那么自己写的程序怎么让Windows弹出此确认呢？

方法一是在manifest中添加特殊的字段，见[这里](http://blog.csdn.net/limiko/article/details/5713182)

        <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
            <security>
                <requestedPrivileges>
                    <requestedExecutionLevel level="requireAdministrator" uiAccess="false"/>
                </requestedPrivileges>
            </security>
        </trustInfo>

在VS2008中只需要动手选择一下就好了,**项目属性->配置属性->连接器->manifest file : 将UAC Execution Level勾选为requireAdministrator**


##[简单绕过UAC](http://blog.csdn.net/magictong/article/details/4804862)
前面的方法大部分情况下能够很好的工作，但是有一些例外，例如开机启动的程序。因为用uac提权需要弹出一个框来用户确认，但是开机启动很显然这做不到，所以系统会直接拦掉。那怎么解决呢？

实际上我们可以用ShellExecuteEx来做到动态提权，对于开机启动的情形，我们可以不使用menifest的requireAdministrator属性，开机用普通用户权限启动，当工作的时候再动态提权。具体参考下面的**runas**。

        SHELLEXECUTEINFO sei = {0};
        sei.cbSize = sizeof(SHELLEXECUTEINFOW);
        // ... other option
        sei.lpFile = your_path;
        sei.lpVerb = _T("runas");
        sei.lpDirectory = NULL;
        ShellExecuteEx(&sei);

#Bug
当我们用ShellExecuteEx去启动某个程序时，若需要提权，那么uac会弹出一个对话框来用户确认，**在这个对话框返回之前，ShellExecuteEx函数不会返回，也就是调用ShellExecuteEx的线程会被阻塞，若该线程有其他后台逻辑，慎用**。

##消息接管
笔者曾用一个后台线程处理各种逻辑，通过Windows的消息机制和其他线程通信（避免使用锁）。此线程平时都阻塞在MsgWaitForMultipleObjects函数中。在这些消息处理程序中，就包含ShellExecuteEx的调用，很明显，当弹出UAC对话框时，整个线程被阻塞了。

此时，其他线程还会向它发送消息，并且PostThreadMessage也是成功的，我最初认为ShellExecuteEx返回之后，MsgWaitForMultipleObjects应该能够立即返回并且收到此前被Post的消息。但实际上这些消息虽然Post成功，但就是没有收到，丢弃了还是被其他地方处理掉了？

据查证，这些消息直接被UAC的那个框给处理掉了。

解决办法就是创建一个子线程，在这个子线程中调用ShellExecuteEx，同时WaitForSingleObject这个线程。虽然在ShellExecuteEx返回之前，子线程不会结束，此线程也就是阻塞的，但是ShellExecuteEx返回之后，阻塞过程中被Post的消息是不会丢失的。MsgWaitForMultipleObjects会立即返回并收到那些消息。


#参考
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms682425(v=vs.85).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/bb762153(v=vs.85).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/bb762154(v=vs.85).aspx>
1. <http://linux.die.net/man/2/fork>
1. <http://blogs.itecn.net/blogs/winvista/archive/2006/07/25/3021.aspx>
1. <http://blog.csdn.net/magictong/article/details/4804862>
1. <http://www.shily.net/topic/bypassing-uac-since-the-launch-on-vista-win7/>