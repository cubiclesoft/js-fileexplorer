FileExplorer Class
==================

The FileExplorer class is exported to the global `window` object and is core widget for File Explorer.  `window.FileExplorer` also exports several internal classes and functions that can be useful for building custom features.

See the main documentation in thie repository for example, common usage and use-cases for the class.  This documentation focuses on the public functions and information available for each instance of the FileExplorer class.

FileExplorer(parentelem, options)
---------------------------------

Category:  Constructor

Parameters:

* parentelem - A DOM node to append the root of the File Explorer widget to.
* options - An object containing options to use to construct the widget.

Returns:  A Javascript function hierarchy.

This function sets up a new File Explorer widget instance.  The 'new' keyword is required in order to use it correctly.  See the repository README to see a list of all options that can be passed in.

FileExplorer.settings
---------------------

Category:  Settings

The `settings` object contains the settings for the instance.  Changing the settings after creating the instance will have little to no effect and changing them is not recommended.  The settings are made public mostly so that tools can set up and trigger appropriate callbacks.

FileExplorer.Translate(str)
---------------------------

Category:  Internationalization

Parameters:

* str - A string to translate using `FileExplorer.settings.langmap`.

Returns:  A replacement string, if available.

Thie function is called whenever an internal string will be displayed to a user to find a replacement string in the language map object.

FileExplorer.FormatStr(format, ...)
-----------------------------------

Category:  Internationalization

Parameters:

* format - A string containing matching placeholders for `{0}`, `{1}`, etc. for the rest of the parameters

Returns:  A formatted string with placeholders replaced with values from the other input parameters.

This function is usually used in conjunction with Translate() to perform more complex multilingual string transformations that require multiple parameters to create.

FileExplorer.EscapeHTML(text)
-----------------------------

Cateogry:  Strings

Parameters:

* text - A string to sanitize for safe insertion into HTML.

Returns:  A HTML-safe string.

This function escapes input content for insertion into HTML.

FileExplorer.GetDisplayFilesize(numbytes, adjustprecision, units)
-----------------------------------------------------------------

Category:  Files

Parameters:

* numbytes - An integer containing the number of bytes to convert for display.
* adjustprecision - A boolean indicating whether or not to adjust the final precision when displaying file sizes to the user.
* units - A string containing one of 'iec_windows', 'iec_formal', or 'si' to specify what units to use when displaying file sizes to the user.

Returns:  A string containing a displayable file size.

This function converts an integer into a file size string (e.g. 1024 becomes 1 KB).

FileExplorer.SetNamedStatusBarText(name, text, timeout)
-------------------------------------------------------

Category:  Status bar

Parameters:

* name - A string containing the status bar segment name to target for the message.
* text - A string containing the text to display.  Should be escaped with EscapeHTML() to avoid issues.
* timeout - An integer containing the number of milliseconds to show the message (Default is infinite).

Returns:  Nothing.

This function updates a named segment in the status bar with the specified text.

When a timeout is specified, the message shows until the timeout or another message replaces it.  A message with a timeout may also momentarily hide other messages such that it fits on the screen.

There are three predefined segments:  folder, selected, message.  The first two should only be used by the widget.  The third (message) should be used primarily for messages that timeout.  Other segments can be added as needed.

FileExplorer.CreateProgressTracker(cancelcallback)
--------------------------------------------------

Category:  Status bar

Parameters:

* cancelcallback - A function to call if the user clicks the cancel button.

Returns:  An object containing a progress tracker.

This function registers a new progress tracker and returns it.  This allows for long-running operations to take place while providing regular visual feedback (e.g. copying files).  If a cancel callback is supplied, the user can cancel running operations by clicking the cancel button.

Each tracker contains the following options:

