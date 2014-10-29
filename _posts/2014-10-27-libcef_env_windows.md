---
layout: post
title: Windows下libcef 环境搭建
category: cpp
---

##下载&编译

首先从<http://cefbuilds.com/>下载预编译好的二进制包，例如 **cef_binary_3.2171.1883_windows32.7z**

解开后目录结构如下

	.
	├── cefclient
	│   ├── binding_test.cpp
	│   ├── ... ...
	├── cefclient2008.sln
	├── cefclient.vcproj
	├── cefsimple
	│   ├── cefsimple.exe.manifest
	│   ├── ... ...
	├── cefsimple.vcproj
	├── Debug
	│   ├── cef_sandbox.lib
	│   ├── d3dcompiler_43.dll
	│   ├── d3dcompiler_46.dll
	│   ├── ffmpegsumo.dll
	│   ├── libcef.dll
	│   ├── libcef.lib
	│   ├── libEGL.dll
	│   ├── libGLESv2.dll
	│   ├── pdf.dll
	│   └── wow_helper.exe
	├── include
	│   ├── base
	│   ├── ... ...
	├── libcef_dll
	│   ├── base
	│   ├── ... ...
	├── libcef_dll_wrapper.vcproj
	├── Release
	│   ├── cef_sandbox.lib
	│   ├── d3dcompiler_43.dll
	│   ├── d3dcompiler_46.dll
	│   ├── ffmpegsumo.dll
	│   ├── libcef.dll
	│   ├── libcef.lib
	│   ├── libEGL.dll
	│   ├── libGLESv2.dll
	│   ├── pdf.dll
	│   └── wow_helper.exe
	└── Resources
	    ├── cef_100_percent.pak
	    ├── cef_200_percent.pak
	    ├── cef.pak
	    ├── devtools_resources.pak
	    ├── icudtl.dat
	    └── locales

里面包含三个工程

1. libcef_dll_wrapper 静态库
1. cefsimple 简单的测试程序
1. cefclient 功能覆盖相对完整的测试程序


由于libcef_wrapper这个库还是需要编译，所以仍然需要打开对应的slu文件编译，生成的**.lib**文件在**/out/Debug/lib**或者**/out/Release/lib**目录

如果只是简单的学习，可以直接用cefsimple/cefclient来学习，下面来说明下如何利用现有的头文件和库文件创建全新的工程

**我用vs2008/vs2010都无法编译出cefsimple/cefclient，后台禁用sandbox成功，具体修改如下代码为0**

	#define CEF_ENABLE_SANDBOX 0

##新工程

1. 创建工程
1. 拷贝libcef压缩包中的include目录到工程目录
1. 根据Debug/Release拷贝压缩包中的Debug/Release目录到工程目录
1. 工程使用unicode
1. 运行库使用MT/MTd
1. 拷贝cefsimpe的所有源码
1. 从cefsimple工程拷贝宏配置，见后面，一些默认的根据需要修改
1. 从cefsimple工程拷贝库配置，见后面，一些库目录根据需要修改
1. 拷贝libcef压缩包中的Resources目录到工程目录
1. 拷贝编译生成的libcef_wrapper.lib到工程目录，并引用之

---

**最终运行时，依赖的dll，Resources内部的文件，以及生成的exe必须在同一个目录下**

