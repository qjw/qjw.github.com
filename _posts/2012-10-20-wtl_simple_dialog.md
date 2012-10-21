---
layout: post
title: 用WTL实现一个简单的Dialog模板
category: wtl
---



        #include "stdafx.h"
        #include <atlcrack.h>
        #include <atlmisc.h>

        template<int RES_ID,typename BASE_VIEW>
        class BaseDlg : public CDialogImpl<BaseDlg<RES_ID,BASE_VIEW> >{
        public:
            enum { IDD = RES_ID };
            typedef CDialogImpl<BaseDlg> BaseClass;

            BEGIN_MSG_MAP(BaseDlg)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_SIZE, OnSize)
                MESSAGE_HANDLER(WM_ERASEBKGND, OnEraseBkgnd)
                MESSAGE_HANDLER(WM_CLOSE, OnClose)
                REFLECT_NOTIFICATIONS()
            END_MSG_MAP()

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                // center the dialog on the screen
                CenterWindow();

                // set icons
                HICON hIcon = (HICON)::LoadImage(_Module.GetResourceInstance(), MAKEINTRESOURCE(IDR_MAINFRAME),
                    IMAGE_ICON, ::GetSystemMetrics(SM_CXICON), ::GetSystemMetrics(SM_CYICON), LR_DEFAULTCOLOR);
                SetIcon(hIcon, TRUE);
                HICON hIconSmall = (HICON)::LoadImage(_Module.GetResourceInstance(), MAKEINTRESOURCE(IDR_MAINFRAME),
                    IMAGE_ICON, ::GetSystemMetrics(SM_CXSMICON), ::GetSystemMetrics(SM_CYSMICON), LR_DEFAULTCOLOR);
                SetIcon(hIconSmall, FALSE);

                // 创建子窗口
                //CRect rect_;
                //this->GetClientRect(&rect_);
                //m_view_.Create(m_hWnd, &rect_,"",WS_CHILD|WS_VISIBLE);
                m_view_.Create(m_hWnd, NULL,_T(""),WS_CHILD|WS_VISIBLE);
                
                m_view_.InitDlg(this);
                return TRUE;
            }
            LRESULT OnSize(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                // 子窗口直接填满整个对话框
        #if 1
                CRect rect_(
                        0,
                        0,
                        GET_X_LPARAM(lParam),
                        GET_Y_LPARAM(lParam));
        #else
                CRect rect_(
                        m_client_rect_.left,
                        m_client_rect_.top,
                        GET_X_LPARAM(lParam) - m_client_rect_.right,
                        GET_Y_LPARAM(lParam) - m_client_rect_.bottom);
        #endif
                m_view_.MoveWindow(&rect_);
                return TRUE;
            }
            LRESULT OnClose(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled){
                int ret_ = m_view_.IsClose();
                if(0xFFFFFFFF != ret_)
                    EndDialog(ret_);
                return TRUE;
            }
            LRESULT OnEraseBkgnd(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled){
                return TRUE;
            }

        protected:
            BASE_VIEW	m_view_;
            //CRect		m_client_rect_;
        };

        class DlgViewBase{
        public:
            // 不用虚函数，派生类可直接覆盖
            int		IsClose(){
                return 0;
            }
            void	InitDlg(CWindow* win){

            }

        };

        class DlgViewTest :
            public CWindowImpl<DlgViewTest, CWindow , CControlWinTraits>,
            public DlgViewBase{
        public:
            BEGIN_MSG_MAP_EX(TestView)
            END_MSG_MAP()
        };
        /*
         * typedef BaseDlg<IDD_MAINDLG,DlgViewTest> Mydlg;
         *	Mydlg dlg_;
         *	dlg_.DoModal();
         */
