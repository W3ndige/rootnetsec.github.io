---
layout:     post
title:      "PicoCTF - JSut Duck It Up"
subtitle:   "Write-Ups"
date:       2017-04-15 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---
<h1>JSut Duck It Up - 100 PTS</h1>

<p>What in the world could this be?!?!</p>

{% highlight text %}
[]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(+(!+[]+!+[]+[+!+[]]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+
.
.
.
{% endhighlight %}

<p>Wow, that's an really strange and huge file. Title of the challenge suggest that it may be connected with JavaScript, so let's do an quick google. After a little bit of fiddling, I've found this: <a href="http://www.jsfuck.com/">JSFuck</a>. It is an esoteric subset of the JavaScript language that uses only six distinct characters in the source code. The characters are +, !, (, ), [, ]. </p>

<p>Pasting this code into the text field, and running it produces an alert. </p>

![JSFuck alert](/img/picoctf/jsfuck.png){:class="img-responsive center-block"}

<p>Easy 100pts, right? Great challenge! </p>