* started - A UNIX timestamp of when the object was created (Default is Date.now()).
* totalbytes - An integer containing the number of bytes transferred/moved (Default is 0).
* showbyterate - A boolean indicating whether or not to show the byte rate after 3 seconds (Default is false).
* uploading - A boolean of false (if not uploading) or an integer that contains how many items are actively being uploaded (Default is false).
* queueditems - An integer containing the number of known items in the queue (Default is 0).
* queuesizeunknown - A boolean indicating whether or not the actual size of the queue is unknown (Default is false).  This is useful when the actual queue size is unknown but some number of items (queueditems) is known.
* itemsdone - An integer containing the number of items that have been processed successfully (Default is 0).
* faileditems - An integer containing the number of items that did not process successfully (Default is 0).
* cancelcallback - A function to call when the user clicks the cancel button (Default is null).
* finishmessage - A string containing the message to display in the status bar when the trackers is done (Default is '').  Useful for showing 'Copying done' or 'Deletion stopped'.

These options are intended to be updated on a regular basis.  UpdateProgressText() is automatically called every second until no trackers are left.

FileExplorer.UpdateProgressText()
---------------------------------

Category:  Status bar

Parameters:  None.

Returns:  Nothing.

This function updates the progress area of the status bar with the combined information of all trackers.  When a tracker exists, this function is automatically called every second until no trackers are left.

FileExplorer.RemoveProgressTracker(tracker, finishmessage)
----------------------------------------------------------

Category:  Status bar

Parameters:

* tracker - An object containing a progress tracker.
* finishmessage - An optional string that contains the message to display if this is the last progress tracker (Default is false).

Returns:  Nothing.

This function removes a progress tracker.  If it is the last progress tracker, the status bar is left alone until an action takes place (mouse move, keyboard event, or touch) and is removed about one second after an action takes place.  This allows a user to see the final result if they stepped away for a moment.

FileExplorer.addEventListener(eventname, callback)
--------------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a FileExplorer event to listen to.
* callback - A function to call when the event fires.

Returns:  Nothing.

This function presents a familiar function for registering for custom events emitted by FileExplorer.

Known events:

* tool_[event] - Reserved for tool-specific events.
* blur - Dispatched when the widget is blurred.
* focus - Dispatched when the widget is focused.
* update_tool - Dispatched when a tool needs to update in response to changes in the current folder.
* open_file - Dispatched when a user double-clicks/taps on one or more selected items.
* selections_changed - Dispatched when the selections in the current folder change.
* refresh_folder - Dispatched when it is time to refresh a specific folder (not necessarily the current folder).
* navigated - Dispatched when navigation takes place.
* history_changed - Dispatched whenever the history stack changes.
* move - Dispatched when moving items.
* copy - Dispatched when copying items.
* upload_error - Dispatched when an upload fails (after all retry attempts fail) or one of the files or directories on the system is invalid/unreadable.
* upload_done - Dispatched when an upload completes successfully.
* update_upload_fileinfo - Dispatched when preparing the XHR request for an upload chunk.
* init_upload - Dispatched when initializing an upload.
* get_download_url - Dispatched when a drag-and-drop or copy/cut operation occurs to get the information needed to build a DownloadURL (Chromium only).
* keydown - Dispatched when the user presses a key.  Primarily used by tools.
* rename - Dispatched when the user renames an item.
* delete - Dispatched when the user deletes/recycles an item.
* destroy - Dispatched when the Destroy() function is called.

Most of these events have equivalents in the settings object and associated callbacks will automatically be registered as event listeners during initialization.

FileExplorer.removeEventListener(eventname, callback)
-----------------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a FileExplorer event to stop listening to.
* callback - A function containing a registered callback to remove.

Returns:  Nothing.

This function presents a familiar function for unregistering from custom events emitted by FileExplorer.

FileExplorer.hasEventListener(eventname)
----------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a FileExplorer event to check.

Returns:  A boolean of true if there are event listeners for the specified event, false otherwise.

This function checks for the existence of an event and whether or not there are any listeners registered for the event.

FileExplorer.ClearSelectedItems(ignorebusy, skipuiupdate)
---------------------------------------------------------

Category:  Item selection

Parameters:

* ignorebusy - An internal boolean that bypasses the busy folder check.  Do not use.
* skipuiupdate - An internal boolean that bypasses updating the toolbar and status bar.  Do not use.

Returns:  Nothing.

