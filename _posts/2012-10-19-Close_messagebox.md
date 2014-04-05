---
layout: post
title: 后台关闭MessageBox
category: wingui
---


具体方案就是枚举所有MessageBox,然后判断自己的子进程

        BOOL myEnumWindow(HWND inHwnd)
        {
            //TCHAR szText[256];

            HWND hwndAfter = NULL;
            while(hwndAfter = ::FindWindowEx(inHwnd,hwndAfter,_T("MyDialog"),NULL))
            {
                //memset(szText,0,256);
                //::SendMessage(hwndAfter,WM_GETTEXT,(WPARAM)256,(LPARAM)szText);
                //printf("%s/t",szText);
                myEnumWindow(hwndAfter);
                if(::GetParent(hwndAfter) == *this)
                    ::PostMessage(hwndAfter,WM_CLOSE,0,0);
            }
            return 1;
        }

        LRESULT OnTimer(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
            myEnumWindow(NULL);
            return TRUE;
        }

为了缩小范围,可以使用自定义MessageBox




