---
layout:     post
title:      "Gynvael Polish Mission 006"
subtitle:   "Write-Ups"
date:       2017-06-29 8:00:00
author:     "W3ndige"
header-img: "img/googlectf-header.jpeg"
permalink: /:title/
category: Write-Ups
---

<h1>Mission</h1>

<p>Another challenge from Gynvael's polish <a href="https://www.youtube.com/watch?v=w-7vLvTKJbI">stream</a>. Let's see what we've got today.</p>

<pre>
Mission 006            goo.gl/te47XT                  DIFFICULTY: ███░░░░░░░ [3/10]
┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅┅
Looking through old documents from the sixties, we've noticed this note:

  c5 c2 c3 c4 c9 c3 40 82 a8 93 82 a8 40 86 81 91 95 a8 40 87 84
  a8 82 a8 40 82 a8 93 40 93 96 87 89 83 a9 95 a8 4b 40 c1 93 85
  40 95 89 85 40 91 85 a2 a3 4b

We're thinking that it's some sentence written in Polish, but we couldn't decode it. Now it's jour job to do it. Good luck!

--

If you decode the answer, put it in the comments under this video! If you write
a blogpost / post your solution online, please add a link in the comments too!

P.S. I'll show/explain the solution on the stream next week.
</pre>

<p>Firstly, let's take a look at different types of encodings used in the sixties. From this <a href="https://en.wikipedia.org/wiki/Character_encoding#History">Wikipedia article</a>, we can learn that: </p>

<pre>
ASCII was introduced in 1963 and is a seven-bit encoding scheme used to encode letters, numerals, symbols, and device control codes as fixed-length codes using integers.
</pre>

<p>But as we know it's not ASCII, we can skip to the next encoding sheme created also in 1963: </p>

<pre>
IBM's Extended Binary Coded Decimal Interchange Code (usually abbreviated as EBCDIC) is an eight-bit encoding scheme developed in 1963.
</pre>

<p>Now we have to check if it's actually this. Looking at this EBCDIC to ASCII table we can see that it will print out polish words, but let's also write a Python script to decode that. </p>

![EBCDIC Table](/img/gynvael-missions/ebcdic-to-ascii.png){:class="img-responsive center-block"}

{% highlight python %}
import codecs

message =  "c5 c2 c3 c4 c9 c3 40 82 a8 93 82 a8 40 86 81 91 95 a8 40 87 84 a8 82 a8 40 82 a8 93 40 93 96 87 89 83 a9 95 a8 4b 40 c1 93 85 40 95 89 85 40 91 85 a2 a3 4b"

chars = message.split(" ")

for letter in chars:
    print(codecs.decode(letter.decode('hex'), "cp500"))
{% endhighlight %}

<p>Where <b>cp500</b> is codec alias for EBCDIC-CP-BE, EBCDIC-CP-CH, IBM500. Running this script we get the message <i>EBCDIC bylby fajny gdyby byl logiczny. Ale nie jest.</i></p>
<p>This translates to <i>EBCDIC would be cool, if it was logical. But it's not.</i></p>
