---
layout: post
title: 使用SetWindowRgn实现异型窗口
category: wtl
---

        /*
            使用SetWindowRgn实现异型窗口
        */

        template<typename T,bool DLG>
        class ShapedWindow
        {
        public:
            BEGIN_MSG_MAP(ShapedWindow)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_PAINT, OnPaint)
                MESSAGE_HANDLER(WM_ERASEBKGND, OnEraseBkgnd)
            END_MSG_MAP()

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled)
            {
                if(!DLG)
                    this->InitSelf();
                bHandled = FALSE;
                return 0;
            }

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled)
            {
                if(DLG)
                    this->InitSelf();
                bHandled = FALSE;
                return 0;
            }

            LRESULT OnPaint(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                T* pT = static_cast<T*>(this);
                CPaintDC dc_(pT->m_hWnd);
                return 0;
            }

            LRESULT OnEraseBkgnd(UINT /*uMsg*/, WPARAM wParam, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                T* pT = static_cast<T*>(this);
                CDCHandle dc_((HDC)wParam);
                CRect clt_rect_;
                pT->GetClientRect(clt_rect_);
                TileBitBlt(dc_,*m_background_,clt_rect_);
                return 0;
            }

            void		SetBgBitmapInfo(CBitmap* bitmap,COLORREF mask){
                ATLASSERT(bitmap);
                m_background_ = bitmap;
                m_mask_ = mask;
            }

        private:
            void		InitSelf(){
                ATLASSERT(m_background_);
                T* pT = static_cast<T*>(this);

                HRGN rgn_ = BitmapToRegion(*m_background_,m_mask_);
                ATLASSERT(rgn_ != NULL);
                pT->SetWindowRgn(rgn_);

                CSize size_;
                m_background_->GetSize(size_);
                int nWidth=GetSystemMetrics(SM_CXSCREEN); //屏幕宽度
                int nHeight=GetSystemMetrics(SM_CYSCREEN); //屏幕高度

                pT->MoveWindow(
                        (nWidth - size_.cx)/2,
                        (nHeight - size_.cy)/2,
                        size_.cx,
                        size_.cy);
            }
        protected:
            CBitmap*	m_background_;
            COLORREF	m_mask_;
        };
        
###Sample
---

        CMainFrame wndMain;

        CBitmap		bitmap_bg_;
        BOOL ret_ = File2CBitmap(&bitmap_bg_,_T("res/bg.bmp"));
        ATLASSERT(ret_);
        wndMain.SetBgBitmapInfo(&bitmap_bg_,RGB(255,0,255));

        if(wndMain.CreateEx() == NULL)
        {
            ATLTRACE(_T("Main window creation failed!\n"));
            return 0;
        }

        wndMain.ShowWindow(nCmdShow);
        
---


        class CMainFrame :
            public CFrameWindowImpl<CMainFrame>,
            public ShapedWindow<CMainFrame,false>,
            public CMessageFilter
        {
            typedef ShapedWindow<CMainFrame,false> BaseClass;
        public:
            static DWORD GetWndStyle(DWORD dwStyle)
            {
                return WS_POPUP;
            }
            static DWORD GetWndExStyle(DWORD dwExStyle)
            {
                return WS_EX_APPWINDOW;
            }

            virtual BOOL PreTranslateMessage(MSG* pMsg)
            {
                return CFrameWindowImpl<CMainFrame>::PreTranslateMessage(pMsg);
            }

            DECLARE_FRAME_WND_CLASS(NULL, IDR_MAINFRAME)

            BEGIN_MSG_MAP(CMainFrame)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_DESTROY, OnDestroy)
                MESSAGE_HANDLER(WM_LBUTTONDOWN, OnDlg)
                CHAIN_MSG_MAP(CFrameWindowImpl<CMainFrame>)
            END_MSG_MAP()

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                // register object for message filtering and idle updates
                CMessageLoop* pLoop = _Module.GetMessageLoop();
                ATLASSERT(pLoop != NULL);
                pLoop->AddMessageFilter(this);
                return 0;
            }

            LRESULT OnDestroy(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled)
            {
                // unregister message filtering and idle updates
                CMessageLoop* pLoop = _Module.GetMessageLoop();
                ATLASSERT(pLoop != NULL);
                pLoop->RemoveMessageFilter(this);

                // 解决wtl不能关闭ws_popup的bug
                PostQuitMessage(0);
                bHandled = FALSE;
                return 1;
            }
            LRESULT OnDlg(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled)
            {
                CAboutDlg dlg;
                dlg.SetBgBitmapInfo(this->m_background_,this->m_mask_);
                dlg.DoModal();
                return 1;
            }
        };
        
---


        class CAboutDlg :
            public CDialogImpl<CAboutDlg>,
            public ShapedWindow<CAboutDlg,true>
        {
            typedef ShapedWindow<CAboutDlg,true> BaseClass;
        public:
            enum { IDD = IDD_DIALOG1 };

            BEGIN_MSG_MAP(CAboutDlg)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_LBUTTONDOWN, OnLButtonDown)
                COMMAND_ID_HANDLER(IDOK, OnCloseCmd)
                COMMAND_ID_HANDLER(IDCANCEL, OnCloseCmd)
            END_MSG_MAP()

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                CenterWindow(GetParent());
                return TRUE;
            }

            LRESULT OnCloseCmd(WORD /*wNotifyCode*/, WORD wID, HWND /*hWndCtl*/, BOOL& /*bHandled*/)
            {
                EndDialog(wID);
                return 0;
            }

            LRESULT OnLButtonDown(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& /*bHandled*/)
            {
                this->PostMessage(WM_NCLBUTTONDOWN,HTCAPTION,lParam);
                return 0;
            }
        };
