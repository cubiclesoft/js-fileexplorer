FileExplorerFSHelper Class
==========================

The FileExplorerFSHelper class is a server-side helper class designed to handle common actions of the FileExplorer Javascript class on a server running PHP where the widget will map directly to the same physical file system that the server software is running on.  For example, returning a directory listing of Folder-compatible entries when a refresh request is made in a `/files` subdirectory of the application.

When using this class, it is recommended that a cron job run at least nightly that clears out old ".tmp" files (failed/abandoned uploads) and removes old files and directories in the recycling bin and thumbnail directories.  What defines a file as old depends on the application and use-case.

Example usage:

```php
<?php
	// Note that this example is insecure as it is missing user checks, XSRF/CSRF token checking/generation, etc.
	// The /files/ and /thumbs/ directories need to be writeable by the web server user (e.g. www-data).

	require_once "server-side-helpers/file_explorer_fs_helper.php";

	$options = array(
		"base_url" => "https://cubiclesoft.com/myapp/files/",
		"protect_depth" => 1,  // Protects base directory + additional directory depth.
		"recycle_to" => "Recycle Bin",
		"temp_dir" => "/tmp",
		"dot_folders" => false,  // .git, .svn, .DS_Store
		"allowed_exts" => ".jpg, .jpeg, .png, .gif, .svg, .txt",
		"allow_empty_ext" => true,
		"thumbs_dir" => "/var/www/myapp/thumbs",
		"thumbs_url" => "https://cubiclesoft.com/myapp/thumbs/",
		"thumb_create_url" => "https://cubiclesoft.com/myapp/?action=file_explorer_thumbnail&xsrftoken=qwerasdf",
		"refresh" => true,
		"rename" => true,
		"new_folder" => true,
		"new_file" => ".txt",
		"upload" => true,
		"upload_limit" => 20000000,  // -1 for unlimited or an integer
		"download" => "user123-" . date("Y-m-d_H-i-s") . ".zip",
		"download_module" => "",  // Server handler for single-file downloads:  "" (none), "sendfile" (Apache), "accel-redirect" (Nginx)
		"download_module_prefix" => "",  // A string to prefix to the filename.  (For URI /protected access mapping for a Nginx X-Accel-Redirect to the system root)
		"copy" => true,
		"move" => true,
		"recycle" => true,
		"delete" => true
	);

	FileExplorerFSHelper::HandleActions("action", "file_explorer_", "/var/www/myapp/files", $options);
?>
<!DOCTYPE html>
<html>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
<head><title>Manage Files</title></head>
<body>
<style type="text/css">
body { font-family: Verdana, Arial, Helvetica, sans-serif; position: relative; color: #222222; font-size: 1.0em; }
</style>

<link rel="stylesheet" type="text/css" href="file-explorer/file-explorer.css">
<script type="text/javascript" src="file-explorer/file-explorer.js"></script>

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

			var $this = this;

			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_refresh',
					path: JSON.stringify(folder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
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

		onrename: function(renamed, folder, entry, newname) {
			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_rename',
					path: JSON.stringify(folder.GetPathIDs()),
					id: entry.id,
					newname: newname,
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
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

		onopenfile: function(folder, entry) {
console.log(entry);
		},

		onnewfolder: function(created, folder) {
			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_new_folder',
					path: JSON.stringify(folder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
console.log(data);

					if (data.success)  created(data.entry);
					else  created(data.error);
				},
				onerror: function(e) {
console.log(e);
					created('Server/network error.');
				}
			});

			xhr.Send();
		},

		onnewfile: function(created, folder) {
			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_new_file',
					path: JSON.stringify(folder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
console.log(data);

					if (data.success)  created(data.entry);
					else  created(data.error);
				},
				onerror: function(e) {
console.log(e);
					created('Server/network error.');
				}
			});

			xhr.Send();
		},

		oninitupload: function(startupload, fileinfo, queuestarted) {
console.log(fileinfo.file);
console.log(JSON.stringify(fileinfo.folder.GetPathIDs()));

			var $this = this;

			if (fileinfo.type === 'dir')
			{
				// Create a directory.  This type only shows up if the directory is empty.
				// Set a URL, headers, and params to send to the server.
				fileinfo.url = window.location.href;

				fileinfo.headers = {
				};

				fileinfo.params = {
					action: 'file_explorer_new_folder',
					path: JSON.stringify(fileinfo.folder.GetPathIDs()),
					name: fileinfo.fullPath,
					xsrftoken: 'asdfasdf'
				};

				fileinfo.currpathparam = 'currpath';

				// Automatic retry count for the directory on failure.
				fileinfo.retries = 3;

				// Create the directory.
				startupload(true);
			}
			else
			{
				var origcurrfolder = $this.GetCurrentFolder();

				// Prepare the file upload on the server.
				var xhr = new this.PrepareXHR({
					url: window.location.href,
					params: {
						action: 'file_explorer_upload_init',
						path: JSON.stringify(fileinfo.folder.GetPathIDs()),
						name: fileinfo.fullPath,
						size: fileinfo.file.size,
						currpath: JSON.stringify(origcurrfolder.GetPathIDs()),
						queuestarted: queuestarted,
						xsrftoken: 'asdfasdf'
					},
					onsuccess: function(e) {
						var data = JSON.parse(e.target.responseText);
console.log(data);

						if (!data.success)  startupload(data.error);
						else
						{
							if (data.entry && $this.IsMappedFolder(origcurrfolder))  origcurrfolder.SetEntry(data.entry);

							// Set a URL, headers, and params to send with the file data to the server.
							fileinfo.url = window.location.href;

							fileinfo.headers = {
							};

							fileinfo.params = {
								action: 'file_explorer_upload',
								path: JSON.stringify(fileinfo.folder.GetPathIDs()),
								name: fileinfo.fullPath,
								size: fileinfo.file.size,
								queuestarted: queuestarted,
								xsrftoken: 'asdfasdf'
							};

							fileinfo.fileparam = 'file';
							fileinfo.currpathparam = 'currpath';

							// Optional:  Send chunked uploads.  Requires the server to know how to put chunks back together.
							fileinfo.chunksize = 1000000;

							// Optional:  Automatic retry count for the file on failure.
							fileinfo.retries = 3;

							// Start the upload.
							startupload(true);
						}
					},
					onerror: function(e) {
console.log(e);
						startupload('Server/network error.');
					}
				});

				xhr.Send();
			}
		},

		onuploaderror: function(fileinfo, e) {
console.log(e);
console.log(fileinfo);
		},

		oninitdownload: function(startdownload, folder, ids, entries) {
			// Set a URL and params to send with the request to the server.
			var options = {};

			// Optional:  HTTP method to use.
//			options.method = 'POST';

			options.url = window.location.href;

			options.params = {
				action: 'file_explorer_download',
				path: JSON.stringify(folder.GetPathIDs()),
				ids: JSON.stringify(ids),
				xsrftoken: 'asdfasdf'
			};

			// Optional:  Control the download via an in-page iframe (default) vs. form only (new tab).
//			options.iframe = false;

			startdownload(options);
		},

		ondownloadstarted: function(options) {
console.log('started');
console.log(options);
		},

		ondownloaderror: function(options) {
console.log('error');
console.log(options);
		},

		oncopy: function(copied, srcpath, srcids, destfolder) {
			var $this = this;

			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_copy_init',
					srcpath: JSON.stringify($this.GetPathIDs(srcpath)),
					srcids: JSON.stringify(srcids),
					destpath: JSON.stringify(destfolder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
console.log(data);

					if (!data.success)  copied(data.error);
					else if (data.overwrite > 0 && !confirm($this.FormatStr($this.Translate('Copying will overwrite {0} ' + (data.overwrite === 1 ? 'item' : 'items') + '.  Proceed?'), data.overwrite)))  copied('Copy cancelled.');
					else
					{
						var runxhr, origcurrfolder;

						var CancelXHR = function() {
							runxhr.Abort();
						};

						var progresstracker = $this.CreateProgressTracker(CancelXHR);

						var options = {
							url: window.location.href,
							params: {
								action: 'file_explorer_copy',
								copykey: data.copykey,
								xsrftoken: 'asdfasdf'
							},
							onsuccess: function(e) {
								var data = JSON.parse(e.target.responseText);
console.log(data);

								if (!data.success)  copied(data.error, data.finalentries);
								else
								{
									progresstracker.totalbytes = data.totalbytes;
									progresstracker.queueditems = data.queueditems;
									progresstracker.queuesizeunknown = data.queuesizeunknown;
									progresstracker.itemsdone = data.itemsdone;
									progresstracker.faileditems = data.faileditems;

									if ($this.IsMappedFolder(origcurrfolder))  origcurrfolder.UpdateEntries(data.currentries);

									if (data.queueditems)  NextRun();
									else
									{
										$this.RemoveProgressTracker(progresstracker, 'Copying done');
										progresstracker = null;

										copied(true, data.finalentries);

										runxhr.Destroy();
										runxhr = null;
									}
								}
							},
							onerror: function(e) {
console.log(e);
								copied('Server/network error.');

								runxhr.Destroy();
								runxhr = null;
							},
							onabort: function(e) {
								progresstracker.queueditems = 0;
								progresstracker.queuesizeunknown = false;

								$this.RemoveProgressTracker(progresstracker, 'Copying stopped');
								progresstracker = null;

								copied(false);

								runxhr.Destroy();
								runxhr = null;
							}
						};

						// Performs another copy operation cycle.
						var NextRun = function() {
							if (runxhr)  runxhr.Destroy();

							origcurrfolder = $this.GetCurrentFolder();
							options.params.currpath = JSON.stringify(origcurrfolder.GetPathIDs());

							runxhr = new $this.PrepareXHR(options);

							runxhr.Send();
						};

						NextRun();
					}
				},
				onerror: function(e) {
console.log(e);
					copied('Server/network error.');
				}
			});

			xhr.Send();
		},

		onmove: function(moved, srcpath, srcids, destfolder) {
			var $this = this;

			var xhr = new this.PrepareXHR({
				url: window.location.href,
				params: {
					action: 'file_explorer_move',
					srcpath: JSON.stringify($this.GetPathIDs(srcpath)),
					srcids: JSON.stringify(srcids),
					destpath: JSON.stringify(destfolder.GetPathIDs()),
					xsrftoken: 'asdfasdf'
				},
				onsuccess: function(e) {
					var data = JSON.parse(e.target.responseText);
console.log(data);

					if (!data.success)  moved(data.error);
					else  moved(true, data.entries);
				},
				onerror: function(e) {
console.log(e);
					moved('Server/network error.');
				}
			});

			xhr.Send();
		},

		ondelete: function(deleted, folder, ids, entries, recycle) {
			var $this = this;

			// Ask the user if they really want to delete/recycle the items.
			if (!recycle && !confirm('Are you sure you want to permanently delete ' + (entries.length == 1 ? '"' + entries[0].name + '"' : entries.length + ' items') +  '?'))  deleted('Cancelled deletion');
			else
			{
				var xhr = new this.PrepareXHR({
					url: window.location.href,
					params: {
						action: (recycle ? 'file_explorer_recycle' : 'file_explorer_delete'),
						path: JSON.stringify(folder.GetPathIDs()),
						ids: JSON.stringify(ids),
						xsrftoken: 'asdfasdf'
					},
					onsuccess: function(e) {
						var data = JSON.parse(e.target.responseText);
console.log(data);

						if (!data.success)  deleted(data.error);
						else  deleted(true);
					},
					onerror: function(e) {
console.log(e);
						deleted('Server/network error.');
					}
				});

				xhr.Send();
			}
		},
	};

	var fe = new window.FileExplorer(elem, options);
})();
</script>
```

