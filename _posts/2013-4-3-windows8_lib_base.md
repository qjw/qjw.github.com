---
layout: post
title:  Windows 8官方Sample框架解析
category: wingui
---


##[下载源码](http://code.msdn.microsoft.com/windowsapps/Windows-8-Modern-Style-App-Samples/file/60706/34/Windows%208%20app%20samples.zip)
以**Windows 8 app samples\File picker sample**为例说明

##default.html
先看下**default.html**【*为了节省篇幅，源码有删减，下同*】。

	<head>
		<script src="/sample-utils/sample-utils.js"></script>
		<script src="/js/default.js"></script>
	</head>
	<body role="application">
		<div id="rootGrid">
			<！-- 通用的header -->
			<div id="header" role="contentinfo" data-win-control="WinJS.UI.HtmlControl" data-win-options="{uri: '/sample-utils/header.html'}"></div>
			
			<div id="content">
				<h1 id="featureLabel"></h1>
				<div id="contentHost"></div>
			</div>
			
			<！-- 通用的footer -->
			<div id="footer" data-win-control="WinJS.UI.HtmlControl" data-win-options="{uri: '/sample-utils/footer.html'}"></div>
		</div>
	</body>

**sample-utils/footer.html** 和 **sample-utils/header.html** 分别定义了footer和header，内容就是普通的html。例如header.html

	<!DOCTYPE html>
	<html xmlns="http://www.w3.org/1999/xhtml">
		<head>
			<title></title>
		</head>
		<body>
			<img src="/images/windows-sdk.png" alt="Windows Logo" />
			<span>Windows SDK Samples</span>
		</body>
	</html>

需要注意的是**WinJS.UI.HtmlControl**控件，它可以显示静态html页面。

接下来就是**/sample-utils/sample-utils.js** 和 **/js/default.js**。其中sample-utils.js在default.js之前。实际上，大部分逻辑都定义在sample-utils.js中。

	(function () {
		"use strict";

		// 标题，在/sample-utils/sample-utils.js被使用。
		var sampleTitle = "File picker JS sample";

		// 具体若干种"文件选择"操作，每一种操作都通过一个html/js/css集合来实现
		var scenarios = [
			{ url: "/html/scenario1.html", title: "Pick a single photo" },
			{ url: "/html/scenario2.html", title: "Pick multiple files" },
			{ url: "/html/scenario3.html", title: "Pick a folder" },
			{ url: "/html/scenario4.html", title: "Save a file" },
			{ url: "/html/scenario5.html", title: "Open a cached file" },
			{ url: "/html/scenario6.html", title: "Update a cached file" }
		];

		// 程序被激活时调用
		function activated(eventObject) {
			if (eventObject.detail.kind === Windows.ApplicationModel.Activation.ActivationKind.launch) {
				// Use setPromise to indicate to the system that the splash screen must not be torn down
				// until after processAll and navigate complete asynchronously.
				eventObject.setPromise(WinJS.UI.processAll().then(function () {
					// Navigate to either the first scenario or to the last running scenario
					// before suspension or termination.
					// 恢复上次的url，若不存在，则使用第一个
					var url = WinJS.Application.sessionState.lastUrl || scenarios[0].url;
					// 浏览该url，这将触发navigated事件，见下面分析
					return WinJS.Navigation.navigate(url);
				}));
			}
		}

		// WinJS.Navigation的 navigated事件处理函数
		WinJS.Navigation.addEventListener("navigated", function (eventObject) {
			var url = eventObject.detail.location;
			
			// 若id=contentHost的元素存在，则清空它
			// 该元素在default.html中定义
			var host = document.getElementById("contentHost");
			// Call unload method on current scenario, if there is one
			host.winControl && host.winControl.unload && host.winControl.unload();
			WinJS.Utilities.empty(host);
			
			eventObject.detail.setPromise(
				// 将改元素重新渲染为新的url（html文件）
				WinJS.UI.Pages.render(url, host, eventObject.detail.state).then(function () {
					// 同时重置最近的url记录
					WinJS.Application.sessionState.lastUrl = url;
				}
			));
		});

		// 将“标题“和”操作“数组放入一个特定的名字空间中，其他js必须通过这个名字空间访问
		WinJS.Namespace.define("SdkSample", {
			sampleTitle: sampleTitle,
			scenarios: scenarios
		});

		// 注册激活事件
		WinJS.Application.addEventListener("activated", activated, false);
		// 开始运行
		WinJS.Application.start();
	})();

##scenario1.html
在最开始时，若不存在最近的url，那么会使用第一个页面，也就是**scenario1.html**，接下来我们将以scenario1.html为例说明，scenario2.html...等类同。代码非常简单：

	<head>
		<title></title>
		<script src="/js/scenario1.js"></script>
	</head>
	<body>
		<div data-win-control="SdkSample.ScenarioInput">
			<p>Prompt the user to pick a single photo.</p>
			<button class="action" id="open">Pick photo</button>
			<br /><br />
		</div>
		<div data-win-control="SdkSample.ScenarioOutput">
		</div>
	</body>
	</html>
	
