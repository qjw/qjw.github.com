---
layout: post
title: MsgWaitForMultipleObjectsEx发送垃圾消息的问题
category: windows
---


####<http://msdn.microsoft.com/en-us/library/windows/desktop/ms684245(v=vs.85).aspx>
 
        DWORD WINAPI MsgWaitForMultipleObjectsEx(
          __in  DWORD nCount, // 句柄数量
          __in  const HANDLE *pHandles, // 句柄
          __in  DWORD dwMilliseconds, // 超时时间（句柄可以为空，此时就是一个定时器）
          __in  DWORD dwWakeMask, // 希望接收的Windows消息
          __in  DWORD dwFlags
        );


 New input of the type specified in the dwWakeMask parameter is available in the thread's input queue. Functions such as PeekMessage, GetMessage, GetQueueStatus, and WaitMessage mark messages in the queue as old messages. Therefore, after you call one of these functions, a subsequent call to MsgWaitForMultipleObjectsEx will not return until new input of the specified type arrives.
 This value is also returned upon the occurrence of a system event that requires the thread's action, such as foreground activation. Therefore, **MsgWaitForMultipleObjectsEx can return even though no appropriate input is available** and even if dwWakeMask is set to 0. If this occurs, call GetMessage or PeekMessage to process the system event before trying the call to MsgWaitForMultipleObjectsEx again.

 当返回WAIT_OBJECT_0 + nCount时，有可能并没有任何消息，而且实际测试中可能很频繁，所以注意这一点，避免因为它导致过多的消耗


另一个是 

        DWORD WINAPI MsgWaitForMultipleObjects(
          __in  DWORD nCount,
          __in  const HANDLE *pHandles,
          __in  BOOL bWaitAll,
          __in  DWORD dwMilliseconds,
          __in  DWORD dwWakeMask
        );

