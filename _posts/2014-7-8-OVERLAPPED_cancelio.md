---
layout: post
title: Windows OVERLAPPED异步IO及取消
category: windows
---

CreateFile需要携带**FILE_FLAG_OVERLAPPED** 标志

	HANDLE hFileSrc = CreateFile(
		_T("test.txt"),
		GENERIC_READ,
		FILE_SHARE_READ,
		NULL,
        	OPEN_EXISTING,
        	FILE_FLAG_OVERLAPPED,
        	NULL);
        	
ReadFile时，需要使用**OVERLAPPED**结构并赋值传入。结果需要判断**GetLastError是否为ERROR_IO_PENDING**

	OVERLAPPED overlap;
	memset(&overlap, 0, sizeof(overlap));
	ReadFile(
		        hFile,
		        buf,
		        READ_SIZE,
		        &numread,
		        &overlap
		    );
		    
若GetLastError为ERROR_IO_PENDING，可以使用**WaitForSingleObject**等待IO完成，这里可以设置timeout。亦可用**GetOverlappedResult**函数等待，但是无法设置超时，最后一个参数可以设置是否阻塞还是立即返回。

若IO超时想取消，不能简单地直接CloseHandle。而要使用**CancelIo**函数取消IO

##参考
1. <http://bbs.csdn.net/topics/390774155>

