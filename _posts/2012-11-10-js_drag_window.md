---
layout: post
title:  使用js来控制原生窗口的拖拽
category: windows
---

##需求
为了让Html完全接管窗口，可以创建一个popup的窗口，然后将整个Html嵌入窗口内，在Html绘制模拟的边框和标题栏，通过js事件并控制C后端的逻辑代码实现标题栏拖动窗口，左，上，右，下，左上，右上，右下，左下的拖动大小。

**JS代码**

**res:/#129** 表示jquery-1.8.0.min.js在资源中的路径，129是资源ID

        <html xmlns="http://www.w3.org/1999/xhtml">
        <head>
        <title>Simple drag demo</title>
        <style>
        #rect {
          cursor:move;
          background:#eee;
          border:1px solid #333;
          padding:10px;
        }
        </style>

        <script type="text/javascript" src="res:/#129"></script>
        <script>


        var drag_x;
        var drag_y;

        function makeDraggable(element,opr,xmodify,ymodify) {

            /* Simple drag implementation */
            element.onmousedown = function(event) 
            {	
                event = event || window.event;
                drag_x = event.clientX;
                drag_y = event.clientY;
                
                document.onmousemove = function(event) 
                {
                    event = event || window.event;
                    window.external.move(
                        event.clientX - drag_x,
                        event.clientY - drag_y,
                        opr);
                    if(xmodify)
                    {
                        drag_x = event.clientX;
                    }
                    if(ymodify)
                    {
                        drag_y = event.clientY;
                    }
                };

                document.onmouseup = function() 
                {
                    document.onmousemove = null;
                    if(element.releaseCapture) 
                    { 
                        element.releaseCapture(); 
                    }
                };

                if(element.setCapture) 
                { 
                    element.setCapture(); 
                }
                    
                
            };

            /* These 3 lines are helpful for the browser to not accidentally 
            * think the user is trying to "text select" the draggable object
            * when drag initiation happens on text nodes.
            * Unfortunately they also break draggability outside the window.
            */
            element.unselectable = "on";
            element.onselectstart = function(){return false};
            element.style.userSelect = element.style.MozUserSelect = "none";
        }

        $(function(){
            makeDraggable($(".title")[0],"move",false,false);
            
            makeDraggable($(".top")[0],"top",false,false);
            makeDraggable($(".left")[0],"left",false,false);
            makeDraggable($(".topleft")[0],"topleft",false,false);
            
            makeDraggable($(".bottom")[0],"bottom",true,true);
            makeDraggable($(".right")[0],"right",true,true);
            makeDraggable($(".bottomright")[0],"bottomright",true,true);
            
            makeDraggable($(".topright")[0],"topright",true,false);
            makeDraggable($(".bottomleft")[0],"bottomleft",false,true);
        });
        </script>
        </head>
        <body>

        <div id="rect" class="title">标题栏</div>
        <div id="rect" class="left">左</div>
        <div id="rect" class="topleft">左上</div>
        <div id="rect" class="top">上</div>
        <div id="rect" class="topright">右上</div>
        <div id="rect" class="right">右</div>
        <div id="rect" class="bottomright">右下</div>
        <div id="rect" class="bottom">下</div>
        <div id="rect" class="bottomleft">左下</div>

        </body>
        </html>

---

        // httpappDlg.h : 头文件
        //

        #pragma once


        // ChttpappDlg 对话框
        class ChttpappDlg : public CDHtmlDialog
        {
        // 构造
        public:
            ChttpappDlg(CWnd* pParent = NULL);	// 标准构造函数

        // 对话框数据
            enum { IDD = IDD_HTTPAPP_DIALOG, IDH = IDR_HTML_HTTPAPP_DIALOG };

            protected:
            virtual void DoDataExchange(CDataExchange* pDX);	// DDX/DDV 支持
            virtual BOOL IsExternalDispatchSafe(){ return TRUE;}

            HRESULT OnButtonOK(IHTMLElement *pElement);
            HRESULT OnButtonCancel(IHTMLElement *pElement);

            void HttpMove(VARIANT& x,VARIANT& y,VARIANT& opr);
        // 实现
        protected:
            HICON m_hIcon;

            // 生成的消息映射函数
            virtual BOOL OnInitDialog();
            afx_msg void OnPaint();
            afx_msg HCURSOR OnQueryDragIcon();
            DECLARE_MESSAGE_MAP()
            DECLARE_DHTML_EVENT_MAP()
            DECLARE_DISPATCH_MAP()
        };

