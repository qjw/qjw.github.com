---
layout: post
title: Chrome源码学习笔记
category: cpp
---

写于 **2009-11-17**

学习 Chrome 源码也有一小段时间了,对该浏览器的 UI 部分小有了解,于是将自己的一些心得贴上来,希望对有需要的朋友有所帮助。

说明:

1. 本人所使用的 chrome 源码比较早,所以下载最新的代码多少会和文中有些差异。
2. 代码在 windows 下使用 visual studio 2008 编译。
3. 源码下载地址 <http://code.google.com/chromium/>

Ui 部分的通用 1 代码主要在目录树的“src/chrome/views”目录下。其中

1. controls 目录封装用到的控件,例如 Label、Textfield 等
2. widget 目录主要封装系统相关的 UI 底层细节,特别是 UI 消息机制。
3. window 目录主要封装 UI Frame 相关的细节,例如窗口的标题栏、系统按钮以及Frame、Dialog 等的代理 接口。

如果想了解 Chrome 绘图技术,可以去 Skia 项目<http://code.google.com/p/skia> 看看

##一个简单的程序

	//test.h

	#include "chrome/views/view.h"
	#include "chrome/views/window/window_delegate.h"

	class TestWindow: public views::View, public views::WindowDelegate {
	public:

		virtual View* GetContentsView() {
			return this;
		}

		virtual gfx::Size GetPreferredSize() {
			return gfx::Size(400, 300);
		}

		static void CreateTestWindow();

	};
	////////////////////////////////////////////////////////

	//test.cpp

	#include "test.h"
	#include "chrome/views/window/window.h"

	void TestWindow::CreateTestWindow() {
		views::Window::CreateChromeWindow(NULL, gfx::Rect(), new TestWindow)->Show();
	}
	
###源码分析

1. TestWindow 需要继承自两个类 views::View 和views::WindowDelegate。
1. 在chrome中，任何控件（这里忽略操作系统原生控件的某些特殊情况）包括一个Frame都必须是一个View。
1. 一个顶层窗口（例如Frame、Dialog（Dialog继承自DialogDelegate，该代理继承自 WindowDelegate））必须继承自views::WindowDelegate以便传递该窗口的一些参数,例如窗口的尺寸。
1. GetPreferredSize()含义很明显是告诉系统初始窗口大小（确切的说是窗口内部的根View的大小，实际窗口大小要大于该值）为（400*300），这里其实可选。
1. GetContentsView()函数将自己作为一个窗口的根View传给所附属的Frame（或Dialog）。
1. 中间的黑色是因为客户区没有任何东西，所以背景贴图和背景贴图未覆盖的地方被显示出来。
1. 内部的白色框是因为系统默认会在客户区画白色的边框。

为了方面其他代码调用，这里提供一个静态函数CreateTestWindow()打开该窗口

##UI消息机制（针对windows平台）

Windows 每一个控件都单独处理消息，chrome一个整的的窗口可以看做一个(这里忽略原生空间的封装)（Windows）控件，所以所有控件的消息都会发到一块去，所以chrome Ui框架有一套机制来分发到具体的（Chrome）控件。

Windows版本的chrome的UI部分基于WTL<http://sourceforge.net/projects/wtl/>。chrome通过在此处(src\chrome\views\widget\widget_win.h)中的绑定来获取windows UI消息。接着Chrome内置的一套消息分发机制将该消息发送到具体的chrome 控件。

##Chrome控件树

前面提到，chrome每一个控件都是一个views::View。views::View可以认为是chrome中所有控件的基类。在这个类中定义了一个通用的操作和默认实现，例如鼠标单击处理函数，如果某控件需要自定义鼠标处理事件，可以在自己的类中覆盖基类的默认实现。

每一个views::View可以包含子views::View。所以每一个都包含一个父views::View和若干子views::View的指针，下面是源码(src\chrome\views\view.h)的定义


	// This view's parent
	View *parent_;

	// This view's children.
	typedef std::vector<View*> ViewList;
	ViewList child_views_;

这棵树的根节点便是views::RootView(src\chrome\views\widget\root_view.h)。这个View的主要用途就是从views::Widget(src\chrome\views\widget\widget.h)中接收UI消息。

这棵树的最初几层可以参考chrome源码(src\chrome\views\window\non_client_view.h)

	////////////////////////////////////////////////////////////////////////////////
	// NonClientView
	//
	// The NonClientView is the logical root of all Views contained within a
	// Window, except for the RootView which is its parent and of which it is the
	// sole child. The NonClientView has two children, the NonClientFrameView which
	// is responsible for painting and responding to events from the non-client
	// portions of the window, and the ClientView, which is responsible for the
	// same for the client area of the window:
	//
	// +- views::Window ------------------------------------+
	// | +- views::RootView ------------------------------+ |
	// | | +- views::NonClientView ---------------------+ | |
	// | | | +- views::NonClientView subclass ---+ | | |
	// | | | | | | | |
	// | | | | << all painting and event receiving >> | | | |
	// | | | | << of the non-client areas of a >> | | | |
	// | | | | << views::Window. >> | | | |
	// | | | | | | | |
	// | | | +----------------------------------------+ | | |
	// | | | +- views::ClientView or subclass --------+ | | |
	// | | | | | | | |
	// | | | | << all painting and event receiving >> | | | |
	// | | | | << of the client areas of a >> | | | |
	// | | | | << views::Window. >> | | | |
	// | | | | | | | |
	// | | | +----------------------------------------+ | | |
	// | | +--------------------------------------------+ | |
	// | +------------------------------------------------+ |
	// +----------------------------------------------------+
	//
	// The NonClientFrameView and ClientView are siblings because due to theme
	// changes the NonClientFrameView may be replaced with different
	// implementations (e.g. during the switch from DWM/Aero-Glass to Vista Basic/
	
