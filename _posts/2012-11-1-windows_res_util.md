---
layout: post
title: Windows 图片处理的部分工具函数
category: wtl
---

        /*
         * resutil.h
         *
         *  Created on: 2012-5-16
         *      Author: king
         */

        #ifndef RESUTIL_H_
        #define RESUTIL_H_

        #include <atlimage.h>
        #include <atlgdi.h>
        #include <atlmisc.h>

        #pragma comment(lib, "gdiplus.lib")
        // 初始化
        Gdiplus::Status	InitUtil(ULONG_PTR* gdiplusToken);
        void	FiniUtil(ULONG_PTR gdiplusToken);
        /////////////////////////////////////////////////////
        // 从文件中读取图片
        // 支持BMP, GIF, JPEG, PNG, and TIFF.
        bool	File2CImage(CImage* img,const TCHAR* path);
        // 平整alpha通道失效的问题
        void	ImplAlpha(CImage* img);
        inline bool	File2CImageAndImplAlpha(CImage* img,const TCHAR* path){
            if(File2CImage(img,path)){
                ImplAlpha(img);
                return true;
            }else{
                return false;
            }
        }

        // 支持bmp
        bool	File2CBitmap(CBitmap* img,const TCHAR* path);

        // 支持BMP, GIF, JPEG, PNG, TIFF, and EMF.
        bool	File2GdiplusImage(Gdiplus::Image** img,const TCHAR* path);
        //end

        /////////////////////////////////////////////////////
        // 平铺图片
        void	TileBitBlt(
                CDCHandle & pDC,
                CBitmap & bitmap,
                const CRect & rect);
        void	TileTransparentBlt(
                CDCHandle & pDC,
                CBitmap & bitmap,
                const CRect & rect,
                COLORREF crTransparent);
        //end

        // 图片到rgn
        HRGN BitmapToRegion (
                HBITMAP hBmp,
                COLORREF cTransparentColor = 0,
                COLORREF cTolerance = 0x101010);


        #endif /* RESUTIL_H_ */

