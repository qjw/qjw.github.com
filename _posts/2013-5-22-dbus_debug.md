---
layout: post
title:  dbus调试方法
category: linux
---
	
Dbus通常用于桌面系统，不过在没有X的系统也是可以使用的。桌面系统会在session中自动启动dbus-daemon以及其他必要的工作，而在没有X的系统中，我们就需要自己完成了。

###安装
	apt-get install dbus dbus-x11 libdbus-1-3 libdbus-1-dev

编译时，务必添加"**`pkg-config --cflags --libs dbus-1`**"选项

	~ <root@debian> 10:58:31 $ pkg-config --cflags --libs dbus-1
	-I/usr/include/dbus-1.0 -I/usr/lib/dbus-1.0/include  -ldbus-1 -lpthread -lrt
	
###启动dbus-daemon
运行命令**DBUS_VERBOSE=1 dbus-daemon --session --print-address**

	~ <root@debian> 11:01:10 $ DBUS_VERBOSE=1 dbus-daemon --session --print-address
	unix:abstract=/tmp/dbus-TtXRxUrtYO,guid=641b6f112873d01e4a4aaec2000007f0
	
启动一个新的shell，并导出环境变量DBUS_SESSION_BUS_ADDRESS,它的值就是上述命令的输出

	export DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-TtXRxUrtYO,guid=641b6f112873d01e4a4aaec2000007f0
	
测试代码见[这里](http://www.matthew.ath.cx/misc/dbus) 或 [这里](http://blog.csdn.net/flowingflying/article/details/5449995)

###sample
	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <dbus/dbus.h>
	#include <unistd.h>

	int send_a_signal( char * sigvalue)
	{
		DBusError err;
		DBusConnection * connection;
		DBusMessage * msg;
		DBusMessageIter arg;
		dbus_uint32_t  serial = 0;
		int ret;

		//步骤1:建立与D-Bus后台的连接 
		/* initialise the erroes */ 
		dbus_error_init(&err); 
		/* Connect to Bus*/ 
		connection = dbus_bus_get(DBUS_BUS_SESSION , &err ); 
		if(dbus_error_is_set(&err)){
			fprintf(stderr,"Connection Err : %s\n",err.message);
			dbus_error_free(&err);
		}
		if(connection == NULL)
			return -1;

		//步骤2:给连接名分配一个well-known的名字作为Bus name，
		//这个步骤不是必须的，可以用if 0来注释着一段代码，
		//我们可以用这个名字来检查，是否已经开启了这个应用的另外的进程。 
	#if 1 
		ret = dbus_bus_request_name(connection,"test.singal.source",DBUS_NAME_FLAG_REPLACE_EXISTING,&err ); 
		if(dbus_error_is_set(&err)){
			fprintf(stderr,"Name Err : %s\n",err.message);
			dbus_error_free(&err);
		}
		if(ret != DBUS_REQUEST_NAME_REPLY_PRIMARY_OWNER)
			return -1;
	#endif 

		//步骤3:发送一个信号
		//根据图，我们给出这个信号的路径（即可以指向对象），接口，以及信号名，创建一个Message 
		if((msg = dbus_message_new_signal ("/test/signal/Object","test.signal.Type1","Test")) == NULL){
			fprintf(stderr,"Message NULL\n");
			return -1;
		}
		//给这个信号（messge）具体的内容 
		dbus_message_iter_init_append (msg,&arg);
		if(!dbus_message_iter_append_basic (&arg,DBUS_TYPE_STRING,&sigvalue)){
			fprintf(stderr,"Out Of Memory!\n");
			return -1;
		}

		//步骤4: 将信号从连接中发送 
		if( !dbus_connection_send (connection,msg,&serial)){
			fprintf(stderr,"Out of Memory!\n");
			return -1;
		}
		dbus_connection_flush (connection);
		printf("Signal Send\n");

		//步骤5: 释放相关的分配的内存。 
		dbus_message_unref(msg ); 
		return 0;
	}


	int main( int argc , char ** argv){
		if(argc > 1)
			send_a_signal(argv[1]);
		return 0;
	}
	
---

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <dbus/dbus.h>
	#include <unistd.h>

	void listen_signal()
	{
		DBusMessage * msg;
		DBusMessageIter arg;
		DBusConnection * connection;
		DBusError err;
		int ret;
		char * sigvalue;

		//步骤1:建立与D-Bus后台的连接 
		dbus_error_init(&err);
		connection = dbus_bus_get(DBUS_BUS_SESSION, &err);
		if(dbus_error_is_set(&err)){
			fprintf(stderr,"Connection Error %s\n",err.message);
			dbus_error_free(&err);
		}
		if(connection == NULL)
			return;

		//步骤2:给连接名分配一个可记忆名字test.singal.dest作为Bus name，
		// 这个步骤不是必须的,但推荐这样处理 
		ret = dbus_bus_request_name(connection,
			"test.singal.dest",DBUS_NAME_FLAG_REPLACE_EXISTING,&err);
		if(dbus_error_is_set(&err)){
			fprintf(stderr,"Name Error %s\n",err.message);
			dbus_error_free(&err);
		}
		if(ret != DBUS_REQUEST_NAME_REPLY_PRIMARY_OWNER)
			return;

		//步骤3:通知D-Bus daemon，希望监听来行接口test.signal.Type的信号 
		dbus_bus_add_match(connection,"type='signal',interface='test.signal.Type'",&err); 
		dbus_bus_add_match(connection,"type='signal',interface='test.signal.Type1'",&err); 
		//实际需要发送东西给daemon来通知希望监听的内容，所以需要flush 
		dbus_connection_flush(connection); 
		if(dbus_error_is_set(&err)){
			fprintf(stderr,"Match Error %s\n",err.message);
			dbus_error_free(&err);
		}

		//步骤4:在循环中监听，每隔开1秒，就去试图自己的连接中获取这个信号。
		// 这里给出的是中连接中获取任何消息的方式，
		// 所以获取后去检查一下这个消息是否我们期望的信号，并获取内容。
		// 我们也可以通过这个方式来获取method call消息。 
		while(1){
			dbus_connection_read_write(connection,0); 
			msg = dbus_connection_pop_message (connection);
			if(msg == NULL){
				sleep(1);
				/*printf("fuck continue\n");*/
				continue;
			}

			if(dbus_message_is_signal(msg,"test.signal.Type","Test") ){
				if(!dbus_message_iter_init(msg,&arg) )
					fprintf(stderr,"Message Has no Param");
				else if(dbus_message_iter_get_arg_type(&arg) != DBUS_TYPE_STRING) 
					printf("Param is not string");
				else
					dbus_message_iter_get_basic(&arg,&sigvalue); 
				printf("Got Singal with value : %s\n",sigvalue);
			}
			else if(dbus_message_is_signal(msg,"test.signal.Type1","Test") ){
				if(!dbus_message_iter_init(msg,&arg) )
					fprintf(stderr,"Message Has no Param");
				else if(dbus_message_iter_get_arg_type(&arg) != DBUS_TYPE_STRING) 
					printf("Param is not string");
				else
					dbus_message_iter_get_basic(&arg,&sigvalue); 
				printf("Got Singal with value1 : %s\n",sigvalue);
			}
			else
			{
				printf("fuck\n");
			}
			dbus_message_unref(msg);
		}//End of while

	}

	int main( int argc , char ** argv){
		listen_signal();
		return 0;
	}
	
###参考
1. <http://blog.csdn.net/jack0106/article/details/5588057>
1. <http://www.matthew.ath.cx/misc/dbus>
1. <http://blog.csdn.net/flowingflying/article/details/5449995>