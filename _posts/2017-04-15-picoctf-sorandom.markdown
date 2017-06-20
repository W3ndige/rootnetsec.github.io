---
layout:     post
title:      "PicoCTF - SoRandom"
subtitle:   "Write-Ups"
date:       2017-04-15 0:00:00
author:     "W3ndige"
header-img: "img/picoctf-header.jpg"
permalink: /:title/
category: PicoCTF 2017
---
<h1>SoRandom - 75 PTS</h1>

<p>We found sorandom.py running at shell2017.picoctf.com:42263. It seems to be outputting the flag but randomizing all the characters first. Is there anyway to get back the original flag?</p>

{% highlight python %}
#!/usr/bin/python -u
import random,string

flag = "FLAG:"+open("flag", "r").read()[:-1]
encflag = ""
random.seed("random")
for c in flag:
  if c.islower():
    #rotate number around alphabet a random amount
    encflag += chr((ord(c)-ord('a')+random.randrange(0,26))%26 + ord('a'))
  elif c.isupper():
    encflag += chr((ord(c)-ord('A')+random.randrange(0,26))%26 + ord('A'))
  elif c.isdigit():
    encflag += chr((ord(c)-ord('0')+random.randrange(0,10))%10 + ord('0'))
  else:
    encflag += c
print "Unguessably Randomized Flag: "+encflag
{% endhighlight %}

<p>How this code works? Firstly it opens the flag file, reading its content, then it creates a loop which will iterate every letter in the flag string. Then, these 3 'if' statements check, whether or not the letter is lowercase, uppercase or digit, and according to that, it will append some random number to the ASCII value of a letter. Those random values were firstly seeded with a work <b>random</b>, meaning that everytime the code starts running, it will generate the same output. </p>

<p>We can simply crack it, by reversing the scheme and simply substracting those values. By firstly, let's connect to the server to get the encrypted flag. </p>

{% highlight bash %}
w3ndige@W3ndige ~> nc shell2017.picoctf.com 42263
Unguessably Randomized Flag: BNZQ:8o149b15764q471k2533971t6w78liec
{% endhighlight %}

<p>Now we can change the code. </p>

{% highlight python %}
#!/usr/bin/python -u
import random,string

flag = "BNZQ:8o149b15764q471k2533971t6w78liec"
encflag = ""
random.seed("random")
for c in flag:
  if c.islower():
    #rotate number around alphabet a random amount
    encflag += chr((ord(c)-ord('a')-random.randrange(0,26))%26 + ord('a'))
  elif c.isupper():
    encflag += chr((ord(c)-ord('A')-random.randrange(0,26))%26 + ord('A'))
  elif c.isdigit():
    encflag += chr((ord(c)-ord('0')-random.randrange(0,10))%10 + ord('0'))
  else:
    encflag += c
print "Unguessably Randomized Flag: "+encflag
{% endhighlight %}

<p>And lastly, let's run this code once again on our local computer. Remember to use <b>python2</b>!</p>

{% highlight bash %}
w3ndige@W3ndige ~/Pobrane> python2 sorandom.py
Unguessably Randomized Flag: FLAG:5d968c92267e701f5846515f1b31deab
{% endhighlight %}

<p>Here we go. </p>