###说明
1. views::Window 表示一个实际的窗口，不过它并不是一个views::View。
1. views::RootView 如前面所述，即整个控件树的根。
1. Views::NonClientView 表示整个实际的控件树的根，实际上views::RootView只有一颗子树Views::NonClientView ，所以个人认为这两个View可以合并在一块。也许是为了从逻辑上分开吧，即RootView向上和OS层打交道，而 NonClientView则专注于下层的控件View
1. views::NonClientView subclass 表示整个非客户区的View。例如窗口的标题栏、窗口图标、关闭按钮、最大化/最小化按钮、边框等就定义在此。实际上chrome默认使用views::NonClientView的子类CustomFrameView (src\chrome\views\window\custom_frame_view.h)。此外chrome还提供原生View的封装NativeFrameView（src\chrome\views\window\native_frame_view.h ，该类本人未作测试。）。
1. views::ClientView or subclass 是窗口客户区的根，上面test实例中的TestWindow 也是一个views::View。该View就是挂在【views::ClientView or subclass】的下面（作为它的子树）。具体的挂在通过函数GetContentsView() 实现。

##Chrome消息分发机制

下面以一个鼠标移入事件说明，views::View中定义了函数OnMouseEntered，当鼠标移入某控件时，该函数被调用。下面是相关的函数声明。

	// This method is invoked when the mouse enters this control.
	//
	// Default implementation does nothing. Override as needed.
	virtual void OnMouseEntered(const MouseEvent& event);
	
当views::RootView收到windows消息时，例如鼠标移动的消息，它便会根据鼠标的位置获取具体的View，然后调用该View的相关处理函数。

因为相关的处理函数都在views::View中有一份默认实现，所以不关系该事件的View可以不理会。而如果需要定制处理函数只需重载相关的处理函数即可。

Chrome查找具体的View通过 GetViewForPoint函数，递归调用。View首先查找所有子View，如果事件的位置在某个子View的区域内，则调用该子View的 GetViewForPoint函数。否则返回自己。下面是RootView分发“鼠标进入”事件的源码

	void RootView::OnMouseMoved(const MouseEvent& e) {
		View* v = GetViewForPoint(e.location());
		// Find the first enabled view.

		while (v && !v->IsEnabled()) {
			v = v->GetParent();
			if (v && v != this) {
				if (v != mouse_move_handler_) {
					if (mouse_move_handler_ != NULL) {
						MouseEvent exited_event(Event::ET_MOUSE_EXITED, 0, 0, 0);
						mouse_move_handler_->OnMouseExited(exited_event);
					}

					mouse_move_handler_ = v;

					MouseEvent entered_event(Event::ET_MOUSE_ENTERED, this,
							mouse_move_handler_, e.location(), 0);
					mouse_move_handler_->OnMouseEntered(entered_event);
				}
			}
		}
	}

##基本控件

前面提到过，chrome的基础控件在目录“ src/chrome/views/controls”目录下。

下面是chrome自带的控件一览

	//按钮
	D:\chrometrunk\chrometrunk\src\chrome\views\controls\button>ls
	button.cc custom_button.cc native_button.cc radio_button.h
	button.h custom_button.h native_button.h text_button.cc
	button_dropdown.cc image_button.cc native_button_win.cc text_button.h
	button_dropdown.h image_button.h native_button_win.h
	checkbox.cc menu_button.cc native_button_wrapper.h
	checkbox.h menu_button.h radio_button.cc

	//菜单
	D:\chrometrunk\chrometrunk\src\chrome\views\controls\menu>ls
	chrome_menu.cc controller.h menu.h
	chrome_menu.h menu.cc view_menu_delegate.h

	//滑动条
	D:\chrometrunk\chrometrunk\src\chrome\views\controls\scrollbar>ls
	bitmap_scroll_bar.cc native_scroll_bar.cc scroll_bar.cc
	bitmap_scroll_bar.h native_scroll_bar.h scroll_bar.h

	//表格
	D:\chrometrunk\chrometrunk\src\chrome\views\controls\table>ls
	group_table_view.cc table_view.cc table_view_unittest.cc
	group_table_view.h table_view.h

	//树
	D:\chrometrunk\chrometrunk\src\chrome\views\controls\tree>ls
	tree_model.h tree_node_iterator_unittest.cc tree_view.cc
	tree_node_iterator.h tree_node_model.h tree_view.h

	//其他控件
	D:\chrometrunk\chrometrunk\src\chrome\views\controls>ls
	button label_unittest.cc native_control_win.h tabbed_pane.h
	combo_box.cc link.cc scroll_view.cc table
	combo_box.h link.h scroll_view.h text_field.cc
	hwnd_view.cc menu scrollbar text_field.h
	hwnd_view.h message_box_view.cc separator.cc throbber.cc
	image_view.cc message_box_view.h separator.h throbber.h
	image_view.h native_control.cc single_split_view.cc tree
	label.cc native_control.h single_split_view.h
	label.h native_control_win.cc tabbed_pane.cc
	
