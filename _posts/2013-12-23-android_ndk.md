---
layout: post
title: android NDK环境搭建
category: other
---

##下载
1. <https://developer.android.com/sdk/index.html>
1. <http://dl.google.com/android/adt/adt-bundle-windows-x86_64-20131030.zip>
1. <http://developer.android.com/tools/sdk/ndk/index.html>
1. <http://dl.google.com/android/ndk/android-ndk-r9c-windows-x86_64.zip>

其中sdk压缩包已包含了eclipse（集成cdt，adt等插件）

解压到同一目录下（非必需，我这里是D:/android)

	$ tree android/ -L 2
	android/
	├── adt-bundle-windows-x86_64-20131030
	│   ├── eclipse
	│   ├── sdk
	│   └── SDK Manager.exe
	└── android-ndk-r9c
		├── build
		├── docs
		├── documentation.html
		├── find-win-host.cmd
		├── GNUmakefile
		├── ndk-build
		├── ndk-build.cmd
		├── ndk-depends.exe
		├── ndk-gdb
		├── ndk-gdb.py
		├── ndk-gdb-py
		├── ndk-gdb-py.cmd
		├── ndk-stack.exe
		├── ndk-which
		├── platforms
		├── prebuilt
		├── README.TXT
		├── RELEASE.TXT
		├── remove-windows-symlink.sh
		├── samples
		├── sources
		├── tests
		└── toolchains

	12 directories, 16 files

## Java HelloWorld

打开android/adt-bundle-windows-x86_64-20131030/eclipse/eclipse 打开开发环境

菜单File->New->New android Application,一直点击**下一步**，运行即可。

## Hello World

安装cygwin，确保gnu toolchain安装好，建议安装常用工具。

打开cygwin终端，增加以下两行（最后两行）

	qjw@qjw-PC /cygdrive/d
	$ cat ~/.bash_profile | tail
	#   MANPATH="${HOME}/man:${MANPATH}"
	# fi

	# Set INFOPATH so it includes users' private info if it exists
	# if [ -d "${HOME}/info" ]; then
	#   INFOPATH="${HOME}/info:${INFOPATH}"
	# fi

	NDK_ROOT=/cygdrive/d/android/android-ndk-r9c/
	export NDK_ROOT

进入**samples/hello-jni**，运行命令**$NDK_ROOT/ndk-build**

	qjw@qjw-PC /cygdrive/d/android/android-ndk-r9c/samples/hello-jni
	$ $NDK_ROOT/ndk-build
	[armeabi-v7a] Gdbserver      : [arm-linux-androideabi-4.6] libs/armeabi-v7a/gdbserver
	[armeabi-v7a] Gdbsetup       : libs/armeabi-v7a/gdb.setup
	[armeabi] Gdbserver      : [arm-linux-androideabi-4.6] libs/armeabi/gdbserver
	[armeabi] Gdbsetup       : libs/armeabi/gdb.setup
	[x86] Gdbserver      : [x86-4.6] libs/x86/gdbserver
	[x86] Gdbsetup       : libs/x86/gdb.setup
	[mips] Gdbserver      : [mipsel-linux-android-4.6] libs/mips/gdbserver
	[mips] Gdbsetup       : libs/mips/gdb.setup
	[armeabi-v7a] Install        : libhello-jni.so => libs/armeabi-v7a/libhello-jni.so
	[armeabi] Install        : libhello-jni.so => libs/armeabi/libhello-jni.so
	[x86] Install        : libhello-jni.so => libs/x86/libhello-jni.so
	[mips] Install        : libhello-jni.so => libs/mips/libhello-jni.so
	
若生成了动态库就正常了。

## 使用Eclipse

菜单File->Import->Android->Android Project from Existing Code。输入根目录**D:\android\android-ndk-r9c\samples\hello-jni**，adt自动搜索到HelloJni和tests两个andriod项目，我们只需要前者即可。最后finish。

若出现[2013-12-23 22:39:14 - HelloJni] Unable to resolve target 'android-3'之类的错误，查看**project.properties**，看是否存在**target=android-19**，其中后面的值若太小，改大即可。

