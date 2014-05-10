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


##参考
1. <http://www.cnblogs.com/kzang/archive/2012/12/02/2798556.html>
1. <http://www.cnblogs.com/lucc/archive/2010/11/24/1886087.html>
1. <http://blog.csdn.net/catxl313/article/details/2204541>

