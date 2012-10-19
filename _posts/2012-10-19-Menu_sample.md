---
layout: post
title: 菜单基本用法
category: windows
---

#Sample
**注意移除（不要删除）系统菜单，否则下次会失败**

        class CMainDlg : public CDialogImpl<CMainDlg>
        {
        public:
            enum { IDD = IDD_MAINDLG };

            BEGIN_MSG_MAP(CMainDlg)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_LBUTTONDOWN, OnButton)
                COMMAND_CODE_HANDLER(0,OnMenu)
                COMMAND_ID_HANDLER(ID_APP_ABOUT, OnAppAbout)
                COMMAND_ID_HANDLER(IDOK, OnOK)
                COMMAND_ID_HANDLER(IDCANCEL, OnCancel)
            END_MSG_MAP()

            LRESULT OnMenu(WORD wNotifyCode, WORD wID, HWND hWndCtl, BOOL& bHandled)
            {
                // 如果是按钮消息
                if(!!hWndCtl){
                    bHandled = FALSE;
                    return TRUE;
                }
                switch(wID){
                case 1:
                    return 0;
                }
                return 0;
            }
            LRESULT OnButton(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& /*bHandled*/)
            {
                CMenu menu;
                menu.CreatePopupMenu();
                menu.AppendMenu(MF_STRING,1,_T("菜单项1"));
                menu.AppendMenu(MF_SEPARATOR);
                menu.AppendMenu(MF_STRING,2,_T("菜单项2"));
                // 特定位置插入
                menu.InsertMenu(1,MF_BYPOSITION|MF_STRING,3,_T("菜单项3"));
                // 删掉第一个菜单
                menu.DeleteMenu(0,MF_BYPOSITION);

                // 添加子菜单
                menu.AppendMenu(MF_BYPOSITION|MF_POPUP|MF_STRING,
                      m_sysmenu_,_T("子菜单"));

                CPoint pt_;
                ::GetCursorPos(&pt_);
                menu.TrackPopupMenu(TPM_LEFTALIGN, pt_.x,pt_.y,m_hWnd);

                // 注意移除（不要删除）系统菜单，否则下次会失败
                menu.RemoveMenu(3,MF_BYPOSITION);

                return 0;
            }
            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                m_sysmenu_.Attach(::GetSystemMenu(m_hWnd,FALSE));
                /*
                All predefined window menu items have identifier numbers 
                greater than 0xF000. If an application adds commands 
                to the window menu, it should use identifier numbers less than 0xF000.
                */
                m_sysmenu_.AppendMenu(MF_STRING,1233,_T("子菜单项"));
                return TRUE;
            }

            CMenuHandle m_sysmenu_;
        };

#关闭菜单

首先用[FindWindowEx](http://msdn.microsoft.com/en-us/library/windows/desktop/ms633500\(v=vs.85\).aspx) 找到句柄，

        An application can call this function in the following way.
        FindWindowEx( NULL, NULL, MAKEINTATOM(0x8000), NULL );
        Note that 0x8000 is the atom for a menu class. When an application calls this function, 
        the function checks whether a context menu is being displayed that the application created.


然后向其发送如下消息：SendMessage(WM_CANCELMODE);

#参考
1. <http://topic.csdn.net/t/20051102/21/4367661.html>




