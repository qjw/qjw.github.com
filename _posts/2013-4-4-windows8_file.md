---
layout: post
title:  Windows 8文件操作
category: wingui
---

##[文件选择](http://msdn.microsoft.com/library/windows/apps/BR207928)

###[FileOpenPicker](http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker)
选择文件

	// Create the picker object and set options
	var openPicker = new Windows.Storage.Pickers.FileOpenPicker();
	openPicker.viewMode = Windows.Storage.Pickers.PickerViewMode.thumbnail;
	openPicker.suggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.picturesLibrary;
	// Users expect to have a filtered view of their folders depending on the scenario.
	// For example, when choosing a documents folder, restrict the filetypes to documents for your application.
	openPicker.fileTypeFilter.replaceAll([".png", ".jpg", ".jpeg"]);

	// Open the picker for the user to pick a file
	openPicker.pickSingleFileAsync().then(function (file) {
		if (file) {
			// Application now has read/write access to the picked file
			WinJS.log && WinJS.log("Picked photo: " + file.name, "sample", "status");
		} else {
			// The picker was dismissed with no selected file
			WinJS.log && WinJS.log("Operation cancelled.", "sample", "status");
		}
	});

FileOpenPicker支持以下配置

<table style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>属性</th><th>访问类型</th><th>描述</th></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker.commitbuttontext"><strong xmlns="http://www.w3.org/1999/xhtml">CommitButtonText</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the label text of the file open picker's commit button.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker.filetypefilter"><strong xmlns="http://www.w3.org/1999/xhtml">FileTypeFilter</strong></a>
</p>
                                </td><td>只读</td><td>Gets the collection of file types that the file open picker displays.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker.settingsidentifier"><strong xmlns="http://www.w3.org/1999/xhtml">SettingsIdentifier</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the settings identifier associated with the state of the file open picker.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker.suggestedstartlocation"><strong xmlns="http://www.w3.org/1999/xhtml">SuggestedStartLocation</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the initial location where the file open picker looks for files to present to the user.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker.viewmode"><strong xmlns="http://www.w3.org/1999/xhtml">ViewMode</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the view mode that the file open picker uses to display items.</td></tr>
</tbody></table>


####ViewMode
文件的显示方式：

1. Windows.Storage.Pickers.PickerViewMode.thumbnail : 缩略图象集。
1. Windows.Storage.Pickers.PickerViewMode.list : 项的列表。

####FileTypeFilter
文件过滤：

	openPicker.fileTypeFilter.replaceAll([".png", ".jpg", ".jpeg"]);

####CommitButtonText
文件选择器**提交**按钮上的文本

	fileOpenPicker.commitButtonText = commitButtonText;
	
####SettingsIdentifier
如果您的应用程序使用文件打开选择器的**多个实例**，您可以用此属性来识别单独实例。

	fileOpenPicker.settingsIdentifier = settingsIdentifier;

####SuggestedStartLocation
初始目录：

<table style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>成员</th><th>值</th><th>描述</th></tr>
<tr><td><strong>DocumentsLibrary</strong> | <strong>documentsLibrary</strong></td><td>0</td><td>
<p><strong>文档</strong>库。</p>
</td></tr>
<tr><td><strong>ComputerFolder</strong> | <strong>computerFolder</strong></td><td>1</td><td>
<p><strong>计算机</strong>文件夹。</p>
</td></tr>
<tr><td><strong>Desktop</strong> | <strong>desktop</strong></td><td>2</td><td>
<p>Windows 桌面。</p>
</td></tr>
<tr><td><strong>Downloads</strong> | <strong>downloads</strong></td><td>3</td><td>
<p><strong>“下载”</strong> 文件夹。</p>
</td></tr>
<tr><td><strong>HomeGroup</strong> | <strong>homeGroup</strong></td><td>4</td><td>
<p>家庭组。</p>
</td></tr>
<tr><td><strong>MusicLibrary</strong> | <strong>musicLibrary</strong></td><td>5</td><td>
<p><strong>音乐</strong>库。</p>
</td></tr>
<tr><td><strong>PicturesLibrary</strong> | <strong>picturesLibrary</strong></td><td>6</td><td>
<p><strong>图片</strong>库。</p>
</td></tr>
<tr><td><strong>VideosLibrary</strong> | <strong>videosLibrary</strong></td><td>7</td><td>
<p><strong>视频</strong>库。</p>
</td></tr>
</tbody></table>

