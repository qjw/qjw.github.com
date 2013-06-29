---
layout: post
title:  DBUS学习笔记之 DBusMessage
category: network
---

##生命周期
###创建

	// Constructs a new message of the given message type. 
	DBusMessage * 	dbus_message_new (int message_type)
	
	// Constructs a new message representing a signal emission. 
	DBusMessage * 	dbus_message_new_signal (const char *path, const char *interface, const char *name)
	
	// Constructs a new message to invoke a method on a remote object. 
	DBusMessage * 	dbus_message_new_method_call (const char *destination, 
		const char *path, const char *interface, const char *method)
	
	// Constructs a message that is a reply to a method call. 
	DBusMessage * 	dbus_message_new_method_return (DBusMessage *method_call)
	// Creates a new message that is an error reply to another message.
	
	DBusMessage * 	dbus_message_new_error (DBusMessage *reply_to, 
		const char *error_name, const char *error_message)
	
	// Creates a new message that is an error reply to another message, 
	// allowing you to use printf formatting.
	DBusMessage * 	dbus_message_new_error_printf (DBusMessage *reply_to, 
		const char *error_name, const char *error_format,...)

###从DBusConnection中获取
通常对于发送的DBusMessage，我们需要明确的创建并设置适当的属性，对于接收的消息，直接使用DBusConnection的方法就可以了。例如
	
	DBusMessage * msg;
    DBusConnection * connection;
	while (1) {
        dbus_connection_read_write(connection, 0);
        msg = dbus_connection_pop_message(connection);
	}
		 
###引用技术
DBUS中很多对象都使用应用技术来管理，DBusMessage也不例外。当dbus_message_unref之后，引用技术变成0，那么该对象就被销毁。

	// Increments the reference count of a DBusMessage. 
	DBusMessage * 	dbus_message_ref (DBusMessage *message)
	
	// Decrements the reference count of a DBusMessage, freeing the message if the count reaches 0. 
	void 	dbus_message_unref (DBusMessage *message)

###其他
	// 拷贝消息
	DBusMessage * 	dbus_message_copy (const DBusMessage *message)
	
	
##消息类型

1. **DBUS_MESSAGE_TYPE_SIGNAL** : 事件
1. **DBUS_MESSAGE_TYPE_METHOD_CALL** ： 方法调用
1. **DBUS_MESSAGE_TYPE_METHOD_RETURN** ： 方法返回
1. **DBUS_MESSAGE_TYPE_ERROR** ： 错误

前面提到，消息只有两种，实际上，**DBUS_MESSAGE_TYPE_METHOD_RETURN**和**DBUS_MESSAGE_TYPE_ERROR**是可以和**DBUS_MESSAGE_TYPE_METHOD_CALL**归为一大类的。

	// 获得消息类型 
	int 	dbus_message_get_type (DBusMessage *message)、
	
	// Checks whether the message is a method call with the given interface and member fields.
	dbus_bool_t 	dbus_message_is_method_call (DBusMessage *message, 
		const char *interface, const char *method)
	
	// Checks whether the message is a signal with the given interface and member fields.
	dbus_bool_t 	dbus_message_is_signal (DBusMessage *message, 
		const char *interface, const char *signal_name)
	
	// Checks whether the message is an error reply with the given error name. 
	dbus_bool_t 	dbus_message_is_error (DBusMessage *message, const char *error_name)
		

##基本属性
###Path

	// Sets the object path this message is being sent to (for DBUS_MESSAGE_TYPE_METHOD_CALL)
	// the one a signal is being emitted from (for DBUS_MESSAGE_TYPE_SIGNAL). 
	dbus_bool_t 	dbus_message_set_path (DBusMessage *message, const char *object_path)
	
 	// Gets the object path this message is being sent to (for DBUS_MESSAGE_TYPE_METHOD_CALL)
	// being emitted from (for DBUS_MESSAGE_TYPE_SIGNAL). 
	char * 	dbus_message_get_path (DBusMessage *message)
	
 	// Checks if the message has a particular object path. 
	dbus_bool_t 	dbus_message_has_path (DBusMessage *message, const char *path)
	
 	// Gets the object path this message is being sent to (for DBUS_MESSAGE_TYPE_METHOD_CALL)
	// being emitted from (for DBUS_MESSAGE_TYPE_SIGNAL) in a decomposed format (one array element per path component). 
	dbus_bool_t 	dbus_message_get_path_decomposed (DBusMessage *message, char ***path)
 	
