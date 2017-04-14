---
layout:     post
title:      "PicoCTF - Leaked Hashes"
subtitle:   "Write-Ups"
date:       2017-04-14 0:00:00
author:     "W3ndige"
header-img: "img/picoctf-header.jpg"
permalink: /:title/
category: PicoCTF 2017
---
<h1>Leaked Hashes - 90 PTS</h1>

<p>Someone got hacked! Check out some service's password hashes that were leaked at hashdump.txt! Do you think they chose strong passwords? We should check... The service is running at shell2017.picoctf.com:61096!</p>

{% highlight text %}
root:be3f7de032d2e398ec542a7df71e0417
christene:cc5847ba595f097885b2bbe5eb940e22
nadia:f3ab9dc32a394eb1cdf922659abcdead
myra:4dff179a4ec49e978ce72e6bf39b7d86
sharell:05635c2378051c67022b9096e0813aeb
lilli:05635c2378051c67022b9096e0813aeb
ilene:b51bc648f9d5e58971705b5efdc098a5
emiko:ee4d5bc25a2771d95fdbd24452e355ad
lindsey:c9f01a6d41d78f79585a30fdc815bf17
deandrea:c9dd062917d9feb763b10fbcb483abfa
honey:74fe6370a9626ef1887c0e3ca4e6b600
diana:9ff497405dbd1cf3c17576629cf35475
lucretia:ac2f556a0eb415745b31e14a91d55d75
.
.
.
{% endhighlight %}

<p>Now we will have to try and crack at least one of these passwords provided in the dump, so we can log into the service. My favourite <a href="https://hashkiller.co.uk/md5-decrypter.aspx">Hashkiller</a> website used to cracking MD5 hashes did it without any problem. </p>

{% highlight text %}
be3f7de032d2e398ec542a7df71e0417 [Not found]
cc5847ba595f097885b2bbe5eb940e22 [Not found]
f3ab9dc32a394eb1cdf922659abcdead MD5 : 0u70u73r
4dff179a4ec49e978ce72e6bf39b7d86 MD5 : s3r0w
05635c2378051c67022b9096e0813aeb MD5 : r3v3l
b51bc648f9d5e58971705b5efdc098a5 MD5 : p0lyp
ee4d5bc25a2771d95fdbd24452e355ad MD5 : r3s4w
c9f01a6d41d78f79585a30fdc815bf17 MD5 : sp337
c9dd062917d9feb763b10fbcb483abfa [Not found]
{% endhighlight %}

<p>Now we can log in using <i>nadia:0u70u73r</i> credentials. </p>

{% highlight bash %}
w3ndige@W3ndige ~> nc shell2017.picoctf.com 61096
enter username:
nadia
nadia's password:0u70u73r
welcome to shady file server. would you like to access the cat ascii art database? y/n
y

     /\__/\
    /`    '\
  === 0  0 ===
    \  --  /    - flag is 1ae2d736242e28658d08e8b47dbf49bf

   /        \
  /          \
 |            |
  \  ||  ||  /
   \_oo__oo_/#######o

              ("`-''-/").___..--''"`-._
               `6_ 6  )   `-.  (     ).`-.__.`)
               (_Y_.)'  ._   )  `._ `. ``-..-'
             _..`--'_..-_/  /--'_.' ,'
           (il),-''  (li),'  ((!.-'

from http://user.xmission.com/~emailbox/ascii_cats.htm

{% endhighlight %}

<p>Here we have the flag! </p>

<p>~ Stay safe!</p>
