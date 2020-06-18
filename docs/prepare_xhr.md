PrepareXHR Class
================

The PrepareXHR class is a basic and convenient wrapper around a XMLHttpRequest (XHR) object.

Example usage:

```js
	var xhr = new PrepareXHR({
		url: '/filemanager/',
		params: {
			action: 'refresh',
			path: JSON.stringify(['/', 'some', 'dir']),
			xsrftoken: 'asdfasdf'
		},
		onsuccess: function(e) {
console.log(e.target.response);
		},
		onerror: function(e) {
console.log(e);
		}
	});

	xhr.Send();
```

PrepareXHR(options)
-------------------

Category:  Constructor

Parameters:

* options - An object of options to use to initialize the class.

Returns:  A Javascript function hierarchy.

This function sets up a new PrepareXHR class instance.

The options object accepts these options:

* method - An optional string containing a HTTP method to use for the request.
* url - A string containing the URL to use.  Must be XHR-compatible (e.g. CORS headers or same domain).
* headers - An optional object containing key-value pairs of custom HTTP headers to send.
* params - An optional object or array or FormData instance containing POST parameters to send.  Not compatible with the body option.
* body - An optional string containing the body of the request to send.  Not compatible with the params option.
* onsuccess - An optional callback function that is run if the request completes successfully.
* onerror - An optional callback function that is run if the request fails.
* onload - An optional callback function that is run if the request completes successfully.
* onabort - An optional callback function that is run if the request is aborted.
* onloadstart - An optional callback function that is run when the request starts.
* onprogress - An optional callback function that is periodically run as the response data arrives.
* ontimeout - An optional callback function that is run if the request times out.
* onloadend - An optional callback function that is run when the request ends (regardless if it succeeds or not).

PrepareXHR.xhr
--------------

Category:  Variable

This variable is the internal-ish XMLHttpRequest (XHR) object instance.

PrepareXHR.upload.addEventListener(type, listener, options)
-----------------------------------------------------------

Category:  Events

Parameters:

* type - A string containing the XHR event type to listen to.
* listener - A callback function to call when the XHR event fires.
* options - A boolean or an object containing standard event listener options.

Returns:  Nothing.

This function adds an event listener to the XHR object instance upload/request event handlers.

PrepareXHR.upload.removeEventListener(type, listener, options)
--------------------------------------------------------------

Category:  Events

Parameters:

* type - A string containing the XHR event type to stop listening to.
* listener - A callback function to remove.
* options - A boolean or an object containing standard event listener options.

Returns:  Nothing.

This function removes an event listener from the XHR object instance upload/request event handlers.

PrepareXHR.addEventListener(type, listener, options)
----------------------------------------------------

Category:  Events

Parameters:

* type - A string containing the XHR event type to listen to.
* listener - A callback function to call when the XHR event fires.
* options - A boolean or an object containing standard event listener options.

Returns:  Nothing.

This function adds an event listener to the XHR object instance response event handlers.

PrepareXHR.removeEventListener(type, listener, options)
-------------------------------------------------------

Category:  Events

Parameters:

* type - A string containing the XHR event type to stop listening to.
* listener - A callback function to remove.
* options - A boolean or an object containing standard event listener options.

Returns:  Nothing.

This function removes an event listener from the XHR object instance response event handlers.

PrepareXHR.GetMethod()
----------------------

Category:  HTTP information

Parameters:  None.

Returns:  A string containing the HTTP method to use.

This function returns the calculated HTTP method to use based on the input constructor options object.

PrepareXHR.PrepareBody()
------------------------

Category:  Actions

Parameters:  None.

Returns:  A value that can be passed to `Send()`.

This function prepares the XHR body to send.  It is automatically called if Send() is not passed anything.

PrepareXHR.Send(xhrbody)
------------------------

Category:  Actions

Parameters:

* xhrbody - Undefined or a value compatible with `XMLHttpRequest.send()`.

Returns:  Nothing.

This function starts sending an asynchronous request to the server.  If `xhrbody` is not defined, this function calls `PrepareBody()` to generate the value to send.

PrepareXHR.Abort()
------------------

Category:  Actions

Parameters:  None.

Returns:  Nothing.

This function aborts a request/upload/response in progress.

PrepareXHR.Destroy()
--------------------

Category:  Destructor

Parameters:  None.

Returns:  Nothing.

This function destroys the PrepareXHR instance.
