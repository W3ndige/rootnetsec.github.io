---
layout:     post
title:      "PicoCTF - Connect The Wiggle"
subtitle:   "Write-Ups"
date:       2017-04-15 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---

<p>Identify the data contained within wigle and determine how to visualize it.</p>

<p>This time we get another sqlite file called <b>wigle</b>.  </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> file wigle
wigle: SQLite 3.x database, last written using SQLite version 3008007
{% endhighlight %}

<p>Now we can open it up, and try to determine, which of the tables in a database should be our target. </p>

{% highlight sql %}
sqlite> SELECT * FROM sqlite_master WHERE type='table';
table|android_metadata|android_metadata|2|CREATE TABLE android_metadata (locale text)
table|location|location|3|CREATE TABLE location (_id int, bssid text, level int, lat double, lon double, altitude double, accuracy float, time long)
table|network|network|4|CREATE TABLE network (bssid text, ssid text, frequency int, capabilities text, lasttime long, lastlat double, lastlon double, type text)
{% endhighlight %}

<p>Great, location table, with some longitude, and latitude columns. Let's view them. </p>

{% highlight sql %}
sqlite> select lat, lon from location;
-30.0|94.04
-29.99|94.04
-29.98|94.04
-29.98|94.045
-29.98|94.05
-29.97|94.04
-29.96|94.04
-29.96|94.05
-29.96|94.06
-30.0|94.14
-29.99|94.14
.
.
.
{% endhighlight %}

<p>Mysterious list of long coordinates. We can try to plot them using <a href="https://www.darrinward.com/lat-long/">Darrinward</a> website. But firstly, we will have to change all the <b>|</b> characters into the <b>,</b>. Then we're ready to plot them on a map.  </p>

![Plot](/img/picoctf/darrinward.png){:class="img-responsive center-block"}

<p>After taking a closer look, we can see that the flag is FLAG{F0UND_M3_33DE7930}. </p>
