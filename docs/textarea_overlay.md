TextareaOverlay Class
=====================

The TextareaOverlay class displays a positionable textarea with optional text with text selection.  Full keyboard, mouse, and touch support.

Example usage:

```js
	options = {
		initvalue: 'hello.txt',
		initselstart: 0,
		initselend: 5,

		onposition: function(textelem) {
			textelem.style.left = 50 + 'px';
			textelem.style.top = 40 + 'px';
			textelem.style.width = 50 + 'px';
			textelem.style.height = Math.min(elem.offsetHeight - 2, elem.scrollHeight - 42, textelem.scrollHeight + 2) + 'px';
		},

		ondone: function(val, lastelem) {
			if (lastelem)  lastelem.focus();
console.log(val);

			this.Destroy();
		},

		oncancel: function(lastelem) {
			if (lastelem)  lastelem.focus();

			this.Destroy();
		},
	};

	var textoverlaytest = new window.FileExplorer.TextareaOverlay(elem, options);
```

TextareaOverlay(parentelem, options)
------------------------------------

Category:  Constructor

Parameters:

* parentelem - A DOM node to append the root of the TextareaOverlay widget to.
* options - An object containing options to use to construct the widget.

Returns:  A Javascript function hierarchy.

This function sets up a new TextareaOverlay widget instance.

The options object accepts these options:

* capturetab - A boolean indicating whether or not to capture the Tab key in the textarea and emit a tab (Default is false).
* multiline - A boolean indicating whether or not to capture the Enter key in the textarea and emit a newline (Default is false).
* initvalue - A string containing the initial value to use for the textarea (Default is '').
* initselstart - An integer specifying the start position of the selected text (Default is -1).  If negative, the start position is at the end.
* initselend - An integer specifying the end position of the selected text (Default is -1).  If negative, the end position is at the end.
* resizewatchers - An optional array of objects compatible with DebounceAttributes watchers.  Useful for repositioning the popup menu during a resize operation.
* onposition - An optional callback function to position and size the textarea.
* ondone - An optional callback function to receive the new textarea value.  Only sent if the value changed.
* oncancel - An optional callback function to be notified if the overlay is cancelled or the value did not change.
* ondestroy - An optional callback function to be notified when the widget is destroyed.

TextareaOverlay.settings
------------------

Category:  Settings

The `settings` object contains the settings for the instance.  Changing the settings after creating the instance will have little to no effect and changing them is not recommended.

TextareaOverlay.addEventListener(eventname, callback)
-----------------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a TextareaOverlay event to listen to.
* callback - A function to call when the event fires.

Returns:  Nothing.

This function presents a familiar function for registering for custom events emitted by TextareaOverlay.

Known events:

* position - Dispatched when the textarea should be positioned.
* done - Dispatched when the textarea is done being edited and the value changed.
* cancelled - Dispatched if the overlay is cancelled or the value did not change.
* destroy - Dispatched when the Destroy() function is called.

Most of these events have equivalents in the settings object and associated callbacks will automatically be registered as event listeners during initialization.

TextareaOverlay.removeEventListener(eventname, callback)
--------------------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a TextareaOverlay event to stop listening to.
* callback - A function containing a registered callback to remove.

Returns:  Nothing.

This function presents a familiar function for unregistering from custom events emitted by TextareaOverlay.

TextareaOverlay.hasEventListener(eventname)
-------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a TextareaOverlay event to check.

Returns:  A boolean of true if there are event listeners for the specified event, false otherwise.

This function checks for the existence of an event and whether or not there are any listeners registered for the event.

TextareaOverlay.UpdatePosition()
--------------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function changes the height of the textarea to 1px and triggers a position update call.

TextareaOverlay.Done(etype)
---------------------------

Category:  Actions

Parameters:

* etype - An optional string containing an event type.

Returns:  Nothing.

This function makes the textarea read only and triggers the done event if it hasn't been disallowed and the text has been modified.  Otherwise, it calls Cancel().  The etype value can be used to, for example, prevent another textarea from immediately appearing when clicking on a button that originally created the overlay.

TextareaOverlay.Cancel(etype)
-----------------------------

Category:  Actions

Parameters:

* etype - An optional string containing an event type.

Returns:  Nothing.

This function triggers the cancelled event if it hasn't been disallowed.  The etype value can be used to, for example, prevent another textarea from immediately appearing when clicking on a button that originally created the overlay.

TextareaOverlay.ResetAllowCancelDone()
--------------------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function resets the textarea to be editable, sets keyboard focus to the textarea, and allows `Done()` and `Cancel()` to be called again in the future.

TextareaOverlay.Destroy()
-------------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the TextareaOverlay instance.  Note that the focused element should be moved to another element prior to calling Destroy().
