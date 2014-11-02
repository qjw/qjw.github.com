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
	
##参考
1. <http://www.iwebrtc.com/blog/chromium-ebedded-framework-javascript-integration/>

