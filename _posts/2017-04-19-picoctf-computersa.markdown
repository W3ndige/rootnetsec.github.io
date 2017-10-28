---
layout:     post
title:      "PicoCTF - ComputeRSA"
subtitle:   "Write-Ups"
date:       2017-04-19 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---
<h1>ComputeRSA - 50 PTS</h1>

<p>RSA encryption/decryption is based on a formula that anyone can find and use, as long as they know the values to plug in. Given the encrypted number 150815, d = 1941, and N = 435979, what is the decrypted number?</p>

<p>In order to decrypt the message we have to take a look at this equation, as shown in <a href="https://en.wikipedia.org/wiki/RSA_(cryptosystem)">Wikipedia</a> entry. </p>

<p><b>m = c^d % n</b></p>

<p>So it's just a matter of computing this <b>m</b> number, as everything is provided in the challenge. </p>

{% highlight bash %}
w3ndige@W3ndige ~> python
Python 3.6.0 (default, Jan 16 2017, 12:12:55)
[GCC 6.3.1 20170109] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> c = 150815
>>> d = 1941
>>> n = 435979
>>> pow(c, d, n)
133337
{% endhighlight %}

<p>Easy and fun, right?</p>