###scenario1.js

	(function () {
		"use strict";
		var page = WinJS.UI.Pages.define("/html/scenario1.html", {
			// 在页面就绪时，给scenario1.html的按钮注册个事件
			ready: function (element, options) {
				document.getElementById("open").addEventListener("click", pickSinglePhoto, false);
			}
		});

		function pickSinglePhoto() {
		}
	})();
	
这其中的重点就是两个**data-win-control**,分别是**SdkSample.ScenarioInput**和**SdkSample.ScenarioOutput**，这两个控件在**sample-utils.js**中定义。

##sample-utils.js

	(function () {

		// 定义控件ScenarioInput
		var ScenarioInput = WinJS.Class.define(
		    // 构造函数
			function (element, options) {
			element.winControl = this;
			this.element = element;

			new WinJS.Utilities.QueryCollection(element)
						.setAttribute("role", "main")
						.setAttribute("aria-labelledby", "inputLabel");
			element.id = "input";

			// 调用三个成员函数
			this.addInputLabel(element);
			this.addDetailsElement(element);
			this.addScenariosPicker(element);
		}, {
			addInputLabel: function (element) {
				var label = document.createElement("h2");
				label.textContent = "Input";
				label.id = "inputLabel";
				element.parentNode.insertBefore(label, element);
			},
			addScenariosPicker: function (parentElement) {
				var scenarios = document.createElement("div");
				scenarios.id = "scenarios";
				// 在这里创建了一个列表选择器，见后续分析
				var control = new ScenarioSelect(scenarios);

				parentElement.insertBefore(scenarios, parentElement.childNodes[0]);
			},
		}
		);


		// 定义控件ScenarioOutput
		var ScenarioOutput = WinJS.Class.define(
			function (element, options) {
		}, {
		}
		);

		var currentScenarioUrl = null;

		// 在导航之前，先设置currentScenarioUrl变量，考虑到
		// scenario1.html,scenario2.html ...中分别定义了一个列表控件，为了同步
		// 所选择的item，这里使用一个变量记录当前选择的item
		// http://msdn.microsoft.com/en-us/library/windows/apps/br229843.aspx
		WinJS.Navigation.addEventListener("navigating", function (evt) {
			currentScenarioUrl = evt.detail.location;
		});

		// Control that populates and runs the scenario selector

		// 定义一个“列表选择”控件，这里使用WinJS.UI.Pages
		var ScenarioSelect = WinJS.UI.Pages.define("/sample-utils/scenario-select.html", {
			ready: function (element, options) {
				var that = this;
				
				// 获得scenario-select.html中id = scenarioSelect的控件
				/*
				<html xmlns="http://www.w3.org/1999/xhtml">
					<head>
						<title></title>
					</head>
					<body>
						<div class="options">
							 <h3 id="listLabel">Select scenario:</h3>
							<select id="scenarioSelect" aria-labelledby="listLabel">
								<!-- scenario list is inserted here -->
							</select>
						</div>
					</body>
				</html>
				*/
				var selectElement = WinJS.Utilities.query("#scenarioSelect", element);
				this._selectElement = selectElement[0];

				// 插入所有记录
				// SdkSample.scenarios在default.js中定义
				SdkSample.scenarios.forEach(function (s, index) {
					// 见成员函数_addScenario
					that._addScenario(index, s);
				});

				//设置事件处理
				selectElement.listen("change", function (evt) {
					var select = evt.target;
					if (select.selectedIndex >= 0) {
						var newUrl = select.options[select.selectedIndex].value;
						
						// 浏览到新的url
						WinJS.Navigation.navigate(newUrl).then(function () {
							setImmediate(function () {
								// 让该控件获得焦点
								document.getElementById("scenarioSelect").setActive();
							});
						});
					}
				});
				
				// 最多显示5条记录，若更多会显示滑动条
				selectElement[0].size = (SdkSample.scenarios.length > 5 ? 5 : SdkSample.scenarios.length);
				if (SdkSample.scenarios.length === 1) {
					// Avoid showing down arrow when there is only one scenario
					selectElement[0].setAttribute("multiple", "multiple");
				}
			},

			_addScenario: function (index, info) {
				var option = document.createElement("option");
				if (info.url === currentScenarioUrl) {
				    // 是否选择
					option.selected = "selected";
				}
				option.text = (index + 1) + ") " + info.title;
				option.value = info.url;
				this._selectElement.appendChild(option);
			}
		});

		// 初始化标题,featureLabel在default.html中定义
		function activated(e) {
			WinJS.Utilities.query("#featureLabel")[0].textContent = SdkSample.sampleTitle;
		}
		// 设置“激活”事件回调
		WinJS.Application.addEventListener("activated", activated, false);

		// 导出两个控件
		// Export public methods & controls
		WinJS.Namespace.define("SdkSample", {
			ScenarioInput: ScenarioInput,
			ScenarioOutput: ScenarioOutput
		});

	})();
