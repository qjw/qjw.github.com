---
layout: post
title: 用UpdateLayeredWindow实现任意异形窗口
category: wtl
---

**前面提到，我们可以用SetWindowRgn或SetLayeredWindowAttributes实现不规则以及半透明的效果**

对于SetWindowRgn，它通过一个Rgn来设置区域，这个Rgn一般可以从图片中读取，在这张图片中，将不需要显示的区域标记为一种特殊的颜色，这里有个问题，必须保证这种颜色没有被正常的区域使用，否则会被误伤。为了解决这个问题，可以考虑用两张图片，增加一张单色的掩码图，这种方案带来了额外的管理开销。SetWindowRgn的好处是效率较高，对于大部分自绘的皮肤，一般只有四个角落有一些不规则，所以用SetWindowRgn是最好的选择。

SetLayeredWindowAttributes可以将特定的窗口设置为某种透明度，也可以用它来过滤某种颜色，匹配的颜色会变成全透明。也就是类似于SetWindowRgn的效果。SetLayeredWindowAttributes直接从DC中获得颜色，所以你需要事先绘制DC。

SetLayeredWindowAttributes过滤颜色后，相关区域虽然不可见，但是不可见的区域可以放置子窗口，这点和SetWindowRgn有所区别。此外若子窗口刷新不及时或其他原因，那么父窗口因为SetLayeredWindowAttributes被隐藏的DC颜色将被浮出水面。

UpdateLayeredWindow直接根据DC中的Alpha通道来实现透明效果，它很好的处理了和背景的Alpha Blend的问题，所以完美的解决了SetWindowRgn的锯齿问题。

#Sample

        template<typename T,bool DLG>
        class ImageFrameT{
        public:
            BEGIN_MSG_MAP(ImageFrameT)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_PAINT, OnPaint)
                MESSAGE_HANDLER(WM_ERASEBKGND, OnEraseBkgnd)
                MESSAGE_HANDLER(WM_LBUTTONDOWN, OnLBttonDown)
            END_MSG_MAP()

            ImageFrameT():m_res_(NULL),m_move_flag_(false){}
            virtual ~ImageFrameT(){}

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                bHandled = FALSE;
                
                // 通过参数获得资源句柄
                LPCREATESTRUCT lpCreateStruct = (LPCREATESTRUCT)lParam;
                if(lpCreateStruct && lpCreateStruct->lpCreateParams)
                    m_res_ = (CImage*)(lpCreateStruct->lpCreateParams);
                ATLASSERT(m_res_);

                if(!DLG)
                    this->InitSelf();

                return 0;
            }

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                bHandled = FALSE;
                if(lParam){
                    // 通过参数获得资源句柄
                    m_res_ = (CImage*)lParam;
                }
                if(DLG)
                    this->InitSelf();
                return 0;
            }

            LRESULT OnPaint(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                // OnPaint不作任何事，转到UpdateLayeredWindow处理
                T* pT = static_cast<T*>(this);
                CPaintDC dc_(pT->m_hWnd);
                return TRUE;
            }
            LRESULT OnEraseBkgnd(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                // 屏蔽背景绘制
                return TRUE;
            }

            LRESULT OnLBttonDown(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled){
                // 是否支持整窗口拖动
                T* pT = static_cast<T*>(this);
                if(this->m_move_flag_)
                    pT->PostMessage(WM_SYSCOMMAND,0xF012,0);
                else
                    bHandled = FALSE;
                return 0;
            }

            void			SetRes(CImage* res){
                // 设置资源
                ATLASSERT(res);
                if(res)
                    this->m_res_ = res;
            }

            void			SetMoveFlag(bool flag){
                // 设置是否可拖动
                this->m_move_flag_ = flag;
            }
        private:
            void			InitSelf(){
                ATLASSERT(m_res_);
                if(m_res_){
                    T* pT = static_cast<T*>(this);

                    // 设置属性WS_EX_LAYERED
                    LONG lWindowLong = ::GetWindowLong(pT->m_hWnd, GWL_EXSTYLE) | WS_EX_LAYERED;
                    ::SetWindowLong(pT->m_hWnd, GWL_EXSTYLE, lWindowLong);
                    // 设置属性WS_POPUP
                    lWindowLong = ::GetWindowLong(pT->m_hWnd, GWL_STYLE) | WS_POPUP;
                    // 去掉一堆其他属性
                    lWindowLong &= ~WS_CHILD;
                    lWindowLong &= ~WS_BORDER;
                    lWindowLong &= ~WS_CAPTION;
                    lWindowLong &= ~WS_SYSMENU;
                    ::SetWindowLong(pT->m_hWnd, GWL_STYLE, lWindowLong);

                    pT->SetWindowPos(HWND_BOTTOM,0,0,
                            m_res_->GetWidth(),m_res_->GetHeight(),
                            SWP_NOMOVE | SWP_NOOWNERZORDER);

                    CClientDC dc_(pT->m_hWnd);
                    CDC mem_dc_;
                    mem_dc_.CreateCompatibleDC(dc_);
                    CBitmap mem_bitmap_;
                    mem_bitmap_.CreateCompatibleBitmap(dc_,
                            m_res_->GetWidth(),
                            m_res_->GetHeight());
                    mem_dc_.SelectBitmap(mem_bitmap_);

                    m_res_->Draw(mem_dc_,0,0);

                    BLENDFUNCTION pb_;
                    pb_.AlphaFormat = 1;
                    pb_.BlendOp = 0;
                    pb_.BlendFlags =0;
                    pb_.SourceConstantAlpha = 0xFF;

                    CPoint 	pt_(0,0);
                    CSize	size_(m_res_->GetWidth(),m_res_->GetHeight());
                    ::UpdateLayeredWindow(pT->m_hWnd,dc_,&pt_,&size_,mem_dc_,&pt_,0,&pb_,ULW_ALPHA );

                    pT->CenterWindow(NULL);
                }
            }
        protected:
            CImage*			m_res_;
            bool			m_move_flag_;
        };