尽管chrome提供丰富的控件，但是如果打算使用chrome这一套UI框架开发自己的程序，这些远远不够用，幸运的是基于chrome开发自定义控件相当的方便。
      	
接着上面的例子，下面的程序在客户区添加一个Button和Label。当点击按钮后，Label显示“点击了“。下面是源码：

	//test.h

	#include "chrome/views/view.h"
	#include "chrome/views/window/window_delegate.h"
	#include "chrome/views/controls/button/button.h"

	namespace views {
	class Label;
	class TextButton;
	}
	class ChromeCanvas;

	class TestWindow: public views::View,
			public views::WindowDelegate,
			public views::ButtonListener {
	public:

		TestWindow();

		virtual View* GetContentsView() {
			return this;
		}

		virtual gfx::Size GetPreferredSize() {
			return gfx::Size(400, 300);
		}

		virtual void Layout();
		virtual void Paint(ChromeCanvas* canvas);
		virtual void ButtonPressed(views::Button* sender);

		static void CreateTestWindow();

	private:

		views::Label * lable_;
		views::TextButton * button_;

	};

	//test.cpp

	#include "test.h"
	#include "chrome/views/window/window.h"
	#include "chrome/views/controls/button/text_button.h"
	#include "chrome/views/controls/label.h"
	#include "chrome/common/gfx/chrome_canvas.h"

	TestWindow::TestWindow() {
		this->lable_ = new views::Label(L"");
		this->button_ = new views::TextButton(this, L"点击");
		AddChildView(this->lable_);
		AddChildView(this->button_);
	}

	void TestWindow::Layout() {
		this->lable_->SetBounds(10, 10, 50, 30);
		this->button_->SetBounds(10, 50, 60, 30);
	}

	void TestWindow::Paint(ChromeCanvas* canvas) {
		canvas->FillRectInt(SK_ColorWHITE, 0, 0, width(), height());
	}

	void TestWindow::ButtonPressed(views::Button* sender) {
		if (sender == this->button_) {
			this->lable_->SetText(L"点击了");
		}
	}

	void TestWindow::CreateTestWindow() {
		views::Window::CreateChromeWindow(NULL, gfx::Rect(), new TestWindow)->Show();
	}
	
###源码解析
1. 包含相关的控件views::Label * lable_ ，views::TextButton * button_;
1. 通过实现 views::ButtonListener接口来处理button_的点击事件
1. 使用 AddChildView 将两个控件添加为子控件。
1. 在buttonPressed函数中添加事件处理的代码。
1. Paint 函数将整个客户区画成白色
1. Layout函数设置子控件的位置。

##Paint&&Layout

Chrome使用Skia作为二位绘图引擎，而绘图体现在每一个View的Paint函数中，views::View的Paint函数屁事都不干，所以在第一个程序中可以看到客户区将背景图片和默认的黑色显示出来，而第二个函数，我们覆盖默认的Paint函数并且将整个View （TestWindow）的区间绘成白色。

每一个View的Layout函数用于定位它所有的子View。当因为某些原因整个窗口需要重构时，根View调用Layout函数，然后递归调用每一个子View的Layout函数。关于Chrome的Layout，chrome还提供一个GridLayout(src\chrome\views\grid_layout.h)和FillLayout(src\chrome\views\fill_layout.h)。具体的用法可以参考chrome中的源码。

##事件处理
         	
chrome控件通常的事件处理方式是通过声明一个接口，然后将该接口指针作为一个成员变量来使用实现。例如按钮事件的接口声明如下：

	// An interface implemented by an object to let it know that a button was
	// pressed.
	class ButtonListener {
	 public:
	  virtual void ButtonPressed(Button* sender) = 0;
	};
	
一般情况下，控件的父View会继承子控件依赖的借口并实现相关的函数，然后将自己作为参数传递给这些控件实现完整的事件处理。一个父View可能包含好几个同类的控件，所以这些接口一般会包含一个sender以便区分不同的控件发送者。
         	
其他控件的使用，可以参考源码中的代码，这里不多叙述。


##原生控件

一般来说，如果有设计良好，使用方便的控件可用就完全不必自己从轮子造起，chrome就是这样。chrome提供的很多控件并不是自己从零绘制的，而是基于原生的操作系统控件，最典型的是TextField(src\chrome\views\controls\text_field.h)。该控件底层的实现完全使用WTL提供的richedit<http://msdn.microsoft.com/en-us/library/bb787873%28VS.85%29.aspx#_win32_Rich_Edit_Version_2.0>。当然为了方便扩展，chrome也提供相关的类方便用户封装其他的原生控件。
       	
下面以封装IE浏览器说明问题，当用户需要在UI中嵌入浏览器，可以将下面的View像上面的代码中Button那样使用。

首先获得WTL对IE的封装（实际上是一个com），相关源码可参考下面的代码<http://devel.openocr.org/svn/openocr/trunk/cuneiform/interface/icrashreport/wtl/samples/tabbrowser/browserview.h>。

