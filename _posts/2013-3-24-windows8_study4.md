---
layout: post
title:  Windows 8学习笔记4 页面导航
category: wingui
---

##Reference
1. [Programming Windows 8 Apps with HTML, CSS, and JavaScript (Second Preview)](http://blogs.msdn.com/b/microsoft_press/archive/2012/08/20/free-ebook-programming-windows-8-apps-with-html-css-and-javascript-second-preview.aspx)
1. <http://code.msdn.microsoft.com/windowsapps/>

This type of navigation(\<a href\>) presents a few problems for WinRT apps, however. For one, navigating to a wholly new page means a **wholly new script context**—all the JavaScript variables from your previous page will be lost. Sure, you can pass state between those pages, but managing this across an entire app likely hurts performance and can quickly become your least favorite programming activity. It’s better and easier, in other words, for client apps to maintain a consistent in-memory state across pages. 

Also, the nature of the HTML/CSS rendering engine is such that a blank screen appears when switching pages with a hyperlink. Users of web applicationss are accustomed to waiting a bit for a browser to acquire a new page (I’ve found many things to do with 15-second intervals!), but this isn’t an appropriate user experience for a fast and fluid WinRT app. Furthermore, such a transition doesn’t allow animation of various elements on and off the screen, which can help provide a sense of continuity between pages if that fits with your design. 

##[WinJS.UI.Pages](http://msdn.microsoft.com/en-us/library/windows/apps/hh770584.aspx)

####[IPageControlMembers interface](http://msdn.microsoft.com/en-us/library/windows/apps/jj126146.aspx)

<table  style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>Method</th><th>Description</th></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770585.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">error</strong></a>
</td><td>
<p> Called if any error occurs during the processing of the page.</p>
</td></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770587.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">init</strong></a>
</td><td>
<p>Initializes the control before the content of the control is set.
                Use the processed method for any initialization that should be done after the content
                of the control has been set.</p>
</td></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770588.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">load</strong></a>
</td><td>
<p>Creates DOM objects from the content in the  specified URI.   This method is called after the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/jj126158.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">PageControl</strong></a> is defined and before the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770587.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">init</strong></a> method is called. </p>
</td></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770589.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">processed</strong></a>
</td><td>
<p>Initializes the control after the content of the control is set.</p>
</td></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770590.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">ready</strong></a>
</td><td>
<p> Called after all initialization and rendering is complete. At this
                time, the element is ready for use.</p>
</td></tr>
<tr><td>
<a href="http://msdn.microsoft.com/en-us/library/windows/apps/jj851231.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">render</strong></a>
</td><td>
<p>Takes the elements returned by the <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh770588.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">load</strong></a> method and attaches them to the specified element.</p>
</td></tr>
</tbody></table>

1. [WinJS.UI.Fragments](http://msdn.microsoft.com/en-us/library/windows/apps/br229781.aspx) contains a low-level “fragment-loading” API, the use of which is necessary only when you want close control over the process (such as which parts of the HTML fragment get which parent). 
1. [WinJS.UI.Pages](http://msdn.microsoft.com/en-us/library/windows/apps/hh770584.aspx) is a higher-level API intended for general use and employed by the templates. Think of this as a generic wrapper around the fragment loader that lets you easily define a “page control”—simply an arbitrary unit of HTML, CSS, and JS—that you can easily pull into the context of another page as you do other controls.

		// define page
		(function () {
			"use strict";

			WinJS.UI.Pages.define("/pages/home/home.html", {
				// This function is called whenever a user navigates to this page. It
				// populates the page elements with the app's data.
				ready: function (element, options) {
					// TODO: Initialize the page here.
				}
			});
		})();
		
		// reader page
		WinJS.UI.Pages.render("/html/noSelection.html", targetElement);
		
		
These APIs provide only the means to load and unload individual pages—they pull HTML in from other files (along with referenced CSS and JS) and attach the contents to an element in the DOM. That’s it. To actually implement a page-to-page navigation structure, we need two additional pieces: something that manages a navigation stack and something that hooks navigation events to the page-loading mechanism of WinJS.UI.Pages. 

##[WinJS.Navigation](http://msdn.microsoft.com/en-us/library/windows/apps/br229778.aspx)

For the first piece, you can turn to **WinJS.Navigation**, which through about 150 lines of CS101-level code supplies a basic navigation stack. This is all it does. The stack itself is just a list of URIs on top of which **WinJS.Navigation** exposes **state**, **location**, **history**, **canGoBack**, and **canGoForward** 
properties. The stack is manipulated through the **forward**, **back**, and **navigate** methods, and the **WinJS.Navigation** object raises a few events—beforenavigate, navigating, and navigated—to anyone who wants to listen (through addEventListener).

##PageControlNavigator

navigator.js, which implements the PageControlNavigator object that supports the navigation model for the Windows Store app JavaScript templates.

		<body> 
			<div id="contenthost" data-win-control="Application.PageControlNavigator" 
				data-win-options="{home: '/pages/home/home.html'}"></div> 
		</body> 
		
---

		WinJS.UI.Pages.define("/pages/home/home.html", {
			// This function is called whenever a user navigates to this page. It
			// populates the page elements with the app's data.
			ready: function (element, options) {
				// TODO: Initialize the page here.
			},

			updateLayout: function (element, viewState, lastViewState) {
				/// <param name="element" domElement="true" />
				/// <param name="viewState" value="Windows.UI.ViewManagement.ApplicationViewState" />
				/// <param name="lastViewState" value="Windows.UI.ViewManagement.ApplicationViewState" />

				// TODO: Respond to changes in viewState.
			},

			unload: function () {
				// TODO: Respond to navigations away from this page.
			}
		});
		
---

        <header aria-label="Header content" role="banner">
            <button class="win-backbutton" aria-label="Back" disabled type="button"></button>
            <h1 class="titlearea win-type-ellipsis">
                <span class="pagetitle">Photo app sample</span>
            </h1>
        </header>

---

		var nav = WinJS.Navigation;
		args.setPromise(WinJS.UI.processAll().then(function () {
			if (nav.location) {
				nav.history.current.initialPlaceholder = true;
				return nav.navigate(nav.location, nav.state);
			} else {
				return nav.navigate(Application.navigator.home);
			}
		}));