---

        class CAboutDlg :
            public CDialogImpl<CAboutDlg>,
            public ImageFrameT<CAboutDlg,true>
        {
            typedef ImageFrameT<CAboutDlg,true> BaseClass;
        public:
            enum { IDD = IDD_DIALOG1 };

            BEGIN_MSG_MAP(CAboutDlg)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_RBUTTONDOWN, OnClose)
            END_MSG_MAP()

            LRESULT OnClose(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                EndDialog(0);
                return 0;
            }
        };
        
---

        CImage		bitmap_bg_;
        BOOL ret_ = File2CImageAndImplAlpha(&bitmap_bg_,_T("res/bk.png"));
        ATLASSERT(ret_);

        CAboutDlg dlg_;
        dlg_.SetMoveFlag(true);
        dlg_.DoModal(this->m_hSubWindow_,(LPARAM)(&bitmap_bg_));

#放置子控件

若只需要最一个简单的窗口，那么上面的代码可以完成要求。UpdateLayeredWindow有一个问题，那就是它上面不能放置任何子窗口，放置上去的任何窗口都不可见。为了解决这个问题，一种简单的办法是自绘，在单个窗口中模拟各种消息。

若不想搞复杂，有一种变通的办法，那就是在上面放置一个**非子窗口**，这种子窗口大小位置保持和它一致，同时这个子窗口用SetLayeredWindowAttributes搞成全透明，接下来我们将所有子控件放到这个全透明的子窗口即可。