接着新建一个View，并封装该原生控件，具体的源码可参考:

	// ie_view.h BrowserView.h 是原生控件的封装头文件

	#include "BrowserView.h"
	#include "string"
	#include "chrome/views/view.h"
	#include "base/scoped_ptr.h"

	namespace views {
	class HWNDView;
	}

	class IEView: public views::View {

	public:

		explicit IEView(const std::wstring & url);
		virtual ~IEView();

		virtual void Layout();
		virtual void ViewHierarchyChanged(bool is_add, views::View* parent,
				views::View* child);

	private:

		void initChildViews();

		scoped_ptr<CBrowserView> iebrowser_;
		views::HWNDView * iebrowser_view_;

		std::wstring url_;

	protected:

		DISALLOW_COPY_AND_ASSIGN (IEView);
	};

	// ie_view.cpp

	#include "ie_view.h"
	#include "base/logging.h"
	#include "chrome/views/controls/hwnd_view.h"
	#include "chrome/views/widget/widget.h"

	IEView::IEView(const std::wstring & url) :
			url_(url) {
	}

	IEView::~IEView() {
	}

	void IEView::Layout() {
		this->iebrowser_view_->SetBounds(0, 0, width(), height());
	}

	void IEView::initChildViews() {

		iebrowser_.reset(new CBrowserView);
		HWND hwnd = GetWidget()->GetNativeView();

		RECT rc = { 0, 0, 100, 100 };
		HWND hwnd_ie = iebrowser_->Create(hwnd, 0, url_.c_str(),
				WS_CHILD | WS_VISIBLE | WS_CLIPSIBLINGS | WS_CLIPCHILDREN
						| WS_VSCROLL | WS_HSCROLL);
		DWORD errcode = GetLastError();
		DCHECK(iebrowser_->IsWindow());

		iebrowser_view_ = new views::HWNDView;
		DCHECK(iebrowser_view_) << "LocationBarView::Init - OOM!";
		AddChildView(iebrowser_view_);
		iebrowser_view_->SetAssociatedFocusView(this);
		iebrowser_view_->Attach(iebrowser_->m_hWnd);

	}

	void IEView::ViewHierarchyChanged(bool is_add, views::View* parent,
			views::View* child) {
		if (is_add && child == this) {
			this->initChildViews();
		}
	}

这个过程中，最关键的View就是 views::HWNDView。该View提供一个机制将原生控件和Chrome View进行绑定，以便用户能够像View一样操作原生控件。

ViewHierarchyChanged函数在它所述的View被add和remove时被调用。上面源码的意思是当 IEView被父View调用AddChildView函数将其加为子View时被调用。之所以不放在构造函数内是因为有些控件的初始化依赖一个窗口句柄，而在构造函数结束之前，这个句柄很可能还没有初始化。

##国际化

###Locale 项目
如果使用virtual studio 2008打开chrome for windows的工程，可以看到如下的项目：

不可否认，Chrome的国际化做的非常优秀，在Chrome中添加一种新的语言支持非常方便。

其中每一个项目对应一种语言支持，所以如果需要添加新的语言支持，只需要新建一个新的语言项目。

实际上，每一个语言项目内的所有文件都是编译生成的中间文件，在文件夹【src\chrome\app\locales】中存放了这所有的项目文件，但每一个项目仅仅存在一个vcproj文件，例如zh-CN.vcproj，项目中包含的文件实际存在于【src\chrome\Debug/Release\grit_derived_sources】目录下。那这些文件怎么生成的呢，这需要了解一下google自己开发的一个python项目。

###GRIT 软件
该项目的源码在目录【src\tools\grit】下，全部使用python语言。该目录下的readme是这么说的：

	GRIT (Google Resource and Internationalization Tool) is a tool for Windows
	 projects to manage resources and simplify the localization workflow.
	 
在命令行输入命令【python grit.py】(该文件是整个grit的入口)有如下输出

	GRIT - the Google Resource and Internationalization Tool
	Copyright (c) Google Inc. 2009

	Usage: grit [GLOBALOPTIONS] TOOL [args to tool]

	Global options:

	-i INPUT Specifies the INPUT file to use (a .grd file). If this is not
	specified, GRIT will look for the environment variable GRIT_INPUT.
	If it is not present either, GRIT will try to find an input file
	named 'resource.grd' in the current working directory.

	-v Print more verbose runtime information.

	-x Print extremely verbose runtime information. Implies -v

	-p FNAME Specifies that GRIT should profile its execution and output the
	results to the file FNAME.

	Tools:

	TOOL can be one of the following:
	build A tool that builds RC files for compilation.
	newgrd Create a new empty .grd file.
	rc2grd A tool for converting .rc source files to .grd files.
	transl2tc Import existing translations in RC format into the TC
	sdiff View differences without regard for translateable portions.
	resize Generate a file where you can resize a given dialog.
	unit Use this tool to run all the unit tests for GRIT.
	count Exports all translateable messages into an XMB file.

	For more information on how to use a particular tool, and the specific
	arguments you can send to that tool, execute 'grit help TOOL'
	
有兴趣可以研究一下代码，python还是很有趣的东东！！！

grit接收一个输入文件，然后生成项目所需的.h和.rc等文件。当然输出什么文件需要用户在输入文件中指定。

###Grd文件

grit接收grd类型的文件作为输入，然后根据输入文件中的指定输出匹配的文件。locale相关的输入文件存放在目录【src\chrome\app\resources】下，最关键的几个文件是【locale_settings.grd】和【generated_resources.grd】(此文件放在目录src\chrome\app下，个人觉得放那很诡异)。在chrome中有若干项目仅仅包含grd文件并将其生成目标文件，而其他一些项目则依赖这些文件。chrome_strings项目就如此。它就将相关的grd文件生成locale下各种语言项目依赖的.h和.rc文件。