FileExplorerFSHelper::GetRequestVar($name)
------------------------------------------

Access:  public static

Parameters:

* $name - A string containing a key to look up in the $_POST and $_GET superglobals.

Returns:  The value in $_POST or $_GET if it exists, a boolean of false otherwise.

This static function returns a request variable.

FileExplorerFSHelper::GetSanitizedPath($basedir, $name, $allowdotfolders = false, $extrapath = "")
--------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $basedir - A string containing a base directory.
* $name - A string containing a key to pass to `GetRequestVar()`.
* $allowdotfolders - A boolean indicating whether or not to allow dot folders (e.g. .git) in the path (Default is false).
* $extrapath - A string containing extra path information from a `GetSanitizedFileAndExtraPath()` call (Default is "").

Returns:  A string containing a safe, sanitized path on success, a boolean of false otherwise.

This static function decodes and sanitizes an incoming JSON encoded array of path IDs for the local file system.  It prevents all attempts to leave the base directory and access other parts of the file system.

FileExplorerFSHelper::CleanFilename($file)
------------------------------------------

Access:  public static

Parameters:

* $file - A string containing a filename to clean.

Returns:  A string with bad filename characters removed, a boolean of false if the resulting string is "", ".", or "..".

This static function removes specific characters from filenames and rejects specific filenames that may be attempts to circumvent defenses.

