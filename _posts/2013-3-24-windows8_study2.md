---
layout: post
title:  Windows 8学习笔记2 Promise
category: wingui
---

##Reference
1. [Programming Windows 8 Apps with HTML, CSS, and JavaScript (Second Preview)](http://blogs.msdn.com/b/microsoft_press/archive/2012/08/20/free-ebook-programming-windows-8-apps-with-html-css-and-javascript-second-preview.aspx)
1. <http://code.msdn.microsoft.com/windowsapps/>
1. <http://msdn.microsoft.com/en-us/library/windows/apps/br211867.aspx>
1. <http://msdn.microsoft.com/en-us/library/windows/apps/hh700330.aspx>


##WinJS.Promise
JavaScript is a single-threaded language. This means that invoking a long-running process blocks all execution until that process completes. UI elements are unresponsive, animations pause, and no other code in the app can run. The solution to this problem is to avoid synchronous execution as much as possible.

One way to do this is to have a function execute at a later time, as with event handlers, which are invoked after another call has raised an event. Callback functions are another kind of asynchronous processing, because they call back into the code that initiated the process.

####Problems when using asynchronous programming
Asynchronous programming can quickly become complicated. Many of the standard JavaScript APIs rely heavily on callbacks, which are often nested, making them difficult to debug. In addition, the use of anonymous inline functions can make reading the call stack problematic. Exceptions that are thrown from within a heavily nested set of callbacks might not be propagated up to a function that initiated the chain. This makes it difficult to determine exactly where a bug is hidden.

All of the Windows Runtime APIs that are exposed to Windows Store apps are wrapped in promise objects. This allows you to use Windows Runtime APIs in a way that you're comfortable with. The Promises in Windows Library for JavaScript are important because many interactions with Windows Runtime are asynchronous.

A promise is an object. The most frequently used method on a promise object is then, which takes three parameters: a function to call when the promise completes successfully, a function to call when the promise completes with an error, and a function to provide progress information. In both the Windows Runtime and the Windows Library for JavaScript you can also use the done function, which takes the same parameters. The difference is that in the case of an error in processing, the then function returns a promise in the error state but does not throw an exception, while the done method throws an exception if an error function is not provided.

	promise.done(onComplete, onError, onProgress);
	promise.then(onComplete, onError, onProgress)
		.done( /* Your success and error handlers */ );
		
You can also group multiple promises. For example, you can use the join function run multiple asynchronous operations that call several web services and then aggregate the results.

	promise.join(values).done( /* Your success and error handlers */ );
	function loadData() {
		var urls = [
			'http://www.example.com/',
			'http://www.example.org/',
			'http://www.example.net'
			];
			
		var promises = urls.map(function (url) {
			return myWebService.get(url);
		});

		WinJS.Promise.join(promises).then(function () {
			//do the aggregation here.
		});
	}
	
##Chaining promises
1. You can chain multiple then functions, because then returns a promise. You cannot chain more than one done method, because it returns undefined.
1. If you do not provide an error handler to done and the operation has an error, an exception is thrown to the event loop. This means that you cannot catch the exception inside a try/catch block, but you can catch it in window.onerror. If you do not provide an error handler to then and the operation has an error, it does not throw an exception but rather returns a promise in the error state.
1. You should prefer **flat promise chains** to nested ones. The formatting of promise chains makes them easier to read, and it's much easier to deal with errors in promise chains.

---
	aAsync()
		.then(function () { return bAsync(); })
		.then(function () { return cAsync(); })
		.done(function () { finish(); });
	// Bad code!
	aAsync().then(function () {
		bAsync().then(function () {
				cAsync().done(function () { finish(); });
		})
	});
	
##Handle errors with promises
1. Turn on JavaScript first-chance exceptions,working only when the app is running in the debugger, so users won't see these exceptions.
1. Add an error handler in a then or done function
1. Add a WinJS.promise.onerror handler

		function activatedHandler() {
			var input = document.getElementById("inUrl");
			input.addEventListener("change", changeHandler);
			WinJS.Promise.onerror = errorHandler
		}
		function errorhandler(event) {
				var ex = event.detail.exception;
				var promise = event.detail.promise;
		}