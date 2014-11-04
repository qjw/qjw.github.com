---
layout: post
title: Libcef CPP导出函数给js
category: cpp
---

##注册回调
为了导出函数，我们需要重新实现**GetRenderProcessHandler**函数，以便接收**OnContextCreated**消息。


	class SimpleApp: public CefApp,
			public CefBrowserProcessHandler,
			public CefRenderProcessHandler{
	public:
		SimpleApp();
		virtual CefRefPtr<CefRenderProcessHandler> GetRenderProcessHandler() OVERRIDE {
			return this;
		}
	
		virtual void OnContextCreated(CefRefPtr<CefBrowser> browser,
			CefRefPtr<CefFrame> frame, CefRefPtr<CefV8Context> context) OVERRIDE;
	}
	

	void SimpleApp::OnContextCreated(CefRefPtr<CefBrowser> browser,
			CefRefPtr<CefFrame> frame, CefRefPtr<CefV8Context> context)
	{
		// Retrieve the context's window object.
		CefRefPtr<CefV8Value> object = context->GetGlobal();

		// Create an instance of my CefV8Handler object.
		CefRefPtr<CefV8Handler> handler = new MyV8Handler();

		// Create the "myfunc" function.
		CefRefPtr<CefV8Value> func = CefV8Value::CreateFunction("myfunc", handler);

		// Add the "myfunc" function to the "window" object.
		object->SetValue("myfunc", func, V8_PROPERTY_ATTRIBUTE_NONE);
	}
	
在注册的CefV8Handler对象的Execute函数中，第三个函数表示js传过来的参数集合。

CefV8Value是个通用对象，既可以表示整形，字符串，也可以表示函数，对象等复杂结构

若是个函数，我们可以调用**ExecuteFunction**函数来执行这个js函数。不过同样需要保留第二个参数object已标示哪个会话

	class MyV8Handler: public CefV8Handler
	{
	public:
		MyV8Handler()
		{
		}

		virtual bool Execute(const CefString& name,
				CefRefPtr<CefV8Value> object,
				const CefV8ValueList& arguments,
				CefRefPtr<CefV8Value>& retval,
				CefString& exception) OVERRIDE
		{
			if (name == "myfunc")
			{

				std::ostringstream strbuf_;
				strbuf_ << name.ToString() << " | " << arguments.size() << " | "
						<< object->GetStringValue().ToString() << " | ";
				size_t size_ = arguments.size() - 1;
				for (size_t i = 0; i < size_; i++)
				{
					CefRefPtr<CefV8Value> tmparg_ = arguments[i];
					strbuf_ << tmparg_->GetStringValue().ToString() << " | ";
				}
				strbuf_ << "omg,fuck you";

				// Return my string value.
				retval = CefV8Value::CreateString(strbuf_.str().c_str());

				// 最后执行最后一个参数传入的函数对象
				CefRefPtr<CefV8Value> tmparg_ = arguments[arguments.size() - 1];
				CefV8ValueList arglist_;
				tmparg_->ExecuteFunction(object,arglist_);
				return true;
			}

			// Function does not exist.
			return false;
		}

		// Provide the reference counting implementation for this class.
		IMPLEMENT_REFCOUNTING(MyV8Handler)
		;
	};
	
---

	<script type="text/javascript" src="jquery-2.1.1.min.js"></script>
	<script type="text/javascript">
		$(function(){
			//alert("fuck1");
			$("#fuck").click(function(){
				$("#fuck").text(window.myfunc("a","b","c",function(){
					$("#fuck2").text("fuck you");
				}));
			});
		});
	</script>
	
##异步回调

在下面的js中，我们传入两个回调函数（假设需求是搜索的音乐文件，第一个回调用于刷新文件列表，第二个用于通知结束）。在调用后立即返回，而c++的代码再异步执行扫描并回调传入的js函数。

	$(function(){
	    $("#submit").click(function(){
		$("#status").text("process");
		var index=0
		window.myfunc(function(path){
		    var str = "<li>" + index + ":" + path + "</li>";
		    index++;
		    $("#files").children().last().append(str);
		},
		function(){
		    $("#status").text("");
		});
	    });
	});
	