FileExplorerFSHelper::GetSanitizedFile($name)
---------------------------------------------

Access:  public static

Parameters:

* $name - A string containing a key to pass to `GetRequestVar()`.

Returns:  A string containing a cleaned filename on success, a boolean of false otherwise.

This static function retrieves and sanitizes a filename.  If the filename may include path symbols, use the `GetSanitizedFileAndExtraPath()` function instead to extract the extra path information.

FileExplorerFSHelper::GetSanitizedFileAndExtraPath(&$extrapath, $name, $allowdotfolders = false)
------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $extrapath - A string containing extracted sanitized extra path information.
* $name - A string containing a key to pass to `GetRequestVar()`.
* $allowdotfolders - A boolean indicating whether or not to allow dot folders (e.g. .git) in the extra path (Default is false).

Returns:  A string containing a cleaned filename and extra path information in $extrapath on success, a boolean of false otherwise.

This static function retrieves and sanitizes a filename after extracting and sanitizing additional extra path information.  Extra path information may be passed into `GetSanitizedPath()`.

FileExplorerFSHelper::GetFileExt($file, $default = false)
---------------------------------------------------------

Access:  public static

Parameters:

* $file - A string containing a filename.
* $default - A default value to use if the input filename does not containing a file extension (Default is false).

Returns:  A string containing the last file extension (minus the dot) on success, the value of $default otherwise.

