---
layout: post
title:  Windows 8学习笔记1 本地/Web会话
category: wingui
---

##Reference
1. [Programming Windows 8 Apps with HTML, CSS, and JavaScript (Second Preview)](http://blogs.msdn.com/b/microsoft_press/archive/2012/08/20/free-ebook-programming-windows-8-apps-with-html-css-and-javascript-second-preview.aspx)
1. <http://code.msdn.microsoft.com/windowsapps/>
1. <http://palermo4.com/post/How-to-Use-IFrames-in-WinRT-Apps.aspx>

##Local and Web Contexts within the App Host
1. HTML content in the app package can be loaded into the **local or web context**
1. The local context has access to the **WinRT API**, among other things, whereas the web context is allowed to load and execute **remote script** but cannot access WinRT.
1. Remote content (referred to with **http[s]://**) always runs in the web context. 
1. **ms-appx:///** always runs in the local context.
1. **ms-appx-web:///** always runs in the web context.
1. The **//**’s in the WinJS paths refer to shared libraries rather than files in you app package, whereas a **single /** refers to the root of your package. 
1. an app’s **home page** always runs in the local context, and any page to which you **navigate directly** (\<a href\> or document.location) must also be in the local context. 
1. Within the local context of an app, **ms-appdata** is a shortcut to the appdata folder wherein exist local, roaming, and temp folders.ms-appdata can be used only for **resources**, namely with the src attribute of img, video, and audio 
elements. 

##Use iFrames in WinRT Apps
1. A local context page can contain an iframe in either local or web context

		<iframe src="ms-appx-web:///web.html" width="300" height="400">
		<!-- running in web context -->
		</iframe>
		<iframe src="http://web.html" width="300" height="400">
		<!-- running in web context -->
		</iframe>
		<iframe src="/web.html" width="300" height="400">
		<!-- running in local context -->
		</iframe>
		<iframe src="ms-appx:///web.html" width="300" height="400">
		<!-- running in local context -->
		</iframe>
		
		
		<!-- iframe in local context with source in the app package --> 
		<!-- this form is only allowed from inside the local context --> 
		<iframe src=" /frame-local.html"></iframe> 
		<iframe src="ms-appx:///frame-local.html"></iframe> 
		 
		<!-- iframe in web context with source in the app package --> 
		<iframe src="ms-appx-web:///frame-web.html"></iframe> 
		 
		<!-- iframe with an external source automatically assigns web context --> 
		<iframe src="http://www.bing.com"></iframe>

		
1. if you use an \<a href="..." target="..."\> tag with target pointing to an iframe, the 
scheme in href determines the context.

		<h1>iFrame Demo</h1>
		<div>
			<a href="http://palermo4.com" target="framed">Palermo4</a> 
			<a href="http://codefoster.com" target="framed">Codefoster</a>
		</div>
		<iframe name="framed" width="900" height="600">
		</iframe>

1. navigating from a web context page to a local context page is **not allowed by default**, but you can enable this by calling the super-secret function [MSApp.addPublicLocalApplicationUri](http://msdn.microsoft.com/en-us/library/windows/apps/hh465759.aspx) (from code in a **local page**)

		//This must be called from the local context 
		MSApp.addPublicLocalApplicationUri("ms-appx:///frame-local.html"); 