若修改了C++代码（*这很正常*），那么又需要切到cygwin中，并且重新运行命令**$NDK_ROOT/ndk-build**，生成动态库。然后切回adt运行。

通过查看**$NDK_ROOT/ndk-build**的输出，我们看到每次都生成了4个动态库，分别对应四种CPU类型，为了减少编译时间，我们可以只编译一种。首先查看**Android Virtual Device Manager**，查看当前的模拟器使用的什么**CPU/ABI**。然后jni/Application.mk.将内容修改如下：

	APP_ABI := all
	APP_ABI := armeabi-v7a
	
修改之后我们可以看到

	qjw@qjw-PC /cygdrive/d/android/android-ndk-r9c/samples/hello-jni
	$ $NDK_ROOT/ndk-build -B
	Android NDK: WARNING: APP_PLATFORM android-19 is larger than android:minSdkVersion 3 in ./AndroidManifest.xml
	[armeabi-v7a] Gdbserver      : [arm-linux-androideabi-4.6] libs/armeabi-v7a/gdbserver
	[armeabi-v7a] Gdbsetup       : libs/armeabi-v7a/gdb.setup
	[armeabi-v7a] Cygwin         : Generating dependency file converter script
	[armeabi-v7a] Compile thumb  : hello-jni <= hello-jni.c
	[armeabi-v7a] SharedLibrary  : libhello-jni.so
	[armeabi-v7a] Install        : libhello-jni.so => libs/armeabi-v7a/libhello-jni.so

## 集成
打开CPP文件，发现头文件无法索引，单文件确实存在。

	qjw@qjw-PC /cygdrive/d/android/android-ndk-r9c/samples/hello-jni
	$ find ../../ -name jni.h
	../../platforms/android-12/arch-arm/usr/include/jni.h
	../../platforms/android-12/arch-mips/usr/include/jni.h
	../../platforms/android-12/arch-x86/usr/include/jni.h

将HelloJni转成C++项目，File->new->other->Convert to a c/c++ Project，由于会默认使用cygwin toolchain，除了jni.h之外的头文件可以解析。

打开Eclipse  Window->Preferences->C/C++->Build->Environment增加两个环境变量。

**C_INCLUDE_PATH/CPLUS_INCLUDE_PATH**等于(*路径根据需要修改*)
D:\android\android-ndk-r9c\platforms\android-19\arch-arm\usr\include

这样头文件就可以解析了，不过由于默认使用cygwin的编译规则，所以编译时会出错，

打开**工程属性页**->**C/C++ Build**，取消**use default build command**,然后输入**D:\cygwin\bin\bash.exe --login -c "NDK=/cygdrive/d/android/android-ndk-r9c && cd $NDK/samples/hello-jni && $NDK/ndk-build"**，这样就可以直接编译了。

然后run->run as->android application

##其他一些环境变量

	_CYGBIN=D:\cygwin
	NDK_ROOT=D:\android\android-ndk-r9c
	ANDROID_SDK=D:\android\adt-bundle-windows-x86_64-20131030\sdk\platforms;\
		D:\android\adt-bundle-windows-x86_64-20131030\sdk\tools;\
		D:\android\adt-bundle-windows-x86_64-20131030\sdk\platform-tools
	CLASSPATH=.;%JAVA_HOME%\lib;
	JAVA_HOME=C:\Program Files\Java\jdk1.7.0_45
	PATH=$PATH;D:\cygwin\bin;%ANDROID_SDK%;C:\Program Files\Java\jdk1.7.0_45\bin

## 参考

1. <http://blog.csdn.net/rexuefengye/article/details/12376037>
1. <http://software.intel.com/en-us/articles/using-the-android-x86-ndk-with-eclipse-and-porting-an-ndk-sample-app>
1. <http://stackoverflow.com/questions/10098049/android-ndk-build-ignoring-app-abi-x86>
1. <http://www.cnblogs.com/luxiaofeng54/archive/2011/08/13/2136982.html>
1. <http://blog.sina.com.cn/s/blog_4cd5d2bb01014tod.html>