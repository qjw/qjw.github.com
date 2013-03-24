---
layout: post
title:  自定义渲染gif实现全透明异型效果
category: wingui
---

	// MainDlg.h : interface of the CMainDlg class
	//
	/////////////////////////////////////////////////////////////////////////////

	#if !defined(VFC_MAINDLG_H__9E8ABA70_0239_4836_AFDF_F4C7840A7648__INCLUDED_)
	#define VFC_MAINDLG_H__9E8ABA70_0239_4836_AFDF_F4C7840A7648__INCLUDED_

	#if _MSC_VER >= 1000
	#pragma once
	#endif // _MSC_VER >= 1000

	#include <atlimage.h>


	class CMainDlg : public CDialogImpl<CMainDlg>
	{
	public:
		enum { IDD = IDD_MAINDLG };
		enum { TimerID = 1111};

		BEGIN_MSG_MAP(CMainDlg)
			MESSAGE_HANDLER(WM_INITDIALOG, OnInitDialog)
			MESSAGE_HANDLER(WM_DESTROY, OnDestory)
			MESSAGE_HANDLER(WM_ERASEBKGND, OnEraseBkgnd)
			MESSAGE_HANDLER(WM_TIMER, OnTimer)
			COMMAND_ID_HANDLER(ID_APP_ABOUT, OnAppAbout)
			COMMAND_ID_HANDLER(IDOK, OnOK)
			COMMAND_ID_HANDLER(IDCANCEL, OnCancel)
		END_MSG_MAP()

	// Handler prototypes (uncomment arguments if needed):
	//	LRESULT MessageHandler(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
	//	LRESULT CommandHandler(WORD /*wNotifyCode*/, WORD /*wID*/, HWND /*hWndCtl*/, BOOL& /*bHandled*/)
	//	LRESULT NotifyHandler(int /*idCtrl*/, LPNMHDR /*pnmh*/, BOOL& /*bHandled*/)

		LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
		{
			// center the dialog on the screen
			CenterWindow();

			// set icons
			HICON hIcon = (HICON)::LoadImage(_Module.GetResourceInstance(), MAKEINTRESOURCE(IDR_MAINFRAME), 
				IMAGE_ICON, ::GetSystemMetrics(SM_CXICON), ::GetSystemMetrics(SM_CYICON), LR_DEFAULTCOLOR);
			SetIcon(hIcon, TRUE);
			HICON hIconSmall = (HICON)::LoadImage(_Module.GetResourceInstance(), MAKEINTRESOURCE(IDR_MAINFRAME), 
				IMAGE_ICON, ::GetSystemMetrics(SM_CXSMICON), ::GetSystemMetrics(SM_CYSMICON), LR_DEFAULTCOLOR);
			SetIcon(hIconSmall, FALSE);

			Gdiplus::GdiplusStartupInput StartupInput;  
			GdiplusStartup(&m_gdiplusToken,&StartupInput,NULL);  

			m_image_ = new Gdiplus::Image(_T("C:/Users/qiuji_000/Desktop/xxx/1.gif"));
			if(m_image_->GetLastStatus() != Gdiplus::Ok)
			{
				m_image_ = NULL;
				return FALSE;
			}
			m_size_.SetSize(m_image_->GetWidth(),m_image_->GetHeight());

			UINT count = m_image_->GetFrameDimensionsCount();
			m_pguid_=(GUID*)new GUID[count];
			m_image_->GetFrameDimensionsList(&m_pguid_[0],count);
			WCHAR strGuid[MAX_PATH];
			StringFromGUID2(m_pguid_[0],strGuid,MAX_PATH);
			m_img_cnt_=m_image_->GetFrameCount(&m_pguid_[0]);

			m_img_interval_ = m_image_->GetPropertyItemSize(PropertyTagFrameDelay);
			m_cur_ = 0;

			m_pitem_=(Gdiplus::PropertyItem*)new BYTE[m_img_interval_];
			m_image_->GetPropertyItem(PropertyTagFrameDelay,m_img_interval_,m_pitem_);

			// 设置属性WS_EX_LAYERED
			LONG lWindowLong = ::GetWindowLong(this->m_hWnd, GWL_EXSTYLE) | WS_EX_LAYERED;
			::SetWindowLong(this->m_hWnd, GWL_EXSTYLE, lWindowLong);
		
			SetTimer(TimerID,this->m_img_interval_,NULL);
			this->Paint();
			return TRUE;
		}

		void	Paint()
		{
			CClientDC dc_(this->m_hWnd);
			CDC mem_dc_;
			mem_dc_.CreateCompatibleDC(dc_);
			CBitmap mem_bitmap_;
	#if 0
			mem_bitmap_.CreateCompatibleBitmap(dc_,
				m_size_.cx,
				m_size_.cy);
	#else
			BITMAPINFOHEADER bmih;
			bmih.biSize                  = sizeof (BITMAPINFOHEADER) ;
			bmih.biWidth                 = m_size_.cx ;
			bmih.biHeight                = m_size_.cy ;
			bmih.biPlanes                = 1 ;
			bmih.biBitCount              = 32 ;  //注意32位
			bmih.biCompression           = BI_RGB ;
			bmih.biSizeImage             = 0 ;
			bmih.biXPelsPerMeter         = 0 ;
			bmih.biYPelsPerMeter         = 0 ;
			bmih.biClrUsed               = 0 ;
			bmih.biClrImportant          = 0 ;
			mem_bitmap_.CreateDIBitmap(dc_,&bmih,0,NULL,NULL,0);
	#endif
			mem_dc_.SelectBitmap(mem_bitmap_);

			// 刷图片
			m_image_->SelectActiveFrame(&m_pguid_[0],m_cur_);
			Gdiplus::Graphics g(mem_dc_);
			g.DrawImage(m_image_,0,0);

			BLENDFUNCTION pb_;
			pb_.AlphaFormat = 1;
			pb_.BlendOp = 0;
			pb_.BlendFlags =0;
			pb_.SourceConstantAlpha = 0xFF;

			CRect rect_;
			this->GetWindowRect(rect_);
			CPoint topleft_(rect_.TopLeft());

			CPoint  pt_(0,0);
			::UpdateLayeredWindow(this->m_hWnd,dc_,&topleft_,&m_size_,mem_dc_,&pt_,0,&pb_,ULW_ALPHA );

		}
	
		LRESULT OnEraseBkgnd(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
		{
			return TRUE;
		}
		LRESULT OnTimer(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
		{
			m_cur_ ++;
			if(m_cur_ >= m_img_cnt_)
				m_cur_ = 0;
			this->Paint();
			return 0;
		}
	
		LRESULT OnDestory(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
		{
			delete[] m_pguid_;
			delete m_image_;
			delete[] (LPBYTE)m_pitem_;

			Gdiplus::GdiplusShutdown(m_gdiplusToken);  
			return 0;
		}

		LRESULT OnAppAbout(WORD /*wNotifyCode*/, WORD /*wID*/, HWND /*hWndCtl*/, BOOL& /*bHandled*/)
		{
			CSimpleDialog<IDD_ABOUTBOX, FALSE> dlg;
			dlg.DoModal();
			return 0;
		}

		LRESULT OnOK(WORD /*wNotifyCode*/, WORD wID, HWND /*hWndCtl*/, BOOL& /*bHandled*/)
		{
			// TODO: Add validation code 
			EndDialog(wID);
			return 0;
		}

		LRESULT OnCancel(WORD /*wNotifyCode*/, WORD wID, HWND /*hWndCtl*/, BOOL& /*bHandled*/)
		{
			EndDialog(wID);
			return 0;
		}

	private:
		ULONG_PTR m_gdiplusToken;  
		//CImage    m_image_;
		Gdiplus::Image*   m_image_;
		CSize			m_size_;
		GUID * m_pguid_;
		Gdiplus::PropertyItem *m_pitem_;
		UINT m_img_cnt_;
		int m_img_interval_;
		int m_cur_;
	};

	/////////////////////////////////////////////////////////////////////////////

	//{{AFX_INSERT_LOCATION}}
	// VisualFC AppWizard will insert additional declarations immediately before the previous line.

	#endif // !defined(VFC_MAINDLG_H__9E8ABA70_0239_4836_AFDF_F4C7840A7648__INCLUDED_)


##参考
1. <http://blog.csdn.net/justin_bkdrong/article/details/5935138>