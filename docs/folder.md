Folder Class
============

The internal Folder class tracks the contents of a single folder in the mapped folders of the FileExplorer class.  The Folder class cannot be instantiated except from within the FileExplorer but instances of this class can be accessed primarily via callbacks from FileExplorer.

Folder(path)
------------

Category:  Constructor

Parameters:

* path - An array of path segments to define the path for this Folder instance.

Returns:  Nothing.

This function sets up a new Folder instance.  Each path segment is an array consisting of `[id, value, attrs]`.  Paths are validated via `FileExplorer.IsValidPath()`.

Folder.addEventListener(eventname, callback)
--------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a Folder event to listen to.
* callback - A function to call when the event fires.

Returns:  Nothing.

This function presents a familiar function for registering for custom events emitted by Folder.

Known events:

* set_attributes - Dispatched when SetAttributes()/SetAttribute() is called.
* set_entries - Dispatched when SetEntries()/SetEntry() is called.
* remove_entry - Dispatched when RemoveEntry() is called.
* destroy - Dispatched when Destroy() is called.

Folder.removeEventListener(eventname, callback)
-----------------------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a Folder event to stop listening to.
* callback - A function containing a registered callback to remove.

Returns:  Nothing.

This function presents a familiar function for unregistering from custom events emitted by Folder.

Folder.hasEventListener(eventname)
----------------------------------

Category:  Events

Parameters:

* eventname - A string containing a name of a Folder event to check.

Returns:  A boolean of true if there are event listeners for the specified event, false otherwise.

This function checks for the existence of an event and whether or not there are any listeners registered for the event.

Folder.lastrefresh
------------------

Category:  Internal variable

This internal integer contains the last refresh timestamp.  FileExplorer manages this value per Folder instance.

Folder.waiting
--------------

Category:  Internal variable

This internal boolean specifies whether or not the Folder is waiting for the first refresh callback to complete.  The Folder instance sets this to true during a SetEntries() call.

Folder.refs
-----------

Category:  Internal variable

This internal integer tracks reference counts by the FileExplorer instance for folder map management.  Do not use.

Folder.SetBusyRef(newval)
-------------------------

Category:  Busy state

Parameters:

* newval - An integer containing the amount to add to the current busy state refcount.

Returns:  Nothing.

This function adds the specified amount to the busy state refcount.  If the refcount becomes <= 0, any queued busy callbacks are fired in the order they were added.  Do not use.

Folder.IsBusy()
---------------

Category:  Busy state

Parameters:  None.

Returns:  A boolean indicating whether or not the folder is busy.

This function checks the busy status of the folder.  Some operations enable the busy state.  For example, when the mouse is used to start drawing a selection box, the busy state is enabled for the current folder so that the precalculated grid can't be modified by a SetEntries() call during the selection box operation, resulting in bizarre selections.  When the mouse button is released, the busy state is cancelled and any queued callbacks fire.

Folder.AddBusyQueueCallback(callback, callbackopts)
---------------------------------------------------

Category:  Busy state

Parameters:

* callback - A callback function to call when the busy state is complete.
* callbackopts - An array of options to pass to the callback.

Returns:  Nothing.

This function pushes the callback and callbackopts onto the busy queue and then calls `SetBusyRef(0)`.

Folder.ClearBusyQueueCallbacks()
--------------------------------

Category:  Busy state

Parameters:  None.

Returns:  Nothing.

This internal function clears the busy queue.  Used by FileExplorer during `Destroy()`.  Do not use.

Folder.GetPath()
----------------

Category:  Path information

Parameters:  None.

Returns:  The internal path array for the Folder instance.

This function simply returns the internal Folder path array as-is.  Do not modify.

Folder.GetPathIDs()
-------------------

Category:  Path information

Parameters:  None.

Returns:  An array of path IDs.

This function iterates over the path and constructs an array of path IDs.  The return value is intended to be useful for sending the path of a folder to a server.

Folder.SetAttributes(newattrs)
------------------------------

Category:  Path attributes

Parameters:

* newattrs - An object containing new attributes to set for the current path.

Returns:  Nothing.

This function overwrites the attributes of the current path item and dispatches the `set_attributes` event.