This function clears all existing selections.  The various boolean options should not be used by external callers (i.e. just call the function like `fe.ClearSelectedItems()`).

FileExplorer.SelectAllItems(skipuiupdate)
-----------------------------------------

Category:  Item selection

Parameters:

* skipuiupdate - An internal boolean that bypasses updating the toolbar and status bar.  Do not use.

Returns:  Nothing.

This function selects all items.  The skipuiupdate option should not be used by external callers (i.e. just call the function like `fe.SelectAllItems()`).

FileExplorer.SelectItemsFromLastAnchor(ignorebusy, skipuiupdate)
----------------------------------------------------------------

Category:  Item selection

Parameters:

* ignorebusy - An internal boolean that bypasses the busy folder check.  Do not use.
* skipuiupdate - An internal boolean that bypasses updating the toolbar and status bar.  Do not use.

Returns:  Nothing.

This function selects all items from the internal anchor position to the currently focused element.  The various boolean options should not be used by external callers (i.e. just call the function like `fe.SelectItemsFromLastAnchor()`).

FileExplorer.SetSelectedItems(ids, keepprev, skipuiupdate)
----------------------------------------------------------

Category:  Item selection

Parameters:

* ids - An array containing item IDs to select.
* keepprev - A boolean that controls whether or not to clear previously selected items (Default is false).
* skipuiupdate - An internal boolean that bypasses updating the toolbar and status bar.  Do not use.

Returns:  Nothing.

This function sets the selected items in the current folder with the specified item IDs.  It clears previous selections by default.  The skipuiupdate option should not be used by external callers.

FileExplorer.ToggleItemSelection(elem, ignorebusy, skipuiupdate)
----------------------------------------------------------------

Category:  Item selection

Parameters:

* elem - A string containing an item ID or a DOM node of an item to toggle selection.  External callers should use the item ID string method.
* ignorebusy - An internal boolean that bypasses the busy folder check.  Do not use.
* skipuiupdate - An internal boolean that bypasses updating the toolbar and status bar.  Do not use.

Returns:  Nothing.

This function toggles selection of a single item.  The various boolean options should not be used by external callers (i.e. just call the function like `fe.ToggleItemSelection(elem)`).

FileExplorer.GetNumSelectedItems()
----------------------------------

Category:  Selected items

Parameters:  None.

Returns:  An integer containing the number of selected items.

This function returns the number of currently selected items in the current folder.

FileExplorer.GetSelectedFolderEntries()
---------------------------------------

Category:  Selected items

Parameters:  None.

Returns:  An array containing the selected folder entries.

This function returns the currently selected entries in the current folder.

FileExplorer.GetSelectedItemIDs()
---------------------------------

Category:  Selected items

Parameters:  None.

Returns:  An array containing the IDs of the currently selected items.

This function returns just the IDs of the currently selected items in the current folder.

FileExplorer.IsSelectedItem(id)
-------------------------------

Category:  Selected items

Parameters:

* id - A string containing an ID to check for currently selected status.

Returns:  A boolean of true if the item associated with the ID is selected, false otherwise.

This function returns whether or not the item associated with the ID is selected.

FileExplorer.OpenSelectedItems()
--------------------------------

Category:  Selected item actions

Parameters:  None.

Returns:  Nothing.

This function triggers an open event for each selected non-folder item.  If a single folder is selected among all selected items, this function will also trigger navigation to that folder.

FileExplorer.RenameSelectedItem()
---------------------------------

Category:  Selected item actions

Parameters:  None.

Returns:  Nothing.

This function starts a rename operation if there is exactly one selected item and at least one 'rename' event listener.  The UI shows a textarea over the existing text with the item name contained within it.  If the name is modified, event listeners are called to handle the rename operation.

FileExplorer.DeleteSelectedItems(recycle)
-----------------------------------------

Category:  Selected item actions

Parameters:

* recycle - A boolean indicating whether or not the selected items should move to a recycling bin.

Returns:  Nothing.

This function starts a delete operation of selected items if there is at least one 'delete' event listener.  The 'recycle' option can be passed to and utilized by event handlers to optionally move items to a recycling bin instead of permanently deleting items off the server.  A cron job could be used to automatically clean up items in a recycling bin folder after they have been in there for a week.

