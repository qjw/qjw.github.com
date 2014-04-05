---
layout: post
title: 《Windows核心编程》内核对象学习笔记
category: windows
---

1. 内核对象与进程相关
1. HANDLE只是内核对象表的一个索引,根据索引可以找到具体内核对象,在这里存储这对象的具体信息,这也是为什么内核对象都可以用HANDLE来表示的原因。
1. 内核对象被所有进程共享，通过应用计数管理，所以CloseHandle之后，内核对象不一定被销毁。
1. 几乎所有的内核对象创建函数都包含一个安全属性的参数，这也是区分内核对象和应用层对象（例如HWND）的一种简单办法。

        typedef struct _SECURITY_ATTRIBUTES { 
          DWORD  nLength;   //结构体长度
          LPVOID lpSecurityDescriptor;  //安全性设置
          BOOL   bInheritHandle;  //可继承性
        } SECURITY_ATTRIBUTES, *PSECURITY_ATTRIBUTES; 

1. 创建/打开内核对象，注意用合适的权限，例如查询一个注册表，就用query权限，而不要用all_access。在Vista以及之后的Windows下可能会存在兼容性问题。
1. 内核对象表【索引（HANDLE)|内核对象内存块|访问掩码|标志】
1. 创建内核对象大部分情形都返回NULL若失败，但部分API却返回INVALID_HANDLE_VALUE(-1)。例如[CreateFile](http://msdn.microsoft.com/en-us/library/windows/desktop/aa363858\(v=vs.85\).aspx)
1. 进程结束，Windows会清理所有的内核对象，其他应用层对象也会一并清理。

##跨进程边界共享内核对象
###使用对象句柄继承
1. 在创建内核对象时，传入的[SECURITY_ATTRIBUTES](http://msdn.microsoft.com/zh-cn/library/windows/desktop/aa379560\(v=vs.85\).aspx)参数第三个字段bInheritHandle必须置为true，并且只有它的子进程才能继承
1. 设置上述标志位后，并非所有的子孙进程会自动继承，需要在[CreateProcess](http://msdn.microsoft.com/en-us/library/windows/desktop/ms682425\(v=vs.85\).aspx)j将参数bInheritHandles传入True。
1. 继承的句柄，Windows保证句柄ID（HANDLE）大小一致。
1. 子进程不知道自己继承了哪些句柄，也就是继承句柄之后必须通过其他方式通知它，例如使用参数或者环境变量。
1. 继承句柄只有在创建进程的那一刻发生，若创建子进程后再创建新的内核对象，那么这个对象不会被继承，无论是否可继承。
1. 使用[SetHandleInformation](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms724935\(v=vs.85\).aspx)可以修改内核对象的部分属性。
1. 使用[GetHandleInformation](http://msdn.microsoft.com/zh-cn/library/windows/desktop/ms724329\(v=vs.85\).aspx)来获取这些属性
  
  **HANDLE_FLAG_INHERIT**  If this flag is set, a child process created with the bInheritHandles parameter of CreateProcess set to TRUE will inherit the object handle.
  
  **HANDLE_FLAG_PROTECT_FROM_CLOSE** If this flag is set, calling the CloseHandle function will not close the object handle.

###对象命名
1. 很多内核对象创建函数包含一个对象名称，可通过这个名称在多个进程中共享，一个典型的应用就是互斥量来实现程序单实例。
1. 若创建一个已经存在的内核对象，会成功，同时返回ERROR_ALREADY_EXISTS。若返回失败，并且返回ERROR_ACCESS_DENY，也可能表示对象已经存在
1. 所有类型的内核对象共享一个名称区间，有两种内置的名称区间 local,global
1. 可以使用Open类型函数来打开已经存在的内核对象，若不存在，返回ERROR_FILE_NOT_FOUND
1. 专有命名空间，不懂！

###复制内核对象[DuplicateHandle](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724251\(v=vs.85\).aspx)

        BOOL WINAPI DuplicateHandle(
          _In_   HANDLE hSourceProcessHandle,
          _In_   HANDLE hSourceHandle,
          _In_   HANDLE hTargetProcessHandle,
          _Out_  LPHANDLE lpTargetHandle,
          _In_   DWORD dwDesiredAccess,
          _In_   BOOL bInheritHandle,
          _In_   DWORD dwOptions
        );

1. 复制后的内核对象，目标进程同样不知道，所以必须使用一定的IPC通知目标进程
1. 源进程切忌不要关闭lpTargetHandle，因为这是相对于目标进程的句柄，源进程关闭有可能会失败，更糟糕的可能会关闭本进程一个合法的其他内核对象。
1. 更多的时候，DuplicateHandle可以在同一进程使用，例如将一个可写的句柄dup出一个只读的对象，并传给一个子系统，以提高程序的健壮性。
