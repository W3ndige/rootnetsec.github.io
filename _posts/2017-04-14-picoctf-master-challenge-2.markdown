---
layout:     post
title:      "PicoCTF - Master Challenge 2"
subtitle:   "Write-Ups"
date:       2017-04-14 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---

<p>Turns out, some of the files back from Master Challenge 1 were corrupted. Restore this one file and find the flag. </p>

<p>We can try to scan the file for any pieces of strings, containing flag or clues for further approach. </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> strings file
.
.
.
fHS
>Pzd
J8Da
W+P.
8FQtf
Y)e^
IEND
flag.pngPK
nottheflag1.pngPK
J$L %
nottheflag2.pngPK
nottheflag3.pngPK
nottheflag4.pngPK
nottheflag5.pngPK
nottheflag6.pngPK
nottheflag7.pngPK

{% endhighlight %}

<p>At the end of the provided output we can see 8 file names with additional <b>PK</b> letters appended to it. Quick google search shows us that PK are the initals of Phil Katz, co-creator of the ZIP file format and author of PKZIP. You can read more <a href="http://www.garykessler.net/library/file_sigs.html">here</a>.</p>

<p>As it's not that easy to just change the file name and extract from the archive, we will have to take a closer look at the structure of the file. I'm going to use <b>Bliss</b> hexeditor for Linux and <b>hexdump</b> for navigating the hexadecimal values. </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> hexdump -C file > texthexdump.txt
{% endhighlight %}

<p>Now we can view the beginning (head) of the texthexdump. </p>

{% highlight text %}
w3ndige@W3ndige ~/Pobrane> head texthexdump.txt
00000000  58 58 58 58 58 58 00 00  08 00 22 44 7f 4a b4 8b  |XXXXXX...."D.J..|
00000010  e4 67 2b 90 00 00 1c 90  00 00 08 00 00 00 66 6c  |.g+...........fl|
00000020  61 67 2e 70 6e 67 00 66  40 99 bf 89 50 4e 47 0d  |ag.png.f@...PNG.|
00000030  0a 1a 0a 00 00 00 0d 49  48 44 52 00 00 02 81 00  |.......IHDR.....|
00000040  00 00 3c 08 02 00 00 00  96 fa f7 6d 00 00 8f e3  |..<........m....|
00000050  49 44 41 54 78 9c ec fd  59 af 6c d9 96 1e 86 8d  |IDATx...Y.l.....|
00000060  31 9b d5 45 1f bb 3f 6d  b6 37 33 6f de ae ea 96  |1..E..?m.73o....|
00000070  cc 2a 56 d9 34 8b b2 20  1b 86 4c 91 80 21 d8 16  |.*V.4.. ..L..!..|
00000080  0c 43 06 fc e6 27 fd 0c  c2 80 fc e2 27 fa c1 10  |.C...'......'...|
00000090  6c 58 76 d1 86 21 c0 b2  45 53 02 2d 36 2e 56 cb  |lXv..!..ES.-6.V.|
{% endhighlight %}

<p>We can clearly see the cause of the problem. Every file starts with the file signature, but this one has been overwritten with <b>X</b> characters. We can even view this error while trying to unzip this file. </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> unzip file
Archive:  file
file #1:  bad zipfile offset (local header sig):  0
  inflating: nottheflag1.png         
  inflating: nottheflag2.png         
  inflating: nottheflag3.png         
  inflating: nottheflag4.png         
  inflating: nottheflag5.png         
  inflating: nottheflag6.png         
  inflating: nottheflag7.png
{% endhighlight %}

<p>Let's repair this with correct signature, which is <b>50 4B 03 04 00 00</b>.</p>

![Editing in bless](/img/picoctf/master-2-bless.png){:class="img-responsive center-block"}

<p>We have everything for the last step - <b>zip -F</b> which will try to somehow repair the file.  </p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> zip -F file.zip --out new.zip
Fix archive (-F) - assume mostly intact archive
Zip entry offsets do not need adjusting
 copying: flag.png
	zip warning: Local Version Needed To Extract does not match CD: flag.png
 copying: nottheflag1.png
 copying: nottheflag2.png
 copying: nottheflag3.png
 copying: nottheflag4.png
 copying: nottheflag5.png
 copying: nottheflag6.png
 copying: nottheflag7.png
{% endhighlight %}

<p>And here's our flag. </p>

![Flag](/img/picoctf/master-2-flag.png){:class="img-responsive center-block"}

<p>Now let's write this down and submit the flag. </p>