This static function extracts the last file extension of the filename, if any.

FileExplorerFSHelper::ExtractAllowedExtensions($exts)
-----------------------------------------------------

Access:  public static

Parameters:

* $exts - A string containing a comma-separated list of file extensions to parse.

Returns:  An array of extracted file extensions.

This static function parses the `allowed_exts` option for various actions into a usable array.

FileExplorerFSHelper::HasAllowedExt(&$allowedexts, $allowempty, $file)
----------------------------------------------------------------------

Access:  public static

Parameters:

* $allowedexts - A boolean or an array of allowed extensions.
* $allowempty - A boolean that indicates whether or not to allow empty file extensions.
* $file - A string containing a filename.

Returns:  A boolean that indicates whether or not the file has an allowed file extension.

This static function calculates whether a file has an allowed file extension.

FileExplorerFSHelper::GetPathDepth($path, $basedir)
---------------------------------------------------

Access:  public static

Parameters:

* $path - A string containing a full file system path.
* $basedir - A string containing a calculated base directory.

Returns:  An integer containing the directory depth.

This static function calculates and returns the directory depth from the base directory.

FileExplorerFSHelper::GetEntryHash($type, $file, &$info)
--------------------------------------------------------

Access:  public static

Parameters:

* $type - A string containing a file type.
* $file - A string containing the filename.
* $info - An array containing the result of a `stat()` call on the associated directory/file.

Returns:  A string containing a MD5 hash.

This static function calculates a basic hash of various bits of information that make the Folder-compatible entry unique.  The hash is used by the FileExplorer class to decide whether or not it needs to update an existing entry in the DOM during folder refreshes.

FileExplorerFSHelper::IsRecycleBin($path, &$options)
----------------------------------------------------

Access:  public static

Parameters:

* $path - A string containing a full file system path.
* $options - An array of options previously processed by various action handlers.

Returns:  A boolean indicating whether or not the path is in the recycling bin.

This static function returns whether or not the path is in the recycling bin.  The recycling bin is a specially-treated folder by this class that can't be directly modified.

FileExplorerFSHelper::ConvertBytesToUserStr($num)
-------------------------------------------------

Access:  public static