所有grd文件都是一个xml文件，格式都符合grit的一个规范，下面是【generated_resources.grd】的部分内容：

	<?xml version="1.0" encoding="UTF-8"?>

	<grit base_dir="." latest_public_release="0" current_release="1"
	      source_lang_id="en" enc_check="m&#246;l">
	  <outputs>
	    <output filename="grit/generated_resources.h" type="rc_header">
	      <emit emit_type='prepend'></emit>
	    </output>
	    <output filename="generated_resources_zh-CN.rc" type="rc_all" lang="zh-CN" />
	    <output filename="generated_resources_zh-TW.rc" type="rc_all" lang="zh-TW" />
	    <output filename="generated_resources_zh-CN.pak" type="data_package" lang="zh-CN" />
	    <output filename="generated_resources_zh-TW.pak" type="data_package" lang="zh-TW" />
	  </outputs>
	  <translations>
	    <file path="resources/generated_resources_zh-CN.xtb" lang="zh-CN" />
	    <file path="resources/generated_resources_zh-TW.xtb" lang="zh-TW" />
	  </translations>
	  <release seq="1" allow_pseudo="false">
	    <messages fallback_to_english="true">
	      <!-- TODO add all of your "string table" messages here. Remember to
	      change nontranslateable parts of the messages into placeholders (using the
	      <ph> element). You can also use the 'grit add' tool to help you identify
	      nontranslateable parts and create placeholders for them. -->

	      <message name="IDS_SHOWFULLHISTORY_LINK" desc="test">
		Show Full History
	      </message>
	    </messages>
	  </release>
	</grit>
	
###说明
1. Output节表示输出文件，例如上面的文件会生成一个 generated_resources.h、若干rc文件（generated_resources_zh-CN.rc）和若干pak文件（generated_resources_zh-CN.pak）。
1. Translations节表示翻译文件，一般来说每一种支持的语言都应该有一个翻译文件。
1. Messages节表示定义的默认字符串(不同的grd文件有不同的作用，其他文件的message节里面的内容不一定是字符串，也可能是其他类型。)，

当grit解析收到locate 相关的grd文件时，首先生成默认的资源文件，这里默认的资源文件是“en-US”。当发现Output节有其他语言的输出时，则查找对应的xtb翻译文件，如果grd文件中的message选项指定需要翻译，则通过message中的name属性查找xtb中对应的record，然后将替换之。

在grd文件中有很多可选的选项，具体可以参考chrome自带的grd文件或者grit源代码。目前本人未找到google官方的帮助文档。

Xtb文件也是一个xml文件，典型的内容格式如下：

	<?xml version="1.0" ?>
	<!DOCTYPE translationbundle>
	<translationbundle lang="zh-CN">
	<translation id="6779164083355903755">删除(&amp;R)</translation>
	<translation id="3581034179710640788">此网站的安全证书已过期！</translation>
	<translation id="8275038454117074363">导入</translation>
	</translationbundle>
	
或者

	<?xml version="1.0" ?>
	<!DOCTYPE translationbundle>
	<translationbundle lang="zh-CN">
	<translation id="IDS_WEB_FONT_FAMILY">Simsun</translation>
	<translation id="IDS_WEB_FONT_SIZE">84%</translation>
	<translation id="IDS_UI_FONT_FAMILY">default</translation> 
	</translationbundle>
	
两者区别主要是ID的表示方式。后者ID和grd中的name是一致的，而前者则通过某种算法将grd中的name转换为由数字组成的ID。

###Grd文件的编译

chrome通过添加“自定义编译规则(Custom Build Rules)”来通过vs2008自动编译所有自定义格式文件。例如grd文件的规则如下：

如图所示，grd使用一个bat文件编译，并且提供两个参数。正如前面提到的，第二个参数就是目标文件的输出路径。

该bat文件的内容如下：

	:: Batch file run as build command for .grd files
	:: The custom build rule is set to expect (inputfile).h and (inputfile).rc
	:: our grd files must generate files with the same basename.
	@echo off

	setlocal

	… 忽略 …

	:: Put cygwin in the path
	call %SolutionDir%\..\third_party\cygwin\setup_env.bat

	%SolutionDir%\..\third_party\python_24\python.exe %SolutionDir%\..\tools\grit\grit.py \
		-i %InFile% build -o %OutDir% %PreProc1% %PreProc2% %PreProc3% %PreProc4% %PreProc5
		
从上面内容可以发现，chrome将python解释器直接放到源码【src\third_party\python_24】里。然后通过它调用grit.py文件实现文件的编译。

关于grd的自定义编译规则文件可以查看文件【src\tools\grit\build\grit_resources.rules】。 msdn上<http://msdn.microsoft.com/en-us/library/03t8bzzy.aspx>也许有些帮助。


##Locale初始化

当通过grd文件生成locale下语言项目依赖的文件后，Chrome将这些项目打包生成一个语言Dll，这些Dll可以在目录【src\chrome\Debug/Release\locales】下找到。当Chrome启动时，它会通过某种途径查找locale类型，然后找到对应的Dll来load。

Chrome查找locale类型通过【src\chrome\common\l10n_util.h】的GetApplicationLocale函数实现。

该函数首先检查命令行有没有通过“--lang”指定locale，如果没有，则检查当前的配置文件中是否指定locale。如果未指定，则获取操作系统的locale，如果再获取失败，则使用默认的en-US。当然在返回之前，Chrome总会检查当前的语言文件目录是否存在该语言的Dll。

