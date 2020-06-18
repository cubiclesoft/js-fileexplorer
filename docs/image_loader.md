ImageLoader Class
=================

The ImageLoader class is a multi-queue delayed image loader that only loads images when it is told to.

Example usage:

```js
	function ImageLoadResult(opts, success)
	{
console.log(opts);
console.log(success);
	}

	var imageloader = new ImageLoader();

	imageloader.AddToQueue({
		src: 'https://picsum.photos/300/200',
		callback: ImageLoadResult,
		info: { id: 'cool' }
	});

	imageloader.AddToQueue({
		src: 'https://picsum.photos/150/400',
		callback: ImageLoadResult,
		info: { id: 'cool2' }
	});

	imageload.ProcessQueue();
```

ImageLoader(options)
--------------------

Category:  Consstructor

Parameters:

* options - An object of options to use to initialize the class.

Returns:  A Javascript function hierarchy.

This function sets up a new ImageLoader class instance.

The options object accepts this option:

* maxactive - An integer containing the maximum number of simultaneous requests to place in the active queue (Default is 10).

ImageLoader.settings
--------------------

Category:  Settings

The `settings` object contains the settings for the instance.

ImageLoader.AddToQueue(opts)
----------------------------

Category:  Queue actions

Parameters:

* opts - An object containing various options.

Returns:  Nothing.

This function assigns an `id` and adds the opts object to the queue.

The opts object accepts these options:

* src - A string containing an image URL to retrieve.
* width - An optional integer containing the width of the img tag.
* height - An optional integer containing the height of the img tag.
* callback - An optional callback function to call on success/failure.

The following are reserved in the opts object for internal use:

* id - An integer containing the ID of the queued item.
* started - An integer containing a timestamp of when the item entered the active queue.
* img - A DOM node containing an img tag.

ImageLoader.ProcessQueue()
--------------------------

Category:  Queue actions

Parameters:  None.

Returns:  Nothing.

This function starts processing the queue.  It moves up to `maxactive` items into the active queue.

ImageLoader.IsActive(id)
------------------------

Category:  Queue information

Parameters:

* id - An integer containing an ID of a queued item.

Returns:  A boolean of true if the item is in the active queue, false otherwise.

This function checks to see if the item with the specified ID is in the active queue.  Useful when wanting to avoid cancelling images already being retrieved.

ImageLoader.RemoveFromQueue(id)
-------------------------------

Category:  Queue actions

Parameters:

* id - An integer containing an ID of a queued item.

Returns:  Nothing.

This function removes the item specified by the ID from whichever queue it is in.  If it is in the active queue, it is cancelled first and then removed.

ImageLoader.Destroy()
---------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the ImageLoader instance.
