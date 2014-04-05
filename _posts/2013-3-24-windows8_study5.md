---
layout: post
title:  Windows 8学习笔记5 基本控件
category: wingui
---

##Reference
1. [Programming Windows 8 Apps with HTML, CSS, and JavaScript (Second Preview)](http://blogs.msdn.com/b/microsoft_press/archive/2012/08/20/free-ebook-programming-windows-8-apps-with-html-css-and-javascript-second-preview.aspx)
1. <http://code.msdn.microsoft.com/windowsapps/>

##Controls
1. You can use those intrinsic **HTML controls** in a Windows 8 app because those apps run on top of the same HTML/CSS rendering engine as Internet Explorer.
1. WinJS and custom controls, lacking the benefit of existing standards, are declared using some root element, typically a **\<div\>** or **\<span\>**, with two custom data-* attributes: **data-win-control** and **data-win-options**. The value of data-win-control specifies the fully qualified name of a public constructor function that creates the actual control as child elements of the root. The second, **data-win-options**, is a JSON string containing key-value pairs separated by commas: **{ \<key1\>: \<value1\>, \<key1\>: \<value2\>, ... }**. 
1. because data-* attributes are, according to the HTML5 specifications, completely ignored by the HTML/CSS rendering engine, some additional processing is necessary to turn an element with these attributes into an actual control in the DOM. And this, as I’ve hinted at before, is exactly the life purpose of the **WinJS.UI.process** and **WinJS.UI.processAll** methods. 

		args.setPromise(WinJS.UI.processAll().then(function () {
			if (nav.location) {
				nav.history.current.initialPlaceholder = true;
				return nav.navigate(nav.location, nav.state);
			} else {
				return nav.navigate(Application.navigator.home);
			}
		}));
		
1. WinJS comes with two parallel stylesheets that provide many default styles and style classes for WinRT apps: **ui-light.css** and **ui-dark.css**. 
1. As you probably know already, there are many developing standards for HTML and CSS. Until these are brought to completion, implementations of those standards in various browsers are typically made available ahead of time with vendor-prefixed names. In addition, browser vendors sometimes add their own extensions to the DOM API for various elements. With WinRT apps, of course, you don’t need to worry about the variances between browsers, but since these apps essentially run on top of the Internet Explorer engines.

##[WinJS Controls](http://msdn.microsoft.com/en-us/library/windows/apps/br229782.aspx)

Windows 8 defines a number of controls that help apps fulfill WinRT app design guidelines. As noted before, these are implemented in WinJS for WinRT apps written in HTML, CSS, and JavaScript, rather than WinRT; this allows those controls to integrate naturally with other DOM elements. 

WinJS controls declared in markup with data-* attributes are not instantiated until you call **WinJS.UI.process(\<element\>)** for a single control or **WinJS.UI.processAll** for all such elements in the DOM. To understand this process, here’s what WinJS.UI.process does for a single element: 

1.  Parse the data-win-options string into an options object. 
1.  Extract the constructor specified in data-win-control, and then call new on that function passing the root element and the options object. 
1.  The constructor creates whatever child elements it needs within the root element. 
1.  The object returned from the constructor—the control object—is stored in the root element’s winControl property.

WinJS has three DOM-traversing functions: WinJS.UI.processAll, WinJS.Binding.processAll, and WinJS.Resources.processAll. Each of these looks for specific data-win-* attributes and then takes additional actions using those contents. Such actions introduce **a risk of injection attack** if a processAll function is called on untrusted HTML, such as arbitrary markup obtained from the web. To mitigate this risk, WinJS has a notion of strict processing that is enforced within all HTML/JavaScript apps.. The effect of strict processing is that any functions indicated in markup that processAll methods might encounter must be “marked for processing” or else processing will fail. The mark itself is simply a **supportedForProcessing** property on the function object that is set to **true**. 

Functions returned from **WinJS.Class.define, WinJS.Class.derive, WinJS.UI.Pages.define, and WinJS.Binding.converter** are **automatically** marked in this manner. For other functions, you can either set a supportedForProcessing property to true directly or use marking functions like so: 

	WinJS.Utilities.markSupportedForProcessing(myfunction); 
	WinJS.UI.eventHandler(myHandler); 
	WinJS.Binding.initializer(myInitializer);

###Instantiate control

	<div id="rating1" data-win-control="WinJS.UI.Rating" 
		data-win-options="{averageRating: 3.4, userRating: 4, onchange: changeRating}"> 
	</div>
	
---

	WinJS.UI.process(document.getElementById("rating1")); 
	WinJS.UI.processAll(); 
	
---

	var element = document.getElementById("rating1"); 
	WinJS.UI.process(element); 
	element.winControl.averageRating = 3.4; 
	element.winControl.userRating = 4; 
	element.winControl.onchange = changeRating; 
	
---

	var newControl = new WinJS.UI.Rating(document.getElementById("rating1")); 
	newControl.averageRating = 3.4; 
	newControl.userRating = 4; 
	newControl.onchange = changeRating; 
	
---

	var newControl = new WinJS.UI.Rating(null, 
		{ averageRating: 3.4, userRating: 4, onchange: changeRating }); 
	newControl.element.id = "rating1"; 
	document.body.appendChild(newControl.element);


###WinJS.UI.Tooltip

With most of the other simple controls—namely the DatePicker, TimePicker, and ToggleSwitch—you can work with them in the same ways as we just saw with Ratings. All that changes are the specifics of their properties and events; The WinJS.UI.Tooltip control is a little different, however.

	<!-- Directly attach the Tooltip to its target element --> 
	<targetElement data-win-control="WinJS.UI.Tooltip"> 
	</targetElement>

	<!-- Place the element inside the Tooltip --> 
	<span data-win-control="WinJS.UI.Tooltip"> 
		<!-- The element that gets the tooltip goes here --> 
	</span> 
	 
	<div data-win-control="WinJS.UI.Tooltip"> 
		<!-- The element that gets the tooltip goes here --> 
	</div>

Second, the contentElement property of the tooltip control can name another element altogether, which will be displayed when the tooltip is invoked. For example, if we have this piece of hidden HTML in our markup (and notice that it contains other controls): 

	<div style="display: none;"> 
		<!--Here is the content element. It's put inside a hidden container 
		so that it's invisible to the user until the tooltip takes it out.--> 
		<div id="myContentElement"> 
			<div id="myContentElement_rating"> 
				<div data-win-control="WinJS.UI.Rating" class="win-small movieRating" 
					data-win-options="{userRating: 3}"> 
				</div> 
			</div> 
			<div id="myContentElement_description"> 
				<p>You could provide any DOM element as content, even with WinJS controls inside. The tooltip 
	control will re-parent the element to the tooltip container, and block interaction events on that element, 
	since that's not the suggested interaction model.</p> 
			</div> 
			<div id="myContentElement_picture"> 
			</div> 
		</div> 
	</div> 

we can reference it like so: 

	<div data-win-control="WinJS.UI.Tooltip" 
	   data-win-options="{infotip: true, contentElement: myContentElement}"> 
        <button id="helloButton">
            Say "Hello"
        </button>
	</div>