##Sample

        #define CHAIN_MSG_MAP_ALT_MEMBER_EX(theChainMember, msgMapID, msg) \
            { \
                if(uMsg == msg && \
                    theChainMember &&\
                    theChainMember->ProcessWindowMessage(hWnd, uMsg, wParam, lParam, lResult, msgMapID)) \
                    return TRUE; \
            }

        // 放置在ImageFrameT上的子窗口
        // 这个类主要处理消息的转发，通过WM_CREATE获得最顶层窗口的指针
        // 并存储在m_message_变量中，然后使用CHAIN_MSG_MAP_ALT_MEMBER_EX
        // 将它子窗口发给它的WM_COMMAND，WM_NOTIFY转发过去
        template<typename T,typename Y>
        class SubWindowT{
        public:
            BEGIN_MSG_MAP(SubWindowT)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_ERASEBKGND, OnEraseBkgnd)
                // 将这个不可见的窗口的WM_COMMAND和WM_NOTIFY消息
                // 转发给最顶层的窗口
                CHAIN_MSG_MAP_ALT_MEMBER_EX(m_message_,1,WM_COMMAND)
                CHAIN_MSG_MAP_ALT_MEMBER_EX(m_message_,1,WM_NOTIFY)
            END_MSG_MAP()

            SubWindowT():m_message_(NULL){};

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                bHandled = FALSE;
                T* pT = static_cast<T*>(this);

                LPCREATESTRUCT lpCreateStruct = (LPCREATESTRUCT)lParam;

                // 初始化m_message_
                if(lpCreateStruct && lpCreateStruct->lpCreateParams)
                    m_message_ = (Y*)(lpCreateStruct->lpCreateParams);
                ATLASSERT(m_message_);

                // 设置属性WS_POPUP
                LONG lWindowLong = ::GetWindowLong(pT->m_hWnd, GWL_STYLE) | WS_POPUP;
                // 去掉一堆其他属性
                lWindowLong &= ~WS_CHILD;
                lWindowLong &= ~WS_BORDER;
                lWindowLong &= ~WS_CAPTION;
                lWindowLong &= ~WS_SYSMENU;
                ::SetWindowLong(pT->m_hWnd, GWL_STYLE, lWindowLong);

                return 0;
            }

            LRESULT OnEraseBkgnd(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                // 屏蔽背景消息
                return TRUE;
            }
            
            // 这个指针指向最顶层的窗口，用它来将紧贴这顶层窗口的不可见窗口的消息
            // 转发给m_message_
            // 注意宏CHAIN_MSG_MAP_ALT_MEMBER_EX
            Y*		m_message_;
        };

        // 这个类在SubWindowT的基础上实现了全透明的效果
        // 利用SetLayeredWindowAttributes可以对某种特定的颜色实现全透明过滤的特性
        template<typename Y>
        class SubWindow1 :
            public CWindowImpl<SubWindow1<Y>,CWindow>,
            public SubWindowT<SubWindow1<Y>,Y>{
            typedef SubWindowT<SubWindow1<Y>,Y> BaseClass;
        public:
            BEGIN_MSG_MAP(SubWindow1)
                CHAIN_MSG_MAP(BaseClass) //连接基类的消息处理逻辑，并优先处理
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_PAINT, OnPaint)
                REFLECT_NOTIFICATIONS()
            END_MSG_MAP()

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                bHandled = FALSE;

                // 设置属性WS_EX_LAYERED
                LONG lWindowLong = ::GetWindowLong(this->m_hWnd, GWL_EXSTYLE) | WS_EX_LAYERED;
                lWindowLong &= ~WS_EX_TRANSPARENT;
                ::SetWindowLong(this->m_hWnd, GWL_EXSTYLE, lWindowLong);

                // 在OnPaint里面将整个窗口刷成RGB(255,0,255)
                // 在这里将此颜色过滤（编程全透明）
                ::SetLayeredWindowAttributes(this->m_hWnd,RGB(255,0,255),0,LWA_COLORKEY);
                return 0;
            }

            LRESULT OnPaint(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                // 在OnPaint里面将整个窗口刷成RGB(255,0,255)
                // 以便让SetLayeredWindowAttributes过滤
                CPaintDC dc_(this->m_hWnd);
                CRect client_rect_;
                this->GetClientRect(client_rect_);
                dc_.FillSolidRect(client_rect_,RGB(255,0,255));
                return TRUE;
            }
        };
        
        template<typename T,bool DLG,template<typename X> class SUB_WINDOW >
        class ImageFrameExT : public ImageFrameT<T,DLG>{
            typedef ImageFrameT<T,DLG> BaseClass;
        public:
            BEGIN_MSG_MAP(ImageFrameExT)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_MOVE, OnMove)
            END_MSG_MAP()

            ImageFrameExT(){
                m_sub_rect_.SetRect(0,0,0,0);
            }

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM lParam, BOOL& bHandled){
                bHandled = FALSE;
                T* pT = static_cast<T*>(this);
                if(!DLG){
                    // 一并创建子窗口
                    // 注意它将pT传给了最后一个参数，这个在
                    // OnCreate传给了SubWindowT
                    m_hSubWindow_.Create(pT->m_hWnd,NULL,NULL,
                            WS_VISIBLE,0,0U,pT);
                }
                return 0;
            }

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled){
                bHandled = FALSE;
                T* pT = static_cast<T*>(this);
                if(DLG){
                    // 一并创建子窗口
                    // 注意这里没有WS_CHILD
                    m_hSubWindow_.Create(pT->m_hWnd,NULL,NULL,
                            WS_VISIBLE,0,0U,pT);
                }
                return 0;
            }

            LRESULT OnMove(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled){
                // 动态更新子窗口
                bHandled = FALSE;
                T* pT = static_cast<T*>(this);
                if(m_hSubWindow_.IsWindow()){
                    CRect win_rect_;
                    pT->GetWindowRect(win_rect_);

                    if(m_sub_rect_.IsRectNull()){
                        m_hSubWindow_.MoveWindow(win_rect_);
                    }else{
                        CRect tmp_ = m_sub_rect_;
                        tmp_.OffsetRect(win_rect_.TopLeft());
                        m_hSubWindow_.MoveWindow(tmp_);
                    }
                }
                return 0;
            }

            SUB_WINDOW<T>*		SubWindow(){
                // 获得子窗口
                return &m_hSubWindow_;
            }
            void				SetSubRect(const CRect& rect){
                // 可以设置让子窗口位于那块区域，而不一定要占满整屏
                m_sub_rect_ = rect;
            }
        protected:
            SUB_WINDOW<T>		m_hSubWindow_;
            CRect				m_sub_rect_;
        };
        