chrome自己弄了一套本地配置系统，下面是一个典型的配置文件格式，内容为语言选项

	{
		"intl": {
			"app_locale": "zh-CN"
		}
	}

当找到locale后，chrome通过【src\chrome\common\resource_bundle_win.cc】中的LoadResources函数加载对应的Dll。

##国际化小结

当程序中需要使用多国语言的字符串时，可以参考下面的代码

	ResourceBundle &rb = ResourceBundle::GetSharedInstance();
	std::wstring str = l10n_util::GetString( IDS_ABCDEFG )
	
其中 IDS_ABCDEFG就是定义在grd中的name 。

Chrome中有两个grd文件比较重要：

1. src\chrome\app\generated_resources.grd ：该文件定义了多国语言字符串（string），例如按钮的文字，窗口的标题等。
1. src\chrome\app\resources\locale_settings.grd ：该文件定义了多语言配置，例如窗口的大小，某些控件的尺寸。考虑到不同语言的字符串差异，同一个窗口在不同语言下要求的宽度等参数可能不一样。此外还包含字体的大小、名称等。

##UI主题

UI主题和国际化一样，同样适用grit 将grd翻译成目标文件，然后再生成主题Dll。生成主题的grd文件是【src\chrome\app\theme\theme_resources.grd】。该文件在项目chrome_resources中。该项目生成的rc文件被项目theme_dll使用并生成一个主题Dll。

主题Dll在目录【src\chrome\Debug/Release\themes】下。在我这个chrome源码版本中，只有一个主题Dll。名字为“default.dll”。这个Dll在【src\chrome\common\resource_bundle_win.cc】中的函数LoadThemeResources()被load。

主题通常都由一些图片组成，这些图片存放在目录【src\chrome\app\theme】下，基本上时png图片格式。

这些图片通过theme_resources.grd 来生成dll。这个文件的部分格式如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<grit latest_public_release="0" current_release="1">
	  <outputs>
	    <output filename="grit/theme_resources.h" type="rc_header">
	      <emit emit_type='prepend'></emit>
	    </output>
	    <output filename="theme_resources.rc" type="rc_all" />
	    <output filename="theme_resources.pak" type="data_package" />
	  </outputs>
	  <release seq="1">
	    <includes>
	      <include name="IDR_BACK" file="back.png" type="BINDATA" />
	      <include name="IDR_BACK_D" file="back_d.png" type="BINDATA" />
	    </includes>
	  </release>
	</grit>

当用户需要使用图片时，可以参考如下代码：

	ResourceBundle &rb = ResourceBundle::GetSharedInstance();
	SkBitmap* img_ = rb.GetBitmapNamed(IDR_ABCDEFG);

其中 IDR_ABCDEFG为在 theme_resources.grd中每一张图片分配的name。

##Chrome的版本信息

如果查看项目生成的chrome.exe、chrome.dll文件的属性，可以发现如下：

实际上这些参数通过项目chrome_exe、chrome_dll中的一个.rc.version文件来获取，之所以讲这些是因为.rc.version的处理和前面提到的grd文件有异曲同工之妙。version文件典型的内容如下：

	//////////////////////////////////////////////
	//
	// Version
	//
	... 省略 ...
	BEGIN
	VALUE "CompanyName", "@COMPANY_FULLNAME@"
	VALUE "FileDescription", "@PRODUCT_FULLNAME@"
	VALUE "FileVersion", "0.0.0.0"
	VALUE "InternalName", "chrome_exe"
	VALUE "LegalCopyright", "@COPYRIGHT@"
	... 省略 ...
	END
	END
	... 省略 ...
	END

###version文件的编译

Version文件的编译命令如下：

	D:\chrometrunk\chrometrunk\src\chrome\/../chrome/tools/build/win/version.bat \
	 "D:\chrometrunk\chrometrunk\src\chrome\/../chrome/" \
	 "D:\chrometrunk\chrometrunk\src\chrome\Debug\obj\chrome_exe" \
	 "D:\chrometrunk\chrometrunk\src\chrome\Debug\obj\chrome_exe/chrome_exe_version.rc"

此类型的编译规则文件见【src\chrome\tools\build\win\version.rules】。该规则的核心就是version.bat文件。

