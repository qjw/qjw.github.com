---
layout: post
title:  Windows 8学习笔记3 生命周期
category: wingui
---

##Reference
1. [Programming Windows 8 Apps with HTML, CSS, and JavaScript (Second Preview)](http://blogs.msdn.com/b/microsoft_press/archive/2012/08/20/free-ebook-programming-windows-8-apps-with-html-css-and-javascript-second-preview.aspx)
1. <http://code.msdn.microsoft.com/windowsapps/>
1. <http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.viewmanagement.applicationviewstate>

##App Activation

	(function () { 
		"use strict"; 
	 
		var app = WinJS.Application; 
		var activation = Windows.ApplicationModel.Activation; 
	 
		app.onactivated = function (args) { 
			if (args.detail.kind === activation.ActivationKind.launch) { 
				if (args.detail.previousExecutionState !==  
					activation.ApplicationExecutionState.terminated) { 
					// TODO: This application has been newly launched. Initialize  
					// your application here. 
				} else { 
					// TODO: This application has been reactivated from suspension.  
					// Restore application state here. 
				} 
				args.setPromise(WinJS.UI.processAll()); 
			} 
		}; 
	 
		app.oncheckpoint = function (args) { 
		}; 
	 
		app.start(); 
	})();
	

If the activation kind is **launch** and the previous state is **notrunning** or **closedByUser**, the app should start up with its default UI and apply any persistent settings (such as those in its Settings panel). With closedByUser, there might be scenarios where the app should perform additional actions (such as updating cached data) after the user explicitly closed the app and left it closed for a while. 

If the activation kind is **launch** and the previous state is **terminated**, the app should start up in the same state as when it was last suspended.

For launch and other activation kinds that include additional arguments or parameters (as with secondary tiles, toast notifications, and contracts), it should initialize itself to serve that purpose by using the additional parameters. The app might already be running, so it won’t necessarily initialize its default state again.

	
1. **[args.detail.kind](http://msdn.microsoft.com/en-us/library/windows/apps/windows.applicationmodel.activation.activationkind.aspx)** identifies the type of activation.

<table style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>Member</th><th>Value</th><th>Description</th></tr>
<tr><td><strong>Launch</strong> | <strong>launch</strong></td><td>0</td><td>
<p>The user launched the app or tapped a content tile.</p>
</td></tr>
<tr><td><strong>Search</strong> | <strong>search</strong></td><td>1</td><td>
<p>The user wants to search with the app.</p>
</td></tr>
<tr><td><strong>ShareTarget</strong> | <strong>shareTarget</strong></td><td>2</td><td>
<p>The app is activated as a target for share operations.</p>
</td></tr>
<tr><td><strong>File</strong> | <strong>file</strong></td><td>3</td><td>
<p>An app launched a file whose file type this app is registered to handle.</p>
</td></tr>
<tr><td><strong>Protocol</strong> | <strong>protocol</strong></td><td>4</td><td>
<p>An app launched a URL whose protocol this app is registered to handle.</p>
</td></tr>
<tr><td><strong>FileOpenPicker</strong> | <strong>fileOpenPicker</strong></td><td>5</td><td>
<p>The user wants to pick files that are provided by the app.</p>
</td></tr>
<tr><td><strong>FileSavePicker</strong> | <strong>fileSavePicker</strong></td><td>6</td><td>
<p>The user wants to save a file and selected the app as the location.</p>
</td></tr>
<tr><td><strong>CachedFileUpdater</strong> | <strong>cachedFileUpdater</strong></td><td>7</td><td>
<p>The user wants to save a file that the app provides content management for.</p>
</td></tr>
<tr><td><strong>ContactPicker</strong> | <strong>contactPicker</strong></td><td>8</td><td>
<p>The user wants to pick contacts.</p>
</td></tr>
<tr><td><strong>Device</strong> | <strong>device</strong></td><td>9</td><td>
<p>The app handles AutoPlay.</p>
</td></tr>
<tr><td><strong>PrintTaskSettings</strong> | <strong>printTaskSettings</strong></td><td>10</td><td>
<p>The app handles print tasks.</p>
</td></tr>
<tr><td><strong>CameraSettings</strong> | <strong>cameraSettings</strong></td><td>11</td><td>
<p>The app captures photos or video from an attached camera.</p>
</td></tr>
<tr><td><strong>PickerReturned</strong> | <strong>pickerReturned</strong></td><td>1000</td><td>
<p>Windows Phone only. The application was activated after the completion of a picker.</p>
</td></tr>
</tbody></table>

1. **[args.detail.previousExecutionState](http://msdn.microsoft.com/en-us/library/windows/apps/windows.applicationmodel.activation.applicationexecutionstate.aspx)** identifies the state of the app prior to this activation, which determines whether to reload state.

<table  style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>Member</th><th>Value</th><th>Description</th></tr>
<tr><td><strong>NotRunning</strong> | <strong>notRunning</strong></td><td>0</td><td>
<p>The app is not running.</p>
</td></tr>
<tr><td><strong>Running</strong> | <strong>running</strong></td><td>1</td><td>
<p>The app is running.</p>
</td></tr>
<tr><td><strong>Suspended</strong> | <strong>suspended</strong></td><td>2</td><td>
<p>The app is suspended.</p>
</td></tr>
<tr><td><strong>Terminated</strong> | <strong>terminated</strong></td><td>3</td><td>
<p>The app was terminated after being suspended.</p>
</td></tr>
<tr><td><strong>ClosedByUser</strong> | <strong>closedByUser</strong></td><td>4</td><td>
<p>The app was closed by the user.</p>
</td></tr>
</tbody></table>


##App Suspending

When an app is not **visible** (in any view state), it will be **suspended** after **five seconds** to conserve battery power. This means it remains wholly in memory but won’t be scheduled for CPU time and thus won’t have network or disk activity (except when using specifically allowed background tasks). When this happens, the app receives the **[Windows.UI.WebUI.WebUIApplication.onsuspending](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.webui.webuiapplication.suspending.aspx)** event, which is also exposed through **[WinJS.Application.oncheckpoint](http://msdn.microsoft.com/en-us/library/windows/apps/br229839.aspx)**. Apps must return from this event within the **five-second ** period, or Windows will assume the app is hung and terminate it. 

		function onSuspending(eventArgs) { /* Your code */ }
		// addEventListener syntax
		webUIApplication.addEventListener("suspending", onSuspending);
		webUIApplication.removeEventListener("suspending", onSuspending);
		// - or -
		webUIApplication.onsuspending = onSuspending;
		
		//////////////////
		
		WinJS.Application.addEventListener("checkpoint", listenerName);
		// - or -
		WinJS.Application.oncheckpoint = listenerName;
		
The best practice is actually to save session state **incrementally (as it changes)** to minimize the work needed within the suspending event, because you have only five seconds to do it. 

##App Resuming

If the user switches back to a suspended app, it receives the **[Windows.UI.WebUI.WebUIApplication.onresuming](http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.webui.webuiapplication.resuming.aspx)** event.

		function onResuming(eventArgs) { /* Your code */ }
		 
		// addEventListener syntax
		webUIApplication.addEventListener("resuming", onResuming);
		webUIApplication.removeEventListener("resuming", onResuming);
		// - or -
		webUIApplication.onresuming = onResuming;
		
##App ViewState

<table style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>Member</th><th>Value</th><th>Description</th></tr>
<tr><td><strong>FullScreenLandscape</strong> | <strong>fullScreenLandscape</strong></td><td>0</td><td>
<p>The current app's view is in full-screen (has no snapped app adjacent to it), and has changed to landscape orientation.</p>
</td></tr>
<tr><td><strong>Filled</strong> | <strong>filled</strong></td><td>1</td><td>
<p>The current app's view has been reduced to a partial screen view as the result of another app snapping.</p>
</td></tr>
<tr><td><strong>Snapped</strong> | <strong>snapped</strong></td><td>2</td><td>
<p>The current app's view has been snapped.</p>
</td></tr>
<tr><td><strong>FullScreenPortrait</strong> | <strong>fullScreenPortrait</strong></td><td>3</td><td>
<p>The current app's view is in full-screen (has no snapped app adjacent to it), and has changed to portrait orientation.</p>
</td></tr>
</tbody></table>