Parameters:

* $num - An integer containing the number of bytes to convert.

Returns:  A string in a compact size format ready for display to a user.

This static function converts an integer value (e.g. 1234567890) to a compact size format (e.g. "1.1 GB") for display.

FileExplorerFSHelper::GetUserInfoByID($uid)
-------------------------------------------

Access:  public static

Parameters:

* $uid - An integer containing a user ID to retrieve information for.

Returns:  An array of information about the user on success, a boolean of false otherwise.

This static function calls `posix_getpwuid` and also caches the response information for later.  This function only works on systems that support the POSIX extension.

FileExplorerFSHelper::GetUserName($uid)
---------------------------------------

Access:  public static

Parameters:

* $uid - An integer containing a user ID to retrieve the username of.

Returns:  A string containing the username on success, an empty string otherwise.

This static function calls `GetUserInfoByID()`.

FileExplorerFSHelper::GetGroupInfoByID($gid)
--------------------------------------------

Access:  public static

Parameters:

* $gid - An integer containing a group ID to retrieve information for.

Returns:  An array of information about the group on success, a boolean of false otherwise.

This static function calls `posix_getgrgid` and also caches the response information for later.  This function only works on systems that support the POSIX extension.

FileExplorerFSHelper::GetGroupName($gid)
----------------------------------------

Access:  public static

Parameters:

* $gid - An integer containing a group ID to retrieve information for.

Returns:  A string containing the username on success, an empty string otherwise.

This static function calls `GetGroupInfoByID()`.

FileExplorerFSHelper::GetDestCropAndSize(&$cropx, &$cropy, &$cropw, &$croph, &$destwidth, &$destheight, $srcwidth, $srcheight, $crop, $maxwidth, $maxheight)
------------------------------------------------------------------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $cropx - The variable that will store an integer containing the final upper-left corner x coordinate.
* $cropy - The variable that will store an integer containing the final upper-left corner y coordinate.
* $cropw - The variable that will store an integer containing the final crop width.
* $croph - The variable that will store an integer containing the final crop height.
* $destwidth - The variable that will store an integer containing the final image width.
* $destheight - The variable that will store an integer containing the final image height.
* $srcwidth - An integer containing the source image width.
* $srcheight - An integer containing the source image height.
* $crop - A comma-separated string containing a crop rectangle.
* $maxwidth - An integer containing the maximum image width of the final image.
* $maxheight - An integer containing the maximum image height of the final image.

Returns:  Nothing.

This static function performs calculations of the final image width and cropping region based on input image width and height, cropping region, and maximum width.

FileExplorerFSHelper::CropAndScaleImage($data, $crop, $maxwidth = false, $maxheight = false)
--------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $data - A string containing an image to crop and scale.
* $crop - A comma-separated string containing a crop rectangle.
* $maxwidth - An integer containing the maximum image width of the final image (Default is false).
* $maxheight - An integer containing the maximum image height of the final image (Default is false).

Returns:  A standard array of information.

This static function uses the best available image library (PECL Imagick, GD) to scale and crop SDK supported image formats.  It is recommended to cache the resulting image and prevent unauthorized use of this function to preserve system resources.

FileExplorerFSHelper::GetTooltip($path, $file, $windows, $type, &$info)
-----------------------------------------------------------------------

Access:  public static

Parameters:

* $path - A string containing a full file system path.
* $file - A string containing the filename.
* $windows - A boolean indicating whether or not the source OS is Windows.
* $type - A string containing one of "folder" or "file".
* $info - An array containing the result of a `stat()` call on the associated directory/file.

Returns:  A multi-line string to use for the tooltip for an entry being built.

This static function calculates and returns the hover tooltip to use for a Folder-compatible entry.  For non-Windows OSes, the mode, owner, and group are also included.  For files, the size is converted to a human-readable string.

FileExplorerFSHelper::BuildEntry($path, $file, $type, $depth, &$options)
------------------------------------------------------------------------

Access:  public static

Parameters:

* $path - A string containing a full file system path.
* $file - A string containing the filename.
* $type - A string containing one of "folder" or "file".
* $depth - An integer containing the depth of the current path.
* $options - An array of options previously processed by various action handlers.