version.bat的内容如下：

	:: Batch file run as build command for vers.vcproj
	@echo off

	setlocal

	set InFile=%~1
	set SolutionDir=%~2
	set IntDir=%~3
	set OutFile=%~4
	set VarsBat=%IntDir%/vers-vars.bat

	:: Put cygwin in the path
	call %SolutionDir%\..\third_party\cygwin\setup_env.bat

	:: Load version digits as environment variables
	cat %SolutionDir%\VERSION | sed "s/\(.*\)/set \1/" > %VarsBat%

	:: Load branding strings as environment variables
	set Distribution="chromium"
	if "%CHROMIUM_BUILD%" == "_google_chrome" set Distribution="google_chrome"
	cat %SolutionDir%app\theme\%Distribution%\BRANDING | sed "s/\(.*\)/set \1/" >> %VarsBat%

	set OFFICIAL_BUILD=0
	if "%CHROME_BUILD_TYPE%" == "_official" set OFFICIAL_BUILD=1

	:: Determine the current repository revision number
	set PATH=%~dp0..\..\..\..\third_party\svn;%PATH%
	svn.exe info | grep.exe "Revision:" | cut -d" " -f2- | sed "s/\(.*\)/set LASTCHANGE=\1/" >> %VarsBat%
	call %VarsBat%

	::echo LastChange: %LASTCHANGE%

	:: output file
	cat %InFile% | sed "s/@MAJOR@/%MAJOR%/" ^
	| sed "s/@MINOR@/%MINOR%/" ^
	| sed "s/@BUILD@/%BUILD%/" ^
	| sed "s/@PATCH@/%PATCH%/" ^
	| sed "s/@COMPANY_FULLNAME@/%COMPANY_FULLNAME%/" ^
	| sed "s/@COMPANY_SHORTNAME@/%COMPANY_SHORTNAME%/" ^
	| sed "s/@PRODUCT_FULLNAME@/%PRODUCT_FULLNAME%/" ^
	| sed "s/@PRODUCT_SHORTNAME@/%PRODUCT_SHORTNAME%/" ^
	| sed "s/@PRODUCT_EXE@/%PRODUCT_EXE%/" ^
	| sed "s/@COPYRIGHT@/%COPYRIGHT%/" ^
	| sed "s/@OFFICIAL_BUILD@/%OFFICIAL_BUILD%/" ^
	| sed "s/@LASTCHANGE@/%LASTCHANGE%/" > %OutFile%

	endlocal

注意上述红色标示的部分

VERSION 部分是获取Chrome预先写好的版本参数文件【src\chrome\VERSION】,内容如下

	MAJOR=2
	MINOR=0
	BUILD=175
	PATCH=0
	
BRANDING 部分是获取Chrome预先写好的公司参数文件【src\chrome\app\theme\chromium\BRANDING】,内容如下

	COMPANY_FULLNAME=The Chromium Authors
	COMPANY_SHORTNAME=The Chromium Authors
	PRODUCT_FULLNAME=Chromium
	PRODUCT_SHORTNAME=Chromium
	COPYRIGHT=Copyright (C) 2006-2009 The Chromium Authors. All Rights Reserved.
	
如果熟悉Linux的话，估计不会对sed这条命令陌生。这条命令存在与目录【src\third_party\cygwin】中。sed命令将上述两个文件的内容经过处理，在每一行之前加入set。然后输出到%VarsBat%文件【src\chrome\Debug\obj\chrome_exe\vers-vars.bat】中。最终 vers-vars.bat文件的格式类似【set MAJOR=2】。很显然，当执行这个bat，即把所有的参数添加的环境变量了。

最后一条命令将version文件中【%***%】替换成同名环境变量的值，并输出到一个rc文件【src\chrome\Debug\obj\chrome_exe/chrome_exe_version.rc"】中。

最后在chrome_exe项目的rc文件【src\chrome\app\chrome_exe.rc】中，【chrome_exe_version.rc】被include进去。源码如下：

	#else // APSTUDIO_INVOKED

	#include "chrome_exe_version.rc"
	#endif
	
##PrefService

和许多其他的程序一样，chrome也包含一系列本地配置文件，这些文件保存程序需要在重启后还能够记忆的参数或者其他数据。Chromium的配置文件存放在路径【C:\Documents and Settings\Username\Local Settings\Application Data\Chromium\User Data\Default】。Chrome典型的配置文件格式如下：

	{
		… 省略 …
		"profile": {
			"exited_cleanly": true,
			"id": "not-signed-in",
			"name": "",
			"nickname": ""
		},
		"session": {
			"urls_to_restore_on_startup": [ ]
		}
	}

chrome本地配置文件通过一个核心的类【chrome\common\pref_service.cc】来实现，在这个类中，它调用解析器JSONStringValueSerializer【chrome\common\pref_service.cc】将PrefService 结构中的内容转换成一个可写入文本文件的字符串。最后通过WriteFile函数【base\file_util_win.cc】将字符串写入文本。需要注意的是这个函数并不是UI线程调用的，而是用后台线程来实现。具体参见下列代码：

【chrome\common\pref_service.cc】

	SaveLaterTask* task = new SaveLaterTask(pref_filename_, data);
	if (thread != NULL) {
		// We can use the background thread, it will take ownership of the task.

		thread->message_loop()->PostTask(FROM_HERE, task);
	} else {
		// In unit test mode, we have no background thread, just execute.

		task->Run();
		delete task;
	}
	
为了方便管理PrefService，chrome提供了一个ProfileManager类【chrome\browser\profile_manager.h】和Profile 类【chrome\browser\profile.h】。这里不做分析，因为这几个类并不通用。


##PrefService使用

为了新建一个在路径为”C:\Preferences”的配置文件，我们可以声明一个PrefService 实例：

	PrefService * pService = new PrefService(L”C:\Preferences”);

当需要往该配置文件增加一个参数时，可以参考下列步骤