接前面的实现，我们新增一个后台线程类

	class MyV8Handler;
	class Test
	{
	public:
		void test()
		{
			// 最后执行最后一个参数传入的函数对象
			for(int i = 0;i<10;i++)
			{
				CefString path_;
				path_.FromString("fuck");
				CefPostTask(TID_RENDERER, NewCefRunnableMethod(
					m_ptr_,
					&MyV8Handler::Updatefile,
					context,
					object,
					m_path_func_,
					path_));
				sleep(1);
			}
			{
				CefPostTask(TID_RENDERER, NewCefRunnableMethod(
					m_ptr_,
					&MyV8Handler::UpdatefileFini,
					context,
					object,
					m_fin_func_));
			}
		}

		static void* Thread(void* arg)
		{
			Test* ptr_ = (Test*)(arg);
			ptr_->test();
			return NULL;
		}

		MyV8Handler* m_ptr_;
		CefRefPtr<CefV8Context> context;
		CefRefPtr<CefV8Value> object;
		CefRefPtr<CefV8Value> m_path_func_;
		CefRefPtr<CefV8Value> m_fin_func_;
	};

如果是直接调用，我们可以不理会**CefV8Context**，异步时必须保留并使用它来调用**ExecuteFunctionWithContext**

	class MyV8Handler: public CefV8Handler
	{
	public:
		MyV8Handler(CefRefPtr<CefBrowser> browser):
			m_browser_(browser)
		{
		}

		void Updatefile(
				CefRefPtr<CefV8Context> context,
				CefRefPtr<CefV8Value> object,
				CefRefPtr<CefV8Value> func,
				const CefString& path)
		{
			CefRefPtr<CefV8Value> arg1_ = CefV8Value::CreateString(path);

			CefV8ValueList arguments;
			arguments.push_back(arg1_);

			func->ExecuteFunctionWithContext(context,object,arguments);
		}
		void UpdatefileFini(
				CefRefPtr<CefV8Context> context,
				CefRefPtr<CefV8Value> object,
				CefRefPtr<CefV8Value> func)
		{
			CefV8ValueList arguments;
			func->ExecuteFunctionWithContext(context,object,arguments);
		}
	
		virtual bool Execute(const CefString& name,
				CefRefPtr<CefV8Value> object,
				const CefV8ValueList& arguments,
				CefRefPtr<CefV8Value>& retval,
				CefString& exception) OVERRIDE
		{
			if (name == "myfunc")
			{
				pthread_t thread_;
				Test* test_ = new Test();
				test_->object = object;
				test_->m_ptr_ = this;
				test_->m_path_func_ = arguments[0];
				test_->m_fin_func_ = arguments[1];
				test_->context = CefV8Context::GetCurrentContext();
				pthread_create(&thread_,NULL,Test::Thread,(void*)test_);

				// Return my string value.
				retval = CefV8Value::CreateString("fuck");
		
				return true;
			}

			// Function does not exist.
			return false;
		}
		
####线程

我们从后台线程给主线程发送任务时，注意是**TID_RENDERER**，因为以上的逻辑都在Render进程中运行，若需要通知Browser进程处理，需要通过消息通知（**IPC**）。

1. TID_UI thread is the main thread in the browser process. This will be the same as the main application thread if CefInitialize() is called with a CefSettings.multi_threaded_message_loop value of false.
1. TID_IO thread is used in the browser process to process IPC and network messages.
1. TID_FILE thread is used in the browser process to interact with the file system.
1. TID_RENDERER thread is the main thread in the renderer process.

##多进程IPC

libcef提供了一种机制，用于在渲染进程(Render Process)和浏览器进程（Browser Process)之间通信，这常见于要控制原生窗口的行为，**例如在Web上实现标题栏拖动，最大化，最小化等**

