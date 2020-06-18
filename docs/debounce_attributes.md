DebounceAttributes Class
========================

The DebounceAttributes class debounces/throttles based on changing attribute values instead of debouncing/throttling DOM events.  The result is a cleaner, more performant solution.

Example usage:

```js
	var statusbarresizewatch = new DebounceAttributes({
		watchers: [
			{ elem: elems.mainwrap, attr: 'offsetWidth', val: -1 }
		],
		interval: 50,
		stopsame: 5,
		callback: CalculateUpdateMultilineStatus,
		intervalcallback: CalculateUpdateMultilineStatus
	});

	window.addEventListener('resize', statusbarresizewatch.Start, true);
```

DebounceAttributes(options)
---------------------------

Category:  Constructor

Parameters:

* options - An object of options to use to initialize the class.

Returns:  A Javascript function hierarchy.

This function sets up a new DebounceAttributes class instance, which efficiently watches one or more element attributes for changes, and stops watching when the changes stop.  A better alternative to traditional debouncing.  The 'new' keyword is required in order to use it correctly.

The options object accepts these options:

* watchers - An array of objects each containing an element and attribute name to watch and an initial value (Default is []).
* interval - An integer containing the number of milliseconds in between value checks (Default is 50).
* stopsame - An integer containing the number of sequential identical values of the attribute before the watcher clears the interval (Default is 1).
* callback - A function to call when the interval is cleared.
* intervalcallback - A function to call each interval.

DebounceAttributes.Start()
--------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function starts the interval timer if it isn't already started to begin watching for attribute changes.

DebounceAttributes.Stop()
-------------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function stops the interval timer.

DebounceAttributes.Destroy()
----------------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the DebounceAttributes instance.