FileExplorer.SetFocusItem(id, updateanchor)
-------------------------------------------

Category:  Focused item

Parameters:

* id - A string containing an item ID to focus or a boolean of false to clear focus.
* updateanchor - A boolean that indicates whether or not to update the anchor position for later Shift key selections.

Returns:  Nothing.

This function sets/clears the focused item.  When there is no focused item, focus reverts to the scroll region.  The anchor position, which is used when holding down the Shift key, can be updated at the same time.

FileExplorer.ScrollToFocusedItem()
----------------------------------

Category:  Focused item

Parameters:  None.

Returns:  Nothing.

This function scrolls the scroll region so that the focused item is fully visible.

FileExplorer.GetFocusedItem()
-----------------------------

Category:  Focused item

Parameters:  None.

Returns:  The DOM node containing the focused item or a boolean of false if not set.

This function returns the internal DOM node of the focused item.  When the focused item is not set, it returns false.

FileExplorer.GetFocusedItemID()
-------------------------------

Category:  Focused item

Parameters:  None.

Returns:  A string containing the folder item ID of the focused item or a boolean of false if the focused item is not set.

This function returns the folder item ID of the focused item.  When the focused item is not set, it returns false.  While it is possible to get the ID directly from a DOM node, it is not recommended.

FileExplorer.HasItemFocus()
---------------------------

Category:  Focused item

Parameters:  None.

Returns:  A boolean of true if there is a set focused item and it is also the `document.activeElement`, false otherwise.

This function returns whether or not the set focused item is also the currently focused item in the browser DOM.

FileExplorer.HasFocus(itemsonly)
--------------------------------

Category:  Focus

Parameters:

* itemsonly - A boolean that can restrict the return value to the folder items area (Default is false).

Returns:  A boolean of true if the widget or items currently have focus.

This function determines whether the currently focused `document.activeElement` is part of the folder items area when `itemsonly` is true or part of the widget otherwise (navigation, path segments, toolbar, etc).

FileExplorer.Focus(itemsonly, alwaysfocus)
------------------------------------------

Category:  Focus actions

Parameters:

* itemsonly - A boolean that indicates focus should be for the folder items area only (Default is false).
* alwaysfocus - A boolean that indicates that focus should always take place (Default is false).  Do not use.

Returns:  Nothing.

This function focuses on the items area if `itemsonly` is true, the widget otherwise.  If the widget/items region already has focus, nothing happens unless `alwaysfocus` is set to true.  The second parameter is only useful for stealing keyboard focus away from, for example, the hidden clipboard capture overlay, which shares the same DOM root as the the main items region.

Folder.GetPathIDs(path)
-----------------------

Category:  Path information

Parameters:

* path - An array of arrays that defines a path.

Returns:  An array of path IDs.

This function iterates over the path and constructs an array of path IDs.  The return value is intended to be useful for sending the path of a folder to a server.

FileExplorer.IsValidPath(path)
------------------------------

Category:  Path

Parameters:

* path - An array of arrays that defines a path.

Returns:  A boolean if the array structure looks correct, false otherwise.

This function checks an input path array for validity.  Used by SetPath() to prevent invalid structures from being used.

FileExplorer.SetPath(newpath)
-----------------------------

Category:  Path actions

Parameters:

* newpath - An array of arrays that defines the path to set.

Returns:  Nothing.

This function performs a path change operation, setting the path to the new path, adding to the history/navigation stack, triggering folder refreshes, and updating the DOM.  For most use-cases, using the 'initpath' option gets things started without needing to call this function directly.

If the calculated new path key is identical to the current path key, the current folder is refreshed in-place without modifying the history stack.

FileExplorer.GetMappedFolderFromPath(path)
------------------------------------------

Category:  Mapped folders

Parameters:

* path - An array of arrays that defines a path.

Returns:  An object containing a Folder instance if the path is mapped, a boolean of false otherwise.