Folder.SetAttribute(key, value)
-------------------------------

Category:  Path attributes

Parameters:

* key - A string containing the attribute key.
* value - A value to assign to the attribute.

Returns:  Nothing.

This function assigns an attribute of the current path item and dispatches the `set_attributes` event.

Folder.GetAttributes()
----------------------

Category:  Path attributes

Parameters:  None.

Returns:  The current attributes object for the path.

This function returns the internal attributes object for the path.  Do not modify directly.

Folder.SetAutoSort(newautosort)
-------------------------------

Category:  Sorting

Parameters:

* newautosort - A boolean that specifies whether or not the Folder instance should automatically sort entries.

Returns:  Nothing.

This function enables/disables automatic sorting of entries.

Folder.SortEntries()
--------------------

Category:  Sorting

Parameters:  None.

Returns:  Nothing.

This function sorts the entries using a locale-sensitive + numeric comparison function.  Entries with a type of 'folder' also sort so they come before 'file' entries.

Folder.SetEntries(newentries)
-----------------------------

Category:  Entries

Parameters:

* newentries - An array of objects containing the entries to set.

Returns:  Nothing.

This function sets the new entries array, automatically sorts the entries (if enabled), rebuilds the entry ID map (ID to position in array), and dispatches the 'set_entries' event.

Each entry consists of an object with the following required key-value pairs:

* id - A unique string for the Folder instance that is DOM dataset compatible.  This string should also be server compatible (e.g. map to a database).
* name - A string to display to the user.  Used by automatic sorting (if enabled).
* type - A string containing one of 'folder' or 'file'.
* hash - A string containing a hash or something uniquely identifiable to use during DOM updates that an item has NOT changed.

Each entry consists of an object with the following optional key-value pairs:

* attrs - An object containing key-value pairs.  Currently, 'canmodify' is the only key-value recognized by FileExplorer.
* size - An integer containing the size, in bytes, of the entry on disk.
* tooltip - A string to use for the title to display as a tooltip when hovering.  Most browsers natively support multi-line tooltips.
* thumb - A string containing a URL to a thumbnail image to display.  A standard icon is displayed while the thumbnail loads in the background.  Thumbnails of large images should be scaled to no smaller than 196x196 to handle 4x pixel densities.
* overlay - A string containing one or an array containing one or more class names to apply to the item in the DOM for purposes of displaying an overlay icon/text using CSS.

Folder.UpdateEntries(updatedentries)
------------------------------------

Category:  Entries

Parameters:

* updatedentries - An array of objects containing the entries to add/update.

Returns:  Nothing.

This function adds/updates the entries array, automatically sorts the entries (if enabled), rebuilds the entry ID map (ID to position in array), and dispatches the 'set_entries' event.

Folder.SetEntry(entry)
----------------------

Category:  Entries

Parameters:

* entry - An object containing the entry to set.

Returns:  Nothing.

This function sets a single entry, automatically sorts the entries (if enabled), rebuilds the entry ID map (ID to position in array), and dispatches the 'set_entries' event.

This is an inefficient function to call repeatedly for setting a bunch of entries.  It's designed for convenience to set a single entry.

Folder.RemoveEntry(id)
----------------------

Category:  Entries

Parameters:

* id - A string containing the ID of the entry to remove.

Returns:  Nothing.

This function removes a single entry, adjusts the entry ID map (ID to position in array), and dispatches the 'remove_entry' event.

This is an inefficient function to call for more than a couple of entries at one time.  It's designed for convenience and performance for removing just one entry at a time.

Folder.GetEntries()
-------------------

Category:  Entries

Parameters:  None.

Returns:  The array of entries.

This function returns the internal array of entries for the Folder instance.

Folder.GetEntryIDMap()
----------------------

Category:  Entries

Parameters:  None.

Returns:  The object of entry IDs to positions into the entry array.

This function returns the internal precalculated ID to position map for the Folder instance.  This function is usually used in conjunction with GetEntries() for high-performance lookups by ID.

Folder.Destroy()
----------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the Folder instance.  Should only be called by FileExplorer when the folder reference count reaches zero.
