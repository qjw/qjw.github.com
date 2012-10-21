---
layout: post
title: ToolTip基本用法
category: windows
---

##Sample

        class CMainDlg : public CDialogImpl<CMainDlg>
        {
        public:
            enum { IDD = IDD_MAINDLG };

            BEGIN_MSG_MAP(CMainDlg)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_RANGE_HANDLER(WM_MOUSEFIRST,WM_MOUSELAST,OnMouseMessage)
            END_MSG_MAP()

            LRESULT   OnMouseMessage(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled) 
            { 
                if(m_tooltip_.IsWindow()) 
                { 
                    MSG msg = {
                        m_hWnd,
                        uMsg,
                        wParam,
                        lParam
                    }; 
                    m_tooltip_.RelayEvent(&msg); 
                } 
                bHandled   =   FALSE; 
                return   1; 
            } 

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {

                // http://msdn.microsoft.com/en-us/library/windows/desktop/bb760246(v=vs.85).aspx
                m_tooltip_.Create(m_hWnd,NULL,NULL,
                    WS_CHILD | WS_VISIBLE | TTS_ALWAYSTIP | WS_POPUP | TTS_BALLOON);
                m_tooltip_.Activate(FALSE);	
                m_tooltip_.AddTool(m_hWnd);
                m_tooltip_.UpdateTipText(_T("tipText"), m_hWnd);
                m_tooltip_.Activate(TRUE);	
                return TRUE;
            }

        private:

            CToolTipCtrl   m_tooltip_;
        };

###小Tips
1. 若没有ws_popup ,tooltip将无法显示
1. 使用tts_alwaystip 将在窗口不在最前也能够显示
1. 使用tts_balloon ，tooltip将显示气球形状

为了更加灵活，可以使用如下工具类

        class CMyTip : public CWindowImpl <CMyTip> 
        {
        public:
            void   Init(HWND hWnd, LPCTSTR lpstrName) 
            { 
                ATLASSERT(::IsWindow(hWnd)); 
                SubclassWindow(hWnd); 

                m_tooltip_.Create(hWnd,NULL,NULL,
                    WS_CHILD | WS_VISIBLE | TTS_ALWAYSTIP | WS_POPUP | TTS_BALLOON);
                m_tooltip_.Activate(FALSE);	
                m_tooltip_.AddTool(hWnd);
                m_tooltip_.UpdateTipText(lpstrName, hWnd);
                m_tooltip_.Activate(TRUE);	
            } 

            BEGIN_MSG_MAP(CMainDlg)
                MESSAGE_RANGE_HANDLER(WM_MOUSEFIRST,WM_MOUSELAST,OnMouseMessage)
            END_MSG_MAP()

            LRESULT   OnMouseMessage(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled) 
            { 
                if(m_tooltip_.IsWindow()) 
                { 
                    MSG msg = {
                        m_hWnd,
                        uMsg,
                        wParam,
                        lParam
                    }; 
                    m_tooltip_.RelayEvent(&msg); 
                } 
                bHandled   =   FALSE; 
                return   1; 
            } 

        private:
            CToolTipCtrl   m_tooltip_; 
        }; 


        class CMainDlg : public CDialogImpl<CMainDlg>
        {
            BEGIN_MSG_MAP(CMainDlg)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
            END_MSG_MAP()

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {	
                m_tp1_.Init(GetDlgItem(IDOK),_T("fuck")); 
                m_tp2_.Init(GetDlgItem(ID_APP_ABOUT),_T("fuck2")); 
                return TRUE;
            }

            CMyTip   m_tp1_; 
            CMyTip   m_tp2_; 
        };
        
tooltip有一个结构[TOOLINFO](http://msdn.microsoft.com/en-us/library/windows/desktop/bb760256\(v=vs.85\).aspx)

        typedef struct {
          UINT      cbSize;
          UINT      uFlags;
          HWND      hwnd;
          UINT_PTR  uId;
          RECT      rect;
          HINSTANCE hinst;
          LPTSTR    lpszText;
        #if (_WIN32_IE >= 0x0300)
          LPARAM    lParam;
        #endif 
        #if (_WIN32_WINNT >= Ox0501)
          void      *lpReserved;
        #endif 
        } TOOLINFO, *PTOOLINFO, *LPTOOLINFO;

它根据不同的平台而又不同的大小，为了不至于出错，可以强制使用最新的版本

        #if defined _M_IX86
          #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' \
            version='6.0.0.0' processorArchitecture='x86' publicKeyToken='6595b64144ccf1df' language='*'\"")
        #elif defined _M_IA64
          #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' \
            version='6.0.0.0' processorArchitecture='ia64' publicKeyToken='6595b64144ccf1df' language='*'\"")
        #elif defined _M_X64
          #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' \
            version='6.0.0.0' processorArchitecture='amd64' publicKeyToken='6595b64144ccf1df' language='*'\"")
        #else
          #pragma comment(linker, "/manifestdependency:\"type='win32' name='Microsoft.Windows.Common-Controls' \
            version='6.0.0.0' processorArchitecture='*' publicKeyToken='6595b64144ccf1df' language='*'\"")
        #endif

##参考：
1. <http://www.cnblogs.com/terminator-studio/archive/2012/04/09/2438780.html>
2. <http://blog.csdn.net/flyflyking/article/details/6272361>
3. <http://topic.csdn.net/u/20090928/17/07ebf98f-a802-4359-a190-9fed8b3d0790.html>
4. <http://topic.csdn.net/t/20021020/13/1109608.html>
4. <http://blog.csdn.net/shihaojie1219/article/details/5788987>



