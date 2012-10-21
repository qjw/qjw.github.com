---
layout: post
title: CBitmap工具函数
category: wtl
---

        #include "stdafx.h"
        #include "atlmisc.h"
        #include "atlcrack.h"
        #include "basedlg.h"
        #include <cassert>

        bool		LoadBitmapFromFile(CBitmap* bitmap,const TCHAR* path);

        void		MakeWindowRgn (CWindow * window,const CBitmap& bitmapBk,COLORREF colorMask);

        bool		TileBitmap(CDCHandle & pDC, const CBitmap & bitmap,const CRect& rect);

        void 		BitmapToRegion (
                const CBitmap& hBmp,
                CRgn* rgn,
                COLORREF cTransparentColor = 0,
                COLORREF cTolerance = 0x000000);
                
---

        #include <string>


        bool		LoadBitmapFromFile(CBitmap* bitmap,const TCHAR* path){
            ATLASSERT(bitmap);
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
            bitmap->Attach(pbitmap_);
            return true;
        }

        void 		BitmapToRegion (
                const CBitmap& hBmp,
                CRgn* rgn,
                COLORREF cTransparentColor,
                COLORREF cTolerance)
        {
            CRgn hRgn;
            ATLASSERT(rgn);
            ATLASSERT(hBmp.m_hBitmap);
            if (hBmp.m_hBitmap) {
                // Create a memory DC inside which we will scan the bitmap conten
                CDC hMemDC_;
                hMemDC_.CreateCompatibleDC(NULL);
                ATLASSERT(hMemDC_.m_hDC);
                if (hMemDC_.m_hDC) {
                    // Get bitmap siz
                    BITMAP bm;
                    hBmp.GetBitmap(bm);

                    // Create a 32 bits depth bitmap and select it into the memory DC
                    BITMAPINFOHEADER RGB32BITSBITMAPINFO = {
                        sizeof(BITMAPINFOHEADER),    // biSize
                        bm.bmWidth,                    // biWidth;
                        bm.bmHeight,                // biHeight;
                        1,                            // biPlanes;
                        32,                            // biBitCount
                        BI_RGB,                        // biCompression;
                        0,                            // biSizeImage;
                        0,                            // biXPelsPerMeter;
                        0,                            // biYPelsPerMeter;
                        0,                            // biClrUsed;
                        0                            // biClrImportant;
                    };
                    VOID * pbits32;
                    CBitmap hbm32_;
                    hbm32_.CreateDIBSection(hMemDC_, (BITMAPINFO *)&RGB32BITSBITMAPINFO, DIB_RGB_COLORS, &pbits32, NULL, 0);
                    ATLASSERT(hbm32_.m_hBitmap);
                    if (hbm32_.m_hBitmap) {
                        HBITMAP holdBmp = hMemDC_.SelectBitmap(hbm32_);

                        // Create a DC just to copy the bitmap into the memory D
                        CDC hDc_;
                        hDc_.CreateCompatibleDC(hMemDC_);
                        ATLASSERT(hDc_.m_hDC);
                        if (hDc_.m_hDC) {
                            // Get how many bytes per row we have for the bitmap bits (rounded up to 32 bits
                            BITMAP bm32;
                            ATLVERIFY(hbm32_.GetBitmap(bm32));
                            while (bm32.bmWidthBytes % 4)
                                bm32.bmWidthBytes++;

                            // Copy the bitmap into the memory D
                            HBITMAP holdBmp = hDc_.SelectBitmap(hBmp);
                            ATLVERIFY(hMemDC_.BitBlt(0, 0, bm.bmWidth, bm.bmHeight, hDc_, 0, 0, SRCCOPY));

                            // For better performances, we will use the ExtCreateRegion() function to create th
                            // region. This function take a RGNDATA structure on entry. We will add rectangles b
                            // amount of ALLOC_UNIT number in this structure
        #define ALLOC_UNIT    100
                            DWORD maxRects = ALLOC_UNIT;
                            HANDLE hData = GlobalAlloc(GMEM_MOVEABLE, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects));
                            RGNDATA *pData = (RGNDATA *)GlobalLock(hData);
                            pData->rdh.dwSize = sizeof(RGNDATAHEADER);
                            pData->rdh.iType = RDH_RECTANGLES;
                            pData->rdh.nCount = pData->rdh.nRgnSize = 0;
                            SetRect(&pData->rdh.rcBound, MAXLONG, MAXLONG, 0, 0);

                            // Keep on hand highest and lowest values for the "transparent" pixel
                            BYTE lr = GetRValue(cTransparentColor);
                            BYTE lg = GetGValue(cTransparentColor);
                            BYTE lb = GetBValue(cTransparentColor);
                            BYTE hr = min(0xff, lr + GetRValue(cTolerance));
                            BYTE hg = min(0xff, lg + GetGValue(cTolerance));
                            BYTE hb = min(0xff, lb + GetBValue(cTolerance));

                            // Scan each bitmap row from bottom to top (the bitmap is inverted vertically
                            BYTE *p32 = (BYTE *)bm32.bmBits + (bm32.bmHeight - 1) * bm32.bmWidthBytes;
                            for (int y = 0; y < bm.bmHeight; y++) {
                                // Scan each bitmap pixel from left to righ
                                for (int x = 0; x < bm.bmWidth; x++) {
                                    // Search for a continuous range of "non transparent pixels"
                                    int x0 = x;
                                    LONG *p = (LONG *)p32 + x;
                                    while (x < bm.bmWidth) {
                                        BYTE b = GetRValue(*p);
                                        if (b >= lr && b <= hr) {
                                            b = GetGValue(*p);
                                            if (b >= lg && b <= hg) {
                                                b = GetBValue(*p);
                                                if (b >= lb && b <= hb)
                                                    // This pixel is "transparent"
                                                    break;
                                            }
                                        }
                                        p++;
                                        x++;
                                    }

                                    if (x > x0) {
                                        // Add the pixels (x0, y) to (x, y+1) as a new rectangle in the regio
                                        if (pData->rdh.nCount >= maxRects) {
                                            GlobalUnlock(hData);
                                            maxRects += ALLOC_UNIT;
                                            ATLVERIFY(hData = GlobalReAlloc(hData, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), GMEM_MOVEABLE));
                                            pData = (RGNDATA *)GlobalLock(hData);
                                            ATLASSERT(pData);
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

                                        // On Windows98, ExtCreateRegion() may fail if the number of rectangles is to
                                        // large (ie: > 4000). Therefore, we have to create the region by multiple steps
                                        if (pData->rdh.nCount == 2000) {
                                            HRGN h = ExtCreateRegion(NULL, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), pData);
                                            ATLASSERT(h);
                                            if (hRgn.m_hRgn) {
                                                hRgn.CombineRgn(hRgn, h, RGN_OR);
                                                DeleteObject(h);
                                            } else
                                                hRgn.Attach(h);
                                            pData->rdh.nCount = 0;
                                            SetRect(&pData->rdh.rcBound, MAXLONG, MAXLONG, 0, 0);
                                        }
                                    }
                                }

                                // Go to next row (remember, the bitmap is inverted vertically
                                p32 -= bm32.bmWidthBytes;
                            }

                            // Create or extend the region with the remaining rectangle
                            HRGN h = ExtCreateRegion(NULL, sizeof(RGNDATAHEADER) + (sizeof(RECT) * maxRects), pData);
                            ATLASSERT(h);
                            if (hRgn.m_hRgn) {
                                hRgn.CombineRgn(hRgn, h, RGN_OR);
                                DeleteObject(h);
                            } else
                                hRgn.Attach(h);

                            // Clean u
                            hDc_.SelectBitmap(holdBmp);
                        }
                        hMemDC_.SelectBitmap(holdBmp);
                    }
                }
            }
            rgn->Attach(hRgn.Detach());
        }

        void		MakeWindowRgn (CWindow * window,const CBitmap& bitmapBk,COLORREF colorMask)
        {
            CRgn rgn_;
            BitmapToRegion( bitmapBk,&rgn_,colorMask);
            window->SetWindowRgn (rgn_, TRUE);
        }

        bool		TileBitmap(CDCHandle & pDC, const CBitmap & bitmap,const CRect& rect)
        {
            int top = rect.top;
            SIZE imgSize_;
            bitmap.GetSize(imgSize_);

            CDC dcMem;
            dcMem.CreateCompatibleDC ( pDC );
            dcMem.SelectBitmap(bitmap);

            int height_;
            while(top < rect.bottom) {
                height_ = imgSize_.cy;
                if( top + height_ > rect.bottom ) {
                    height_ = rect.bottom - top;
                }

                int width_;
                int left_ = rect.left;
                int right_ = rect.right;
                while( left_ < right_) {
                    width_ = imgSize_.cx;
                    if( left_ + width_ > right_ ) {
                        width_ = right_ - left_;
                    }

                    pDC.BitBlt(left_,top ,width_ ,height_ , dcMem, 0, 0, SRCCOPY);
                    left_ += width_;
                }

                top += height_;
            }

            return true;
        }