1. 声明一个合适的类型，chrome提供了若干种类型，例如BooleanPrefMember、 IntegerPrefMember、 StringPrefMember等。当需要存取一个bool类型。可以声明一个BooleanPrefMember。具体的实现见【chrome\common\pref_member.cc】。
1. 向prefService结构里注册一个pref变量，并提供默认值。每一个pref变量必须有一个名字，该名字对应于该变量在配置文件中的“路径”。pref名字可以直接输入。但是保险的做法是将所有pref的名字存放在一个统一的地方以方便管理。chrome的所有prefName存放在文件【chrome\common\pref_names.h】中。perfName通过英语句号分割，每一节表示一层。例如perfName【L"general.luaguage.language"】在实际的配置文件的表现如下。
1. 初始化这个类型。初始化时pref类型需要提供三个参数，该变量的名字，prefService的指针和一个回调。一般情况下，变量的名字和第二步注册的名字相同，prefService指针也和注册该pref变量使用的prefService一致。这一步的目的就是将一个本地变量和某一个全局的prefService中的某一个变量关联起来。既然他们是两个不同的变量（而不是两个指针指向同一个变量），那么就存在同步的问题。当prefService中的变量被其他线程偷偷的修改了怎么办呢，后面再分析。 <!-- @page { margin: 2cm } P { margin-bottom: 0.21cm } --> 
1. 对这个类型进行读写操作。例如成员函数GetValue()和SetValue(value)。这两个函数对bool，int，string等类型都适用，底层使用很容易想到用模版类，如果研究过stl源码，这个其实是很容易理解。
1. 一般情况下，prefService总是作为一个全局变量存储在内存中，它在程序初始化时从磁盘读入数据，在程序退出时，将数据写入磁盘。很明显。如果程序中途崩溃或者断电等因素。这之前作出的配置改动都会丢失。貌似chrome有一个周期性将这些数据写入磁盘的机制，具体没去研究。
1. 当一个新的程序安装后，磁盘没有任何配置参数，但是每一个pref变量在注册到prefService时都指定了一个默认值，新程序即使用这个值来初始化程序。也许有人会注意到，程序运行一次以后磁盘还是没有写入任何东西，是不是没有在退出程序时写入磁盘啊？chrome对每一个pref变量会判断是否该修改，如果未修改，它实际上什么都不做。很显然，前面的情况是因为用户或者程序根本没有修改任何一个pref参数。


	{
		"general": {
			"luaguage": {
				"language": "zh-CN"
			}
		}
	}
	
	
##Pref同步

将配置参数封装成PrefMember等类的一个重要原因是支持实时同步。该类实际上使用了一个”OBSERVER”<观察者>设计模式。

在PrefMember初始化时，它会将自己注册至赋值给自己的 PrefService对象，以便在信息发生变化时能够尽快得到通知。而外部应用同样可以在初始化pref变量注册一个回调，以便在该pref变量被其他人修改时及时得到通知。

##PrefService内部实现
PrefService将所有从对应配置文件读取的信息存储在内部的一个【PreferenceSet prefs_】【typedef std::set<Preference*, PreferencePathComparator> PreferenceSet;】中， Preference则是封装了每一个pref变量的Name和value等信息：

Preference类中的部分成员【chrome\common\pref_service.h】

	Value::ValueType type_;
	std::wstring name_;
	scoped_ptr<Value> default_value_;
	
Value这个类值得研究一下，它可用于表示所有类型，不过底层实现并不是模版类，而是继承和多态来实现。Value只不过是众多具体实现类的一个基类。
chrome这一套pref机制具有很好的扩展性。当新的需要导致一个新的pref变量时，除了需要在【chrome\common\pref_names.h】文件中增加一个pref名字以外，其他的处理都可以在“当地”进行。

##PathService

前面提到，chrome包含若干配置文件。这些文件的路径怎么指定，直接在程序输入地址么，这并不是一个好的办法。chrome提供了一个PathService类【src\base\path_service.cc】。通过这个类的Get函数，其他模块可以很方便地获取一个Path（路径）。

在PathService中，每一个Path都有一个唯一的PathID（int类型）。

在pathService中，所有的path都存储在一个由结构 Provider构成的链表中，每一个Provider中包含一个ProviderFunc函数指针。当用户通过PathService的Get函数查询一个path时，PathService以此查询每一个Provider，而Provider由通过函数func来获得实际的path。

	struct Provider {
		PathService::ProviderFunc func;
		// ... ...
		struct Provider* next;
		bool is_static;
	};

为了获得更好的性能，chrome在PathService中内置了path cache。下面的 PathData结构存储了PathService的所有数据。 providers是Provider的首节点。 cache即上面提到的缓存.

	struct PathData {
		Lock lock;
		PathMap cache; // Track mappings from path key to path value.
		PathSet overrides; // Track whether a path has been overridden.
		Provider* providers;
	}
	
Chrome默认提供三个Provider。

1. base_provider ：【base\path_service.cc】
1. base_provider_win ：【base\path_service.cc】
1. PathProvider ：【chrome\common\chrome_paths.cc】

用户可以通过RegisterProvider函数【base\path_service.cc】注册自己的Provider。 PathProvider的注册代码如下：

	void RegisterPathProvider() {
		PathService::RegisterProvider(PathProvider, PATH_START, PATH_END);
	}
	
###自定义Provider

如果用户需要注册自定义Provider，只需实现一个函数【 typedef bool (*ProviderFunc)(int, FilePath*)】。这个函数收到一个PathID。如果该ID在本函数内合法，则返回true，并且将Path通过第二个参数传回来。否则返回false。具体实现参考PathProvider函数【chrome\common\chrome_paths.cc】。

然后通过 PathService::RegisterProvider函数注册自己即可。

##完整文档

见[这里](http://pan.baidu.com/s/1o6ogh8u)

##参考
1. <http://pan.baidu.com/s/1o6ogh8u>
1. <http://www.cnblogs.com/duguguiyu/archive/2008/10/02/1303095.html>
