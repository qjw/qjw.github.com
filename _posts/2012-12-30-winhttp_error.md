---
layout: post
title:  winhttp的陷阱
category: network
---
         
WinHttpOpenRequest用于打开一个Http请求，原型如下：

	HINTERNET WINAPI WinHttpOpenRequest(
	  _In_  HINTERNET hConnect,
	  _In_  LPCWSTR pwszVerb,
	  _In_  LPCWSTR pwszObjectName,
	  _In_  LPCWSTR pwszVersion,
	  _In_  LPCWSTR pwszReferrer,
	  _In_  LPCWSTR *ppwszAcceptTypes,
	  _In_  DWORD dwFlags
	);
	
其中第二个参数MSDN有如下说明

	pwszVerb [in]
	Pointer to a string that contains the HTTP verb to use in the request. 
	If this parameter is NULL, the function uses GET as the HTTP verb.
	
	Note  This string should be all uppercase. Many servers treat HTTP verbs 
	as case-sensitive, and the Internet Engineering Task Force (IETF) 
	Requests for Comments (RFCs) spell these verbs using uppercase characters only.
	
即这个字符串必须**全部大写**，否则会出现某些兼容性问题。
	
##参考
1. <http://msdn.microsoft.com/en-us/library/windows/desktop/aa384099(v=vs.85).aspx>