---
layout: post
title: 在标题栏添加控件实现
category: wtl
---

        class NcWidgetBase{
        public:
            virtual void OnCreate(CWindow* window){}
            virtual void OnDestroy(CWindow* window){}
            virtual void OnPaint(CWindow* window,CWindowDC* dc){}
            virtual void OnMouseMove(CWindow* window,const CPoint& point){}
            virtual void OnMouseEnter(CWindow* window,const CPoint& point){}
            virtual void OnMouseLeave(CWindow* window){}
            virtual void OnLButtonDown(CWindow* window,const CPoint& point){}
            virtual void OnLButtonDblClk(CWindow* window,const CPoint& point){}
            virtual void OnLButtonUp(CWindow* window,const CPoint& point){}
            // 返回true表示接收消息
            virtual bool OnHitTest(const CPoint& point){return true;}


            /////////////////////////////////////////////////////
            // wrapper
            /////////////////////////////////////////////////////
            // OnHitTest
            bool 		OnNcHitTestBase(const CPoint& point,CPoint* actual){
                ATLASSERT(actual);
                if(TRUE == this->m_position_.PtInRect(point)){
                    ATLASSERT(!m_position_.IsRectNull());
                    *actual = point;
                    // 将父窗口坐标改成当前窗口坐标
                    actual->Offset(-m_position_.TopLeft());
                    return this->OnHitTest(*actual);
                }else
                    return false;
            }

            // OnPaint
            void OnPaintBase(CWindow* window,CWindowDC* dc){
                ATLASSERT(window);
                ATLASSERT(dc);
                // 创建客户区的rgn
                CRgn rgn_;
                rgn_.CreateRectRgnIndirect(m_position_);
                dc->SelectClipRgn(rgn_, RGN_AND);
                /*
                MyDbgStr(_T("OnPaintBase %d,%d,%d,%d\n"),
                        m_position_.left,
                        m_position_.right,
                        m_position_.top,
                        m_position_.bottom);
                 */

                CPoint point_;
                dc->SetViewportOrg(m_position_.TopLeft(),&point_);
                this->OnPaint(window,dc);
                dc->SetViewportOrg(point_);
            }

            /////////////////////////////////////////////////////
            // util
            /////////////////////////////////////////////////////
            // 重绘
            void		RePaint(CWindow* window){
                ATLASSERT(window);
                window->SendMessage(WM_NCPAINT,0,0);
            }
            // layout,设置窗口位置
            void		LayoutWindow(const CRect& rect){
                if(!this->m_position_.EqualRect(rect)){
                    this->m_position_ = rect;
                }
            }
            // 将widget坐标转换为全局坐标
            void		PointToGlobal(CWindow* window,CPoint* point){
                ATLASSERT(window);
                ATLASSERT(point);
                ATLASSERT(!m_position_.IsRectNull());

                point->Offset(m_position_.TopLeft());

                // 屏幕坐标转换成窗口坐标
                CRect rcWindow;
                window->GetWindowRect(rcWindow);
                point->Offset(rcWindow.TopLeft());

            }
        private:
            CRect		m_position_;
        };

        template <class T,bool OWNER_DRAW>
        class TitleWidget{
        public:
            typedef NcWidgetBase* 					Entry;
            typedef std::vector<Entry>				Vector;
            typedef Vector::iterator				Iter;
            typedef Vector::reverse_iterator		RIter;
        public:
            TitleWidget():
                m_move_hittest_(NULL),
                m_lbtn_hittest_(NULL),
                m_mouse_tracking_(false){}

            BEGIN_MSG_MAP(TitleWidget)
                MESSAGE_HANDLER(WM_NCDESTROY, OnNcDestroy)
                //MESSAGE_HANDLER(WM_NCCALCSIZE,OnNcCalcSize)
                MESSAGE_HANDLER(WM_NCACTIVATE, OnNcPaint)
                MESSAGE_HANDLER(WM_NCPAINT, OnNcPaint)
                MESSAGE_HANDLER(WM_NCMOUSEMOVE, OnNcMouseMove)
                MESSAGE_HANDLER(WM_NCMOUSELEAVE, OnNcMouseLeave)
                MESSAGE_HANDLER(WM_NCLBUTTONDOWN, OnNcLButtonDown)
                MESSAGE_HANDLER(WM_NCLBUTTONDBLCLK, OnNcLButtonDblclk)
                MESSAGE_HANDLER(WM_NCLBUTTONUP, OnNcLButtonUp)
                MESSAGE_HANDLER(WM_LBUTTONUP, OnLButtonUp)
            END_MSG_MAP()


            /*
            LRESULT OnNcCalcSize(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled) {
                bHandled = FALSE;

                NCCALCSIZE_PARAMS* pParams = NULL;
                RECT* pRect = NULL;

                BOOL bValue = static_cast<BOOL>(wParam);
                if(bValue)
                    pParams = reinterpret_cast<NCCALCSIZE_PARAMS*>(lParam);
                else
                    pRect = reinterpret_cast<RECT*>(lParam);

                if(bValue){
                    pRect = &pParams->rgrc[0];
                }

                pRect->left = pRect->left + 0;
                pRect->top = pRect->top + 0;
                pRect->right = pRect->right + 0;
                pRect->bottom = pRect->bottom + 0;

                MyDbgStr(_T("OnNcCalcSize %d,%d,%d,%d\n"),
                        pRect->left,
                        pRect->right,
                        pRect->top,
                        pRect->bottom);

                if(bValue)
                    pParams->rgrc[1] = pParams->rgrc[0];
                return TRUE;
            }
            */

            LRESULT OnNcDestroy(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled)
            {
                bHandled = FALSE;
                T* pT = static_cast<T*>(this);
                for(RIter iter_ = m_widgets_.rbegin();
                        iter_ != m_widgets_.rend();
                        ++ iter_){
                    //调用子控件destroy函数
                    ATLASSERT(*iter_);
                    (*iter_)->OnDestroy(pT);
                }
                return TRUE;
            }

            LRESULT OnNcPaint(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled)
            {
                //bHandled = FALSE;
                T* pT = static_cast<T*>(this);
                if(OWNER_DRAW)
                    ;
                else
                    ::DefWindowProc(pT->m_hWnd, uMsg, wParam, lParam);

                CRgn rgn;
                {
                    CRect rcWindow, rcClient;
                    pT->GetWindowRect(rcWindow);

                    // layout
                    pT->OnNcLayout(pT,(UINT)wParam,rcWindow.Size());

                    pT->GetClientRect(rcClient);
                    pT->ClientToScreen(rcClient);
                    rcClient.OffsetRect(-rcWindow.TopLeft());
                    rcWindow.OffsetRect(-rcWindow.TopLeft());

                    // 创建整个窗口的rgn
                    rgn.CreateRectRgn(rcWindow.left, rcWindow.top, rcWindow.right, rcWindow.bottom);

                    // 创建客户区的rgn
                    CRgn clt_rgn_;
                    clt_rgn_.CreateRectRgn(rcClient.left, rcClient.top, rcClient.right, rcClient.bottom);

                    // 排除客户区的rgn
                    rgn.CombineRgn(clt_rgn_,RGN_DIFF);
                }

                // 依次绘制子窗口
                CWindowDC dc(pT->m_hWnd);
                for(Iter iter_ = m_widgets_.begin();
                        iter_ != m_widgets_.end();
                        ++ iter_){
                    dc.SelectRgn(rgn);
                    (*iter_)->OnPaintBase(pT,&dc);
                }
                dc.SelectRgn(rgn);
                return TRUE;
            }


            LRESULT OnNcMouseMove(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                T* pT = static_cast<T*>(this);
                if(!m_mouse_tracking_){
                    TRACKMOUSEEVENT   tme;
                    tme.cbSize = sizeof(tme);
                    tme.dwFlags = TME_LEAVE | TME_NONCLIENT; //注册非客户区离开
                    tme.hwndTrack = pT->m_hWnd;
                    tme.dwHoverTime = HOVER_DEFAULT; //只对HOVER有效
                    ::TrackMouseEvent(&tme);
                    m_mouse_tracking_ = true;
                }

                CPoint point_;
                GetPosition(&point_,lParam);

                // 依次处理每一个widget
                CPoint widget_point_;
                for(RIter iter_ = m_widgets_.rbegin();
                        iter_ != m_widgets_.rend();
                        ++ iter_){
                    if((*iter_)->OnNcHitTestBase(point_,&widget_point_)){

                        if(*iter_ != m_move_hittest_){
                            // 如果老widget存在，则发送OnMouseLeave
                            if(m_move_hittest_){
                                m_move_hittest_->OnMouseLeave(pT);
                                m_move_hittest_ = NULL;
                            }

                            // 如果是新widget，则发送MouseEnter
                            (*iter_)->OnMouseEnter(pT,widget_point_);
                            m_move_hittest_ = *iter_;
                        }

                        (*iter_)->OnMouseMove(pT,widget_point_);
                        return TRUE;
                    }
                }

                // 如果老widget存在，则发送OnMouseLeave
                if(m_move_hittest_){
                    m_move_hittest_->OnMouseLeave(pT);
                    m_move_hittest_ = NULL;
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }


            LRESULT OnNcMouseLeave(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                T* pT = static_cast<T*>(this);
                m_mouse_tracking_ = false;

                // 如果老widget存在，则发送OnMouseLeave
                if(m_move_hittest_){
                    m_move_hittest_->OnMouseLeave(pT);
                    m_move_hittest_ = NULL;
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }

            LRESULT OnNcLButtonDown(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                T* pT = static_cast<T*>(this);

                CPoint point_;
                GetPosition(&point_,lParam);

                // 依次处理每一个widget
                CPoint widget_point_;
                for(RIter iter_ = m_widgets_.rbegin();
                        iter_ != m_widgets_.rend();
                        ++ iter_){
                    if((*iter_)->OnNcHitTestBase(point_,&widget_point_)){
                        this->NotifyLButtonDown(*iter_,pT,widget_point_);
                        return TRUE;
                    }
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }

            void		NotifyLButtonDown(NcWidgetBase* hittest,CWindow* window,const CPoint& point){
                ATLASSERT(hittest);
                ATLASSERT(window);
                hittest->OnLButtonDown(window,point);
                m_lbtn_hittest_ = hittest;
            }
            void		NotifyLButtonUp(NcWidgetBase* hittest,CWindow* window,const CPoint& point){
                ATLASSERT(hittest);
                ATLASSERT(window);
                hittest->OnLButtonUp(window,point);
                m_lbtn_hittest_ = NULL;
            }

            LRESULT OnNcLButtonDblclk(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                T* pT = static_cast<T*>(this);

                CPoint point_;
                this->GetPosition(&point_,lParam);

                // 依次处理每一个widget
                CPoint widget_point_;
                for(RIter iter_ = m_widgets_.rbegin();
                        iter_ != m_widgets_.rend();
                        ++ iter_){
                    if((*iter_)->OnNcHitTestBase(point_,&widget_point_)){
                        (*iter_)->OnLButtonDblClk(pT,widget_point_);
                        return TRUE;
                    }
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }

            LRESULT OnNcLButtonUp(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                T* pT = static_cast<T*>(this);

                CPoint point_;
                GetPosition(&point_,lParam);

                // 先处理m_lbtn_hittest_
                if(m_lbtn_hittest_){
                    // 这里的point_是不准确的
                    this->NotifyLButtonUp(m_lbtn_hittest_,pT,point_);
                    return TRUE;
                }

                // 依次处理每一个widget
                CPoint widget_point_;
                for(RIter iter_ = m_widgets_.rbegin();
                        iter_ != m_widgets_.rend();
                        ++ iter_){
                    if((*iter_)->OnNcHitTestBase(point_,&widget_point_)){
                        this->NotifyLButtonUp(*iter_,pT,widget_point_);
                        return TRUE;
                    }
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }

            LRESULT OnLButtonUp(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){

                T* pT = static_cast<T*>(this);
                // 先处理m_lbtn_hittest_
                if(m_lbtn_hittest_){
                    pT->SendMessage(WM_NCLBUTTONUP,wParam,lParam);
                    return TRUE;
                }

                // 否则交给默认处理
                bHandled = FALSE;
                return TRUE;
            }

            ////////////////////////////////////////////////////////////
            // 子窗口相关
            void			AddWidget(NcWidgetBase* widget){
                ATLASSERT(widget);
                this->m_widgets_.push_back(widget);

                // 调用Create函数
                T* pT = static_cast<T*>(this);
                widget->OnCreate(pT);
            }

        private:
            void			GetPosition(CPoint* point,LPARAM lParam){
                CPoint point_(GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
                T* pT = static_cast<T*>(this);
                // 屏幕坐标转换成窗口坐标
                CRect rcWindow;
                pT->GetWindowRect(rcWindow);
                point_.Offset(-rcWindow.TopLeft());
                *point = point_;
            }
        private:
            Vector			m_widgets_;
            // 跟踪鼠标hover的控件
            NcWidgetBase*	m_move_hittest_;

            // 跟踪被左键单击的控件
            NcWidgetBase*	m_lbtn_hittest_;

            // 跟踪鼠标移出
            bool			m_mouse_tracking_;
        };

        class BitmapBtn : public NcWidgetBase{
        public:
            virtual void OnCreate(CWindow* window){

                LoadBitmapFromFile(&m_bitmap_normal_,_T("res\\close_normal.bmp"));
                LoadBitmapFromFile(&m_bitmap_hover_,_T("res\\close_hover.bmp"));
                LoadBitmapFromFile(&m_bitmap_press_,_T("res\\close_press.bmp"));
                LoadBitmapFromFile(&m_bitmap_disable_,_T("res\\close_disable.bmp"));

                m_mask_color_ = RGB(255, 0, 255);

                m_hover_ = false;
                m_pressed_ = false;
                m_disable_ = false;
            }
            virtual void OnPaint(CWindow* window,CWindowDC* dc)
            {
                CBitmap* bitmap_ = NULL;
                if(m_disable_)
                    bitmap_ = &m_bitmap_disable_;
                if(m_pressed_)
                    bitmap_ = &m_bitmap_press_;
                else if(m_hover_)
                    bitmap_ = &m_bitmap_hover_;
                else
                    bitmap_ = &m_bitmap_normal_;

                CSize size_;
                this->GetPrefSize(&size_);

                CDC dcMemory;
                dcMemory.CreateCompatibleDC(*dc);
                dcMemory.SelectBitmap(*bitmap_);

                dc->TransparentBlt(
                        0,
                        0,
                        size_.cx,
                        size_.cy,
                        dcMemory,
                        0,
                        0,
                        size_.cx,
                        size_.cy,
                        m_mask_color_);
            }
            virtual void OnMouseEnter(CWindow* window,const CPoint& point){
                m_hover_ = true;
                RePaint(window);
            }
            virtual void OnMouseLeave(CWindow* window){
                m_hover_ = false;
                RePaint(window);
            }
            virtual void OnLButtonDown(CWindow* window,const CPoint& point){
                m_pressed_ = true;
                RePaint(window);
                window->SetCapture();
            }
            virtual void OnLButtonUp(CWindow* window,const CPoint& point){
                m_pressed_ = false;
                RePaint(window);
                ::ReleaseCapture();
            }
            virtual void OnLButtonDblClk(CWindow* window,const CPoint& point){
                CPoint local_p_(point);
                PointToGlobal(window,&local_p_);
                window->SendMessage(WM_NCLBUTTONDOWN,HTNOWHERE,MAKELPARAM(local_p_.x,local_p_.y));
            }
            void		GetPrefSize(CSize* size){
                ATLASSERT(size);
                m_bitmap_normal_.GetSize(*size);
            }
        private:
            CBitmap		m_bitmap_normal_;
            CBitmap		m_bitmap_hover_;
            CBitmap		m_bitmap_press_;
            CBitmap		m_bitmap_disable_;
            COLORREF 	m_mask_color_;

            bool		m_hover_;
            bool		m_pressed_;
            bool		m_disable_;
        };

        class BitmapStatic : public NcWidgetBase{
        public:
            virtual void OnCreate(CWindow* window){
                LoadBitmapFromFile(&m_bitmap_,_T("res\\static.bmp"));
                m_mask_color_ = RGB(255, 0, 255);
            }
            virtual void OnPaint(CWindow* window,CWindowDC* dc)
            {
                CSize size_;
                this->GetPrefSize(&size_);

                CDC dcMemory;
                dcMemory.CreateCompatibleDC(*dc);
                dcMemory.SelectBitmap(m_bitmap_);

                dc->TransparentBlt(
                        0,
                        0,
                        size_.cx,
                        size_.cy,
                        dcMemory,
                        0,
                        0,
                        size_.cx,
                        size_.cy,
                        m_mask_color_);
            }
            void		GetPrefSize(CSize* size){
                ATLASSERT(size);
                m_bitmap_.GetSize(*size);
            }

            virtual bool OnHitTest(const CPoint& point){return false;}
        private:
            CBitmap		m_bitmap_;
            COLORREF 	m_mask_color_;
        };
        
---

        #include "title_fuck.h"


        class CMainDlg : 
            public CDialogImpl<CMainDlg>,
            public TitleWidget<CMainDlg,false>
        {
            typedef TitleWidget<CMainDlg,false> BaseClass;
        public:
            enum { IDD = IDD_MAINDLG };

            BEGIN_MSG_MAP(CMainDlg)
                CHAIN_MSG_MAP(BaseClass)
                MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
                COMMAND_ID_HANDLER(IDOK, OnOK)
                COMMAND_ID_HANDLER(IDCANCEL, OnCancel)
            END_MSG_MAP()
            
            LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
            {
                // ... ...
                
                this->AddWidget(&m_nc_button_);
                this->AddWidget(&m_nc_static_);

                return TRUE;
            }
            
            void 	OnNcLayout(CWindow* window,UINT nType, CSize size){
                CSize size_;
                m_nc_button_.GetPrefSize(&size_);

                CRect rect_(0,1,size_.cx,size_.cy);
                rect_.OffsetRect(size.cx - rect_.Width() - 100,0);
                m_nc_button_.LayoutWindow(rect_);


                m_nc_static_.GetPrefSize(&size_);
                rect_.SetRect(0,1,size_.cx,size_.cy);
                rect_.OffsetRect(size.cx - rect_.Width() - 140,0);
                m_nc_static_.LayoutWindow(rect_);
            }

        private:
            BitmapBtn		m_nc_button_;
            BitmapStatic	m_nc_static_;
        };