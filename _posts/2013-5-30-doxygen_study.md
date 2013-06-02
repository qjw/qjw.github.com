---
layout: post
title:  doxygen学习笔记
category: cpp
---

##安装
Windows下安装**cygwin**并且选择**[doxygen](http://www.stack.nl/~dimitri/doxygen/index.html)**，若debian，可**apt-get install doxygen**。

##配置
打开Cygwin Terminal,cd到特定目录。运行命令**doxygen -g xx.conf**(*xx.conf自定*)。生成一份默认配置。

对xx.conf做以下修改

	// 项目名称
	PROJECT_NAME           = "QJW TEST"
	// 项目版本
	PROJECT_NUMBER         = "BETA"
	// 输出语言（若使用中文，选择Chinese）
	OUTPUT_LANGUAGE        = Chinese
	// 若是C系语言，选择此项
	OPTIMIZE_OUTPUT_FOR_C  = YES
	// 若源码是GB编码，使用GB2312，否则保持UTF-8
	INPUT_ENCODING         = GB2312
	// 不要生成latex
	GENERATE_LATEX         = NO
	// 需要生成的文件类型
	FILE_PATTERNS          = *.c *.cc *.cpp *.c++ *.h

	// 生成选项
	EXTRACT_ALL            = YES
	EXTRACT_PRIVATE        = YES
	EXTRACT_STATIC         = YES
	EXTRACT_LOCAL_METHODS  = YES
	// 将源码一并打包
	SOURCE_BROWSER         = YES
	
运行**doxygen xx.conf**生成网页版的文档。

##简单用法

	/**
	 * func1 示例
	 * @pre 已经初始化
	 * @warning 不要传空指针
	 * @post 完了可以干坏事了
	 * @param [in] a - 参数1
	 * @param [out] b - 参数2
	 * @param [inout] c - 参数3
	 * @return 0 - 成功
	 * @return 1 - 如何如何
	 * @return -1 - 再如何如何
	 */
	int func1(int a,int* b,int* c)
	{
		return 0;
	}

	/**
	 * @brief func2 示例 \n
	 *  fjjfklasdffsd
	 * @details fjalsdfad
	 * @param [in] a - 参数1
	 * @see func1
	 * @return 0 - 成功
	 */
	int func2(int a)
	{
		return 0;
	}

	/**
	 * 间接调用func1
	 * @code
	 *  int main()
	 *  {
	 *      return 0;
	 *  }
	 * @endcode
	 *
	 * @param X - 参数1
	 * @param Y - 参数2
	 */
	#define FUNC(X,Y) \
		func1(111,Y,Z)

	/** 简要说明文字 */
	#define FLOAT float

	/**
	 * ##枚举
	 * ###枚举1
	 * fjafajsdfj
	 *
	 * ---
	 * <http://qjw.qiujinwu.com>
	 * @see \n
	 *  - func1 \n
	 *  - func2
	 */
	typedef enum
	{
		/** enum1 value */
		enum1 = 1,
		enum2 = 2, /**< enum2 value*/
	}Enum;


	/**
	 * 定义个类型2
	 * @see Enum
	 */
	typedef struct 
	{
		/** 成员i */
		int i;
		double j; /**< 成员j */
	}Cls2;

	/**
	 * 定义个联合
	 * @see Cls2
	 */
	typedef struct 
	{
		/** 成员i */
		int i;
		double j; /**< 成员j */
	}union;

doxygen不支持C语言默认的帮助，需要在此基础上做进一步的tag，例如可以使用

	/**
	 * sth
	 */
	 
	/** sth */

还有其他，具体见<http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html>

###常用@选项
1. [@return](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdreturn) 函数返回值，可多次使用
1. [@param](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdparam) 函数参数，**可选地**使用[in],[out],[inout]标记方向
1. [@warning](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdwarning) 给出适当的警告
1. [@attention](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdattention) 给出适当的注意事项
1. [@pre](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdpre) 前置条件
1. [@post](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdpost) 后置条件
1. [@see](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdsee) 引用
1. [@brief](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdbrief) 简要说明
1. [@details](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmddetails) 详细说明
1. [@code](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdcode),[@endcode](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdendcode) 若希望在帮助中引用代码，可以将代码插入以上两个tag之间（多行）

###常用格式

1. 在帮助的第一行开始的部分，被默认为是@brief，所以该tag可以省略。
1. 若需要明确的换行，可以使用"\n"
1. 若需要使用列表，可以使用-，例如

---
	/**
	 * @see \n
	 *  - func1 \n
	 *  - func2
	 */

###具体语法
1. 函数，define可使用相同的定义
1. struct，enum，union 可使用相同的定义，对于这些类型的内部成员，以下两种方式都可以(**后者注意符号'<'**)

---
	/**
	 * 定义个类型2
	 * @see Enum
	 */
	typedef struct 
	{
		/** 成员i */
		int i;
		double j; /**< 成员j */
	}Cls2;

##格式
doxygen使用[markdown](http://daringfireball.net/projects/markdown/syntax)格式，具体的语法规范见<http://daringfireball.net/projects/markdown/syntax>

下面总结部分常用的，doxygen关于markdown的介绍见<http://www.stack.nl/~dimitri/doxygen/manual/markdown.html>

1. 若没有任何修饰，默认是**段落**，空行结束段落
1. 若想使用**标题格式**，在前面加#。加多少个#就表示几级标题。最多六级，#和正文用空格隔开。
1. 若使用**列表**，可以使用"-","+"或者"*"。同时支持多级列表，根据缩进区别。空行结束列表
1. **分割线**使用"---"(三个'-')
1. **粗体**使用" \*\*content\*\* "（一边两个'\*'）,**斜体**使用" \*conteng\* "（一边一个'\*'）。
1. **超链接**使用**\[txt\]\(url\)** 或者 **\<url\>**

###列表

	/** 
	 *  A list of events:
	 *    - mouse events
	 *         + mouse move event
	 *         + mouse click event\n
	 *            More info about the click event.
	 *         + mouse double click event
	 *    - keyboard events
	 *         1. key down event
	 *         2. key up event
	 *    - keyboard events
	 *         * key down event
	 *         * key up event
	 *
	 *  More text here.
	 */
	
##分组[grouping](http://www.stack.nl/~dimitri/doxygen/manual/grouping.html)

###模块[modules](http://www.stack.nl/~dimitri/doxygen/manual/grouping.html#modules)

	/** @addtogroup group1 grou1 actual name
	 * Additional documentation for group 'group1'
	 *  @{
	 */

	/** * func1 */
	void func1(){}

	/** * func2 */
	void func2(){}

	/** @}*/

	/** @addtogroup group2
	 * Additional documentation for group 'group2'
	 *  @{
	 */

	/** * func3 */
	void func3(){}

	/** * func4 */
	void func4(){}

	/** @}*/


使用[@addtogroup](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdaddtogroup)和对应的结束标记抱起来，内部所有的定义都属于这个模块。@defgroup和@addtogroup的区别是前者若发现该group已经存在会报错。

通常用结束标记括起来可以让内部所有元素自动归为此group，不过你也可以分开处理。此时手动添加[@ingroup](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdingroup)标记。

	/** @addtogroup group1 group actual name1
	 * Additional documentation for group 'group1'
	 */
	/** @addtogroup group2 group actual name2
	 * Additional documentation for group 'group2'
	 */

	/** 
	 * @ingroup group1
	 * func1 
	 */
	void func1(){}

	/** 
	 * @ingroup group2
	 * func2 
	 */
	void func2(){}

###成员组[memgroup](http://www.stack.nl/~dimitri/doxygen/manual/grouping.html#memgroup)
	/** A class. Details */
	class Test
	{
	  public:
		/** Same documentation for both members. Details */
		void func1InGroup1();
		void func2InGroup1();

		/** Function without group. Details. */
		void ungroupedFunction();
		void func1InGroup2();
	  protected:
		void func2InGroup2();
	};

	/** @name Group1
	 *  Description of group 1. 
	 */
	///@{
	/** Function 2 in group 2. Details. */
	void Test::func1InGroup1() {}
	void Test::func2InGroup1() {}
	///@}

	/** @name Group2
	 *  Description of group 2. 
	 */
	/** @{ */
	/** Function 2 in group 2. Details. */
	void Test::func2InGroup2() {}
	/** Function 1 in group 2. Details. */
	void Test::func1InGroup2() {}
	/** @} */

在类里面，public，private等本身就是一种"成员组"，我们可以自定义成员组。成员组使用[@name](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdname)命名，也可以匿名。然后使用"/\*\* @{ \*/"将组内的元素括起来。

若名下的所有元素都属于一个public/private/protected组，那么这个新组会变成前者的一个子组，否则会创建一个和前者齐平的组。

###子页面[Subpaging](http://www.stack.nl/~dimitri/doxygen/manual/grouping.html#subpaging)

	/**
	@mainpage A simple manual

	Some general info.

	This manual is divided in the following sections:
	- @subpage intro
	- @subpage advanced "Advanced usage"
	*/

	//-----------------------------------------------------------

	/** @page intro Introduction
	This page introduces the user to the topic.
	Now you can proceed to the \ref advanced "advanced section".
	*/

	//-----------------------------------------------------------

	/** @page advanced Advanced Usage
	This page is for advanced users.
	Make sure you have first read \ref intro "the introduction".
	*/
	
[@mainpage](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdmainpage)即主页，在第一页显示,在里面可以用[@subpage](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdsubpage)指向[@page](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdpage)。(*经笔者测试，subpage亦可指向group，见[@addtogroup](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdaddtogroup)*)。

通过@mainpage和@subpage，我们可以建立一颗树。

除了根节点（@mainpage）之外，每一个节点都是一个@page。通过@subpage链接。若不想单独搞个page，也可以用[@section](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdsection)

除了使用@subpage，我们也可以使用[@ref](http://www.stack.nl/~dimitri/doxygen/manual/commands.html#cmdref)

##[输出](http://www.stack.nl/~dimitri/doxygen/manual/output.html)

doxygen可直接输出以下格式

1. **HTML**: Generated if **GENERATE_HTML** is set to YES in the configuration file.
1. **Latex**: Generated if **GENERATE_LATEX** is set to YES in the configuration file.
1. **Man**: pages Generated if **GENERATE_MAN** is set to YES in the configuration file. 
1. **RTF**: Generated if **GENERATE_RTF** is set to YES in the configuration file.Note that the RTF output probably only looks nice with Microsoft's Word. If you have success with other programs, please let me know.
1. **XML**: Generated if **GENERATE_XML** is set to YES in the configuration file.
1. **Docbook**: Generated if **GENERATE_DOCBOOOK** is set to YES in the configuration file.

其他一些格式则需要依赖与第三方的工具，最常用的是chm格式。

###chm
为了将html编译成chm，首先需安装hhc工具，下载地址<http://download.microsoft.com/download/0/A/9/0A939EF6-E31C-430F-A3DF-DFAE7960D564/htmlhelp.exe>。安装后配置

	CHM_INDEX_ENCODING     = GBK
	HHC_LOCATION           = "C:/Program Files (x86)/HTML Help Workshop/hhc.exe"
	GENERATE_HTMLHELP      = YES
	
运行doxygen，就会在html目录生成一个index.chm文件。

为了编译正常，需要下载windows原生的doxygen，下载地址<http://ftp.stack.nl/pub/users/dimitri/doxygen-1.8.4.windows.x64.bin.zip>

cygwin版本的也可以，暂无发现问题，配置和上面一样，或者将HHC_LOCATION改成如下路径

	HHC_LOCATION = "/cygdrive/c/Program Files (x86)/HTML Help Workshop/hhc.exe"
