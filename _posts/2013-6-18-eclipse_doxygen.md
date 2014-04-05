---
layout: post
title:  eclipse中自动生成doxygen帮助
category: other
---

打开首选项**window->preferences->C/C++->Editor->Documentation tool comments**

将**workspace default**改成**Doxygen**。

当新写一个函数后，在函数上面输入**/\*\*** 并按回车，并会自动补齐基本的帮助信息，例如

	/**
	 * 
	 * @param c
	 * @param msg
	 * @param data
	 * @return
	 */
	static DBusHandlerResult filter_func(
			DBusConnection *c,
			DBusMessage *msg,
			void *data)
			
##参考
1. <http://stackoverflow.com/questions/3537542/a-doxygen-eclipse-plugin-with-automatic-code-completion>			