---
layout: post
title: 工作区大小变化的相关消息
category: windows
---

#分辨率改变[WM_DISPLAYCHANGE](http://msdn.microsoft.com/en-us/library/windows/desktop/dd145210\(v=vs.85\).aspx)
 

#session改变[WM_WTSSESSION_CHANGE](http://msdn.microsoft.com/en-us/library/windows/desktop/aa383828\(v=vs.85\).aspx)

这个消息默认不会出发，需要显式地注册，见[这里](http://msdn.microsoft.com/en-us/library/windows/desktop/aa383841\(v=vs.85\).aspx)

        WTSRegisterSessionNotification(m_hWnd,NOTIFY_FOR_ALL_SESSIONS)
        Wtsapi32.h
        Wtsapi32.lib

#工作区改变（任务栏/自动隐藏任务栏）
[WM_SETTINGCHANGE](http://msdn.microsoft.com/en-us/library/windows/desktop/ms725497\(v=vs.85\).aspx)

wParam == [SPI_SETWORKAREA](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724947\(v=vs.85\).aspx)

#获得工作区大小
        RECT rt;
        SystemParametersInfo(SPI_GETWORKAREA,   0,   &rt,   0) ;   // 获得工作区大小

#全屏显示
        int full_x = GetSystemMetrics(SM_CXSCREEN);
        int full_y = GetSystemMetrics(SM_CYSCREEN);
        ::SetWindowPos(hWnd,HWND_TOPMOST,0,0,full_x,full_y,0 );

#任务栏
        CRect   rect; 
        HWND hwnd=  ::FindWindow("Shell_TrayWnd", "");     // 调用Findwindow函数，返回窗口指针
        ::GetWindowRect(hwnd,&rect);

#任务栏编程[SHAppBarMessage](http://msdn.microsoft.com/en-us/library/windows/desktop/bb762108\(v=vs.85\).aspx)

        void AutoHideTaskBar(BOOL bHide)
        {
              //这三句视情况加于不加
              #ifndef   ABM_SETSTATE  
              #define   ABM_SETSTATE             0x0000000a  
              #endif
               LPARAM lParam;
               if(bHide == TRUE)
               {
                      lParam = ABS_AUTOHIDE;//自动隐藏
               }
               else
               {
                      lParam = ABS_ALWAYSONTOP;//取消自动隐藏
               }
               APPBARDATA  apBar;  
               memset(&apBar,0,sizeof(apBar));  
               apBar.cbSize  =  sizeof(apBar);  
               apBar.hWnd  =  FindWindow("Shell_TrayWnd", NULL);
               if(apBar.hWnd  !=  NULL)  
               {  
                      apBar.lParam   =   lParam;  
                      SHAppBarMessage(ABM_SETSTATE,&apBar);  //设置任务栏自动隐藏
               }
        }

#电源消息[WM_POWERBROADCAST](http://msdn.microsoft.com/en-us/library/windows/desktop/aa373247\(v=vs.85\).aspx)