###Interface

 	// Sets the interface this message is being sent to (for DBUS_MESSAGE_TYPE_METHOD_CALL)
	// the interface a signal is being emitted from (for DBUS_MESSAGE_TYPE_SIGNAL). 
	dbus_bool_t 	dbus_message_set_interface (DBusMessage *message, const char *interface)
	
 	// Gets the interface this message is being sent to (for DBUS_MESSAGE_TYPE_METHOD_CALL)
	// being emitted from (for DBUS_MESSAGE_TYPE_SIGNAL). 
	const char * 	dbus_message_get_interface (DBusMessage *message)
	
 	// Checks if the message has an interface. 
	dbus_bool_t 	dbus_message_has_interface (DBusMessage *message, const char *interface)
	
###Member
 	// Sets the interface member being invoked (DBUS_MESSAGE_TYPE_METHOD_CALL)
	// emitted (DBUS_MESSAGE_TYPE_SIGNAL). 
	dbus_bool_t 	dbus_message_set_member (DBusMessage *message, const char *member)
	
 	// Gets the interface member being invoked (DBUS_MESSAGE_TYPE_METHOD_CALL)
	// emitted (DBUS_MESSAGE_TYPE_SIGNAL). 
	const char * 	dbus_message_get_member (DBusMessage *message)

 	// Checks if the message has an interface member. 
	dbus_bool_t 	dbus_message_has_member (DBusMessage *message, const char *member)

###发送接收方
对于方法调用，需要明确地指定发送接收方

 	// Checks whether the message has the given unique name as its sender. 
	dbus_bool_t 	dbus_message_has_sender (DBusMessage *message, const char *name)
	
 	// Sets the message sender. 
	dbus_bool_t 	dbus_message_set_sender (DBusMessage *message, const char *sender)
	
 	// Gets the unique name of the connection which originated this message, or NULL if unknown or inapplicable. 
	const char * 	dbus_message_get_sender (DBusMessage *message)
	
 	// Checks whether the message was sent to the given name. 
	dbus_bool_t 	dbus_message_has_destination (DBusMessage *message, const char *name)
	
	dbus_bool_t 	dbus_message_set_destination (DBusMessage *message, const char *destination)
 	// Sets the message's destination. 
	
 	// Gets the destination of a message or NULL if there is none set. 
	const char * 	dbus_message_get_destination (DBusMessage *message)
	
dbus_message_new函数很少用，不过在使用它创建DBusMessage之后，在配合上述属性设置的方法，也可以实现dbus_message_new_signal等函数相同的效果。

##私有数据
DBusMessage可以携带一些私有数据在各个程序逻辑中传递，方便管理（*注意和参数相区分，后者会被发送到连接对端*）

	// Allocates an integer ID to be used for storing application-specific data on any DBusMessage. 
	dbus_bool_t 	dbus_message_allocate_data_slot (dbus_int32_t *slot_p)

	// Deallocates a global ID for message data slots. 
	void 	dbus_message_free_data_slot (dbus_int32_t *slot_p)

	// Stores a pointer on a DBusMessage, along with an optional function to be used 
	// for freeing the data when the data is set again, or when the message is finalized. 
	dbus_bool_t 	dbus_message_set_data (DBusMessage *message, 
		dbus_int32_t slot, void *data, DBusFreeFunction free_data_func)
		
	// Retrieves data previously set with dbus_message_set_data(). 
	void * 	dbus_message_get_data (DBusMessage *message, dbus_int32_t slot)

