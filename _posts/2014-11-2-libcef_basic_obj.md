---
layout: post
title: Libcef 基本对象
category: cpp
---

##CefApp

Libcef需要一个CefApp实例，通常为了监听某些事件，还需要继承其他的一些process Handler

1. CefBrowserProcessHandler
1. CefRenderProcessHandler

	class SimpleApp: public CefApp,
		public CefBrowserProcessHandler,
		public CefRenderProcessHandler{

	// Entry point function for all processes.
	int main(int argc, char* argv[]) {
		// SimpleApp implements application-level callbacks. It will create the first
		// browser instance in OnContextInitialized() after CEF has initialized.
		CefRefPtr<SimpleApp> app(new SimpleApp);
	
		// CEF applications have multiple sub-processes (render, plugin, GPU, etc)
		// that share the same executable. This function checks the command-line and,
		// if this is a sub-process, executes the appropriate logic.
		int exit_code = CefExecuteProcess(main_args, app.get(), NULL);
	
		// Initialize CEF for the browser process.
		CefInitialize(main_args, settings, app.get(), NULL);

在CefApp中，声明了一些获取对象的接口。他们都返回空。这里需要注意，**继承的那些Handler的消息，是以来于CefApp中获取特殊对象的接口实现，而不是简单地返回一个NULL**

	///
	// Return the handler for resource bundle events. If
	// CefSettings.pack_loading_disabled is true a handler must be returned. If no
	// handler is returned resources will be loaded from pack files. This method
	// is called by the browser and render processes on multiple threads.
	///
	/*--cef()--*/
	virtual CefRefPtr<CefResourceBundleHandler> GetResourceBundleHandler()
	{
		return NULL;
	}

	///
	// Return the handler for functionality specific to the browser process. This
	// method is called on multiple threads in the browser process.
	///
	/*--cef()--*/
	virtual CefRefPtr<CefBrowserProcessHandler> GetBrowserProcessHandler()
	{
		return NULL;
	}

	///
	// Return the handler for functionality specific to the render process. This
	// method is called on the render process main thread.
	///
	/*--cef()--*/
	virtual CefRefPtr<CefRenderProcessHandler> GetRenderProcessHandler()
	{
		return NULL;
	}
	
##CefClient

创建浏览器对象时需传入一个CefClient，通常需要继承一些其他对象来监听消息
	

	void SimpleApp::OnContextInitialized()
	{
		// SimpleHandler implements browser-level callbacks.
		CefRefPtr<SimpleHandler> handler(new SimpleHandler());
	
		// Create the first browser window.
		CefBrowserHost::CreateBrowser(window_info, handler.get(), url,
				browser_settings, NULL);
	}	
	
	class SimpleHandler : public CefClient,
		public CefDisplayHandler,
		public CefLifeSpanHandler,
		public CefLoadHandler {	
		
###更新标题栏

通常都实现了改函数，因为桌面系统的任务栏文字和标题栏的文字是一致的（*通常使用cef时都不会保留标题栏*）

	///
	// Implement this interface to handle events related to browser display state.
	// The methods of this class will be called on the UI thread.
	///
	/*--cef(source=client)--*/
	class CefDisplayHandler: public virtual CefBase
	{
	public:

		///
		// Called when the page title changes.
		///
		/*--cef(optional_param=title)--*/
		virtual void OnTitleChange(CefRefPtr<CefBrowser> browser,
				const CefString& title)
		{
		}
		
1. CefContextMenuHandler，回调类，主要用于处理 Context Menu 事件。
1. CefDialogHandler，回调类，主要用来处理对话框事件。
1. CefDisplayHandler，回调类，处理与页面状态相关的事件，如页面加载情况的变化，地址栏变化，标题变化等事件。
1. CefDownloadHandler，回调类，主要用来处理文件下载。
1. CefFocusHandler，回调类，主要用来处理焦点事件。
1. CefGeolocationHandler，回调类，用于申请 geolocation 权限。
1. CefJSDialogHandler，回调类，主要用来处理 JS 对话框事件。
1. CefKeyboardHandler，回调类，主要用来处理键盘输入事件。
1. CefLifeSpanHandler，回调类，主要用来处理与浏览器生命周期相关的事件，与浏览器对象的创建、销毁以及弹出框的管理。
1. CefLoadHandler，回调类，主要用来处理浏览器页面加载状态的变化，如页面加载开始，完成，出错等。
1. CefRenderHandler，回调类，主要用来处在在窗口渲染功能被关闭的情况下的事件。
1. CefRequestHandler，回调类，主要用来处理与浏览器请求相关的的事件，如资源的的加载，重定向等。

##CefBrowser && CefFrame

通常我们在CefLifeSpanHandler的**OnAfterCreated**和**OnBeforeClose**消息中保留CefBrowser的指针。

每个浏览器都有一个CefFrame对象，以及若干子Frame

The CefBrowser and CefFrame objects are used for sending commands to the browser and for retrieving state information in callback methods. Each CefBrowser object will have a single main CefFrame object representing the top-level frame and zero or more CefFrame objects representing sub-frames. For example, a browser that loads two iframes will have three CefFrame objects (the top-level frame and the two iframes).

---

	void ClientHandler::OnAfterCreated(CefRefPtr<CefBrowser> browser)
	{
		// 保存 browser 指针
	}

	void ClientHandler::OnBeforeClose(CefRefPtr<CefBrowser> browser)
	{
		// 释放 browser 指针
	}

	///
	// Class used to represent a browser window. When used in the browser process
	// the methods of this class may be called on any thread unless otherwise
	// indicated in the comments. When used in the render process the methods of
	// this class may only be called on the main thread.
	///
	/*--cef(source=library)--*/
	class CefBrowser: public virtual CefBase
	{
		///
		// Returns the main (top-level) frame for the browser window.
		///
		/*--cef()--*/
		virtual CefRefPtr<CefFrame> GetMainFrame() =0;
		
		///
		// Returns the browser host object. This method can only be called in the
		// browser process.
		///
		/*--cef()--*/
		virtual CefRefPtr<CefBrowserHost> GetHost() =0;
	};

####操作浏览器行为

	///
	// Class used to represent a browser window. When used in the browser process
	// the methods of this class may be called on any thread unless otherwise
	// indicated in the comments. When used in the render process the methods of
	// this class may only be called on the main thread.
	///
	/*--cef(source=library)--*/
	class CefBrowser: public virtual CefBase
	{
		///
		// Returns true if the browser can navigate backwards.
		///
		/*--cef()--*/
		virtual bool CanGoBack() =0;
	
		///
		// Navigate backwards.
		///
		/*--cef()--*/
		virtual void GoBack() =0;
	
		///
		// Returns true if the browser can navigate forwards.
		///
		/*--cef()--*/
		virtual bool CanGoForward() =0;
	
		///
		// Navigate forwards.
		///
		/*--cef()--*/
		virtual void GoForward() =0;
	
		///
		// Returns true if the browser is currently loading.
		///
		/*--cef()--*/
		virtual bool IsLoading() =0;
	
		///
		// Reload the current page.
		///
		/*--cef()--*/
		virtual void Reload() =0;
	
		///
		// Reload the current page ignoring any cached data.
		///
		/*--cef()--*/
		virtual void ReloadIgnoreCache() =0;
	
		///
		// Stop loading the page.
		///
		/*--cef()--*/
		virtual void StopLoad() =0;
	
		///
		// Returns the globally unique identifier for this browser.
		///
		/*--cef()--*/
		virtual int GetIdentifier() =0;
	
		// ... ...
	};


####获得窗口原生句柄

**CefBrowserHost::GetWindowHandle**

	///
	// Class used to represent a browser window. When used in the browser process
	// the methods of this class may be called on any thread unless otherwise
	// indicated in the comments. When used in the render process the methods of
	// this class may only be called on the main thread.
	///
	/*--cef(source=library)--*/
	class CefBrowser: public virtual CefBase
	{	
		///
		// Returns the browser host object. This method can only be called in the
		// browser process.
		///
		/*--cef()--*/
		virtual CefRefPtr<CefBrowserHost> GetHost() =0;
	};

	///
	// Class used to represent the browser process aspects of a browser window. The
	// methods of this class can only be called in the browser process. They may be
	// called on any thread in that process unless otherwise indicated in the
	// comments.
	///
	/*--cef(source=library)--*/
	class CefBrowserHost : public virtual CefBase 
	{
		///
		// Retrieve the window handle for this browser.
		///
		/*--cef()--*/
		virtual CefWindowHandle GetWindowHandle() =0;
	}


##参考
1. <https://code.google.com/p/chromiumembedded/wiki/GeneralUsage>
1. <https://code.google.com/p/chromiumembedded/wiki/JavaScriptIntegration>

