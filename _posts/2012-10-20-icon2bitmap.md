---
layout: post
title: Icon转换成Bitmap
category: wingui
---


        HICON icon_ = (HICON)LoadImage(
            NULL, 
            L"c:/a.ico",
            IMAGE_ICON, 
            0,0,
            LR_LOADFROMFILE); 

        // 这个大小不一定对
        ICONINFO icon_info_;
        ::GetIconInfo(icon_,&icon_info_);
        // 获得实际的大小
        BITMAP bitmap_;
        GetObject(icon_info_.hbmColor, sizeof(BITMAP), &bitmap_);
        DeleteObject(icon_info_.hbmColor);
        DeleteObject(icon_info_.hbmMask);

        // 这里判断是否有alpha通道
        if(bitmap_.bmBitsPixel != 32)
            return TRUE;

        CDC dc_;
        dc_.CreateCompatibleDC(NULL);

        m_img_.Create(
            bitmap_.bmWidth,
            bitmap_.bmHeight,
            32,
            // 若ico不支持alpha通道，这里不要使用此标志位，否则会导致全透明
            CImage::createAlphaChannel);

        dc_.SelectBitmap(m_img_);
        dc_.DrawIconEx(0, 0, icon_,
            bitmap_.bmWidth,
            bitmap_.bmHeight,
            0, 
            NULL,
            DI_NORMAL);
        
        DestroyIcon(icon_);

        return TRUE;

这种办法对于某些icon似乎还是有些问题，可能是图片格式不太对，可以参考[icon_util.h](http://src.chromium.org/chrome/branches/355/src/gfx/icon_util.h)和[icon_util.cc](http://src.chromium.org/chrome/branches/355/src/gfx/icon_util.cc)


        SkBitmap* IconUtil::CreateSkBitmapFromHICON(HICON icon, const gfx::Size& s) {
          // We start with validating parameters.
          ICONINFO icon_info;
          if (!icon || !(::GetIconInfo(icon, &icon_info)) ||
              !icon_info.fIcon || s.IsEmpty()) {
            return NULL;
          }

          // Allocating memory for the SkBitmap object. We are going to create an ARGB
          // bitmap so we should set the configuration appropriately.
          SkBitmap* bitmap = new SkBitmap;
          DCHECK(bitmap);
          bitmap->setConfig(SkBitmap::kARGB_8888_Config, s.width(), s.height());
          bitmap->allocPixels();
          bitmap->eraseARGB(0, 0, 0, 0);
          SkAutoLockPixels bitmap_lock(*bitmap);

          // Now we should create a DIB so that we can use ::DrawIconEx in order to
          // obtain the icon's image.
          BITMAPV5HEADER h;
          InitializeBitmapHeader(&h, s.width(), s.height());
          HDC dc = ::GetDC(NULL);
          unsigned int* bits;
          HBITMAP dib = ::CreateDIBSection(dc,
                                           reinterpret_cast<BITMAPINFO*>(&h),
                                           DIB_RGB_COLORS,
                                           reinterpret_cast<void**>(&bits),
                                           NULL,
                                           0);
          DCHECK(dib);
          HDC dib_dc = CreateCompatibleDC(dc);
          DCHECK(dib_dc);
          ::SelectObject(dib_dc, dib);

          // Windows icons are defined using two different masks. The XOR mask, which
          // represents the icon image and an AND mask which is a monochrome bitmap
          // which indicates the transparency of each pixel.
          //
          // To make things more complex, the icon image itself can be an ARGB bitmap
          // and therefore contain an alpha channel which specifies the transparency
          // for each pixel. Unfortunately, there is no easy way to determine whether
          // or not a bitmap has an alpha channel and therefore constructing the bitmap
          // for the icon is nothing but straightforward.
          //
          // The idea is to read the AND mask but use it only if we know for sure that
          // the icon image does not have an alpha channel. The only way to tell if the
          // bitmap has an alpha channel is by looking through the pixels and checking
          // whether there are non-zero alpha bytes.
          //
          // We start by drawing the AND mask into our DIB.
          size_t num_pixels = s.GetArea();
          memset(bits, 0, num_pixels * 4);
          ::DrawIconEx(dib_dc, 0, 0, icon, s.width(), s.height(), 0, NULL, DI_MASK);

          // Capture boolean opacity. We may not use it if we find out the bitmap has
          // an alpha channel.
          bool* opaque = new bool[num_pixels];
          DCHECK(opaque);
          for (size_t i = 0; i < num_pixels; ++i)
            opaque[i] = !bits[i];

          // Then draw the image itself which is really the XOR mask.
          memset(bits, 0, num_pixels * 4);
          ::DrawIconEx(dib_dc, 0, 0, icon, s.width(), s.height(), 0, NULL, DI_NORMAL);
          memcpy(bitmap->getPixels(), static_cast<void*>(bits), num_pixels * 4);

          // Finding out whether the bitmap has an alpha channel.
          bool bitmap_has_alpha_channel = PixelsHaveAlpha(
              static_cast<const uint32*>(bitmap->getPixels()), num_pixels);

          // If the bitmap does not have an alpha channel, we need to build it using
          // the previously captured AND mask. Otherwise, we are done.
          if (!bitmap_has_alpha_channel) {
            unsigned int* p = static_cast<unsigned int*>(bitmap->getPixels());
            for (size_t i = 0; i < num_pixels; ++p, ++i) {
              DCHECK_EQ((*p & 0xff000000), 0);
              if (opaque[i])
                *p |= 0xff000000;
              else
                *p &= 0x00ffffff;
            }
          }

          delete [] opaque;
          ::DeleteDC(dib_dc);
          ::DeleteObject(dib);
          ::ReleaseDC(NULL, dc);

          return bitmap;
        }
        
        
        void IconUtil::InitializeBitmapHeader(BITMAPV5HEADER* header, int width,
                                              int height) {
          DCHECK(header);
          memset(header, 0, sizeof(BITMAPV5HEADER));
          header->bV5Size = sizeof(BITMAPV5HEADER);

          // Note that icons are created using top-down DIBs so we must negate the
          // value used for the icon's height.
          header->bV5Width = width;
          header->bV5Height = -height;
          header->bV5Planes = 1;
          header->bV5Compression = BI_RGB;

          // Initializing the bitmap format to 32 bit ARGB.
          header->bV5BitCount = 32;
          header->bV5RedMask = 0x00FF0000;
          header->bV5GreenMask = 0x0000FF00;
          header->bV5BlueMask = 0x000000FF;
          header->bV5AlphaMask = 0xFF000000;

          // Use the system color space.  The default value is LCS_CALIBRATED_RGB, which
          // causes us to crash if we don't specify the approprite gammas, etc.  See
          // <http://msdn.microsoft.com/en-us/library/ms536531(VS.85).aspx> and
          // <http://b/1283121>.
          header->bV5CSType = LCS_WINDOWS_COLOR_SPACE;

          // Use a valid value for bV5Intent as 0 is not a valid one.
          // <http://msdn.microsoft.com/en-us/library/dd183381(VS.85).aspx>
          header->bV5Intent = LCS_GM_IMAGES;
        }