---

        class CMainFrame :
            public CFrameWindowImpl<CMainFrame>,
            public ImageFrameExT<CMainFrame,false,SubWindow1 >
        {
            typedef ImageFrameExT<CMainFrame,false,SubWindow1 > BaseClass;
        public:
            DECLARE_FRAME_WND_CLASS(NULL, IDR_MAINFRAME)

            BEGIN_MSG_MAP(CMainFrame)
                //REFLECT_NOTIFICATIONS()
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                MESSAGE_HANDLER(WM_DESTROY, OnDestroy)
                CHAIN_MSG_MAP(CFrameWindowImpl<CMainFrame>)
                ALT_MSG_MAP(1)
                COMMAND_CODE_HANDLER(BN_CLICKED,OnClick)
            END_MSG_MAP()

            LRESULT OnCreate(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                m_close_.Create(this->m_hSubWindow_,
                        CRect(200,200,50 + 200,26 + 200),
                        _T("Close"),
                        WS_CHILD | WS_VISIBLE);
                m_help_.Create(this->m_hSubWindow_,
                        CRect(260,200,50 + 260,26 + 200),
                        _T("Help"),
                        WS_CHILD | WS_VISIBLE);
                m_help2_.Create(this->m_hSubWindow_,
                        CRect(320,200,50 + 320,26 + 200),
                        _T("Help2"),
                        WS_CHILD | WS_VISIBLE);

                return 0;
            }

            LRESULT OnDestroy(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& bHandled)
            {
                // 解决wtl不能关闭ws_popup的bug
                PostQuitMessage(0);
                bHandled = FALSE;
                return 1;
            }

            LRESULT OnClick(WORD wNotifyCode, WORD wID, HWND hWndCtl,
                  BOOL& bHandled){
                if(m_close_ == hWndCtl){
                    this->PostMessage(WM_CLOSE);
                }else if(m_help_ == hWndCtl){

                }else if(m_help2_ == hWndCtl){

                }
                return 0;
            }
            CButton		m_close_;
            CButton		m_help_;
            CButton		m_help2_;
        };
        
#其他问题

若上面放置普通的矩形控件，并且不支持透明，那么啥问题都没有，然后若要实现不规则或者透明控件，那么**全透明窗口**被过滤的颜色将被显示出来

一个变通的办法就是将需要填充子控件的区域抠出来，并交给这个子窗口来画（这就要求放置控件区域的地方没有半透明，大部分需求都只是希望能够处理简单的异形，并且消除锯齿）。

        // 将父窗口中间抠出来，交给子窗口来画
        template<typename Y>
        class SubWindow2 :
            public CWindowImpl<SubWindow2<Y>,CWindow>,
            public SubWindowT<SubWindow2<Y>,Y>{
            typedef SubWindowT<SubWindow2<Y>,Y> BaseClass;
        public:
            BEGIN_MSG_MAP(SubWindow2)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_PAINT, OnPaint)
                REFLECT_NOTIFICATIONS()
            END_MSG_MAP()

            LRESULT OnPaint(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                CPaintDC dc_(this->m_hWnd);
                ATLASSERT(m_res_);
                if(m_res_)
                    m_res_->Draw(dc_,0,0);
                return TRUE;
            }

            inline void	SetRes(CImage* res){
                ATLASSERT(res);
                if(res)
                    this->m_res_ = res;
            }
        private:
            CImage*		m_res_;
        };
        
---
        
        class CAboutDlgEx :
            public CDialogImpl<CAboutDlgEx>,
            public ImageFrameExT<CAboutDlgEx,true,SubWindow2>
        {
            typedef ImageFrameExT<CAboutDlgEx,true,SubWindow2> BaseClass;
        public:
            enum { IDD = IDD_DIALOG1 };

            BEGIN_MSG_MAP(CAboutDlgEx)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                MESSAGE_HANDLER(WM_RBUTTONDOWN, OnClose)
                ALT_MSG_MAP(1)
            END_MSG_MAP()

            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                bool ret_ = m_res_.InitFromFile(
                        _T("res/imgbtn.png"),
                        _T("res/imgbtn_h.png"),
                        _T("res/imgbtn_p.png"),
                        _T("res/imgbtn_d.png"));
                ATLASSERT(ret_);

                m_btn_.Create(this->m_hSubWindow_,NULL,NULL,0,0,0U,&m_res_);
                m_btn_.MoveWindow(120,120,0,0,TRUE);

                return 0;
            }
            LRESULT OnClose(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/){
                EndDialog(0);
                return 0;
            }


            ImageButton<ButtonRes,true>		m_btn_;
            ButtonRes		m_res_;
        };
        
经测试，这种方案在拖动时，两个窗口交接处有明显的刷新不一致存在。