---

        // httpappDlg.cpp : 实现文件
        //

        BEGIN_DISPATCH_MAP(ChttpappDlg, CDHtmlDialog)
            DISP_FUNCTION(ChttpappDlg,"move",HttpMove,VT_EMPTY,VTS_VARIANT VTS_VARIANT VTS_VARIANT)
        END_DISPATCH_MAP()

        void ChttpappDlg::HttpMove(VARIANT& x,VARIANT& y,VARIANT& opr)
        {
            if(x.vt == VT_I4 && y.vt == VT_I4 && opr.vt == VT_BSTR)
            {
                CString opr_(opr.bstrVal);
                CPoint point_(x.lVal,y.lVal);
            
                CRect rect_;
                this->GetWindowRect(&rect_);
                const int min_height_ = 100;
                const int min_width_ = 200;
                if(opr_ == "move")
                {
                    rect_.OffsetRect(point_);
                }
                else if(opr_ == "top")
                {
                    rect_.top += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.top = rect_.bottom - min_height_;
                }
                else if(opr_ == "left")
                {
                    rect_.left += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.left = rect_.right - min_width_;
                }
                else if(opr_ == "topleft")
                {
                    rect_.left += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.left = rect_.right - min_width_;
                    rect_.top += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.top = rect_.bottom - min_height_;
                }
                else if(opr_ == "right")
                {
                    rect_.right += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.right = rect_.left + min_width_;
                }
                else if(opr_ == "bottom")
                {
                    rect_.bottom += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.bottom = rect_.top + min_height_;
                }
                else if(opr_ == "bottomright")
                {
                    rect_.right += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.right = rect_.left + min_width_;
                    rect_.bottom += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.bottom = rect_.top + min_height_;
                }
                else if(opr_ == "bottomleft")
                {
                    rect_.left += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.left = rect_.right - min_width_;
                    rect_.bottom += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.bottom = rect_.top + min_height_;
                }
                else if(opr_ == "topright")
                {
                    rect_.right += point_.x;
                    if(rect_.Width() < min_width_)
                        rect_.right = rect_.left + min_width_;
                    rect_.top += point_.y;
                    if(rect_.Height() < min_height_)
                        rect_.top = rect_.bottom - min_height_;
                }
                else
                {
                    return;
                }
                
                this->MoveWindow(rect_.left,
                    rect_.top,
                    rect_.Width(),
                    rect_.Height());
            }
        }

        ChttpappDlg::ChttpappDlg(CWnd* pParent /*=NULL*/)
            : CDHtmlDialog(ChttpappDlg::IDD, ChttpappDlg::IDH, pParent)
        {
            m_hIcon = AfxGetApp()->LoadIcon(IDR_MAINFRAME);
            EnableAutomation();
        }

        BOOL ChttpappDlg::OnInitDialog()
        {
            CDHtmlDialog::OnInitDialog();
            SetExternalDispatch(GetIDispatch(TRUE));

            // TODO: 在此添加额外的初始化代码

            return TRUE;  // 除非将焦点设置到控件，否则返回 TRUE
        }

#Bug
正常情况下，Mouse Up会正常触发，但是一些特殊的情况，例如按下Win键，或者Atl+Ctrl切换到其他程序，那么可能导致mouse up丢失。所以在web中必须监听**onlosecapture**事件。类似于Windows的窗口消息**WM_CAPTURECHANGED**。


#参考
1. <http://www.cnblogs.com/thinkingfor/archive/2010/11/08/1871736.html>
1. <http://stackoverflow.com/questions/1685326/responding-to-the-onmousemove-event-outside-of-the-browser-window-in-ie/1745382#1745382>
1. <http://msdn.microsoft.com/en-us/library/ie/ms536943\(v=vs.85\).aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/ms645605\(v=vs.85\).aspx>