##参数

	#include <stdio.h>
	#include <stdlib.h>
	#include <string.h>
	#include <dbus/dbus.h>
	#include <unistd.h>

	int main()
	{
		DBusMessage * msg;
		msg = dbus_message_new_signal(
					"/test/signal/Object1fasdf",
					"test.signal.Type", "Test");


		//给这个信号（messge）具体的内容
		{
			DBusMessageIter iter;
			DBusMessageIter subiter;
			dbus_message_iter_init_append(msg, &iter);
			// 基本类型，字符串
			const char * msg_ = "hello world";
			dbus_message_iter_append_basic(&iter, DBUS_TYPE_STRING, &msg_);

			// 结构
			dbus_message_iter_open_container(
					&iter,
					DBUS_TYPE_STRUCT,
					NULL,
					&subiter);
			dbus_message_iter_append_basic(&subiter, DBUS_TYPE_STRING, &msg_);
			dbus_message_iter_append_basic(&subiter, DBUS_TYPE_STRING, &msg_);
			dbus_message_iter_close_container(&iter,&subiter);

			// 基本类型 int
			int intval_ = 123456;
			dbus_message_iter_append_basic(&iter, DBUS_TYPE_INT32, &intval_);

			// 数组
			/**
			 * For variants, the contained_signature should be the type of
			 * the single value inside the variant. For structs and dict entries,
			 *  contained_signature should be NULL;
			 *  it will be set to whatever types you write into the struct.
			 *   For arrays, contained_signature should be the type
			 *    of the array elements.
			 */
			dbus_message_iter_open_container(&iter,
					DBUS_TYPE_ARRAY,
					"i", // i 对应 DBUS_TYPE_INT32
					&subiter);
			int array[] = { 1, 2, 3 ,4 ,5};
			int* fuck_ = array;
			dbus_message_iter_append_fixed_array (
					&subiter,
					DBUS_TYPE_INT32,
					&fuck_,
					sizeof(array)/sizeof(array[0]));
			dbus_message_iter_close_container(&iter,&subiter);

			// 基本类型 浮点
			double doubleval_ = 1.234567;
			dbus_message_iter_append_basic(&iter, DBUS_TYPE_DOUBLE, &doubleval_);
		}
		{
			// 遍历参数
			DBusMessageIter iter;
			dbus_message_iter_init(msg, &iter);
			int current_type;
			int cnt_ = 1;
			while ((current_type = dbus_message_iter_get_arg_type(&iter))
					!= DBUS_TYPE_INVALID)
			{
				// 基本类型
				if(DBUS_TYPE_STRING == current_type)
				{
					char* value_;
					dbus_message_iter_get_basic(&iter, &value_);
					printf("iter %d string '%s'\n",cnt_,value_);
				}
				else if(DBUS_TYPE_INT32 == current_type)
				{
					int value_;
					dbus_message_iter_get_basic(&iter, &value_);
					printf("iter %d int32 '%d'\n",cnt_,value_);
				}
				else if(DBUS_TYPE_DOUBLE == current_type)
				{
					double value_;
					dbus_message_iter_get_basic(&iter, &value_);
					printf("iter %d int32 '%f'\n",cnt_,value_);
				}
				// 数组类型
				else if(DBUS_TYPE_ARRAY == current_type)
				{
					// 先获取数组元素的类型
					if(DBUS_TYPE_INT32 == dbus_message_iter_get_element_type(&iter))
					{
						// 跳进去
						DBusMessageIter subiter;
						dbus_message_iter_recurse(&iter, &subiter);
						int* array_;
						int array_len_;
						// 获得地址和数组长度
						dbus_message_iter_get_fixed_array(&subiter, &array_,&array_len_);
						printf("iter %d array len '%d' type\n",
								cnt_,
								array_len_);
						int index = 0;
						for(;index < array_len_;index++)
						{
							printf("iter %d array [%d] = '%d'\n",
									cnt_,
									index,
									array_[index]);
						}
					}
					else
					{
						printf("iter %d array type\n");
					}
				}
				// 结构类型
				else if(DBUS_TYPE_STRUCT == current_type)
				{
					printf("iter %d struct type\n");

					// 跳进去
					DBusMessageIter subiter;
					int subcnt_ = 0;
					dbus_message_iter_recurse(&iter, &subiter);

					// 遍历，同上
					int sub_current_type;
					while ((sub_current_type = dbus_message_iter_get_arg_type(&subiter))
							!= DBUS_TYPE_INVALID)
					{
						printf("sub iter %d struct type\n",subcnt_);
						dbus_message_iter_next(&subiter);
						subcnt_ ++;
					}
				}
				else
				{
					printf("iter %d unknown type\n");
				}
				dbus_message_iter_next(&iter);
				cnt_ ++;
			}
		}

		return 0;
	}



##参考
1. <http://dbus.freedesktop.org/doc/api/html/group__DBusMessage.html>
1. <http://hi.baidu.com/9562512/item/f93cac0be4849cdcdce5b076>
1. <http://dbus.freedesktop.org/doc/api/html/group__DBusProtocol.html#ga7eb77066dadf5415896b44c56d93acce>
	