---

        /*
         * resutil.cpp
         *
         *  Created on: 2012-5-16
         *      Author: king
         */

        #include "resutil.h"

        // http://msdn.microsoft.com/en-us/library/ms534077(v=vs.85).aspx
        Gdiplus::Status	InitUtil(ULONG_PTR* gdiplusToken){
            ATLASSERT(gdiplusToken);
            Gdiplus::GdiplusStartupInput gdiplusStartupInput;
            return Gdiplus::GdiplusStartup(gdiplusToken, &gdiplusStartupInput, NULL);
        }

        void	FiniUtil(ULONG_PTR gdiplusToken){
            Gdiplus::GdiplusShutdown(gdiplusToken);
        }

        // http://msdn.microsoft.com/en-us/library/tf4bytf8.aspx
        bool	File2CImage(CImage* img,const TCHAR* path){
            ATLASSERT(img);
            ATLASSERT(path);
            if(!img->IsNull()){
                ATLASSERT(0);
                img->Destroy();
            }
            return S_OK == img->Load(path);
        }
        // 平整alpha通道失效的问题
        void	ImplAlpha(CImage* img){
            ATLASSERT(img);
            CSize size_(img->GetWidth(),img->GetHeight());
            //处理CImage不能透明问题;
            unsigned char* pColor;
            for(int i = 0;i < size_.cx; i++)
            {
                for(int j = 0; j < size_.cy; j++)
                {
                    pColor = (unsigned char *) img->GetPixelAddress(i,j);
                    if(pColor[3] < 0xFF){
                        pColor[0] = pColor[0] * pColor[3]/0xFF;
                        pColor[1] = pColor[1] * pColor[3]/0xFF;
                        pColor[2] = pColor[2] * pColor[3]/0xFF;
                    }
                }
            }
        }

        bool	File2CBitmap(CBitmap* img,const TCHAR* path){
            ATLASSERT(img);
            ATLASSERT(path);
            HBITMAP pbitmap_ = (HBITMAP) LoadImage(
                    NULL,
                    path,
                    IMAGE_BITMAP,
                    0,
                    0,
                    LR_CREATEDIBSECTION | LR_LOADFROMFILE);
            if (pbitmap_ == NULL)
                return false;
            img->Attach(pbitmap_);
            return true;
        }

        // http://msdn.microsoft.com/en-us/library/ms535370(v=vs.85).aspx
        bool	File2GdiplusImage(Gdiplus::Image** img,const TCHAR* path){
            ATLASSERT(img);
            ATLASSERT(path);
            *img = Gdiplus::Image::FromFile(path);
            if(!(*img))
                return false;
            return Gdiplus::Ok == (*img)->GetLastStatus();
        }

        ///////////////////////////////////////////////////////////////
        typedef BOOL (*BltTemp)(
                CDCHandle & ,
                int, int, int, int,
                HDC,
                int, int,
                COLORREF);

        inline 	BOOL TransparentBltImp(
                CDCHandle & pDC,
                int x, int y, int nWidth, int nHeight,
                HDC hSrcDC,
                int xSrc, int ySrc,
                COLORREF crTransparent){
            return pDC.TransparentBlt(
                    x,y,nWidth,nHeight,
                    hSrcDC,
                    xSrc,ySrc,nWidth,nHeight,
                    crTransparent);
        }
        inline 	BOOL BitBltImp(
                CDCHandle & pDC,
                int x, int y, int nWidth, int nHeight,
                HDC hSrcDC,
                int xSrc, int ySrc,
                COLORREF){
            return pDC.BitBlt(x,y,nWidth,nHeight,hSrcDC,xSrc,ySrc,SRCCOPY);
        }

        template<BltTemp Func>
        void	TileBitmapT(
                CDCHandle & pDC,
                CBitmap & bitmap,
                const CRect & rect,
                COLORREF colorMask)
        {

            CSize imgSize_;
            bitmap.GetSize(imgSize_);

            CDC dcMem;
            dcMem.CreateCompatibleDC ( pDC );
            dcMem.SelectBitmap(bitmap);

            LONG top_ = rect.top,height_;
            while(top_ < rect.bottom) {
                height_ = imgSize_.cy;
                if( top_ + height_ > rect.bottom ) {
                    height_ = rect.bottom - top_;
                }

                LONG width_;
                LONG left_ = rect.left;
                int right_ = rect.right;
                while( left_ < right_) {
                    width_ = imgSize_.cx;
                    if( left_ + width_ > right_ ) {
                        width_ = right_ - left_;
                    }
                    Func(pDC,left_,top_,width_,height_,dcMem, 0, 0,colorMask);
                    left_ += width_;
                }
                top_ += height_;
            }
        }

        void	TileBitBlt(
                CDCHandle & pDC,
                CBitmap & bitmap,
                const CRect & rect)
        {
            TileBitmapT<BitBltImp>(pDC,bitmap,rect,0);
        }

        void	TileTransparentBlt(
                CDCHandle & pDC,
                CBitmap & bitmap,
                const CRect & rect,
                COLORREF crTransparent){
            TileBitmapT<TransparentBltImp>(pDC,bitmap,rect,crTransparent);
        }

        // http://www.codeguru.com/cpp/g-m/bitmap/usingregions/article.php/c1751/Converting-a-bitmap-to-a-region.htm
        //
        //	BitmapToRegion :	Create a region from the "non-transparent" pixels of a bitmap
        //	Author :			Jean-Edouard Lachand-Robert (http://www.geocities.com/Paris/LeftBank/1160/resume.htm), June 1998.
        //
        //	hBmp :				Source bitmap
        //	cTransparentColor :	Color base for the "transparent" pixels (default is black)
        //	cTolerance :		Color tolerance for the "transparent" pixels.
        //
        //	A pixel is assumed to be transparent if the value of each of its 3 components (blue, green and red) is
        //	greater or equal to the corresponding value in cTransparentColor and is lower or equal to the
        //	corresponding value in cTransparentColor + cTolerance.
        //
        HRGN BitmapToRegion (HBITMAP hBmp, COLORREF cTransparentColor, COLORREF cTolerance)
        {
            HRGN hRgn = NULL;

            if (hBmp)
            {
                // Create a memory DC inside which we will scan the bitmap content
                HDC hMemDC = CreateCompatibleDC(NULL);
                if (hMemDC)
                {
                    // Get bitmap size
                    BITMAP bm;
                    GetObject(hBmp, sizeof(bm), &bm);

                    // Create a 32 bits depth bitmap and select it into the memory DC
                    BITMAPINFOHEADER RGB32BITSBITMAPINFO = {
                            sizeof(BITMAPINFOHEADER),	// biSize
                            bm.bmWidth,					// biWidth;
                            bm.bmHeight,				// biHeight;
                            1,							// biPlanes;
                            32,							// biBitCount
                            BI_RGB,						// biCompression;
                            0,							// biSizeImage;
                            0,							// biXPelsPerMeter;
                            0,							// biYPelsPerMeter;
                            0,							// biClrUsed;
                            0							// biClrImportant;
                    };
                    VOID * pbits32;
                    HBITMAP hbm32 = CreateDIBSection(hMemDC, (BITMAPINFO *)&RGB32BITSBITMAPINFO, DIB_RGB_COLORS, &pbits32, NULL, 0);
                    if (hbm32)
                    {
                        HBITMAP holdBmp = (HBITMAP)SelectObject(hMemDC, hbm32);

                        // Create a DC just to copy the bitmap into the memory DC
                        HDC hDC = CreateCompatibleDC(hMemDC);
                        if (hDC)
                        {
                            // Get how many bytes per row we have for the bitmap bits (rounded up to 32 bits)
                            BITMAP bm32;
                            GetObject(hbm32, sizeof(bm32), &bm32);
                            while (bm32.bmWidthBytes % 4)
                                bm32.bmWidthBytes++;

                            // Copy the bitmap into the memory DC
                            HBITMAP holdBmp = (HBITMAP)SelectObject(hDC, hBmp);
                            BitBlt(hMemDC, 0, 0, bm.bmWidth, bm.bmHeight, hDC, 0, 0, SRCCOPY);

                            // For better performances, we will use the ExtCreateRegion() function to create the
                            // region. This function take a RGNDATA structure on entry. We will add rectangles by
                            // amount of ALLOC_UNIT number in this structure.
                            #define ALLOC_UNIT	100
                            DWORD maxRects = ALLOC_UNIT;
                            HANDLE hData = GlobalAlloc(GMEM_MOVEABLE, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects));
                            RGNDATA *pData = (RGNDATA *)GlobalLock(hData);
                            pData->rdh.dwSize = sizeof(RGNDATAHEADER);
                            pData->rdh.iType = RDH_RECTANGLES;
                            pData->rdh.nCount = pData->rdh.nRgnSize = 0;
                            SetRect(&pData->rdh.rcBound, MAXLONG, MAXLONG, 0, 0);

                            // Keep on hand highest and lowest values for the "transparent" pixels
                            BYTE lr = GetRValue(cTransparentColor);
                            BYTE lg = GetGValue(cTransparentColor);
                            BYTE lb = GetBValue(cTransparentColor);
                            BYTE hr = min(0xff, lr + GetRValue(cTolerance));
                            BYTE hg = min(0xff, lg + GetGValue(cTolerance));
                            BYTE hb = min(0xff, lb + GetBValue(cTolerance));

                            // Scan each bitmap row from bottom to top (the bitmap is inverted vertically)
                            BYTE *p32 = (BYTE *)bm32.bmBits + (bm32.bmHeight - 1) * bm32.bmWidthBytes;
                            for (int y = 0; y < bm.bmHeight; y++)
                            {
                                // Scan each bitmap pixel from left to right
                                for (int x = 0; x < bm.bmWidth; x++)
                                {
                                    // Search for a continuous range of "non transparent pixels"
                                    int x0 = x;
                                    LONG *p = (LONG *)p32 + x;
                                    while (x < bm.bmWidth)
                                    {
                                        BYTE b = GetRValue(*p);
                                        if (b >= lr && b <= hr)
                                        {
                                            b = GetGValue(*p);
                                            if (b >= lg && b <= hg)
                                            {
                                                b = GetBValue(*p);
                                                if (b >= lb && b <= hb)
                                                    // This pixel is "transparent"
                                                    break;
                                            }
                                        }
                                        p++;
                                        x++;
                                    }

                                    if (x > x0)
                                    {
                                        // Add the pixels (x0, y) to (x, y+1) as a new rectangle in the region
                                        if (pData->rdh.nCount >= maxRects)
                                        {
                                            GlobalUnlock(hData);
                                            maxRects += ALLOC_UNIT;
                                            hData = GlobalReAlloc(hData, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), GMEM_MOVEABLE);
                                            pData = (RGNDATA *)GlobalLock(hData);
                                        }
                                        RECT *pr = (RECT *)&pData->Buffer;
                                        SetRect(&pr[pData->rdh.nCount], x0, y, x, y+1);
                                        if (x0 < pData->rdh.rcBound.left)
                                            pData->rdh.rcBound.left = x0;
                                        if (y < pData->rdh.rcBound.top)
                                            pData->rdh.rcBound.top = y;
                                        if (x > pData->rdh.rcBound.right)
                                            pData->rdh.rcBound.right = x;
                                        if (y+1 > pData->rdh.rcBound.bottom)
                                            pData->rdh.rcBound.bottom = y+1;
                                        pData->rdh.nCount++;

                                        // On Windows98, ExtCreateRegion() may fail if the number of rectangles is too
                                        // large (ie: > 4000). Therefore, we have to create the region by multiple steps.
                                        if (pData->rdh.nCount == 2000)
                                        {
                                            HRGN h = ExtCreateRegion(NULL, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), pData);
                                            if (hRgn)
                                            {
                                                CombineRgn(hRgn, hRgn, h, RGN_OR);
                                                DeleteObject(h);
                                            }
                                            else
                                                hRgn = h;
                                            pData->rdh.nCount = 0;
                                            SetRect(&pData->rdh.rcBound, MAXLONG, MAXLONG, 0, 0);
                                        }
                                    }
                                }

                                // Go to next row (remember, the bitmap is inverted vertically)
                                p32 -= bm32.bmWidthBytes;
                            }

                            // Create or extend the region with the remaining rectangles
                            HRGN h = ExtCreateRegion(NULL, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), pData);
                            if (hRgn)
                            {
                                CombineRgn(hRgn, hRgn, h, RGN_OR);
                                DeleteObject(h);
                            }
                            else
                                hRgn = h;

                            // Clean up
                            SelectObject(hDC, holdBmp);
                            DeleteDC(hDC);
                        }

                        DeleteObject(SelectObject(hMemDC, holdBmp));
                    }

                    DeleteDC(hMemDC);
                }
            }

            return hRgn;
        }
