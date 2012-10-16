---
layout: post
title: 自定义对话框
category: wtl
---

##对话框资源（注意CLASS属性）
        IDD_MESSAGEBOX_DLG DIALOGEX 0, 0, 316, 185
        CLASS "MyDialog"
        STYLE DS_SETFONT | DS_MODALFRAME | DS_FIXEDSYS | WS_POPUP | WS_CAPTION | WS_SYSMENU
        CAPTION "Dialog"
        FONT 8, "MS Shell Dlg", 400, 0, 0x1
        BEGIN
            PUSHBUTTON      "IDCANCEL",IDCANCEL,22,20,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDHELP",IDHELP,38,113,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDNO",IDNO,109,65,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDOK",IDOK,171,102,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDRETRY",IDRETRY,182,53,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDYES",IDYES,153,17,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDINGORE",IDINGORE,221,18,50,14,NOT WS_VISIBLE
            PUSHBUTTON      "IDABORT",IDABORT,121,122,50,14,NOT WS_VISIBLE
            LTEXT           "IDC_STATIC_INFO",IDC_STATIC_INFO,216,73,62,8
            ICON            "",IDC_STATIC_ICON,146,146,20,20
        END
        
##使用
        // 注册类型，否则特殊的对话框会打印不出来
        WNDCLASS wcx;
        memset(&wcx, 0, sizeof(wcx));
        GetClassInfo(NULL, WC_DIALOG, &wcx);
        wcx.lpszClassName = _T("MyDialog");//修改为自己定义的ClassName
        RegisterClass(&wcx);	// 注册类型，否则特殊的对话框会打印不出来
        WNDCLASS wcx;
        memset(&wcx, 0, sizeof(wcx));
        GetClassInfo(NULL, WC_DIALOG, &wcx);
        wcx.lpszClassName = _T("MyDialog");//修改为自己定义的ClassName
        RegisterClass(&wcx);

    

