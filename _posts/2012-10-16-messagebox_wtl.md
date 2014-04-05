---
layout: post
title: 用WTL实现自定义MessageBox
category: wtl
---

##参考[DlgMsg.cpp in MsgBoxCtrl.rar](http://www.codeforge.com/read/194327/DlgMsg.cpp__html)
        #include <atlmisc.h>

        const int BTNS_BAR_HEIGHT = 30;
        const int ICON_WIDTH = 50;
        const int ICON_HEIGHT = 34;
        const int MAX_WIDTH = 600;
        const int MIN_INFO_HEIGHT = 25;
        const int BORDER_SPACE = 10;
        const int CAPTION_HEIGHT = 25;

        class CDlgMsg : public CDialogImpl<CDlgMsg>
        {
        public:
            enum { IDD = IDD_MESSAGEBOX_DLG };

            BEGIN_MSG_MAP(CMainDlg)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_TIMER, OnTimer)
                COMMAND_CODE_HANDLER(BN_CLICKED,OnClick)
            END_MSG_MAP()

            CDlgMsg(LPCTSTR lpszText, LPCTSTR lpszCaption, UINT nType){
                m_szInfo		= lpszText;
                m_szCaption		= lpszCaption;
                m_nType			= nType;
                m_nMsg			= IDCANCEL;
            }

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                
                // 初始化控件
                this->InitWidget();

                // 设置按钮文字
                this->SetButtonsText();

                // 设置文字
                this->m_stcInfo.SetWindowText(this->m_szInfo);
                this->SetWindowText(this->m_szCaption);

                // 响一声
                this->Beep(m_nType);

                // 设置图片和layout
                this->ResizeDialog();

                // 设置默认按钮
                this->PostMessage(WM_TIMER);
                return TRUE;
            }

            LRESULT OnTimer(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                this->SetDefaultButton();
                return TRUE;
            }

            LRESULT OnClick(WORD, WORD wID, HWND hWndCtl, BOOL& bHandled){
                if(!hWndCtl){
                    bHandled = FALSE;
                    return TRUE;
                }
                ::EndDialog(*this,wID);
                return TRUE;
            }
            

            void		InitWidget(){
                m_cmdIgnore = GetDlgItem(IDINGORE);
                m_cmdYes = GetDlgItem(IDYES);
                m_cmdRetry = GetDlgItem(IDRETRY);
                m_cmdOK = GetDlgItem(IDOK);
                m_cmdNo = GetDlgItem(IDNO);
                m_cmdHelp = GetDlgItem(IDHELP);
                m_cmdCancel = GetDlgItem(IDCANCEL);
                m_cmdAbort = GetDlgItem(IDABORT);

                m_cmdIgnore.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdYes.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdRetry.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdOK.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdNo.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdOK.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdHelp.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdCancel.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_cmdAbort.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));

                m_stcInfo = GetDlgItem(IDC_STATIC_INFO);
                m_stcInfo.SetFont(AtlGetStockFont(DEFAULT_GUI_FONT));
                m_stcIcon = GetDlgItem(IDC_STATIC_ICON);

                return;
            }

            void Beep(UINT nType)
            {
                UINT nIconID = _GetIconID(nType);
                UINT uType = -1;

                if (nIconID == MB_ICONINFORMATION || nIconID == MB_ICONASTERISK)
                    uType = MB_ICONASTERISK;
                else if (nIconID == MB_ICONEXCLAMATION || nIconID == MB_ICONWARNING)
                    uType = MB_ICONEXCLAMATION;
                else if (nIconID == MB_ICONQUESTION)
                    uType = MB_ICONQUESTION;
                else if (nIconID == MB_ICONSTOP || nIconID == MB_ICONERROR || nIconID == MB_ICONERROR)
                    uType = MB_ICONHAND;

                MessageBeep(uType);
            }

            HICON GetIcon(UINT nType)
            {
                UINT nIconID = _GetIconID(nType);
                HICON hIcon = NULL;
                if (nIconID == MB_ICONINFORMATION || nIconID == MB_ICONASTERISK)
                    hIcon = LoadIcon(NULL, IDI_ASTERISK);
                else if (nIconID == MB_ICONEXCLAMATION || nIconID == MB_ICONWARNING)
                    hIcon = LoadIcon(NULL, IDI_EXCLAMATION);
                else if (nIconID == MB_ICONQUESTION)
                    hIcon = LoadIcon(NULL, IDI_QUESTION);
                else if (nIconID == MB_ICONSTOP || nIconID == MB_ICONERROR || nIconID == MB_ICONERROR)
                    hIcon = LoadIcon(NULL, IDI_ERROR);
                
                return hIcon;
            }

            void SetButtonsText()
            {
                m_cmdOK.SetWindowText(_T("确定"));
                m_cmdCancel.SetWindowText(_T("取消"));
                m_cmdYes.SetWindowText(_T("是(&Y)"));
                m_cmdNo.SetWindowText(_T("否(&N)"));
                m_cmdAbort.SetWindowText(_T("终止(&A)"));
                m_cmdRetry.SetWindowText(_T("重试(&R)"));
                m_cmdIgnore.SetWindowText(_T("忽略(&I)"));
                m_cmdHelp.SetWindowText(_T("帮助(&H)"));
            }

            void SetDefaultButton()
            {
                UINT nBtnsStyle = _GetButtonsStyle(m_nType);
                UINT nDefBtn = _GetDefButton(m_nType);

                switch (nBtnsStyle)
                {
                case MB_OK:
                default:
                    m_cmdOK.SetFocus();
                    break;
                case MB_OKCANCEL:
                    if (nDefBtn == MB_DEFBUTTON2)
                    {
                        m_cmdCancel.SetFocus();
                    }
                    else
                    {
                        m_cmdOK.SetFocus();
                    }
                    break;
                case MB_YESNO:
                    if (nDefBtn == MB_DEFBUTTON2)
                    {
                        m_cmdNo.SetFocus();
                    }
                    else
                    {
                        m_cmdYes.SetFocus();
                    }
                    break;
                case MB_YESNOCANCEL:
                    if (nDefBtn == MB_DEFBUTTON3)
                    {
                        m_cmdCancel.SetFocus();
                    }
                    else if (nDefBtn == MB_DEFBUTTON2)
                    {
                        m_cmdNo.SetFocus();
                    }
                    else
                    {
                        m_cmdYes.SetFocus();
                    }
                    break;
                case MB_ABORTRETRYIGNORE:
                    if (nDefBtn == MB_DEFBUTTON3)
                    {
                        m_cmdIgnore.SetFocus();
                    }
                    else if (nDefBtn == MB_DEFBUTTON2)
                    {
                        m_cmdRetry.SetFocus();
                    }
                    else
                    {
                        m_cmdAbort.SetFocus();
                    }
                    break;
                }
            }

            void ResizeDialog()
            {
                UINT nButtonsStyle = _GetButtonsStyle(m_nType);
                HICON hIcon = GetIcon(m_nType);

                CRect rc, rcCtl;
                int nIconWidth = 0;
                int nIconHeight = 0;
                int nInfoWidth = 0;
                int nInfoHeight = 0;
                int nDialogWidth = 0;
                int nDialogHeight = 0;

                if (!hIcon)
                {
                    nIconWidth = 0;
                    nIconHeight = 0;
                }
                else
                {
                    nIconWidth = ICON_WIDTH;
                    nIconHeight = ICON_HEIGHT;
                }

                if (m_szInfo.IsEmpty())
                {
                    nInfoWidth = 0;
                    nInfoHeight = MIN_INFO_HEIGHT;
                }
                else
                {
                    CPaintDC dc(*this);
                    CFont font = GetFont();
                    dc.SelectFont(font);

                    m_szInfo.Replace(_T("\r\n"), _T("\r"));
                    
                    int nPrevPos = 0;
                    CSize size;
                    dc.GetTextExtent(m_szInfo,-1,&size);
                    int nCharHeight = size.cy;
                    size.cx = 0;
                    size.cy = BORDER_SPACE;

                    for (int iPos = 0; iPos < m_szInfo.GetLength(); iPos++)
                    {
                        TCHAR c = _T('\0');
                        c = m_szInfo.GetAt(iPos);
                        if ((c == _T('\r') || c == _T('\n')) || iPos == m_szInfo.GetLength() - 1)
                        {
                            CString szTemp = m_szInfo.Mid(nPrevPos, iPos - nPrevPos + 1);
                            CSize sizeTemp;
                            dc.GetTextExtent(szTemp,-1,&sizeTemp);
                            sizeTemp.cx = sizeTemp.cx;
                            if (size.cx < sizeTemp.cx)
                                size.cx = sizeTemp.cx + 12;
                            nPrevPos = iPos;
                            size.cy += sizeTemp.cy;
                        }
                    }

                    nInfoHeight = size.cy;
                    if (size.cx > MAX_WIDTH)
                    {
                        nInfoWidth = MAX_WIDTH;
                        nInfoHeight += (size.cx / MAX_WIDTH) * nCharHeight; 
                    }
                    else
                        nInfoWidth = size.cx;
                }

                int nMinWidth = 0;
                int nMinHeight = 0;
                int nBtnWidth = 0;
                int nBtnHeight = 0;
                
                switch (nButtonsStyle)
                {
                case MB_OK:
                default:
                    m_nMsg = IDCANCEL;
                    m_cmdOK.ShowWindow(SW_SHOW);
                    m_cmdOK.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = rc.Width() + 2 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;
                    rcCtl.left = rc.left + (rc.Width() - nBtnWidth) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - (rc.Width() - nBtnWidth) / 2;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdOK.MoveWindow(rcCtl);

                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                case MB_OKCANCEL:
                    m_nMsg = IDCANCEL;
                    m_cmdOK.ShowWindow(SW_SHOW);
                    m_cmdCancel.ShowWindow(SW_SHOW);

                    m_cmdOK.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = 2 * rc.Width() + 3 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;

                    rcCtl.left = rc.left + (rc.Width() - 2 * nBtnWidth - BORDER_SPACE) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rcCtl.left + nBtnWidth;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdOK.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdCancel.MoveWindow(rcCtl);
                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                case MB_YESNO:
                    m_nMsg = IDNO;
                    m_cmdYes.ShowWindow(SW_SHOW);
                    m_cmdNo.ShowWindow(SW_SHOW);

                    m_cmdYes.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = 2 * rc.Width() + 3 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;

                    rcCtl.left = rc.left + (rc.Width() - 2 * nBtnWidth - BORDER_SPACE) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rcCtl.left + nBtnWidth;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdYes.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdNo.MoveWindow(rcCtl);
                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                case MB_YESNOCANCEL:
                    m_nMsg = IDCANCEL;
                    m_cmdYes.ShowWindow(SW_SHOW);
                    m_cmdNo.ShowWindow(SW_SHOW);
                    m_cmdCancel.ShowWindow(SW_SHOW);

                    m_cmdYes.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = 3 * rc.Width() + 4 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;

                    rcCtl.left = rc.left + (rc.Width() - 3 * nBtnWidth - 2 * BORDER_SPACE) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rcCtl.left + nBtnWidth;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdYes.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdNo.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdCancel.MoveWindow(rcCtl);
                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                case MB_ABORTRETRYIGNORE:
                    m_cmdAbort.ShowWindow(SW_SHOW);
                    m_cmdRetry.ShowWindow(SW_SHOW);
                    m_cmdIgnore.ShowWindow(SW_SHOW);

                    m_cmdAbort.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = 3 * rc.Width() + 4 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;

                    rcCtl.left = rc.left + (rc.Width() - 3 * nBtnWidth - 2 * BORDER_SPACE) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rcCtl.left + nBtnWidth;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdAbort.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdRetry.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdIgnore.MoveWindow(rcCtl);
                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                case MB_HELP:
                    m_cmdOK.ShowWindow(SW_SHOW);
                    m_cmdHelp.ShowWindow(SW_SHOW);

                    m_cmdOK.GetWindowRect(rc);
                    nBtnWidth = rc.Width();
                    nBtnHeight = rc.Height();

                    nMinWidth = 2 * rc.Width() + 3 * BORDER_SPACE;
                    nMinHeight = rc.Height() + 3 * BORDER_SPACE + CAPTION_HEIGHT + (nIconHeight > nInfoHeight ? nIconHeight : nInfoHeight);
                    nDialogWidth = nMinWidth > nIconWidth + nInfoWidth + 2 * BORDER_SPACE ? nMinWidth : nIconWidth + nInfoWidth + 2 * BORDER_SPACE ;
                    nDialogHeight = nMinHeight;

                    rc.left = 0;
                    rc.top = 0;
                    rc.right = nDialogWidth;
                    rc.bottom = nDialogHeight;
                    MoveWindow(rc);
                    GetClientRect(rc);

                    rcCtl.left = rc.left + BORDER_SPACE;
                    rcCtl.top = rc.bottom - 2 * BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rc.right - BORDER_SPACE;
                    rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight + 2;

                    rcCtl.left = rc.left + (rc.Width() - 2 * nBtnWidth - 1 * BORDER_SPACE) / 2;
                    rcCtl.top = rc.bottom - BORDER_SPACE - nBtnHeight;
                    rcCtl.right = rcCtl.left + nBtnWidth;
                    rcCtl.bottom = rc.bottom - BORDER_SPACE;
                    m_cmdOK.MoveWindow(rcCtl);
                    rcCtl.OffsetRect(rcCtl.Width() + BORDER_SPACE, 0);
                    m_cmdHelp.MoveWindow(rcCtl);
                    if (hIcon)
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.left + BORDER_SPACE + nIconWidth;
                        rcCtl.bottom = rc.top + BORDER_SPACE + nIconHeight;
                        m_stcIcon.MoveWindow(rcCtl);
                        m_stcIcon.SetIcon(hIcon);

                        if (nInfoHeight < nIconHeight)
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top + (nIconHeight - nInfoHeight) / 2;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rcCtl.top + nInfoHeight;
                        }
                        else
                        {
                            rcCtl.left = rcCtl.right;
                            rcCtl.top = rcCtl.top;
                            rcCtl.right = rc.right - BORDER_SPACE;
                            rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        }
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    else
                    {
                        rcCtl.left = rc.left + BORDER_SPACE;
                        rcCtl.top = rc.top + BORDER_SPACE;
                        rcCtl.right = rc.right - BORDER_SPACE;
                        rcCtl.bottom = rc.bottom - 2 * BORDER_SPACE - nBtnHeight - 2;
                        m_stcInfo.MoveWindow(rcCtl);
                    }
                    break;
                }
                CenterWindow();
            }

            // util
            UINT _GetButtonsStyle(UINT nType)
            {
                if (nType & MB_HELP)
                    return MB_HELP;

                return nType & 0xF;
            }

            UINT _GetDefButton(UINT nType)
            {
                return nType & 0xF00;
            }

            UINT _GetIconID(UINT nType)
            {
                return nType &= 0xF0;
            }
        private:
            CButton	m_cmdIgnore;
            CButton	m_cmdYes;
            CButton	m_cmdRetry;
            CButton	m_cmdOK;
            CButton	m_cmdNo;
            CButton	m_cmdHelp;
            CButton	m_cmdCancel;
            CButton	m_cmdAbort;
            
            CStatic	m_stcInfo;
            CStatic	m_stcIcon;

            UINT		m_nMsg;
            CString		m_szInfo;
            CString		m_szCaption;
            UINT		m_nType;
        };