This function returns the Folder object associated with a path if it is in the folder map.  A folder is placed into the internal widget folder map when it is referenced by a path segment somewhere in the history stack.  During the lifetime of the widget, mapped folders can come and go.  It is not safe to access a mapped folder reference beyond the lifetime of a function callback.

FileExplorer.GetCurrentFolder()
-------------------------------

Category:  Mapped folders

Parameters:  None.

Returns:  The current Folder object that is associated with the DOM.

This function returns the current Folder object that is associated with the DOM.  Updating the entries in this object triggers event callbacks within the widget that reflect in DOM updates.

FileExplorer.IsMappedFolder(folder)
-----------------------------------

Category:  Mapped folders

Parameters:

* folder - A valid Folder object instance.

Returns:  A boolean of true if the object is in the folder map, false otherwise.

This function checks the folder map to see if the input object is a mapped folder.  It actually first compares the input object with the current folder for efficiency and then falls back to walking through the folder map to find a match.  When using asynchronous logic, it is possible for a folder to be removed from the folder map before the asynchronous action completes (e.g. the user navigates to another folder during an operation).

FileExplorer.RefreshFolders(forcecurrfolder)
--------------------------------------------

Category:  Mapped folder actions

Parameters:

* forcecurrfolder - A boolean indicating whether or not to ignore the 5 minute refresh timer for the current folder (Default is false).

Returns:  Nothing.

This function triggers refresh events for folders in the folder map that were last refreshed over 5 minutes ago.

FileExplorer.HistoryBack(e)
---------------------------

Category:  Navigation actions

Parameters:

* e - An optional browser event object to call preventDefault() on.

Returns:  Nothing.

This function navigates back through the history stack one level.

FileExplorer.HistoryForward(e)
------------------------------

Category:  Navigation actions

Parameters:

* e - An optional browser event object to call preventDefault() on.

Returns:  Nothing.

This function navigates forward through the history stack one level.

FileExplorer.NavigateUp(e)
--------------------------

Category:  Navigation actions

Parameters:

* e - An optional browser event object to call preventDefault() on.

Returns:  Nothing.

This function navigates up one path segment level.

FileExplorer.CreateNode(tag, classes, attrs, styles)
----------------------------------------------------

Category:  Miscellaneous exports

Parameters:

* tag - A string containing the tag to use for the new DOM node.
* classes - An optional string or array of strings containing classes to assign to the new DOM node.
* attrs - An optional object of key-value pairs to assign to attributes of the new DOM node.
* styles - An optional object of key-value pairs to assign to styles of the new DOM node.

Returns:  A newly created, detached DOM node.

This function creates a new DOM node with classes, attributes, and styles.  Can be useful for building some custom tools.

FileExplorer.DebounceAttributes
-------------------------------

Category:  Miscellaneous exports

See:  [DebounceAttributes Class](docs/debounce_attributes.md)

FileExplorer.PrepareXHR(options)
--------------------------------

Category:  Miscellaneous exports

See:  [PrepareXHR Class](docs/prepare_xhr.md)

FileExplorer.GetElements()
--------------------------

Category:  Miscellaneous exports

Parameters:  None.

Returns:  The internal elems object for the widget.

This function returns the internal elements object for use with certain tools.  While element keys are not likely to change, there is no guarantee as this is an internal structure.

FileExplorer.GetScrollLineHeight()
----------------------------------

Category:  Miscellaneous exports

Parameters:  None.

Returns:  An integer containing the pixel count line height of a 1em unstyled character to be used for scroll calculations.

This function returns the precalculated value of the pixel count line height of a 1em unstyled character to be used for scroll calculations.  This is used, for example, to convert a vertical mouse wheel event into a horizontal scroll of the correct amount in the path segment bar.

FileExplorer.ShowClipboardPasteBox()
------------------------------------

Category:  Miscellaneous exports

Parameters:  None.

Returns:  Nothing.

This function shows the clipboard paste box.  Used by the paste tool.

FileExplorer.HideClipboardPasteBox(e)
-------------------------------------

Category:  Miscellaneous exports

Parameters:

* e - An optional browser event object to compare the target of for focus handling.

Returns:  Nothing.