---

	_DEBUG
	V8_DEPRECATION_WARNINGS
	BLINK_SCALE_FILTERS_AT_RECORD_TIME
	_WIN32_WINNT=0x0602
	WINVER=0x0602
	WIN32
	_WINDOWS
	NOMINMAX
	PSAPI_VERSION=1
	_CRT_RAND_S
	CERT_CHAIN_PARA_HAS_EXTRA_FIELDS
	WIN32_LEAN_AND_MEAN
	_ATL_NO_OPENGL
	_HAS_EXCEPTIONS=0
	_SECURE_ATL
	CHROMIUM_BUILD
	TOOLKIT_VIEWS=1
	USE_AURA=1
	USE_ASH=1
	USE_DEFAULT_RENDER_THEME=1
	USE_LIBJPEG_TURBO=1
	USE_MOJO=1
	ENABLE_ONE_CLICK_SIGNIN
	ENABLE_REMOTING=1
	ENABLE_WEBRTC=1
	ENABLE_PEPPER_CDMS
	ENABLE_CONFIGURATION_POLICY
	ENABLE_INPUT_SPEECH
	ENABLE_NOTIFICATIONS
	ENABLE_HIDPI=1
	ENABLE_EGLIMAGE=1
	__STD_C
	_CRT_SECURE_NO_DEPRECATE
	_SCL_SECURE_NO_DEPRECATE
	NTDDI_VERSION=0x06020000
	_USING_V110_SDK71_
	ENABLE_TASK_MANAGER=1
	ENABLE_EXTENSIONS=1
	ENABLE_PLUGIN_INSTALLATION=1
	ENABLE_PLUGINS=1
	ENABLE_SESSION_SERVICE=1
	ENABLE_THEMES=1
	ENABLE_AUTOFILL_DIALOG=1
	ENABLE_BACKGROUND=1
	ENABLE_AUTOMATION=1
	ENABLE_GOOGLE_NOW=1
	CLD_VERSION=2
	ENABLE_FULL_PRINTING=1
	ENABLE_PRINTING=1
	ENABLE_SPELLCHECK=1
	ENABLE_CAPTIVE_PORTAL_DETECTION=1
	ENABLE_APP_LIST=1
	ENABLE_SETTINGS_APP=1
	ENABLE_MANAGED_USERS=1
	ENABLE_MDNS=1
	ENABLE_SERVICE_DISCOVERY=1
	USING_CEF_SHARED
	__STDC_CONSTANT_MACROS
	__STDC_FORMAT_MACROS
	DYNAMIC_ANNOTATIONS_ENABLED=1
	WTF_USE_DYNAMIC_ANNOTATIONS=1

---

	wininet.lib
	dnsapi.lib
	version.lib
	msimg32.lib
	ws2_32.lib
	usp10.lib
	psapi.lib
	dbghelp.lib
	winmm.lib
	shlwapi.lib
	kernel32.lib
	gdi32.lib
	winspool.lib
	comdlg32.lib
	advapi32.lib
	shell32.lib
	ole32.lib
	oleaut32.lib
	user32.lib
	uuid.lib
	odbc32.lib
	odbccp32.lib
	delayimp.lib
	credui.lib
	netapi32.lib
	comctl32.lib
	rpcrt4.lib
	opengl32.lib
	glu32.lib
	libcef_dll_wrapper.lib
	cef_sandbox.lib
	libcef.lib

---

##浏览器窗口大小

默认情况下，libcef会创建一个Windows窗口并且将浏览器至于整个客户区，默认的代码如下

	// Information used when creating the native window.
	CefWindowInfo window_info;
	
	#if defined(OS_WIN)
	// On Windows we need to specify certain flags that will be passed to
	// CreateWindowEx().
	// 传一个空hwnd，第二个是标题栏字符
	window_info.SetAsPopup(NULL, "cefsimple");
	#endif
	
	// Create the first browser window.
	CefBrowserHost::CreateBrowser(window_info, handler.get(), url,
			browser_settings, NULL);
			
若希望定制外层的窗口，例如去掉这个默认的窗口，例如让浏览器占据整个窗口

      CefWindowInfo info;

      // 手动创建一个Windows窗口，hwnd为hWnd变量
      // rect为占据的区域，不一定是整个客户区
      // Initialize window info to the defaults for a child window.
      info.SetAsChild(hWnd, rect);

      // Creat the new child browser window
      CefBrowserHost::CreateBrowser(info, g_handler.get(),
          g_handler->GetStartupURL(), settings, NULL);



##参考
1. <http://www.cnblogs.com/liulun/p/3681241.html>

