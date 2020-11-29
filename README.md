Folder and File Explorer
========================

A zero dependencies, customizable, pure Javascript widget for navigating, managing (move, copy, delete), uploading, and downloading files and folders or other hierarchical object structures on any modern web browser.  Choose from a MIT or LGPL license.

[![Screenshot of CubicleSoft File Explorer](https://user-images.githubusercontent.com/1432111/83955166-e3003a80-a804-11ea-806a-07b927297ccb.png)](https://cubiclesoft.com/demos/js-fileexplorer/demo.html)

[Live Demo](https://cubiclesoft.com/demos/js-fileexplorer/demo.html) | [The making of this widget](https://cubicspot.blogspot.com/2020/06/building-complex-javascript-widgets.html)

Experience a clean, elegant presentation of folders and files in a mobile-friendly layout that looks and feels great on all devices.  CubicleSoft File Explorer is easily connected to any web application that needs to manage hierarchical objects (folders and files, database records, or even JSON, XML, etc).

Check out a working product that uses this widget:  [PHP File Manager and Editor](https://github.com/cubiclesoft/php-filemanager)

[![Donate](https://cubiclesoft.com/res/donate-shield.png)](https://cubiclesoft.com/donate/) [![Discord](https://img.shields.io/discord/777282089980526602?label=chat&logo=discord)](https://cubiclesoft.com/product-support/github/)

If you use this project, don't forget to donate to support its development!

Features
--------

* Clean, elegant presentation of folders and files in a mobile-friendly layout.
* Zero dependencies.  No other libraries required!
* Can handle displaying thousands of files in a folder.
* Full keyboard, mouse, and touch support.  Secure from fake/simulated keyboard, mouse, and touch events.
* Keyboard shortcuts.  Lots of keyboard shortcuts.
* Folder history tracking and navigation.  Optionally navigate through folder history in the widget using the back and forward buttons on the mouse.
* Customizable toolbar.
* Create new files and folders.
* Easily rename and delete/recycle files and folders.
* Double-click/tap or press Enter to open items in your application-defined manner.
* Powerful file and folder selection tools.  Selection boxes with variable-speed scrolling on desktop browsers.
* Multi-select checkboxes on touch devices.  Can be enabled for non-touch devices too.
* Cut/Copy/Paste items using:  Keyboard shortcuts, mouse right-click menu, or toolbar buttons.
* Cut/Copy/Paste items between compatible widget instances...even across different web browsers!
* Drag-and-drop support between compatible widget instances.
* Drag-and-drop bulk uploading support directly from the OS for both files and folders.
* Drag-and-drop direct URL downloading support (Chromium only).
* Chunked file upload support.
* Download files and folders via a toolbar button.
* Items can be displayed as a thumbnail image, including folders.  Thumbnails are intelligently bulk loaded in the background using a custom lazy loader.
* Items can have detailed tooltips, overlays, size information, etc.
* Automatic item sorting (can be disabled per folder).
* Provides useful feedback on the status bar - items in a folder, number selected, active upload information, etc.
* Can easily be connected to a WebSocket backend (e.g. [Data Relay Center](https://github.com/cubiclesoft/php-drc)) to efficiently watch for folder updates.
* Event-handler style callbacks.  Way more callbacks than you'll probably need.
* Plenty of automation possibilities too via the extensive [FileExplorer API](docs/file_explorer.md).
* Multilingual support.
* And much, much more.

Getting Started
---------------

To use this widget, you should be quite comfortable with writing both client-side Javascript and secure server-side code.  Client-side Javascript is never a proper defense mechanism for proper server-side security and assume someone will try to break into your server by sending bad folder and file paths to the server to read/write data they shouldn't have access to.

Also note that this widget only provides the client-side (web browser) portion of the equation.  You will need to supply your own server-side handlers (e.g. PHP) but that's the comparatively easy part.

With those caveats out of the way, download/clone this repo, put the `file-explorer` directory on a server, and add these lines to your page to load the widget core:

```html
<link rel="stylesheet" type="text/css" href="file-explorer/file-explorer.css">
<script type="text/javascript" src="file-explorer/file-explorer.js"></script>
```

Next, create an instance of the FileExplorer class inside a closure for security reasons:

```html
<div id="filemanager" style="height: 50vh; max-height: 400px; position: relative;"></div>

<script type="text/javascript">
(function() {
	var elem = document.getElementById('filemanager');

	var options = {
		initpath: [
			[ '', 'Projects (/)', { canmodify: false } ]
		],

		onrefresh: function(folder, required) {
			// Optional:  Ignore non-required refresh requests.  By default, folders are refreshed every 5 minutes so the widget has up-to-date information.
//			if (!required)  return;

			// Maybe notify a connected WebSocket here to watch the folder on the server for changes.
			if (folder === this.GetCurrentFolder())
			{
			}

			// Make a call to your server here to get some entries to diplay.
			// this.PrepareXHR(options) could be useful for doing that.  Example:

			var $this = this;

			var xhr = new this.PrepareXHR({
				url: '/yourapp/',
				params: {
					action: 'file_explorer_refresh',
					path: JSON.stringify(folder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.response);
console.log(data);

					if (data.success)  folder.SetEntries(data.entries);
					else if (required)  $this.SetNamedStatusBarText('folder', $this.EscapeHTML('Failed to load folder.  ' + data.error));
				},
				onerror: function(e) {
					// Maybe output a nice message if the request fails for some reason.
//					if (required)  $this.SetNamedStatusBarText('folder', 'Failed to load folder.  Server error.');

console.log(e);
				}
			});

			xhr.Send();
		},

		// This will be covered in a moment...
//		onrename: function(renamed, folder, entry, newname) {
//		},
	};

	var fe = new window.FileExplorer(elem, options);
})();
</script>
```

That code will produce a less-than-exciting 'Loading...' view and also doesn't show the toolbar since there are no event listeners for the tools (yet):

![A loading screen](https://user-images.githubusercontent.com/1432111/83955169-e4c9fe00-a804-11ea-9275-a19db61ec4a7.png)

The next step is to connect the widget to the backend server.  There are many ways to do that.  The widget instance itself exports the [PrepareXHR class](docs/prepare_xhr.md) to make AJAX calls.  Or you could use a framework specific mechanism of your choice (jQuery, Vue, whatever) or talk to a WebSocket server or whatever makes sense for your application.  The entries returned from the server should ideally be compatible with [Folder.SetEntries](docs/folder.md#foldersetentriesnewentries).

If using PHP on a server and this widget will reflect a physical file system on the same server, there is the useful [FileExplorerFSHelper PHP class](docs/file_explorer_fs_helper.md) that can simplify connecting the widget to the backend server:

```php
<?php
	require_once "server-side-helpers/file_explorer_fs_helper.php";

	$options = array(
		"base_url" => "https://yoursite.com/yourapp/files/",
		"protect_depth" => 1,  // Protects base_dir + additional directory depth.
		"recycle_to" => "Recycle Bin",
		"temp_dir" => "/tmp",
		"dot_folders" => false,  // Set to true to allow things like:  .git, .svn, .DS_Store
		"allowed_exts" => ".jpg, .jpeg, .png, .gif, .svg, .txt",
		"allow_empty_ext" => true,
		"thumbs_dir" => "/var/www/yourapp/thumbs",
		"thumbs_url" => "https://yoursite.com/yourapp/thumbs/",
		"thumb_create_url" => "https://yoursite.com/yourapp/?action=file_explorer_thumbnail&xsrftoken=qwerasdf",
		"refresh" => true,
		"rename" => true,
		"file_info" => false,
		"load_file" => false,
		"save_file" => false,
		"new_folder" => true,
		"new_file" => ".txt",
		"upload" => true,
		"upload_limit" => 20000000,  // -1 for unlimited or an integer
		"download" => true,
		"copy" => true,
		"move" => true,
		"delete" => true
	);

	FileExplorerFSHelper::HandleActions("action", "file_explorer_", "/var/www/yourapp/files", $options);
```

Once entries are populating in the widget, various bits of navigation functionality should start working.  The widget is starting to come to life but is mostly read only.  Let's add an `onrename` handler so users can press F2 or click on the text of a selected item to rename the item:

```js
	// Note:  'entry' is a copy of the original, so it is okay to modify any aspect of it, including 'id'.
	onrename: function(renamed, folder, entry, newname) {
		var xhr = new this.PrepareXHR({
			url: '/yourapp/',
			params: {
				action: 'file_explorer_rename',
				path: JSON.stringify(folder.GetPathIDs()),
				id: entry.id,
				newname: newname,
				xsrftoken: 'asdfasdf'
			},
			onsuccess: function(e) {
				var data = JSON.parse(e.target.response);
console.log(data);

				// Updating the existing entry or passing in a completely new entry to the renamed() callback are okay.
				if (data.success)  renamed(data.entry);
				else  renamed(data.error);
			},
			onerror: function(e) {
console.log(e);
				renamed('Server/network error.');
			}
		});

		xhr.Send();
	},
```

A lot is going on here.  When the user finishes renaming an item, `onrename` is called, which hands the request off to an AJAX request to the server to handle.  During this operation, the textarea is marked read only and the busy state on the folder is enabled.  When the AJAX operation completes, it must call `renamed()` to let the widget know that the operation has been completed and indicate success by passing in a compatible entry object - either a modified `entry` or a new entry from the server.  On failure, a boolean of false or a string that is passed to renamed() to be used as part of the error message displayed to the user.

The above examples and documentation should be enough to get the ball rolling.  Most event handler callbacks utilize a similar approach:  Receive an event callback, make a server call or two, and finally call the completion callback function with the result of the operation.  Callbacks always have the 'this' context as the FileExplorer instance.

See a complete, functional implementation of all of the important callbacks in the [FileExplorerFSHelper PHP class](docs/file_explorer_fs_helper.md) documentation.  It's an excellent starting point when utilizing the FileExplorerFSHelper class.

FileExplorer Options
--------------------

The `options` object passed to the FileExplorer class accepts the following options:

* group - An optional string containing a group name.  When specified, this must be a unique string that indicates a compatible widget backend to allow cross-widget drag-and-drop and cut/copy/paste to function properly.  When not specified, the instance receives a group name unique to the instance that disables the aforementioned cross-widget features.
* alwaysfocused - A boolean that indicates whether or not the widget's appearance never loses focus (Default is false).  This only affects visual appearance and does not stop onfocus/onblur event callbacks from firing.
* capturebrowser - A boolean that indicates whether or not to capture the hardward back and forward mouse buttons when the mouse is hovering over the widget (Default is false).  Read the 'Known Limitations' section before enabling this feature.
* messagetimeout - An integer containing the number of milliseconds to show short-lived messages in the status bar (Default is 2000).
* displayunits - A string containing one of 'iec_windows', 'iec_formal', or 'si' to specify what units to use when displaying file sizes to the user (Default is 'iec_windows').
* adjustprecision - A boolean indicating whether or not to adjust the final precision when displaying file sizes to the user (Default is true).
* initpath - An optional array of arrays containing the initial path segments to set.  Each path segment is an array consisting of `[id, value, attrs]`.  This path is passed to [FileExplorer.SetPath](docs/file_explorer.md#fileexplorer-setpath).
* onfocus(e) - An optional callback function that is called when the widget or one of its components receives focus.
	* e - The browser event object that triggered the event.
* onblur(e) - An optional callback function that is called when the widget loses focus.  This callback can be used to hide or `Destroy()` the widget instance if it is used in a popup overlay.  Note that `onblur` will not fire if the window itself loses focus - only if the widget loses focus to another element on the page.
	* e - The browser event object that triggered the event.
* onrefresh(folder, required) - An optional callback function that is called when a folder refresh should happen.
	* folder - A Folder instance to refresh.
	* refresh - A boolean indicating that the refresh is required for the widget to function properly.
* onselchanged(folder, selecteditemsmap, numselecteditems) - An optional callback function that is called whenever the user changes the item selections.  Rarely used.
	* folder - The current Folder instance.
	* selecteditemsmap - The internal selecteditemsmap object.  Do not modify.
	* numselecteditems - An integer containing the number of selected items in the selecteditemsmap.
* onrename(renamed, folder, entry, newname) - An optional callback function that is called when the user renames an item.
	* renamed(entry) - A callback function to call upon completion of renaming or on failure.
	* folder - The Folder in which the entry to rename is in.
	* entry - The entry to rename.
	* newname - A string containing the name that was entered by the user.
* onopenfile(folder, entry) - An optional callback function that is called when the user double-clicks/taps on an item.
	* folder - The Folder that the entry to open is in.
	* entry - The entry to open.
* tools - An optional object containing key-value pairs where the value is a boolean that indicates whether or not to show the tool and the keys are:  new_folder, new_file, upload, download, copy, paste, cut, delete, item_checkboxes.  Most toolbar tools show up automatically when an onhandler is defined and override any "don't show" setting.  The only tool that doesn't show as a result of a handler being defined is 'item_checkboxes'.
* onnewfolder(created, folder) - An optional callback function that is called when the user clicks the New Folder icon or presses Ctrl + Ins.
	* created(success) - A callback function to call upon completion of creating a new folder or on failure.
	* folder - The Folder in which to create a new folder.
* onnewfile(created, folder) - An optional callback function that is called when the user clicks the New File icon or presses Ins.
	* created(success) - A callback function to call upon completion of creating a new file or on failure.
	* folder - The Folder in which to create a new file.
* oninitupload(startupload, fileinfo, queuestarted) - An optional callback function that is called when the user clicks the Upload icon or presses Ctrl + U.
	* startupload(fileinfo, process) - A callback function to call upon completion of initializing the upload or on failure.  The 'process' option can be used to tell the callback to process the upload itself or skip it if it was handled in the callback (e.g. creating an empty directory).  When 'process' is a string, it is treated as an error message to display to the user.
	* fileinfo - The file information object of the file to initialize for uploading.
	* queuestarted - An integer containing a UNIX timestamp of when the current upload queue was started.  Can be useful to pass to a server that can use the value to copy files that are going to be overwritten to a recycling bin folder before the overwrite happens.
* onfinishedupload(finalize, fileinfo) - An optional callback function that is called when the upload finishes successfully.  Can be used to finalize an uploaded file on a server (e.g. move from a temporary directory to its final location).
	* finalize(success, entry) - A callback function to call upon completion of finalizing the upload or on failure.  The 'entry' parameter is an optional Folder entry to set in the current folder.
	* fileinfo - The file information object of the file upload to finalize.
* onuploaderror(fileinfo, e) - An optional callback function that is called when the upload fails.  Can be used to remove an associated temporary file that was created during initialization.
	* fileinfo - The file information object of the file upload that had an error.
	* e - An optional error event object (not all failure conditions include this parameter).
* concurrentuploads - An integer containing the maximum number of concurrent uploads to perform simultaneously (Default is 4).
* oninitdownload(startdownload, folder, ids, entries) - An optional callback function that is called when the user clicks the Download icon.  Since browsers don't do well with downloading a lot of individual files, it is recommended that folders and files be sent from the server in a single, compressed file format (e.g. ZIP).
	* startdownload(xhroptions) - A callback function to call upon completion of preparing the download or on failure.  When an object is passed for xhroptions, it must be a set of options compatible with [PrepareXHR](docs/prepare_xhr.md).
	* folder - The Folder in which the ids are located.
	* ids - An array containing the selected entry IDs to download.
	* entries - An array containing the selected entries to download.
* ondownloadstarted(options) - An optional callback function that is called when the download has started.
	* options - The xhroptions object passed to startdownload().
* ondownloaderror(options) - An optional callback function to that is called if the download failed to start.
	* options - The xhroptions object passed to startdownload().
* ondownloadurl(result, folder, ids, name) - An optional synchronous callback function that is called when a user starts a drag operation or cuts/copies one or more selected items.  This applies a DownloadURL MIME type to the content.  This allows for dragging/pasting folders and files out of the browser and onto the desktop.  Currently only supported in Chromium browsers and may not use asynchronous methods like AJAX to calculate the URL.
	* result - An object that should have 'name' and 'url' strings assigned to it.
	* folder - The Folder in which the ids are located.
	* ids - An array containing the selected entry IDs to download.
	* entry - The first folder entry in the associated IDs list.  Can be used to set the 'name' of the file.
* oncopy(copied, srcpath, srcids, destfolder) - An optional callback function that is called when the user copies folders/files from one folder to another.
	* copied(success, entries) - A callback function to call upon completion of copying the folders and files or on failure.  The 'entries' should contain the successfully copied entries to add regardless of success/failure.
	* srcpath - An array of arrays representing the source path of the IDs.
	* srcids - An array of entry IDs in the source path to copy.
	* destfolder - A Folder to which the entries are to be copied.  Note that the destination folder may be the same as the source, in which case the server may choose to copy the files to new files with different names or reject the request.
* onmove(moved, srcpath, srcids, destfolder) - An optional callback function that is called when the user moves folders/files from one folder to another.
	* moved(success, entries) - A callback function to call upon completion of moving the folders and files or on failure.  The 'entries' should contain the successfully moved entries to add regardless of success/failure.
	* srcpath - An array of arrays representing the source path of the IDs.
	* srcids - An array of entry IDs in the source path to move.
	* destfolder - A Folder to which the entries are to be moved.  Note that the destination folder may be the same as the source, in which case the server should reject the request.
* ondelete(deleted, folder, ids, entries, recycle) - An optional callback function that is called when the user deletes folders/files.
	* deleted(success) - A callback function to call upon completion of the deletion operation.
	* folder - The Folder in which the ids/entries are located.
	* ids - An array containing the selected entry IDs to delete.
	* entries - An array containing the selected entries to delete.
	* recycle - A boolean hinting at whether to send items to a recycling bin (Delete) or permanently delete them (Shift + Delete).  The server is free to ignore this parameter and do whatever makes the most sense.
* langmap - An object containing translation strings.  Support exists for most of the user interface (Default is an empty object).

The [Live Demo](https://cubiclesoft.com/demos/js-fileexplorer/demo.html) utilizes nearly all of the available callbacks.  The [Live Demo source code](demo.html) was designed so as keep this documentation to a minimum and to provide decent example usage without incurring AJAX calls.

Making Custom Tools
-------------------

While most of the widget is not really intended to be modified externally, the toolbar is designed to be extensible.  Building a new tool involves:

* An icon to display and an associated CSS class.
* Some Javascript that handles clicks and possibly a keyboard shortcut.
* Being efficient.  The toolbar buttons are updated regularly via toolbar events that fire from FileExplorer.
* Cleaning up when the 'destroy' event fires.
* Registering the new tool with the registered tools.

The simplest approach to developing a new tool is to look at the existing tools.  However, the Delete tool is fairly simple and short:

```js
(function() {
	// Tools receive the FileExplorer instance as the only option passed to the tool.
	var FileExplorerTool_Delete = function(fe) {
		if (!(this instanceof FileExplorerTool_Delete))  return new FileExplorerTool_Delete(fe);

		// Do not create the tool if deleting is disabled.
		if (!fe.hasEventListener('delete') && !fe.settings.tools.delete)  return;

		var enabled = false;

		// Register a toolbar button with File Explorer.
		var node = fe.AddToolbarButton('fe_fileexplorer_folder_tool_delete', fe.Translate('Delete (Del)'));

		// Handle clicks.
		var ClickHandler = function(e) {
			if (e.isTrusted && enabled)  fe.DeleteSelectedItems(!e.shiftKey);
		};

		node.addEventListener('click', ClickHandler);

		// Efficiently handle toolbar updates - only adding/removing the disabled class if it is different from its previous state.
		var UpdateToolHandler = function(currfolder, attrs) {
			var prevenabled = enabled;

			enabled = (!currfolder.waiting && (!('canmodify' in attrs) || attrs.canmodify) && fe.GetNumSelectedItems());

			if (prevenabled !== enabled)
			{
				if (enabled)  node.classList.remove('fe_fileexplorer_disabled');
				else  node.classList.add('fe_fileexplorer_disabled');

				// Notify File Explorer that the state was updated.
				// When the event callback finishes, it will know that there is some work to do.
				fe.ToolStateUpdated();
			}
		};

		fe.addEventListener('update_tool', UpdateToolHandler);

		// Cleanly handle the destroy event.
		var DestroyToolHandler = function() {
			node.removeEventListener('click', ClickHandler);
		};

		fe.addEventListener('destroy', DestroyToolHandler);
	};

	// Register the tool in the second group of tools (0-based).
	window.FileExplorer.RegisterTool(1, FileExplorerTool_Delete);
})();
```

Some additional comments were added to the code above to aid in understanding what is going on.  There are more complex tools to look at in the FileExplorer source code (e.g. FileExplorerTool_Download).  The Class Documentation section below will be quite useful when developing a custom tool.

For custom tools, you might want to prefix custom settings object keys with something like a company abbreviation so the likelihood of a naming conflict is reduced.

One good idea for a custom tool might be a HTML embed tool.  If a user selects a single item and clicks the embed tool, the clipboard receives some HTML code that can be pasted into another website to embed the item into a post.

Class Documentation
-------------------

* [FileExplorer class](docs/file_explorer.md) - The core FileExplorer class + tools.  Does most of the heavy-lifting.  Exported as `window.FileExplorer`.
* [DebounceAttributes class](docs/debounce_attributes.md) - Debounces/throttles attribute changes.  More efficient than most debounce functions that rely on events.
* [PrepareXHR class](docs/prepare_xhr.md) - A basic and convenient wrapper around a XMLHttpRequest (XHR) object.
* [ImageLoader class](docs/image_loader.md) - A multi-queue delayed image loader that only loads images when it is told to.  Also exported as `window.FileExplorer.ImageLoader` for reusability purposes.
* [PopupMenu class](docs/popup_menu.md) - Displays a popup menu of items.  Used for Recent Locations and path segment expansion.  Full keyboard, mouse, and touch support.  Also exported as `window.FileExplorer.PopupMenu` for reusability purposes.
* [TextareaOverlay class](docs/textarea_overlay.md) - Displays a positionable textarea with optional text with text selection.  Full keyboard, mouse, and touch support.  Also exported as `window.FileExplorer.TextareaOverlay` for reusability purposes.
* [Folder class](docs/folder.md) - Tracks the contents of a single folder in the mapped folders of the FileExplorer class.  The Folder class cannot be instantiated except from within the FileExplorer but instances of this class can be accessed primarily via callbacks from FileExplorer.
* [FileExplorerFSHelper class](docs/file_explorer_fs_helper.md) - PHP server-side helper class to simplify interacting with server-local, physical file systems.

Known Limitations
-----------------

CubicleSoft File Explorer is a complex piece of software written in Javascript.  As with every complex piece of software written in Javascript, there are going to be problems with certain combinations of OS + device + web browser for a garden variety of reasons.  Web browsers have lots of bugs and the specifications that browsers follow don't cover all edge cases, which leaves things open to interpretation for the browser vendor.  What follows are known limitations of the widget due to conditions beyond nearly everyone's direct control and most of the listed problems have reasonable workarounds.  Please do not open issues on the issue tracker for these items unless you actually solve them.

* Some devices and web browsers lack HTML 5 drag-and-drop support, notably any browser on iOS as the iOS webview implementation lacks support.  It is NOT recommended to use a [polyfill](https://github.com/Bernardo-Castilho/dragdroptouch) with this widget to support HTML 5 drag-and-drop.  There are several situations where a drag-and-drop polyfill will probably break certain touch-based features of this widget and/or break native drag-and-drop elsewhere.  Instead, use the cut/copy/paste feature on devices/browsers where HTML 5 drag-and-drop is not natively supported.

* The right-click context menu for cut/copy/paste can cause the main UI area to get stuck on some platforms until a click or keyboard event is registered, notably Mac OSX.  There is a near-invisible textarea overlay that captures right-click events for cut/copy/paste operations but it doesn't always go away until a second mousedown/keyup event is registered.  This happens because there is no such thing as an 'exitcontextmenu' event to detect that the context menu has closed and all known methods for detection are hacks.  The solution is to either put up with the extra click or just not use right-click and use keyboard shortcuts or the toolbar buttons instead.

* There is extremely limited "file paste" support from OS file systems into the widget.  Raw image data that is stored on the clipboard (e.g. right-click on an image in a website then click Copy) can be pasted into the widget and it will be uploaded but files from the OS itself do not drop.  Here's a [Chromium bug that's been open since 2013](https://bugs.chromium.org/p/chromium/issues/detail?id=316472) about the issue and has received almost no attention other than duplicates being merged into it.  Dragging and dropping files from the OS into the browser is the most feature-complete file transfer mechanism available to date.

* The iframe-based downloading method requires initiating a second request to the server for the same content.  This kind of hack has to be done because there is no web browser event available to know that page navigation was cancelled when the download starts.  The XHR request terminates as soon as it receives its first 'progress' event, usually in the first few KB and it assumes the iframe form submission equivalent, which started before the XHR request, completed in a similar amount of time and has started downloading the file.  It is recommended that download requests that take more than a few seconds to process on the server (e.g. generating a compressed file) be run separately and the generated file stored on the server somewhere prior to calling `startdownload(fileinfo)` in the client so that both the form and XHR requests return quickly, the download begins, and the iframe is removed all within a few seconds.

* The hardware back/forward mouse button capture logic modifies browser history.  There is no way to capture the hardware mouse buttons except to modify browser history so that there are three distinct history push states (back, current, forward) while hovering over the widget and using mouseenter/mouseleave to enable/disable the back/forward button capture.  There are a couple of extremely difficult to track down bugs as well.  The first bug is that the `window.history.scrollRestoration` of the real page sometimes gets stuck on `manual` instead of being restored to `auto`, which causes page reload jumping issues and also causes Edge to jump during scrolling.  That bug only seems to show up if lots of reloads happen.  The second bug is the page the widget is on occasionally completely messes up the parent pushState and the widget decides it needs to back up too far in the history stack and ends up leaving the page altogether.  I have no idea what causes either bug to occur.  Also, during development of this specific feature, Firefox literally crashed a half-dozen times.  Since browser history is modified, which may conflict with other software, the hardware back/forward mouse button capture feature is disabled by default.  It is a pretty cool feature when it works.