Returns:  An array containing a Folder-compatible entry on success, a boolean of false otherwise.

This static function builds and returns a Folder-compatible entry.  The function only fails if the stat() call for the path + file fails.

This function also creates thumbnails of image files until 5 seconds of server runtime has passed.  After that point, thumbnail URLs are generated if the option is configured.

FileExplorerFSHelper::ProcessRefreshAction(&$options)
-----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.

Returns:  A standard array of information.

This internal static function processes the refresh action, returning a series of Folder-compatible entries that match the rules in the options array (e.g. allowed file extensions).

FileExplorerFSHelper::ProcessThumnailAction(&$options)
------------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.
* id - A string containing an image filename.

Returns:  Redirects on success, a standard array of information on failure.

This internal static function processes the thumbnail action.  If the image is valid and under ~10MB, a thumbnail is generated for it and the browser is redirected to the generated thumbnail to retrieve and display.

FileExplorerFSHelper::ProcessRenameAction(&$options)
----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.
* id - A string containing a filename to rename.
* newname - A string containing the new name for the file.

Returns:  A standard array of information.

This internal static function processes the rename action, which renames an item and returns a Folder-compatible entry on success.

FileExplorerFSHelper::ProcessNewFolderAction(&$options)
-------------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.
* name - An optional string containing the name of the new folder (Default is "New Folder[ NUM]").

Returns:  A standard array of information.

This internal static function processes the new folder action, which creates a new folder in the specified path.

FileExplorerFSHelper::ProcessNewFileAction(&$options)
-----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.

Returns:  A standard array of information.

This internal static function processes the new file action, which creates a new file in the specified path.  The file is named "New File[ (NUM)].EXT" where EXT is the extension specified by `$options["new_file"]`.

FileExplorerFSHelper::ProcessUploadAction(&$options)
----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.
* name - A string containing extra path + the name for the file being uploaded.
* size - An integer containing the size of the file.
* queuestarted - An integer containing a UNIX timestamp of when the client-side queue started for the current upload session.
* currpath - An optional string containing a JSON encoded array of path IDs that matches the currently viewed folder.

Returns:  A standard array of information.

This internal static function processes both the upload initialization and upload actions.  When an upload starts/completes, the current path is used to determine if an entry should be added to the response.

FileExplorerFSHelper::ProcessDownloadAction(&$options)
------------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* path - A string containing a JSON encoded array of path IDs.
* ids - A string containing a JSON encoded array of directories/filenames to download.

Returns:  Outputs a single file or a ZIP file on success and exits, returns a standard array of information otherwise.

This internal static function processes the download action, which triggers a direct download of a single file or streams a ZIP file containing the requested directories and files.

FileExplorerFSHelper::ProcessCopyInitAction(&$options)
------------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* srcpath - A string containing a JSON encoded array of source path IDs.
* srcids - A string containing a JSON encoded array of directories/filenames in the source path to copy.
* destpath - A string containing a JSON encoded array of destination path IDs.

Returns:  A standard array of information.

This internal static function processes the copy initialization and copy without copykey actions.

Copying files is a complex operation since it can take a while to actually perform the copy.  The operation is split into an initialization stage that calculates how many files will be overwritten during the copy and sets up progress information tracking for later copy calls and the copy stage where files are actually copied.  This is done so that the user interface can be updated regularly with the progress of the operation.

FileExplorerFSHelper::ProcessCopyAction(&$options)
--------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* copykey - A string containing a copykey returned from a copy initialization action.
* currpath - An optional string containing a JSON encoded array of path IDs that matches the currently viewed folder.

Returns:  A standard array of information.

This internal static function processes the copy action, which copies files from the source path to the destination path.  When operating in multiple-request mode, this will run for a few seconds and then return with updated progress information.  When the recycling bin is enabled, files are first moved to the recycling bin before they are overwritten to minimize accidental data loss.

FileExplorerFSHelper::ProcessMoveAction(&$options)
--------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* srcpath - A string containing a JSON encoded array of source path IDs.
* srcids - A string containing a JSON encoded array of directories/filenames in the source path to move.
* destpath - A string containing a JSON encoded array of destination path IDs.

Returns:  A standard array of information.

This internal static function processes the move action, which moves items from one path to another.

