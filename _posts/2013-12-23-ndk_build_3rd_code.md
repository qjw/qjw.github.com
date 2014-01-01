---
layout: post
title: NDK编译libevent等开源库
category: other
---

##简单编译libevent

从**[这里](https://github.com/ventureresearch/libevent)**下载修改后的libevent源码，然后在解压后的目录下创建目录**jni**。将当前目录的**Android.mk**拷贝到jni目录下。修改Android.mk如下：

	LOCAL_PATH := $(call my-dir)
	include $(CLEAR_VARS)
	LOCAL_MODULE:= libevent2
	LOCAL_MODULE_TAGS:= optional
	LOCAL_SRC_FILES := \
		../epoll.c \
		../epoll_sub.c \
		../evdns.c \
		../event.c \
		../event_tagging.c \
		../evmap.c \
		../evrpc.c \
		../evthread.c \
		../evthread_pthread.c \
		../evutil.c \
		../evutil_rand.c \
		../log.c \
		../poll.c \
		../select.c \
		../signal.c \
		../strlcpy.c

	LOCAL_C_INCLUDES := \
		$(LOCAL_PATH) \
		$(LOCAL_PATH)/../android \
		$(LOCAL_PATH)/../include 
	LOCAL_CFLAGS := -DHAVE_CONFIG_H -DANDROID -fvisibility=hidden
	include $(BUILD_STATIC_LIBRARY)

若有编译错误，直接注释以下代码

	#ifdef _EVENT_HAVE_SYS_EVENTFD_H
	//#include <sys/eventfd.h>
	#endif
	
---

	qjw@qjw-PC /cygdrive/d/android/android-ndk-r9c/samples/hello-jni/jni/libevent
	$ $NDK_ROOT/ndk-build
	[armeabi] Compile thumb  : event2 <= event.c
	[armeabi] Compile thumb  : event2 <= event_tagging.c
	[armeabi] Compile thumb  : event2 <= evmap.c
	[armeabi] Compile thumb  : event2 <= evrpc.c
	[armeabi] Compile thumb  : event2 <= evthread.c
	[armeabi] Compile thumb  : event2 <= evthread_pthread.c
	[armeabi] Compile thumb  : event2 <= evutil.c
	[armeabi] Compile thumb  : event2 <= evutil_rand.c
	jni/../evutil_rand.c: In function 'ev_arc4random_buf':
	jni/../evutil_rand.c:62:2: warning: 'return' with a value, in function returning void [enabled by default]
	[armeabi] Compile thumb  : event2 <= log.c
	[armeabi] Compile thumb  : event2 <= poll.c
	[armeabi] Compile thumb  : event2 <= select.c
	[armeabi] Compile thumb  : event2 <= signal.c
	[armeabi] Compile thumb  : event2 <= strlcpy.c
	[armeabi] StaticLibrary  : libevent2.a

##使用libevent

修改 /cygdrive/d/android/android-ndk-r9c/samples/hello-jni/jni/Android.mk

	LOCAL_PATH := $(call my-dir)
	include $(CLEAR_VARS)    ####
	LOCAL_MODULE    := event2   ####  
	LOCAL_SRC_FILES := $(LOCAL_PATH)/libevent/obj/local/armeabi/libevent2.a   ####
	include $(PREBUILT_STATIC_LIBRARY)   ####

	include $(CLEAR_VARS)

	LOCAL_C_INCLUDES := $(LOCAL_PATH)/libevent/include \ ####
		$(LOCAL_PATH)/libevent/android ####
	LOCAL_MODULE    := hello-jni
	LOCAL_SRC_FILES := hello-jni.c

	LOCAL_STATIC_LIBRARIES := event2   ####
	
##CPP支持
若使用诸如protobuf等c++实现的库，使用时，需要在**Application.mk** 增加以下内容

	APP_STL := gnustl_static
	
同时可以根据需要在**Android.mk**增加以下内容

	LOCAL_CPPFLAGS += -frtti -std=c++11 -fexceptions

	
增加代码/cygdrive/d/android/android-ndk-r9c/samples/hello-jni/jni/hello-jni.c

	#include <event2/event.h>
	#include <signal.h>

	void fuck()
	{
		struct event_base *base;
		struct event *signal_event;
		base = event_base_new();
		signal_event = evsignal_new(base, SIGINT, NULL, (void *)base);
		event_base_dispatch(base);
		event_free(signal_event);
		event_base_free(base);
	}
	
##通用的办法

运行以下命令生成独立的toolchain

	$NDK_ROOT/build/tools/make-standalone-toolchain.sh \
		--platform=android-19 \
		--install-dir=/cygdrive/d/android/toolchain \
		--system=windows-x86_64
		
cd到libevent官方源码的根目录，运行以下命令

	./configure --host=arm \
		CC=/cygdrive/d/android/toolchain/bin/arm-linux-androideabi-gcc.exe \
		CPP=/cygdrive/d/android/toolchain/bin/arm-linux-androideabi-cpp.exe \
		CXX=/cygdrive/d/android/toolchain/bin/arm-linux-androideabi-g++.exe
	make
	
编译结束后，会在.lib目录下生成libevent_core.a和libevent.a等一堆库文件。

##参考
1. <http://blog.csdn.net/wjr2012/article/details/6887559>
1. <http://stackoverflow.com/questions/11655911/cross-compiling-libevent-for-android>
1. <https://github.com/ventureresearch/libevent>
1. <http://jituo666.blog.163.com/blog/static/2948172120120423236660/>
1. <http://blog.csdn.net/smfwuxiao/article/details/6587709>