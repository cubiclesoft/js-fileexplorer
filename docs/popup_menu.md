PopupMenu Class
===============

The PopupMenu class displays a popup menu of items.  Full keyboard, mouse, and touch support.

Example usage:

```js
	options = {
		items: [
			{ id: 0, name: 'Test' },
			{ id: 1, name: 'Test 2', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			'split',
			{ id: 2, name: '(Empty)', icon: 'fe_fileexplorer_popup_item_icon_folder', enabled: false },
			'split',
			{ id: 3, name: 'Test 3', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 4, name: 'Test 4', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 5, name: 'Test 5', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 6, name: 'Test 6', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 7, name: 'Test 7', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 8, name: 'Test 8', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 9, name: 'Test 9', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 10, name: 'Test 10', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 11, name: 'Test 11', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 12, name: 'Test 12', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 13, name: 'Test 13', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 14, name: 'Test 14', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 15, name: 'Test 15', icon: 'fe_fileexplorer_popup_item_icon_folder' },
			{ id: 16, name: 'Test 16', icon: 'fe_fileexplorer_popup_item_icon_folder' },
		],

		onposition: function(popupelem) {
			var maxleft = elem.offsetWidth - popupelem.offsetWidth;

			popupelem.style.left = (maxleft < 900 ? maxleft : 900) + 'px';
			popupelem.style.top = (elem.offsetHeight - 25) + 'px';
		},

		onselected: function(id, item, lastelem, etype) {
			if (lastelem)  lastelem.focus();

console.log(id);
console.log(item);

			this.Destroy();
		},

		oncancel: function(lastelem, etype) {
			if (lastelem)  lastelem.focus();

			this.Destroy();
		},
	};

	var popuptest = new window.FileExplorer.PopupMenu(elem, options);
```

PopupMenu(parentelem, options)
------------------------------

Category:  Constructor

Parameters:

* parentelem - A DOM node to append the root of the PopupMenu widget to.
* options - An object containing options to use to construct the widget.

Returns:  A Javascript function hierarchy.

This function sets up a new PopupMenu widget instance.

The options object accepts these options:

* items - An array of objects and strings containing the items to show (Default is []).
* resizewatchers - An optional array of objects compatible with DebounceAttributes watchers.  Useful for repositioning the popup menu during a resize operation.
* onposition - An optional callback function to position the popup menu.
* onselchanged - An optional callback function to be notified whenever the selection is changed.
* onselected - An optional callback function to receive the selected item.
* oncancel - An optional callback function to be notified if the popup is cancelled.
* onleft - An optional callback function to be notified if the user presses the left arrow key.
* onright - An optional callback function to be notified if the user presses the right arrow key.
* ondestroy - An optional callback function to be notified when the widget is destroyed.

PopupMenu.settings
------------------

Category:  Settings

The `settings` object contains the settings for the instance.  Changing the settings after creating the instance will have little to no effect and changing them is not recommended.

PopupMenu.addEventListener(eventname, callback)
-----------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a PopupMenu event to listen to.
* callback - A function to call when the event fires.

Returns:  Nothing.

This function presents a familiar function for registering for custom events emitted by PopupMenu.

Known events:

* position - Dispatched when the popup menu should be positioned.
* selection_changed - Dispatched when the selection is changed.
* selected - Dispatched when the user selects an item.
* cancelled - Dispatched if the popup menu is cancelled.
* left - Dispatched when the user presses the left arrow key.
* right - Dispatched when the user presses the right arrow key.
* destroy - Dispatched when the Destroy() function is called.

Most of these events have equivalents in the settings object and associated callbacks will automatically be registered as event listeners during initialization.

PopupMenu.removeEventListener(eventname, callback)
--------------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a PopupMenu event to stop listening to.
* callback - A function containing a registered callback to remove.

Returns:  Nothing.

This function presents a familiar function for unregistering from custom events emitted by PopupMenu.

PopupMenu.hasEventListener(eventname)
-------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a PopupMenu event to check.

Returns:  A boolean of true if there are event listeners for the specified event, false otherwise.

This function checks for the existence of an event and whether or not there are any listeners registered for the event.

PopupMenu.UpdatePosition()
--------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function moves the popup menu off screen and triggers a position update call.

PopupMenu.Cancel(etype)
-----------------------

Category:  Actions

Parameters:

* etype - An optional string containing an event type.

Returns:  Nothing.

This function triggers the cancelled event if it hasn't been prevented.  The etype value can be used to, for example, prevent another popup menu from immediately appearing when clicking on the original button that created the popup menu.

PopupMenu.PreventCancel()
-------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function prevents Cancel() from having any effect.  Useful for moving focus to a specific element which would cause Cancel() to be called, which would trigger a callback, which would set focus to a different element.

PopupMenu.Destroy()
-------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the PopupMenu instance.  Note that the focused element should be moved to another element prior to calling Destroy().
