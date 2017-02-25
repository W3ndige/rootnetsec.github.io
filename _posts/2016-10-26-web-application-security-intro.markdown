---
layout:     post
title:      "Web Application Security"
subtitle:   "Security Headers"
date:       2016-10-26 0:00:00
author:     "W3ndige"
header-img: "img/web-application-security-intro-header.jpg"
category: [Web Security]
---
<h1>Introduction</h1>

<p>Today we're going to start with Web Security - in particular I want to discuss security headers - small steps to keep your web application more secure. Without wasting your time, let's move to the content.  </p>

<h1>Specify the doctype</h1>

<p>But why is it so important and how can it improve security of my web application? </p>
<p>Answer to this is quite easy, without the specified doctype web browser will try to guess the type of the document based on the content of the website. Sometimes it may guess good, but sometimes, in the worst case, browser can generate our document in so called <a href="https://en.wikipedia.org/wiki/Quirks_mode">Quirks Mode</a>. Goal of quirks mode is to preserve the backwards compatibility with times, where web was just starting. As you may think, this may lead to data leaks, or even make the site vulnerable to XSS (even if it was coded properly). </p>

<p>Also remember that you don't have to write older versions of doctype like in HTML4:  </p>
{% highlight html %}
 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
{% endhighlight %}

<p>In HTML5 you only have to write: </p>
{% highlight html %}
 <!DOCTYPE HTML>
{% endhighlight %}

<p>Easy, right? </p>
<h1>Subresource Integrity</h1>

<p>Lots of web applications are based on the external libraries or frameworks like AngularJS, jQuery, BootStrap etc and loads of web developers instead of downloading them into the server, often link them from CDN's (Content Delivery Network) of creators, from where the script is downloaded and executed during the process of rendering webpage. Here's the example of linking AngularJS framework: </p>

{% highlight html %}
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js"></script>
{% endhighlight %}

<p>But what if someone attacks the ajax.googleapis.com and changes the script into more malicious one, for example putting browser exploit? Then our vistiors are also attacked, and it's our goal to make it impossible.  </p>

<p>To prevent these situtations, Subresource Integrity allows us to add additional attribute to the <b>script</b> or <b>link</b> tags, where we should enter encoded in Base64 hash (SHA-256, SHA-384, SHA-512) for the external file. Then when the hash doesn't match the hash of the downloaded script, browser won't run it. In our example SRI script will look like this:  </p>

{% highlight html %}
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js" integrity="sha384-V6/dyDFv85/V/Ktq3ez5B80/c9ZY7jV9c/319rqwNOz3h9CIPdd2Eve0UQBYMMr/"></script>>
{% endhighlight %}

<p>Try this site to generate SRI hash: <a href="https://www.srihash.org/">SRI Hash Generator</a></p>

<h1>X-XSS-Protection</h1>

<p>Google Chrome and Internet Explorer have built-in filter preventing Reflected XSS attacks. They work quite easily, checking, if there are HTML tags in the HTTP query, or even different fragments of JavaScript code, and then if this code also shows up in the answer. If yes, then the browser assumes that this is XSS attack, and blocks this piece of code.   </p>

<p>Both in IE and Chrome, you can enable the filters using X-XSS-Protection header. </p>

<ul>
<li>X-XSS-Protection: 0 - XSS filter is disabled and the browser won't try to detect XSS attacks</li>
<li>X-XSS-Protection: 1 - XSS filter is enabled - and this is the default option. It blocks the piece of code suspected for being XSS payload. </li>
<li>X-XSS-Protection: 1; mode=block - XSS filter, which after detecting XSS attack, will stop rendering the web page. </li>
</ul>

<p>The best option to choose would be the last one, since it doesn't only blocks XSS attacks, it also lets us dodge vulnerabilities described in <a href="http://www.slideshare.net/masatokinugawa/xxn-en">X-XSS-Nightmare</a> written by Msato Kinugawy. </p>


<h1>X-Content-Type-Options</h1>
<p>This header disables the guessing  MIME type of the site, by the browser. It also protects us from uploading files different than they should be (JavaScript file as a photo etc). Example: </p>

<p>We have the web application allowing users to upload images. Attacker after uploading malicious .png file, tries to change the source of the script into his malicious script. </p>

{% highlight html %}
<script src="http://www.example.com/uploads/malicious.png”></script>
{% endhighlight %}

<p>But the X-Content-Type-Options won't allow this operation, since image (MIME type image/png) doesn't match the script type. X-Content-Type-Options has only one possible value: <b>nosniff</b></p>

{% highlight html %}
X-Content-Type-Options: nosniff
{% endhighlight %}

<h1>X-Frame-Options</h1>

<p>X-Frame-Options header was introduced as an response to clickjacking attacks, but is also used to protect from many different kinds of attacks. This header reduces the number of domens, where our web application can be put into <b>frame</b>, <b>iframe</b> or <b>object</b> tags. </p>

<ul>
<li>X-Frame-Options: Deny - web page connot be put into frames on any different sites.</li>
<li>X-Frame-Options: SameOrigin - web page can only be put into frames on the the webpages originating from the same domain. </li>
<li>X-Frame-Options: Allow-From url - web page can be put only on specified domain. </li>
</ul>

<p>Best option would be SameOrigin or Deny, unless you want to share any widgets. </p>

<h1>HttpOnly/Secure Flags</h1>

<p>For HTTP cookies we can add 2 flags making them more secure - HttpOnly and Secure. </p>

<p>HttpOnly flag lets us avoid one of the most popular result of XSS attacks - stealing session cookies. Attackers payload can be done using JavaScript <b>document.cookie</b> and sending it to his server, but with HttpOnly cookie will be only sent in queries to server, it won't show up using <b>document.cookie</b></p>

<p>Secure flag guarantees us that, when application works both in HTTP and HTTPS, it will only send cookies using HTTPS. </p>

<p>Here's and example of Set-Cookie, with both flags: </p>

{% highlight html %}
Set-Cookie: PHPSESSID=el4ukv0kqbvoirg7nkp4dncpk3; HttpOnly; Secure
{% endhighlight %}

<p>Setting up flags for session cookies depends on used technology, but specific instructions should be presented in documentation. </p>

<h1>Strict-Transport-Security</h1>

<p>Last but not least Strict-Transport-Security header comes handy, when we're building web application, that should never be runned with HTTP. After defining this header, we gain 2 mechanisms protecting us from man in the middle attacks. </p>

<p>We are sure that web browser will never send queries using HTTP protocol.</p>

<p>When negotiation SSL connection results in an error (ex invalid certificate), browser won't let user to accept such certificate. </p>

<p>While implementing Strict-Transport-Security header, it's essential to enter <b>max-age</b> parameter, saying how long should the header be significant (in seconds). After that time there will be possibility to once again perform queries using HTTP protocol. </p>

<p>Example of such header valid for one year: </p>

{% highlight html %}
Strict-Transport-Security: max-age=31536000
{% endhighlight %}


<hr>
<p>I hope that after this read, you gained better understanding of secure headers. In the future I'll write about Content-Security-Policy, but it is material for a whole new post, so keep tuned ;)</p>

<p>Thanks for reading and as always...</p>

<p>~Stay safe! </p>