FileExplorerFSHelper::ProcessRecycleAction(&$options)
-----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* srcpath - A string containing a JSON encoded array of source path IDs.
* srcids - A string containing a JSON encoded array of directories/filenames in the source path to move.

Returns:  A standard array of information.

This internal static function processes the recycle action, which moves items from the source path to the recycling bin.

FileExplorerFSHelper::ProcessDeleteAction(&$options)
----------------------------------------------------

Access:  protected static

Parameters:

* $options - An array of options processed by `HandleActions()`.

Request Parameters:

* srcpath - A string containing a JSON encoded array of source path IDs.
* srcids - A string containing a JSON encoded array of directories/filenames in the source path to move.

Returns:  A standard array of information.

This internal static function processes the delete action, which permanently deletes items.  The recycling bin is excluded to prevent accidental data loss.

FileExplorerFSHelper::HandleActions($requestvar, $requestprefix, $basedir, $options)
------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $requestvar - A string containing the $_POST or $_GET key to use.
* $requestprefix - A string containing prefix string to look for.
* $basedir - A string containing the absolute path to the base directory to use for all actions.
* $options - An array of options.

Returns:  Outputs a JSON encoded object and exits if an action handles the request, a boolean of false if the request was not handled, or a standard array of information on error.

This function normalizes the input options and attempts to handle the request from the browser.

The $options array accepts these options:

* base_url - An optional string containing a base URL to the files.  Used mostly by the thumbnail calculator to avoid generating a thumbnail if the existing image file is already fairly small.
* protect_depth - An optional integer that protects the base directory + additional directory depth (Default is 0).  Can be useful if the application needs to protect parts of the hierarchy close to the root from being modified.
* recycle_to - An optional string containing the name of the recycling bin folder.
* temp_dir - A string containing a path to a folder for storing temporary files.  Used mostly by the copy_init/copy actions.
* dot_folders - A boolean that indicates whether or not to allow "dot folders" such as .git, .svn, .DS_Store (Default is false).
* allowed_exts - An optional boolean that indicates whether or not to allow/deny all file extensions or a string containing a comma-separated list of allowed file extensions (Default is true).
* allow_empty_ext - An optional boolean that indicates whether or not to allow files with empty extensions (Default is true).
* thumbs_dir - An optional string containing the absolute path to a directory for thumbnails.  Needs to be a web-accessible directory.
* thumbs_url - An optional string containing the base URL of the thumbnail directory referenced by `thumbs_dir`.  Required when using the `thumbs_dir` option.
* thumb_create_url - An optional string containing the URL to the thumbnail creation endpoint.  Recommended when using the `thumbs_dir` option.
* refresh - A boolean that indicates whether or not the refresh handler is enabled.  Pretty much always set to true.
* rename - A boolean that indicates whether or not the rename handler is enabled.
* new_folder - A boolean that indicates whether or not the new folder handler is enabled.
* new_file - A string containing the file extension to use for new files or a boolean of false to disable the new file handler.
* upload - A boolean that indicates whether or not the upload handler is enabled.
* upload_limit - An optional integer that specifies the maximum file size to allow in the upload handler.
* download - A string containing the name of the ZIP file to use for the download if a ZIP file is generated or a boolean of false to disable the download handler.
* download_module - A string containing one of "" (none), "sendfile" (Apache), "accel-redirect" (Nginx) (Default is "").  Used to control the delivery mechanism of a single file download.
* download_module_prefix - A string containing a prefix to apply to the filename.  Used for URI /protected access mapping for a Nginx X-Accel-Redirect to the system root.
* copy - A boolean that indicates whether or not the copy_init and copy handlers are enabled.
* move - A boolean that indicates whether or not the move handler is enabled.
* recycle - A boolean that indicates whether or not the recycle handler is enabled.
* delete - A boolean that indicates whether or not the delete handler is enabled.

FileExplorerFSHelper::FETranslate($format, ...)
-----------------------------------------------

Access:  _internal_ static

Parameters:

* $format - A string containing valid sprintf() format specifiers.

Returns:  A string containing a translation.

This internal static function takes input strings and translates them from English to some other language if CS_TRANSLATE_FUNC is defined to be a valid PHP function name.
