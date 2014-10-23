---
layout: post
title: Fltk源码学习笔记
category: cpp
---

写于 **2010年09月18日**

本文基于Windows操作系统下的Fltk 2.0版本,使用VS2008。该版本和之前的版本改动了不少地方,其招牌的FL_前缀去掉了,增加了贴图支持和更好的非英文字符支持。不过代码还是一如既往的“乱”。


在深入源码之前,先学习一下Fltk的基本用法。一个简单的Sample如下:

	int main(int argc, char **argv) {
		Window *window = new Window(300, 180);
		window->begin();
		Widget *box = new Widget(20, 40, 260, 100, "Hello, World!");
		box->box(UP_BOX);
		box->labelfont(HELVETICA_BOLD_ITALIC);
		box->labelsize(36);
		box->labeltype(SHADOW_LABEL);
		window->end();
		window->show(argc, argv);
		return run();
	}

这个Sample简单的展现了Fltk程序的大致流程:

1. 创建Window(窗口)
2. 向该窗口塞控件
3. 显示窗口
4. 进行Messsage Loop

##窗口

Fltk窗口使用Window类表示,该类派生自Widget。不过和普通的Widget不同的是,Window加入了窗口操作函数,比如最大化窗口等。同时Window作为一个控件树(本身也是一个控件)的根节点,它包含一个定位控件的逻辑,即在收到消息后定位具体的控件,并将消息派发给它。

##控件树

Fltk不用显式地往控件中插入子控件,这个操作通过 Group类( Window 类派生自它)的begin和end成员函数来完成。

它的实现原来相当之简单,在Group类中定义了一个静态成员 static Group *current_,当调用begin函数时,就将该静态变量赋值为自己的指针( this),当调用end函数时,将它赋值为父控件的指针。当一个控件创建时,会查询上述静态变量是否为空,如否则将自己插入上述容器控件,具体见Widget类的构造函数。

在Fltk中,有普通控件Widget和容器控件Group 之分,区别是Group可以包含子控件。Widget包含一个Group指针来指向父控件,Group则还包含一个Widget**指针来包含子控件。Fltk就通过上述结构来构造一棵控件树。

##窗口创建

Window类的构造仅仅只是初始化一些数据,而真正的窗口并未创建。这个过程在Window类的函数void show(int argc, char **argv)实现。除了窗口创建( CreateWindow ),窗口显示( ShowWindow )也在这一并实现。

为了实现跨平台,Ftlk不可能直接实现操作系统API来做上述事情,所以在Window类下面又整了一个CreatedWindow类( fltk\win32.h ),在这个类的(静态)成员函数封装操作系统API。实际的窗口创建操作在静态成员函数CreatedWindow::create(Window* window)中实现,具体的Api函数在函数指针对象__CreateWindowExW中( Windows 使用 API CreateWindowEx )。


##多窗口管理

Fltk支持多窗口,为了做到这一点,CreatedWindow类包含一个静态成员 first。这是一个CreatedWindow指针。每一次创建新窗口(会生成一个 CreatedWindow 实例)后,就将当前的CreatedWindow指针放在这个链表的最前(窗口 Destroy 时从链表中清除)。整个链表就存储着所有已经创建的窗口。

当收到窗口消息时,Fltk就通过窗口ID查询上述链表获得具体的窗口类实例,然后将消息派发给它。在Windows操作系统下,这个ID是HWND。而在X11平台下,这个ID是XWindow ( src\run.cxx )。

##消息循环

Fltk通过运行全局函数run来进入消息循环。首先检测是否还有窗口实例,如有则继续wait,然后处理一些idle和timer等杂七杂八的消息,最后在操作系统层面的消息循环中等待。


##消息

Fltk在分发消息时,仅仅传递了一个消息ID,而没有一些附加的消息,例如鼠标消息的坐标,按键消息的KeyValue等等。Fltk定义了一堆全局变量( src\run.cxx ),将这些附加信息放在那,每次收到新的窗口消息后,首先会更新需要更新的变量。在后面的控件处理中,可以去这些变量去查询和获取需要的值(这种方式可能比较 “ 省 ” ,可是做法鄙人不敢苟同)。

对于鼠标消息,Fltk通过鼠标位置和控件的位置比较来定位。后添加的控件Z轴在上,所以如果两控件重叠,后添加的控件将获得鼠标消息。

Fltk消息种类見<http://www.ibm.com/developerworks/cn/linux/l-fltk/>

具体的定义在(fltk\events.h)

Fltk所有的消息通过函数bool fltk::handle(int event, Window* window)来分发。Fltk和控件通过int Widget::send(int event)函数来传递消息,在该函数中会调用虚函数 Widget::handle。

##回调

为了处理业务逻辑,需要向控件注册回调。和MFC/WTL或者WxWidget等UI框架相比,FLTK的回调绑定机制相当简单。这个绑定通过Widget类的成员函数callback实现。Fltk定义了几个回调函数类型:

	typedef void (Callback )(Widget*, void*);
	typedef Callback* Callback_p; // needed for BORLAND
	typedef void (Callback0)(Widget*);
	typedef void (Callback1)(Widget*, long);
	
在widget类中包含一个成员Callback*callback_,callback函数所做的就是给它赋值。不过这种实现也是有些不足,一是无法同时绑定多个回调,而是对类成员函数(非静态)和仿函数支持不足。当然既然人家都叫Fltk,那也就无可厚非了。
      
Fltk针对回调定义了一些Flag:

	enum { // Widget::when() values
		WHEN_NEVER = 0,
		WHEN_CHANGED = 1,
		WHEN_RELEASE = 4,
		WHEN_RELEASE_ALWAYS = 6,
		WHEN_ENTER_KEY = 8,
		WHEN_ENTER_KEY_ALWAYS = 10,
		WHEN_ENTER_KEY_CHANGED = 11,
		WHEN_NOT_CHANGED = 2 // modifier bit to disable changed() test
	};
	
	
这些标志通过widget类的成员函数when来设置,这些标志可以影响回调的调用机制。例如Button默认在键盘Release时触发回调,而设置WHEN_CHANGED标志后Press和Release都会触发回调。
      
Fltk这套回调机制虽然使用简单,不过个人认为大可不必在widget这一层支持回调。试想Static控件一般情况下是不需要回调的,所以完全没必要 给每一个控件强制分配一个回调指针。第二个,控件的回调需要什么格式很难用几个简单的定义覆盖,而且这些标志位也很难覆盖所有的需求,试想我需要在鼠标移 入按钮时触发回调用标志位怎么处理(虽然这个需求比较猥琐)?所以最好是将回调绑定交给控件自己去实现,底层仅仅将具体的消息分发给它。