####pickSingleFileAsync
打开单个文件，回调的参数是[StorageFile类](http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.storagefile)

	// Open the picker for the user to pick a file
	openPicker.pickSingleFileAsync().then(function (file) {
		if (file) {
			// Application now has read/write access to the picked file
			WinJS.log && WinJS.log("Picked photo: " + file.name, "sample", "status");
		} else {
			// The picker was dismissed with no selected file
			WinJS.log && WinJS.log("Operation cancelled.", "sample", "status");
		}
	});
	
####pickMultipleFilesAsync
打开多个文件，回调的参数是个数组，同pickSingleFileAsync

	// Open the picker for the user to pick a file
	openPicker.pickMultipleFilesAsync().then(function (files) {
		if (files.size > 0) {
			// Application now has read/write access to the picked file(s)
			var outputString = "Picked files:\n";
			for (var i = 0; i < files.size; i++) {
				outputString = outputString + files[i].name + "\n";
			}
			WinJS.log && WinJS.log(outputString, "sample", "status");
		} else {
			// The picker was dismissed with no selected file
			WinJS.log && WinJS.log("Operation cancelled.", "sample", "status");
		}
	});

##[FolderPicker](http://msdn.microsoft.com/library/windows/apps/BR207928)
枚举目录，只能枚举单个目录，用法同[FileOpenPicker](http://msdn.microsoft.com/zh-CN/library/windows/apps/windows.storage.pickers.fileopenpicker)
	
	// Create the picker object and set options
	var folderPicker = new Windows.Storage.Pickers.FolderPicker;
	folderPicker.suggestedStartLocation = Windows.Storage.Pickers.PickerLocationId.desktop;
	// Users expect to have a filtered view of their folders depending on the scenario.
	// For example, when choosing a documents folder, restrict the filetypes to documents for your application.
	folderPicker.fileTypeFilter.replaceAll([".docx", ".xlsx", ".pptx"]);

	folderPicker.pickSingleFolderAsync().then(function (folder) {
		if (folder) {
			// Application now has read/write access to all contents in the picked folder (including sub-folder contents)
			// Cache folder so the contents can be accessed at a later time
			Windows.Storage.AccessCache.StorageApplicationPermissions.futureAccessList.addOrReplace("PickedFolderToken", folder);
			WinJS.log && WinJS.log("Picked folder: " + folder.name, "sample", "status");
		} else {
			// The picker was dismissed with no selected file
			WinJS.log && WinJS.log("Operation cancelled.", "sample", "status");
		}
	});
	
##[FileSavePicker](http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.aspx)
保存文件

<table  style="border: 1px solid #000000;padding:3px;">
<tbody><tr><th>属性</th><th>访问类型</th><th>描述</th></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.commitbuttontext.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">CommitButtonText</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the label text of the commit button in the file pickerUI.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.defaultfileextension.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">DefaultFileExtension</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the default file name extension that the fileSavePicker gives to files to be saved.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.filetypechoices.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">FileTypeChoices</strong></a>
</p>
                                </td><td>只读</td><td>Gets the collection of valid file types that the user can choose to assign to a file.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.settingsidentifier.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">SettingsIdentifier</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the settings identifier associated with the current FileSavePicker instance.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.suggestedfilename.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">SuggestedFileName</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the file name that the file save picker suggests to the user.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.suggestedsavefile.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">SuggestedSaveFile</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the storageFile that the file picker suggests to the user for saving a file.</td></tr>
<tr><td>
                                    <p>
<a href="http://msdn.microsoft.com/zh-cn/library/windows/apps/windows.storage.pickers.filesavepicker.suggestedstartlocation.aspx"><strong xmlns="http://www.w3.org/1999/xhtml">SuggestedStartLocation</strong></a>
</p>
                                </td><td>读取/写入</td><td>Gets or sets the location that the file save picker suggests to the user as the location to save a file.</td></tr>
</tbody></table>

####SuggestedFileName
默认名字

	// Default file name if the user does not type one in or select a file to replace
	savePicker.suggestedFileName = "New Document";
	
####FileTypeChoices
存储和过滤的文件类型

	// Dropdown of file types the user can save the file as
	savePicker.fileTypeChoices.insert("Plain Text", [".txt"]);
	
##Snapped View
	// Verify that we are currently not snapped, or that we can unsnap to open the picker
	var currentState = Windows.UI.ViewManagement.ApplicationView.value;
	if (currentState === Windows.UI.ViewManagement.ApplicationViewState.snapped &&
		!Windows.UI.ViewManagement.ApplicationView.tryUnsnap()) {
		// Fail silently if we can't unsnap
		return;
	}