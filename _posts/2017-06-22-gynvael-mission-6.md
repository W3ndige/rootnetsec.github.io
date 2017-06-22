---
layout:     post
title:      "Gynvael Mission 006"
subtitle:   "Write-Ups"
date:       2017-06-22 8:00:00
author:     "W3ndige"
header-img: "img/googlectf-header.jpeg"
permalink: /:title/
category: Write-Ups
---

<h1>Mission</h1>

<p>Challenge from Gynvael's <a href="https://www.youtube.com/watch?v=KvyBn4Btv8E">stream</a>. Let's take a look at it. </p>
<pre>
Welcome back agent 1336.
No mail.
> mission --take
MISSION 006               goo.gl/Z4u9cv             DIFFICULTY: ███████░░░ [7╱10]
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅

Finally we hired a new secret agent and moved agent Huffman to do some harmless
paperwork. That's the good news. The bad news is that agent Huffman made it his
mission to train our new agent and this went exactly as expected - i.e. we cannot
decode the report received from the new agent.

Long story short, agent Huffman Junior sent us this file:

  http://goo.gl/S395x6

We don't really know how to decode it, nor do we care at this point. Please take
a look and decode it if you can.

GOOD LUCK!

If you decode the answer, put it in the comments under this video! If you write
a blogpost / post your solution online, please add a link in the comments too!

P.S. I'll show/explain the solution on the stream next week.
</pre>

<p>After viewing the file, I can see something looking like coordinates. </p>

<pre>
[11, 24]
[6, 6]
[18, 22]
[12, 10]
[24, 13]
[17, 14]
[12, 8]
</pre>

<p>And after lots of math in this year, they quite remind me of vectors. By looking at the data, we can also assume that the biggest number is 24, so image should not be bigger than that. Now I decided to run another test, to see if all of the entries are unique and it's possible using Linux command line. </p>

{% highlight bash %}
[w3ndige@w3ndige Programming]$ cat mission006.data | uniq > test
[w3ndige@w3ndige Programming]$ diff test mission006.data
{% endhighlight %}

<p>Firstly we pipe the file to the uniq tool that will print only unique entres, and save it to test file. After that we're going to run diff tool to see if there are any differences. And there are none, which means that the file is made of only unique entres. </p>

<p>Now, if all my understaning is correct, we can start plotting them on a bitmap using Python. To make script a little easier I converted it to new format using Atom's find and replace :D </p>

<pre>
11,24
6,6
18,22
</pre>

<p>Now we're ready to write a script! </p>

{% highlight python %}
from PIL import Image, ImageDraw

file = open("mission006.data", "r")

coords = []

for line in file:
    line = line.split(',')
    coords.append((int(line[0]),int(line[1])))

img = Image.new("RGB", (25,25), "white")
draw = ImageDraw.Draw(img)

for (x,y) in coords:
    draw.rectangle([x,y,x,y], fill="black")

img.save('flag.png')
{% endhighlight %}

<p>And our flag image looks like QR Code now. Let's open it with Gimp to make it a little bigger. Wait, is it vertically flipped? </p>

![QR Code](/img/gynvael-006/qrcode.png){:class="img-responsive center-block"}

<p>Now it's much better, we can finally scan it. </p>

<p><b>Mirrored QR? Seriously?!</b></p>

<p>Great challenge, I would like to thank Gynvael for his amazing streams and this fun task! </p>
