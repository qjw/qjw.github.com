---
layout: post
title:  DBUS学习笔记之概述
category: network
---

#总体结构
dbus是一个树状结构，每个客户端通过dbus_bus_get请求获取一条连接(**bus**)。我们可以认为就是和后台**dbus-daemon**建立的一条连接，至于连接使用何种方式对上层透明，具体可以是Unix域协议或者TCP，两者可以通过**bus addresses**看出差异，不过对上层而言那仅仅是个字符串。

当我们运行以下命令时，根据输出的信息，我们基本可以判定下层使用Unix域协议，不过我们不用关心这些具体的实现细节。

	~ <root@debian> 11:01:10 $ DBUS_VERBOSE=1 dbus-daemon --session --print-address
	unix:abstract=/tmp/dbus-TtXRxUrtYO,guid=641b6f112873d01e4a4aaec2000007f0

为了区分每个bus，在请求返回后会分配一个随机的名字(**bus addresses**)，为了便于其他客户定位，我们使用**dbus_bus_request_name**请求一个众所周知的名称，这个名称在dbus-daemon范围内必须唯一。

	// "test.singal.dest" 就是新名字
    ret = dbus_bus_request_name(connection, "test.singal.dest", 
            DBUS_NAME_FLAG_REPLACE_EXISTING, &err);

有两种bus，系统bus（**system bus**）和会话bus（**session bus**）。

在每个dbus下，有若干对象（**object**），对象有个路径（**path**），可以理解为文件系统路径，不过它只要求在bus内唯一。对象路径的格式和文件系统一致，例如/test/signal/Object"

每个对象可以有若干接口（**interface**），接口有个名称，在对象内唯一。名称使用单词句号分隔的方式表示，例如"test.signal.type"。

在接口（**interface**）之下，就是一个个具体的成员(**member**)，例如一个事件，通常使用一个字符串来描述成员(**member**)。

#连接
dbus使用[DBusMessage](http://dbus.freedesktop.org/doc/api/html/group__DBusConnection.html)来抽象连接。我们可以通过它接收或发送消息。一个连接可以阻塞当前线程以便等待下一条消息到来，也可以和其他消息循环（Message Loop）一起协同工作，具体见[DBusTimeout](http://dbus.freedesktop.org/doc/api/html/group__DBusTimeout.html)和[DBusWatch](http://dbus.freedesktop.org/doc/api/html/group__DBusWatch.html)。

#消息
dbus使用[DBusConnection](http://dbus.freedesktop.org/doc/api/html/group__DBusMessage.html)来抽象消息。消息包含两种类型，DBUS_MESSAGE_TYPE_METHOD_CALL(**方法调用**)和DBUS_MESSAGE_TYPE_SIGNAL(**事件**)。**消息**就属于前面提到的对象（**object**）

##事件

**事件**可以是一对一，也可以是一对多，这取决与接收方。发送者将事件发送到dbus-daemon，感兴趣的接收方可以设置过滤条件并注册到dbus daemon，当dbus daemon收到消息后就根据已经注册的策略向各个客户端发送消息。这实际上简介地实现广播。

设置过滤条件

	dbus_bus_add_match(connection,"type='signal',interface='test.signal.Type1'",&err); 
	
可以设置多次，结果是**累加**而不是覆盖的。所有的过滤条件通过key/value对，用","分割，并拼成一个字符串设置。

###[过滤条件](http://dbus.freedesktop.org/doc/dbus-specification.html#message-bus-routing-match-rules)

1. **type**,消息类型，包括四种：signal, method_call, method_return, error
1. **interface**,发送或接收方接口
1. **path**,对象路径
1. **member**,函数或者信号名称
1. **sender**,发送方名称
1. 其他参考<http://dbus.freedesktop.org/doc/dbus-specification.html#message-bus-routing-match-rules>

例如以下代码中，**"/test/signal/Object"**就是对象路径**path**，**"test.signal.Type1"**就是接口**interface**。**"Test"**就是信号名称**member**

	msg = dbus_message_new_signal ("/test/signal/Object","test.signal.Type1","Test")
	
当我们使用**dbus_bus_get**函数获取一条connection时，会随机分配一个**bus addresses**。我们可以使用**dbus_bus_request_name**请求分配一个众所周知的名称，这个名称就是上述过滤条件中的**sender**。例如

	dbus_bus_request_name(connection,"test.singal.source",DBUS_NAME_FLAG_REPLACE_EXISTING,&err );
	
##方法调用
**方法调用**只能是一对一。所以和消息相比，它还需要明确地指定接收方。

	msg = dbus_message_new_method_call(
			"test.method.server", // target for the method call(接收方)
			"/test/method/Object", // object to call on
			"test.method.Type", // interface to call on
			"Method"); // method name
			
和**事件**不一样的是，**方法调用**可以是同步的，也就是可以等待对方执行特定的**方法**并返回一个结果。

无论是何种消息，都可以附加**参数**(*这里的参数非函数参数，可以理解为消息的附加数据*)，例如

	// append arguments onto signal
	dbus_message_iter_init_append(msg, &args);
	if (!dbus_message_iter_append_basic(&args, DBUS_TYPE_STRING, &sigvalue)) { 
	  fprintf(stderr, "Out Of Memory!\n"); 
	  exit(1);
	}


#参考
1. <http://www.freedesktop.org/wiki/IntroductionToDBus/>
1. <http://pythonhosted.org/txdbus/dbus_overview.html>
1. <http://dbus.freedesktop.org/doc/dbus-specification.html>
1. <http://hi.baidu.com/zengzhaonong/item/18179f0f704ee7cf9157184f>
	