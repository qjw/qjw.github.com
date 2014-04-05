---
layout: post
title:  从资源中读取图片
category: wingui
---

##Bitmap
直接使用**[LoadImage](http://msdn.microsoft.com/en-us/library/windows/desktop/ms648045\(v=vs.85\).aspx)**函数从资源中加载图片，

	hBitmap = (HBITMAP)::LoadImage(::AfxGetInstanceHandle(),
		MAKEINTRESOURCE(IDB_BITMAP1), 
		IMAGE_BITMAP, 0,0,
		LR_CREATEDIBSECTION);
		
##从资源中获取流对象

	// 查找资源
    HRSRC hRsrc = ::FindResource(AfxGetResourceHandle(), MAKEINTRESOURCE(nResID), lpTyp);
    if (hRsrc == NULL) return false;

    // 加载资源
    HGLOBAL hImgData = ::LoadResource(AfxGetResourceHandle(), hRsrc);
    if (hImgData == NULL)
    {
        ::FreeResource(hImgData);
        return false;
    }

    // 锁定内存中的指定资源
    LPVOID lpVoid    = ::LockResource(hImgData);

    LPSTREAM pStream = NULL;
    DWORD dwSize    = ::SizeofResource(AfxGetResourceHandle(), hRsrc);
    HGLOBAL hNew    = ::GlobalAlloc(GHND, dwSize);
    LPBYTE lpByte    = (LPBYTE)::GlobalLock(hNew);
    ::memcpy(lpByte, lpVoid, dwSize);

    // 解除内存中的指定资源
    ::GlobalUnlock(hNew);

    // 从指定内存创建流对象
    HRESULT ht = ::CreateStreamOnHGlobal(hNew, TRUE, &pStream);
    if ( ht != S_OK )
    {
        GlobalFree(hNew);
    }
    else
    {
        // 加载图片
        //pImage->Load(pStream);
		//////
        GlobalFree(hNew);
    }
    // 释放资源
    ::FreeResource(hImgData);
	
##CImage
该类有一个成员函数**[Load](http://msdn.microsoft.com/en-US/library/tf4bytf8\(v=vs.80\).aspx)**，或者直接使用**[LoadFromResource](http://msdn.microsoft.com/en-US/library/y5da5112\(v=vs.80\).aspx)**函数。

##Gdiplus::Image
该类有个构造函数直接从[IStream](http://msdn.microsoft.com/en-us/library/windows/desktop/ms535410\(v=vs.85\).aspx)对象中初始化。也可以使用工具函数[FromStream](http://msdn.microsoft.com/en-us/library/windows/desktop/ms535371\(v=vs.85\).aspx)工具函数

##参考
1. <http://blog.csdn.net/hanjiangying/article/details/5489500>