This function hides the clipboard paste box and removes event handlers.

FileExplorer.ProcessFilesAndUpload(targettype, targetnode, dt)
--------------------------------------------------------------

Category:  Miscellaneous exports

Parameters:

* lasttype - A string containing one of 'currfolder', 'path', or 'item'.  External callers should use 'currfolder'.
* lastnode - A DOM node for 'path' or 'item' types or null.
* dt - A DataTransfer object instance containing 'items' and/or 'files' arrays.

Returns:  Nothing.

This function processes the incoming data transfer files/folders and starts uploading them to the specified target.  With some minor effort, it is possible to upload a Blob instance from outside the widget to the current folder using this function (e.g. save an image from a canvas element).

The 'path' type requires pointing at a valid DOM node in the path segment region.  The 'item' type requires pointing at a valid folder item DOM node.  These types are mostly internal for handling drag-and-drop operations.

FileExplorer.StartOperationIndicator()
--------------------------------------

Category:  Miscellaneous exports

Parameters:  None.

Returns:  Nothing.

This function starts a wait indicator when hovering over the widget for up to 15 seconds or until `StopOperationIndicator()` is called, whichever comes first.  This function is refcounted, so equal numbers of Start/Stop calls need to be made to stop the indicator.

FileExplorer.StopOperationIndicator()
-------------------------------------

Category:  Miscellaneous exports

Parameters:  None.

Returns:  Nothing.

This function reduces the refcount of StartOperationIndicator() by 1.  When it reaches 0, the wait indicator class is removed.

FileExplorer.DispatchToolEvent(eventname, params)
-------------------------------------------------

Category:  Tool events

Parameters:

* eventname - A string containing a name of a tool event to dispatch.
* params - A single option or an array of options to pass to callbacks.

Returns:  Nothing.

This function dispatches a tool event.  It should only be called by registered toolbar tools.

FileExplorer.addToolEventListener(eventname, callback)
------------------------------------------------------

Category:  Tool events

Parameters:

* eventname - A string containing a name of a tool event to listen to.
* callback - A function to call when the event fires.

Returns:  Nothing.

This function presents a familiar function for registering for custom events emitted by toolbar tools.

Most events have equivalents in the settings object and associated callbacks will automatically be registered as event listeners during initialization.

FileExplorer.removeToolEventListener(eventname, callback)
---------------------------------------------------------

Category:  Tool events

Parameters:

* eventname - A string containing a name of a tool event to stop listening to.
* callback - A function containing a registered callback to remove.

Returns:  Nothing.

This function presents a familiar function for unregistering from custom events emitted by FileExplorer.

FileExplorer.hasToolEventListener(eventname, callback)
------------------------------------------------------

Category:  Tool events

Parameters:

* eventname - A string containing a name of a tool event to check.

Returns:  A boolean of true if there are event listeners for the specified tool event, false otherwise.

This function checks for the existence of a tool event and whether or not there are any listeners registered for the event.

FileExplorer.ToolStateUpdated()
-------------------------------

Category:  Tool notifications

Parameters:  None.

Returns:  Nothing.

This function is used by tools to let the FileExplorer class instance know that at least one tool has toggled between enabled/disabled states and it should adjust tabIndex attributes and element focus as needed.

FileExplorer.AddToolbarButton(classname, title)
-----------------------------------------------

Category:  Toolbar

Parameters:

* classname - A string containing a class name to use for the toolbar button.
* title - A string containing a title to use for the button (tooltip).

Returns:  The created DOM node.

This function is used by tools to add a toolbar button during setup of the tool.

FileExplorer.Destroy()
----------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This is a hefty function that tears down the entire widget.  Calling other functions except IsDestroyed() after this function will result in an error or undefined behavior.  Given the nature of the widget, it might make more sense to hide it and show it again later if it is needed rather than destroying it so users aren't forced to constantly navigate to specific paths.

FileExplorer.IsDestroyed()
--------------------------

Category:  Destructor

Parameters:  None.

Returns:  A boolean that indicates whether or not Destroy() was called.

This function tracks whether or not an instance has been destroyed.
