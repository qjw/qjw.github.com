---
layout: post
title:  DBUS学习笔记之 DBusConnection
category: network
---

DBusConnection是Dbus对连接的抽象，站在socket的角度上，我们可以理解为一条TCP连接，通过它可以发送和接收消息。

DBusConnection维护着两条队列，发送队列和接收队列。

##生命周期
通常我们使用**[dbus_bus_get](http://dbus.freedesktop.org/doc/api/html/group__DBusBus.html#ga77ba5250adb84620f16007e1b023cf26)**来创建一个连接，第一个参数我们指定类型，

1. **DBUS_BUS_SESSION** ：The login session bus.
1. **DBUS_BUS_SYSTEM** ：The systemwide bus.

和dbus_bus_get相比，**[dbus_bus_get_private](http://dbus.freedesktop.org/doc/api/html/group__DBusBus.html#ga9c62186f19cf3bd3c7c604bdcefb4e09)**总是创建一个全新的DBusConnection，而dbus_bus_get总是去尝试打开一个已有的DBusConnection，并使用引用技术管理。

你也可以使用**[dbus_connection_open](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#gacd32f819820266598c6b6847dfddaf9c)**或者**[dbus_connection_open_private](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga434e3fc7ee420fd30e2f05e57ff26b1d)**打开已有的连接。

和其他DBus对象一样，DBusConnection也使用**引用技术**，所以**[dbus_connection_unref](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga6385ff09bc108238c4429e7c195dab25)**即可销毁连接。

DBusConnection也可以使用DBusMessage类似的**私有数据**，具体见**[dbus_connection_allocate_data_slot](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga728b15c71a492ad244e5a480f1156088)**系列函数。

默认情况下，若DBusConnection收到一个**disconnect信号**，会调用_exit()终止函数，可以使用**[dbus_connection_set_exit_on_disconnect](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga19091beb74f1504b0e862a7ad10e71cd)**修改此行为。

##发送消息
通常我们可以使用**[dbus_connection_send](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#gae1cb64f4cf550949b23fd3a756b2f7d0)**发送消息，不过和send,sendto之类的socket函数不一样的是，它并不会立即发送数据，而只是简单地将Message扔到**发送队列**。待主循环下次运行时被发送出去。为了强制立即发送，可以紧跟着一个**[dbus_connection_flush](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga10e68d9d2f41d655a4151ddeb807ff54)**。

    if( !dbus_connection_send (connection,msg,&serial)){
        fprintf(stderr,"Out of Memory!\n");
        return -1;
    }
    dbus_connection_flush (connection);
	
##接收消息
接收消息根据需要我们可以使用同步或者异步的方式。

通常对于简单的程序，自己维护这消息循环，那么同步方式是最简单可行的方法。例如

	while(1){
		dbus_connection_read_write(connection,0); 
		msg = dbus_connection_pop_message (connection);
	}
	
**[dbus_connection_read_write](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga371163b4955a6e0bf0f1f70f38390c14)**阻塞当前线程，直到有消息过来，读取消息并解析数据成Message并放入接收队列再返回，接下来就使用**[dbus_connection_pop_message](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga1e40d994ea162ce767e78de1c4988566)**从接收队列中读取一条消息。

和dbus_connection_pop_message不同，**[dbus_connection_borrow_message](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga9d07083c520e291591a68adb78f64094)**获得消息之后，并不会从接收队列中删除。

除了dbus_connection_read_write之外，我们也可以使用**[dbus_connection_read_write_dispatch](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga580d8766c23fe5f49418bc7d87b67dc6)**来阻塞并接收消息。和dbus_connection_read_write不同的是，dbus_connection_read_write_dispatch会额外的执行一次Dispatch操作（使用**[dbus_connection_dispatch](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga66ba7df50d75f4bda6b6e942430b81c7)**)。

每一次dbus_connection_read_write_dispatch或者dbus_connection_dispatch调用只会从接收队列中处理**一条Message**。

何为Dispatch呢？我们先了解下Dispatch的[流程](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga66ba7df50d75f4bda6b6e942430b81c7)：

1. First, any method replies are passed to DBusPendingCall or dbus_connection_send_with_reply_and_block() in order to complete the pending method call.
1. Second, any filters registered with dbus_connection_add_filter() are run. If any filter returns DBUS_HANDLER_RESULT_HANDLED then processing stops after that filter.
1. Third, if the message is a method call it is forwarded to any registered object path handlers added with dbus_connection_register_object_path() or dbus_connection_register_fallback().

我们重点关注二三点。第二点是说我们可以注册一些过滤器（函数）来接收消息，这类似于MFC/WTL中的PreTranslateMessage。若在钩子函数中返回**DBUS_HANDLER_RESULT_HANDLED**，那么消息不会继续往下传递。第三点说可以注册一些钩子来关注特定的对象(使用对象路径匹配)。

	DBusHandlerResult filter_func(DBusConnection *c, DBusMessage *msg,void *data) 
	{
		return DBUS_HANDLER_RESULT_HANDLED;
	}
	char *init(void)
	{
		dbus_connection_add_filter(connection, filter_func, NULL, NULL);
	}
	
---

	DBusHandlerResult message_handler(DBusConnection *connection, 
			DBusMessage *message, 
			void *user_data)
	{
	}
	char *dbus_init(void)
	{
		DBusObjectPathVTable dnsmasq_vtable = {NULL, &message_handler, NULL, NULL, NULL, NULL };
		if (!dbus_connection_register_object_path(connection,  DNSMASQ_PATH, 
					&dnsmasq_vtable, NULL))
			return _("could not register a DBus message handler");
	}
	
**[dbus_connection_register_object_path](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#ga24730ca6fd2e9132873962a32df7628c)**用于关注特定的对象，而**[dbus_connection_register_fallback](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#gac4473b37bfa74ccf7459959d27e7bc59)**可以认为是通配符，匹配路径的一个子集。

##异步接收
然而更多的时候，我们希望挂在其他主循环工作，例如一个GUI事件循环，或者一个socket多路IO等待(epoll/select)。此时我们就必须使用异步方式。

具体做法是，使用[dbus_connection_set_watch_functions](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#gaebf031eb444b4f847606aa27daa3d8e6)注册DBusWatch监视函数。当DBus创建句柄（例如socket句柄），那么注册的会调会被触发，此时我们可以将这些句柄加入到我们自己的事件循环中，例如下例中的libevent。

当使用自定义的消息循环监听到IO事件后，我们需要调用**[dbus_watch_handle](http://dbus.freedesktop.org/doc/api/html/group__DBusWatch.html#gac2acdb1794450ac01a43ec4c3e07ebf7)**来通知dbus，这个过程可以理解为“外部接口告诉DBus有数据可以读取，然后Dbus就读取这些数据并解析之，最后塞入接收队列”。

接收之后，我们还要继续处理这些消息。可以简单的使用dbus_connection_read_write。更多时候我们使用dispatch接口（钩子）。

若依赖于定时器，我们还需要实现**[DBusTimeout](http://dbus.freedesktop.org/doc/api/html/group__DBusTimeout.html)**接口。具体使用**[dbus_connection_set_timeout_function](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html#gab3cbc68eec427e9ce1783b25d44fe93c)**。DBusTimeout并非libevent之类的定时器，而是Dbus自己需要使用定时器，但是又不控制主循环，所以只好在需要定时器的时候触发dbus_connection_set_timeout_function的回调，通知上层帮它实现定时器。当上层实现的定时器到期之后，再调用**[dbus_timeout_handle](http://dbus.freedesktop.org/doc/api/html/group__DBusTimeout.html)**通知Dbus，这点和dbus_watch_handle类似。

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>

	#include "event2/event.h"
	#include "event2/event_compat.h"
	#include "event2/event_struct.h"

	#include <dbus/dbus.h>
	#include <unistd.h>

	static struct watch * s_watchs_;
	struct watch {
		DBusWatch *watch;
		struct event event;
		struct watch *next;
		void* data;
	};

	static DBusHandlerResult filter_func(DBusConnection *c, DBusMessage *msg,
			void *data) {
		printf("if: %s, member: %s, path: %s\n",
				dbus_message_get_interface(msg),
				dbus_message_get_member(msg),
				dbus_message_get_path(msg));


		DBusMessageIter arg;
		char * sigvalue;
		if (dbus_message_is_signal(msg, "test.signal.Type", "Test")) {
			if (!dbus_message_iter_init(msg, &arg))
				fprintf(stderr, "Message Has no Param");
			else if (dbus_message_iter_get_arg_type(&arg) != DBUS_TYPE_STRING)
				printf("Param is not string");
			else
				dbus_message_iter_get_basic(&arg, &sigvalue);
			printf("Got Singal with value : %s\n", sigvalue);
		} else if (dbus_message_is_signal(msg, "test.signal.Type1", "Test")) {
			if (!dbus_message_iter_init(msg, &arg))
				fprintf(stderr, "Message Has no Param");
			else if (dbus_message_iter_get_arg_type(&arg) != DBUS_TYPE_STRING)
				printf("Param is not string");
			else
				dbus_message_iter_get_basic(&arg, &sigvalue);
			printf("Got Singal with value1 : %s\n", sigvalue);
		} else {
			printf("fuck\n");
		}

		return DBUS_HANDLER_RESULT_HANDLED;
	}

	static void            udp_cb(int fd, short event, void *argc)
	{
		printf("recv msg '%d'\n",__LINE__);
		struct watch * w = (struct watch*)argc;

		int flags = 0;
		if (event | EV_READ)
			flags |= DBUS_WATCH_READABLE;

		if (event | EV_WRITE)
			flags |= DBUS_WATCH_WRITABLE;

		if (flags != 0)
			dbus_watch_handle(w->watch, flags);

		DBusConnection *connection = (DBusConnection *)w->data;
		if (connection) {
			dbus_connection_ref(connection);
			DBusDispatchStatus ret = 0;
			do
			{
				ret = dbus_connection_dispatch(connection);
			}while ( ret == DBUS_DISPATCH_DATA_REMAINS);
			dbus_connection_unref(connection);
		}
	}

	static dbus_bool_t add_watch(DBusWatch *watch, void *data) {
		int fd = dbus_watch_get_unix_fd(watch);
		int flags = dbus_watch_get_flags(watch);
		printf("add watch %d\n",fd);
		struct watch *w;

		if (!(w = malloc(sizeof(struct watch))))
			return FALSE;

		w->watch = watch;
		w->next = s_watchs_;
		s_watchs_ = w;

		event_set(
				&(w->event),
				dbus_watch_get_unix_fd(watch),
				EV_PERSIST | EV_READ,
				udp_cb,
				w);
		event_add(&(w->event), NULL);

		w->data = data; /* no warning */
		return TRUE;
	}

	static void remove_watch(DBusWatch *watch, void *data) {
		printf("remove watch %d\n",__LINE__);
		struct watch **up, *w, *tmp;

		for (up = &(s_watchs_), w = s_watchs_; w; w = tmp) {
			tmp = w->next;
			if (w->watch == watch) {
				*up = tmp;
				free(w);
			} else
				up = &(w->next);
		}

		w = data; /* no warning */
	}

	void listen_signal() {
		DBusMessage * msg;
		DBusMessageIter arg;
		DBusConnection * connection;
		DBusError err;
		int ret;
		char * sigvalue;

		//步骤1:建立与D-Bus后台的连接
		dbus_error_init(&err);
		connection = dbus_bus_get(DBUS_BUS_SESSION, &err);
		dbus_connection_flush(connection);
		dbus_connection_add_filter(connection, filter_func, NULL, NULL);
		dbus_connection_set_watch_functions(connection, add_watch, remove_watch,
				NULL, connection, NULL);
	}

	int main(int argc, char ** argv) {
		struct event_base *base = event_init();
		listen_signal();
		event_base_dispatch(base);
		return 0;
	}

##调试工具

dbus-send可以用来发送消息和方法调用

	dbus-send  --session --type=signal /test/signal/Object test.signal.Type.Test string:"hello world"
	
---

    //步骤3:发送一个信号
    if((msg = dbus_message_new_signal ("/test/signal/Object","test.signal.Type","Test")) == NULL){                                   
        fprintf(stderr,"Message NULL\n");
        return -1; 
    }   
    //给这个信号（messge）具体的内容 
	const char* sigvalue = "hello world"
    dbus_message_iter_init_append (msg,&arg);
    if(!dbus_message_iter_append_basic (&arg,DBUS_TYPE_STRING,&sigvalue)){
        fprintf(stderr,"Out Of Memory!\n");
        return -1; 
    } 
	
dbus-monitor可以监视总线上流动的消息。


##参考
1. <http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html>
1. <http://stackoverflow.com/questions/9378593/dbuswatch-and-dbustimeout-examples>
	