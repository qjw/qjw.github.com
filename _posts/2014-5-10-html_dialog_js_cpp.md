---
layout: post
title: CHtmlDialog C++调用js函数
category: windows
---

##简单的调用
通常的静态全局js函数调用比较简单

	<script language="javascript" type="text/javascript">
	function Test()
	{
	    alert("你调用了Test");
	}
	</script>

---

	HRESULT CallJSFunction(                                                         
		CString bstrFunc,
		DISPPARAMS dispParams,
		VARIANT* varResult)
	{                                                                                  
	    if( m_spHtmlDoc==NULL )                                                        
	    {                                                                              
		return S_FALSE;                                                            
	    }                                                                              
		                                                                           
	    CComPtr<IDispatch> spScript;                                                   
	    HRESULT hr = m_spHtmlDoc->get_Script(&spScript);                               
	    if(FAILED(hr))                                                                 
	    {                                                                              
		return hr;                                                                 
	    }                                                                              
		                                                                           
	    DISPID dispid = NULL;                                                          
	    hr = spScript->GetIDsOfNames(IID_NULL,&bstrFunc,1,                             
		    LOCALE_SYSTEM_DEFAULT,&dispid);                                        
	    if(FAILED(hr))                                                                 
	    {                                                                              
		return S_FALSE;                                                            
	    }                                                                              
		                                                                           
	    hr = spScript->Invoke(dispid,                                                  
		    IID_NULL,                                                              
		    LOCALE_USER_DEFAULT,                                                   
		    DISPATCH_METHOD,                                                       
		    &dispParams,                                                           
		    varResult,                                                             
		    NULL,                                                                  
		    NULL);                                                                 
	    return hr;                                                                  
	}  


##动态函数

若js函数动态生成，例如某全局对象中的某个成员函数，或者嵌套更深

	<script language="javascript" type="text/javascript">
	var obj={
		subobj:{
			func:function()
			{
			}
		}
	}
	</script>

---

	DISPID CWebBrowserBase::FindId(IDispatch *pObj, LPOLESTR pName)
	{
	    DISPID id = 0;
	    if(FAILED(pObj->GetIDsOfNames(IID_NULL,&pName,1,LOCALE_SYSTEM_DEFAULT,&id))) id = -1;
	    return id;
	}

	HRESULT CWebBrowserBase::GetProperty(IDispatch *pObj, LPOLESTR pName, VARIANT *pValue)
	{
	    DISPID dispid = FindId(pObj, pName);
	    if(dispid == -1) return E_FAIL;

	    DISPPARAMS ps;
	    ps.cArgs = 0;
	    ps.rgvarg = NULL;
	    ps.cNamedArgs = 0;
	    ps.rgdispidNamedArgs = NULL;

	    return pObj->Invoke(dispid, IID_NULL, LOCALE_SYSTEM_DEFAULT, DISPATCH_PROPERTYGET, &ps, pValue, NULL, NULL);
	}

	HRESULT CWebBrowserBase::InvokeMethod(IDispatch *pObj, LPOLESTR pName, VARIANT *pVarResult, VARIANT *p, int cArgs)
	{
	    DISPID dispid = FindId(pObj, pName);
	    if(dispid == -1) return E_FAIL;

	    DISPPARAMS ps;
	    ps.cArgs = cArgs;
	    ps.rgvarg = p;
	    ps.cNamedArgs = 0;
	    ps.rgdispidNamedArgs = NULL;

	    return pObj->Invoke(dispid, IID_NULL, LOCALE_SYSTEM_DEFAULT, DISPATCH_METHOD, &ps, pVarResult, NULL, NULL);
	}

	//获取globalObject
	VARIANT globalObject;
	CWebBrowserBase::GetProperty(pHtmlWindow, L"globalObject", &globalObject);
	//调用globalObject.Test
	CWebBrowserBase::InvokeMethod(globalObject.pdispVal, L"Test", &ret, params, 0);

##Tab键和粘贴
在某些系统中，快捷键无法使用，这是因为外部对话框直接“拿走”了这些快捷键,参考<http://www.experts-exchange.com/Software/Internet_Email/Web_Browsers/Q_24711581.html>，<http://blog.csdn.net/tr0j4n/article/details/4565953>.

####WebBrowser   Keystroke   Problems     
lot   of   people   have   noticed   that   when   they're   hosting   the   WebBrowser   control   in   MFC,   ATL,   or   standard   C++   applications,   they   sometimes   run   into   trouble   with   certain   keys.   These   keystroke   problems   usually   occur   when   typing   into   intrinsic   controls   such   as   text   boxes   that   reside   on   Web   pages   loaded   by   the   WebBrowser   control.   Usually   these   keys   are   accelerator   keys   such   as   Backspace,   Delete,   and   Tab.   The   problem   is   that   the   intrinsic   controls   on   a   Web   page   do   not   automatically   receive   these   accelerator   keys.   When   the   WebBrowser   control   receives   an   accelerator   key   message,   it   does   not   automatically   pass   it   to   child   controls   on   a   Web   page.   Therefore,   you   must   somehow   let   the   WebBrowser   control   know   that   it   should   pass   these   messages   to   controls   on   your   Web   page.           The   solution   is   always   the   same   whether   you   are   hosting   the   control   in   MFC,   ATL,   or   standard   C++:   call   the   TranslateAcclerator   method   of   the   IOleInPlaceActiveObject   interface   that   is   implemented   by   the   WebBrowser   control.   But   where   and   how   you   should   do   this   is   often   unclear.   Let's   see   how   to   work   around   keystroke   problems   in   MFC,   ATL,   and   standard   C++   applications   that   are   hosting   the   WebBrowser   control.   

具体改法是在MFC的PreTranslateMessage加入如下代码

	BOOL   CMyView::PreTranslateMessage(MSG*   pMsg)   
	{   
	    if   (IsDialogMessage(pMsg))   
		    return   TRUE;   
	    else   
		    return   CWnd::PreTranslateMessage(pMsg);   
	}   

##参考
1. <http://www.cnblogs.com/kzang/archive/2012/12/02/2798556.html>
1. <http://www.cnblogs.com/lucc/archive/2010/11/24/1886087.html>
1. <http://blog.csdn.net/catxl313/article/details/2204541>

