---
layout: post
title: 用WTL实现的Layout
category: wtl
---

        /*
         * basewidget.h
         *
         *  Created on: 2012-4-14
         *      Author: king
         */

        #ifndef BASEWIDGET_H_
        #define BASEWIDGET_H_

        #include "stdafx.h"
        #include <atlcrack.h>
        #include <atlmisc.h>

        #include <vector>
        #include <cassert>
        #include <cstdint>

        template <typename T>
        class LayoutBase{
        protected:
            //类型
            enum Layout_t{
                Layout_Fill = 1,
                Layout_Top,
                Layout_Center,
                Layout_Bottom,
                Layout_Left,
                Layout_Right
            };

            typedef std::pair<CWindow*,uint8_t> 	WinEntry;
            typedef std::vector<WinEntry> 			WinVector;
            typedef WinVector::iterator				WinIter;

        public:
            LayoutBase():m_horizontal_(true){}
            virtual ~LayoutBase(){}

            BEGIN_MSG_MAP(LayoutBase)
                MESSAGE_HANDLER(WM_SIZE,OnSize)
            END_MSG_MAP()
            // 需要实现的接口，注意“不要”用虚函数
            //void	StartInitRect(const CSize& size){};
            //bool	InitRect(size_t index,const CSize& size,CRect* rect){};

            LRESULT OnSize(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                //
                if(Empty())
                    return TRUE;
                CSize size_(GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
                size_t count_ = Size();
                static_cast<T*>(this)->StartInitRect(size_);

                WinIter win_iter_ = m_windows_.begin();

                for(size_t i = 0;i < count_;i++){
                    CRect rect_;

                    //获得此控件位置
                    if(static_cast<T*>(this)->InitRect(i,size_,&rect_) &&
                            win_iter_->first){
                        ResetWindow(rect_,win_iter_->second,win_iter_->first);
                    }

                    ++ win_iter_;
                }
                return TRUE;
            }

        protected:
            inline uint16_t	CSize2Len(const CSize& size){
                return IsHorizontal()?size.cx:size.cy;
            }

            // 不要直接使用
            inline void		AddMember(CWindow* win,Layout_t hlayout,Layout_t vlayout){
                this->m_windows_.push_back(WinEntry(win,GenLayoutFlag(hlayout,vlayout)));
            }

            inline uint8_t GenLayoutFlag(Layout_t hlayout,Layout_t vlayout){
                // 括号不能省，下同
                return ((0x0F&hlayout) + ((0x0F&vlayout) << 4));
            }
            inline int HLayoutFlag(uint8_t flag){
                return (flag & 0x0F);
            }
            inline int VLayoutFlag(uint8_t flag){
                return ((flag >> 4) & 0x0F);
            }

            // 根据Flag重置Window
            void			ResetWindow(
                    const CRect& rect,
                    uint8_t flag,
                    CWindow* win){
                CRect rect_(rect);
                //横向
                if(Layout_Fill != HLayoutFlag(flag)){
                    CRect sub_win_rect_;
                    win->GetWindowRect(&sub_win_rect_);
                    if(Layout_Left == HLayoutFlag(flag)){
                        rect_.right = rect_.left + sub_win_rect_.Width();
                    }else if(Layout_Center == HLayoutFlag(flag)){
                        rect_.left = rect_.left +
                                (rect_.Width() - sub_win_rect_.Width())/2;
                        rect_.right = rect_.left + sub_win_rect_.Width();
                    }else if(Layout_Right == HLayoutFlag(flag)){
                        rect_.left = rect_.right - sub_win_rect_.Width();
                    }else{
                        assert(0);
                        return;
                    }
                }

                //竖向
                if(Layout_Fill != VLayoutFlag(flag)){
                    CRect sub_win_rect_;
                    win->GetWindowRect(&sub_win_rect_);
                    if(Layout_Top == VLayoutFlag(flag)){
                        rect_.bottom = rect_.top + sub_win_rect_.Height();
                    }else if(Layout_Center == VLayoutFlag(flag)){
                        rect_.top = rect_.top +
                                (rect_.Height() - sub_win_rect_.Height())/2;
                        rect_.bottom = rect_.top + sub_win_rect_.Height();
                    }else if(Layout_Bottom == VLayoutFlag(flag)){
                        rect_.top = rect_.bottom - sub_win_rect_.Height();
                    }else{
                        assert(0);
                        return;
                    }
                }
                win->MoveWindow(&rect_);
            }

            // 设置横向的layout
            inline void		SetHorizontal(bool horizontal){
                this->m_horizontal_ = horizontal;
            }
            //是否横向的layout
            inline bool		IsHorizontal() const{
                return this->m_horizontal_;
            }

            // 元素大小
            inline bool		Empty() const{
                return this->m_windows_.empty();
            }
            // 元素个数
            inline size_t		Size() const{
                return this->m_windows_.size();
            }

        private:
            WinVector			m_windows_;
            bool				m_horizontal_;
        };

        #if 0
        class BoxLayout :
            public CWindowImpl<BoxLayout, CWindow , CControlWinTraits>,
            public LayoutBase<BoxLayout>{
        public:
            BoxLayout():m_per_length_(0),m_offset_(0){}

            typedef LayoutBase<BoxLayout> BaseClass;
            BEGIN_MSG_MAP(BoxLayout)
                CHAIN_MSG_MAP(BaseClass)
            END_MSG_MAP()

            void	StartInitRect(const CSize& size){
                m_per_length_ = (IsHorizontal()?size.cx:size.cy)/Size();
                m_offset_ = 0;
            }
            bool	InitRect(size_t index,const CSize& size,CRect* rect){
                if(IsHorizontal())
                    rect->SetRect(m_offset_,0,m_offset_ + m_per_length_,size.cy);
                else
                    rect->SetRect(0,m_offset_,size.cx,m_offset_ + m_per_length_);

                // 最后一个让其填满
                if(index == Size() - 1){
                    if(IsHorizontal())
                        rect->right = size.cx;
                    else
                        rect->bottom = size.cy;
                }

                m_offset_ += m_per_length_;
                return true;
            }

        private:
            LONG 		m_per_length_;
            LONG		m_offset_;
        };


        class BoxLayoutTest : public BoxLayout{
        public:
            BEGIN_MSG_MAP(BoxLayoutTest)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                CHAIN_MSG_MAP(BoxLayout)
            END_MSG_MAP()

            LRESULT OnCreate(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                m_btn1_.Create(m_hWnd,CRect(0,0,50,30),"btn1",WS_CHILD|WS_VISIBLE);
                m_btn2_.Create(m_hWnd,CRect(0,0,50,30),"btn2",WS_CHILD|WS_VISIBLE);
                m_btn3_.Create(m_hWnd,CRect(0,0,50,30),"btn3",WS_CHILD|WS_VISIBLE);
                m_btn4_.Create(m_hWnd,CRect(0,0,50,30),"btn4",WS_CHILD|WS_VISIBLE);

                this->AddMember(&m_btn1_,Layout_Fill,Layout_Fill);
                this->AddMember(&m_btn2_,Layout_Left,Layout_Top);
                this->AddMember(&m_btn3_,Layout_Center,Layout_Center);
                this->AddMember(&m_btn4_,Layout_Right,Layout_Bottom);

                this->SetHorizontal(false);
                //先初始化自己
                //this->DefWindowProc(uMsg,wParam,lParam);
                return TRUE;
            }

            int		IsClose(){
                return 0;
            }
            void	InitDlg(CWindow* win){

            }
        private:
            CButton 	m_btn1_;
            CButton		m_btn2_;
            CButton 	m_btn3_;
            CButton 	m_btn4_;
        };
        #endif


        class RatioLayout :
            public CWindowImpl<RatioLayout, CWindow , CControlWinTraits>,
            public LayoutBase<RatioLayout>{

            typedef std::vector<uint16_t> RatioVector;
            typedef RatioVector::iterator RatioIter;
        public:
            RatioLayout():m_offset_(0),m_total_(0){}

            typedef LayoutBase<RatioLayout> BaseClass;
            BEGIN_MSG_MAP(RatioLayout)
                CHAIN_MSG_MAP(BaseClass)
            END_MSG_MAP()

            void	StartInitRect(const CSize& size){
                m_offset_ = 0;
            }
            bool	InitRect(size_t index,const CSize& size,CRect* rect){
                LONG this_length_ = CSize2Len(size) * m_ratio_vector_[index]/m_total_;
                if(IsHorizontal()){
                    rect->SetRect(m_offset_,0,m_offset_ + this_length_,size.cy);
                }else{
                    rect->SetRect(0,m_offset_,size.cx,m_offset_ + this_length_);
                }

                // 最后一个让其填满
                if(index == Size() - 1){
                    if(IsHorizontal())
                        rect->right = size.cx;
                    else
                        rect->bottom = size.cy;
                }

                m_offset_ += this_length_;
                return true;
            }

            void		AddMemberEx(
                    CWindow* win,
                    Layout_t hlayout,
                    Layout_t vlayout,
                    uint16_t len){

                assert(len);
                assert(m_total_ + len > m_total_);
                if(m_total_ + len > m_total_){
                    BaseClass::AddMember(win,hlayout,vlayout);
                    m_ratio_vector_.push_back(len);
                    m_total_ += len;
                }
            }

        private:
            LONG		m_offset_;

            uint16_t	m_total_;
            RatioVector	m_ratio_vector_;
        };

        class RatioLayoutTest : public RatioLayout,public DlgViewBase{
        public:
            BEGIN_MSG_MAP(RatioLayoutTest)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                CHAIN_MSG_MAP(RatioLayout)
            END_MSG_MAP()

            LRESULT OnCreate(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                m_btn1_.Create(m_hWnd,CRect(0,0,50,30),_T("btn1"),WS_CHILD|WS_VISIBLE);
                m_btn2_.Create(m_hWnd,CRect(0,0,50,30),_T("btn2"),WS_CHILD|WS_VISIBLE);
                m_btn3_.Create(m_hWnd,CRect(0,0,50,30),_T("btn3"),WS_CHILD|WS_VISIBLE);

                this->AddMemberEx(&m_btn1_,Layout_Fill,Layout_Fill,1);
                this->AddMemberEx(&m_btn2_,Layout_Fill,Layout_Fill,1);
                this->AddMemberEx(&m_btn3_,Layout_Fill,Layout_Fill,2);

                this->SetHorizontal(false);
                return TRUE;
            }

        private:
            CButton 	m_btn1_;
            CButton		m_btn2_;
            CButton 	m_btn3_;
        };



        class ExBoxLayout :
            public CWindowImpl<ExBoxLayout, CWindow , CControlWinTraits>,
            public LayoutBase<ExBoxLayout>{
            ;
        private:
            class Entry{
            public:
                // 适合动态调整大小的窗口
                Entry(bool fixflag,bool spaceflag,uint16_t size):
                    m_is_fixd_(fixflag),
                    m_is_space_(spaceflag),
                    m_size_(size){
                }

                inline bool	IsFix() const{
                    return this->m_is_fixd_;
                }
                inline bool	IsWindow() const{
                    return !(this->m_is_space_);
                }

                inline uint16_t	Size() const{
                    return this->m_size_;
                }
            private:
                // 是否固定大小
                bool			m_is_fixd_;

                bool			m_is_space_;

                uint16_t		m_size_;
            };

            typedef std::vector<Entry> EntryVector;
            typedef EntryVector::iterator EntryIter;
        public:
            ExBoxLayout():m_offset_(0),m_var_len_(0),m_total_(0){
                this->m_iter_ = this->m_win_infos_.end();
            }

            typedef LayoutBase<ExBoxLayout> BaseClass;
            BEGIN_MSG_MAP(ExBoxLayout)
                CHAIN_MSG_MAP(BaseClass)
            END_MSG_MAP()

            void	StartInitRect(const CSize& size){

                this->m_var_len_ = 0;
                if(this->m_total_){
                    // 获得所有定常区间的总长度
                    LONG fix_size_ = 0;
                    for(this->m_iter_ = this->m_win_infos_.begin();
                            this->m_iter_ != this->m_win_infos_.end();
                            ++ this->m_iter_){
                        if(this->m_iter_->IsFix())
                            fix_size_ += this->m_iter_->Size();
                    }

                    // 计算变长区域每一个单位的像素值
                    LONG var_len_ = CSize2Len(size);
                    var_len_ = (var_len_ >= fix_size_)?(var_len_ - fix_size_):0;
                    this->m_var_len_ = double(var_len_)/this->m_total_;
                }

                // 给所有动态变量清零
                this->m_offset_ = 0;
                this->m_iter_ = this->m_win_infos_.begin();

            }
            bool	InitRect(size_t index,const CSize& size,CRect* rect){

                LONG len_ = 0;
                if(m_iter_->IsFix()){
                    len_ = this->m_iter_->Size();
                }else{
                    len_ = this->m_var_len_ * this->m_iter_->Size();

                    // 如果是最后一个变长区域
                    if(index == this->Size() - 1){
                        if(this->m_offset_ + len_ < CSize2Len(size))
                            len_ = CSize2Len(size) - this->m_offset_;
                    }

                }

                if(this->IsHorizontal()){
                    rect->SetRect(
                            this->m_offset_,
                            0,
                            this->m_offset_ + len_,
                            size.cy);
                }else{
                    rect->SetRect(
                            0,
                            this->m_offset_,
                            size.cx,
                            this->m_offset_ + len_);
                }

                this->m_offset_ += len_;

                // 如果非space，则需要重置窗口
                bool resize_flag_ = this->m_iter_->IsWindow();
                ++ this->m_iter_;
                return resize_flag_;
            }

            void		AddMemberFix(
                    CWindow* win,	//可以为空，表示space
                    Layout_t hlayout,
                    Layout_t vlayout,
                    uint16_t size){

                BaseClass::AddMember(win,hlayout,vlayout);

                //若win == NULL，则表示space
                m_win_infos_.push_back(Entry(true,!win,size));
            }

            void		AddMemberVar(
                    CWindow* win,	//可以为空，表示space
                    Layout_t hlayout,
                    Layout_t vlayout,
                    uint16_t percent){

                assert(percent);
                assert(m_total_ + percent > m_total_);
                if(m_total_ + percent > m_total_){

                    BaseClass::AddMember(win,hlayout,vlayout);

                    //若win == NULL，则表示space
                    m_win_infos_.push_back(Entry(false,!win,percent));
                    m_total_ += percent;
                }
            }

        private:
            // 用户支持baseclass接口
            LONG			m_offset_;
            //可变区域的单位像素
            double			m_var_len_;
            //当前控件
            EntryIter		m_iter_;
            // 固定的
            uint16_t		m_total_;
            EntryVector		m_win_infos_;
        };

        class ExBoxLayoutTest : public ExBoxLayout,public DlgViewBase{
        public:
            BEGIN_MSG_MAP(ExBoxLayoutTest)
                MESSAGE_HANDLER(WM_CREATE, OnCreate)
                CHAIN_MSG_MAP(ExBoxLayout)
            END_MSG_MAP()

            LRESULT OnCreate(UINT uMsg, WPARAM wParam, LPARAM lParam, BOOL& bHandled){
                m_btn1_.Create(m_hWnd,CRect(0,0,50,30),_T("btn1"),WS_CHILD|WS_VISIBLE);
                m_btn2_.Create(m_hWnd,CRect(0,0,50,30),_T("btn2"),WS_CHILD|WS_VISIBLE);
                m_btn3_.Create(m_hWnd,CRect(0,0,50,30),_T("btn3"),WS_CHILD|WS_VISIBLE);

                this->AddMemberFix(NULL,Layout_Fill,Layout_Fill,100);
                this->AddMemberVar(&m_btn1_,Layout_Fill,Layout_Center,100);
                this->AddMemberFix(&m_btn2_,Layout_Fill,Layout_Fill,100);
                this->AddMemberVar(&m_btn3_,Layout_Fill,Layout_Center,100);

                //this->SetHorizontal(false);
                return TRUE;
            }

        private:
            CButton 	m_btn1_;
            CButton		m_btn2_;
            CButton 	m_btn3_;
        };

        #endif /* BASEWIDGET_H_ */