A message sent from the browser process to the render process will arrive in **CefRenderProcessHandler::OnProcessMessageReceived()**. A message sent from the render process to the browser process will arrive in **CefClient::OnProcessMessageReceived()**.

Use **PID_BROWSER** instead when sending a message to the browser process.


	// Create the message object.
	CefRefPtr<CefProcessMessage> msg= CefProcessMessage::Create("my_message");

	// Retrieve the argument list object.
	CefRefPtr<CefListValue> args = msg->GetArgumentList();

	// Populate the argument values.
	args->SetString(0, "my string");
	args->SetInt(0, 10);

	// Send the process message to the render process.
	// Use PID_BROWSER instead when sending a message to the browser process.
	m_browser_->SendProcessMessage(PID_BROWSER, msg);
	
---

	class SimpleHandler: public CefClient,
			public CefDisplayHandler,
			public CefLifeSpanHandler,
			public CefLoadHandler
	{

		bool OnProcessMessageReceived(
				CefRefPtr<CefBrowser> browser,
				CefProcessId source_process,
				CefRefPtr<CefProcessMessage> message)
		{
			const std::string& message_name = message->GetName();
			if (message_name == "my_message")
			{
			}
		}
	}
	
---

	///
	// Implement this interface to provide handler implementations.
	///
	/*--cef(source=client,no_debugct_check)--*/
	class CefClient: public virtual CefBase
	{
		///
		// Called when a new message is received from a different process. Return true
		// if the message was handled or false otherwise. Do not keep a reference to
		// or attempt to access the message outside of this callback.
		///
		/*--cef()--*/
		virtual bool OnProcessMessageReceived(CefRefPtr<CefBrowser> browser,
				CefProcessId source_process, CefRefPtr<CefProcessMessage> message)
		{
			return false;
		}
	};

	///
	// Class used to implement render process callbacks. The methods of this class
	// will be called on the render process main thread (TID_RENDERER) unless
	// otherwise indicated.
	///
	/*--cef(source=client)--*/
	class CefRenderProcessHandler: public virtual CefBase
	{
		///
		// Called when a new message is received from a different process. Return true
		// if the message was handled or false otherwise. Do not keep a reference to
		// or attempt to access the message outside of this callback.
		///
		/*--cef()--*/
		virtual bool OnProcessMessageReceived(CefRefPtr<CefBrowser> browser,
				CefProcessId source_process, CefRefPtr<CefProcessMessage> message)
		{
			return false;
		}
	};
	
	
在<https://code.google.com/p/chromiumembedded/wiki/GeneralUsage#Asynchronous_Bindings>提到一种message_router的IPC(**Generic Message Router**)，个人感觉没啥用


Starting with trunk revision 1574 CEF provides a generic implementation for routing asynchronous messages between JavaScript running in the renderer process and C++ running in the browser process. An application interacts with the router by passing it data from standard CEF C++ callbacks (OnBeforeBrowse, OnProcessMessageRecieved, OnContextCreated, etc). The renderer-side router supports generic JavaScript callback registration and execution while the browser-side router supports application-specific logic via one or more application-provided Handler instances.
	
####强制单进程

1. 在运行程序加入命令行**--single-process**
2. 在程序中设置CefSettings的single_process属性

CEF3 supports a single-process run mode for debugging purposes via the "--single-process" command-line flag. Platform-specific debugging tips are also available for Windows, Mac OS X and Linux.

	// Specify CEF global settings here.
	CefSettings settings;
	settings.single_process = 1;


	
##参考
1. <http://www.iwebrtc.com/blog/chromium-ebedded-framework-javascript-integration/>
1. <https://code.google.com/p/chromiumembedded/wiki/GeneralUsage>
1. <https://code.google.com/p/chromiumembedded/wiki/JavaScriptIntegration>

