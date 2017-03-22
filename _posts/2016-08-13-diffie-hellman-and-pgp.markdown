---
layout:     post
title:      "Diffie-Hellman and PGP"
subtitle:   "Introduction to modern cryptography"
date:       2016-08-13 19:00:00
author:     "W3ndige"
header-img: "img/diffie-hellman-pgp-header.jpg"
permalink: /:title/
category: Cryptography
---
<h1>Introduction to Diffie-Hellman</h1>

<p>Alice and Bob - two friends from IRC (Internet Relay Chat) just learned about this powerful cipher that no computer can ever crack. As they're concerned about privacy of their conversations they want to implement it for everyday use. But they encountered one problem - they need to come up with secret key and as they are living 13516 km from each other they just can't talk about it in real life. They need to come up with a key through their messages. That's where Diffie-Hellman key exchange comes in. </p>

<p>Diffie-Hellman is a key exchange protocol developed by Ralph Merkle and named after Whitfield Diffie and Martin Hellman - two cryptographers. The purpose of Diffie-Hellman is to allow anybody to exchange a secret over a public channel without having anything shared beforehand, and without the possibility of someone eavesdropping. Let's explore basics behind this concept.</p>


<ul>
<li>Alice comes up with two prime numbers <b>g</b> and <b>p</b> and sends them to Bob.</li>
<li>Bob then picks a secret number <b>a</b>. Then he calculates <b>g<sup>a</sup> mod p</b> and sends that result (which we're going to call "A") back to Alice.</li>
<li>Then Alice does the same thing <b>g<sup>b</sup> mod p</b> but with her own secret number (b) and sends the result (which we're going to call "B") to Bob.</li>
<li>Now, Bob takes the number Alice sent him and does the  <b>B<sup>a</sup> mod p</b>. </li>
<li>Finally Alice does the same operation with the result Bob sent her <b>A<sup>b</sup> mod p</b>.</li>
</ul>

<p>What the trick does is that both final calculations that Alice and Bob do, will end up with same results for both of them. If you don't believe me, let's perform this operations with some example numbers. </p>

<ul>
<li>Alice: <b>g = 5</b> and <b>p = 7</b></li>
<li>Bob: <b>a = 11</b> and operaton <b>A</b>: <b>g<sup>a</sup> mod p</b> --> <b>5<sup>11</sup> mod 7 = 3 </b></li>
<li>Alice: <b>b = 13</b> and operation <b>B</b>: <b>g<sup>b</sup> mod p = 5<sup>13</sup> mod 7 = 5</b> </li>
<li>Bob: <b>B<sup>a</sup> mod p = 5<sup>11</sup> mod 7 = 3</b></li>
<li>Alice: <b>A<sup>b</sup> mod p = 3<sup>13</sup> mod 7 = 3 </b> </li>

</ul>

<p>If you still don't understand check out this paint analogy ;) </p>
![gpa](/img/diffie-hellman-pgp/colors.png){:class="img-responsive"}


<p>That's it! Now both Alice and Bob have the same private key. But you can ask, is it really that secure? Here's when we add the third person - Eve - our malicous spy that wants to get encryption key. </p>

<p>Eve was pretty lucky to intercept key exchange between Alice and Bob. Firstly she gets two prime numbers - <b>p</b> and <b>g</b>, then she is also able to get two other numbers <b>A</b> and <b>B</b>. What she can do with it? Nothing, she has their <b>public</b> keys but she still needs their <b>private</b> keys which are safely in Alice's and Bob's houses, and she can't calculate the secret key.</p>

<p>For better security, numbers used for there operations are very big prime numbers, which simply makes this process impossible to reverse for the eavesdropper.</p>

<p>The beauty of Diffie-Hellman is that after each person does this independently, they will both end up with the exact key which they can later use for any encryption algorithm they want to use for their communication, without sharing it over the wire. Beautiful, right?</p>

<h1>Everyday application</h1>
<p>Modern version of Diffie-Hellman key exchange is used in the PGP (Pretty Good Privacy), with much more complex keys. Let's see how to generate PGP key for yourself with GNU/Linux.</p>

<p>Let's start by opening terminal and installing two tools: <b>GPA</b> and <b>GNUPG2</b>, essential for process of creating PGP keys. You can do this by typing this command in any Debian based distribution.</p>

{% highlight bash %}
sudo apt-get install gpa gnupg2
{% endhighlight %}

<p>After that we are ready to generate our first key.</p>

{% highlight bash %}
gpg --gen-key
{% endhighlight %}

<p>Now let's walk through the process. First thing you are going to be asked is what type of key you want to use. </p>
<p>We will go with RSA and RSA, option (1). RSA is another form of encryption that is also public-key based similar to the concept of Diffie-Hellman’s key exchange.</p>

gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

{% highlight bash %}
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
{% endhighlight %}

<p>After that it will ask you about the length of the key. Obviously, we will choose 4096 bits as nowadays most people use it. </p>

{% highlight bash %}
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
{% endhighlight %}

<p>Next step is to choose how long the key should be vaild. If you want to create key that does not expire enter <i>0</i>, otherwire go with other options.</p>

{% highlight bash %}
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y
{% endhighlight %}

<p>Then enter your credentials such as your name, mail address and comment and your passphrase to protect your key.</p>

{% highlight bash %}
You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: testing
Email address: test@test.test
Comment: test
You selected this USER-ID:
    "testing (test) <test@test.test>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
{% endhighlight %}

<p>Last step before key is done is to generate entrophy essential for generating random key. If you somehow have problems with collecting it, you can make use of <b>rng-tools</b> before generating keys.</p>

{% highlight bash %}
sudo apt-get install rng-tools
sudo rngd -r /dev/urandom
{% endhighlight %}

<p>And we've generated our key!</p>

{% highlight bash %}
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key C6CFC3C4 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   4096R/C6CFC3C4 2016-08-13
      Key fingerprint = C294 18B6 8445 1128 15A3  10F0 580B 973A C6CF C3C4
uid                  testing (test) <test@test.test>
sub   4096R/BE43FEFA 2016-08-13
{% endhighlight %}

<p>But what to do next?</p>
<p>Let's obtain our secret key using GPA tool.</p>
{% highlight bash %}
sudo gpa
{% endhighlight %}

<p>Now you should be able to view all your keys.</p>
![gpa](/img/diffie-hellman-pgp/gpa.png){:class="img-responsive"}
<p>Press the one you have made, click <b>Keys</b> and then <b>Export Keys</b> and then enter the directory where you want your key to be saved. To view it navigate to that directory and view it with any text editor like Gedit.</p>

{% highlight bash %}
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2

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
=0LI3
-----END PGP PUBLIC KEY BLOCK-----
{% endhighlight %}
<p>That's how your key should look like. But what if you want to commicate with somebody. How can you use this tool to encrypt your conversations?</p>

<p>First agree on a shared key, it can be your public key or their public key. Let's pretend it's theirs public key.</p>
<p>Import their public key by pressing <b>Keys</b> and then <b>Import Keys</b></p>
<p>Select <b>Windows</b> and then <b>Clipboard</b>. This window will show up:</p>
![clipboard](/img/diffie-hellman-pgp/clipboard.png){:class="img-responsive"}
<p>Then write the message you want to hide, and press <b>Encrypt the buffer text</b>, which is the blue envelop and choose their public key. You can also sign it with your key for additional safety.  </p>

{% highlight bash %}
-----BEGIN PGP MESSAGE-----
Version: GnuPG v2

hQIMAybVW9O+Q/76AQ/7BBZxTP0ZumqQ8KPyfLWKoiCSPlmgBS77Xr7cc+DQTtUL
etFpWB1BNYhetaeAwx2Afs8VXM9UaZjQdsWTFVZwIwgZ8t6CO1FGJuw+nxQa9eEa
BmVMipOke65PeUAgpg3qST8mOUIebgIb9T5lkRbpmDl4+VphzQBXlF34yce20/Cy
qqnpvC2EXdgg8s+MtJZrl0NGYJxPnwCvtHqvzklhfRX5dYEd50Wuj4Xx5jBC1yxO
20r0eWM9tDgCCFZvgM9bxZlHVl5au8FEa/HQXqbJ0t6u9mLEf/Rswnw5X/NtmfYn
GLwWffD8enUtpIn4ub4TCd8N0zu4SvwELMtJEpYmDjbWFb9DfxzwHs3n9sSRaZ6g
K3DxzOO9Ke4ZlWt+cgtgmg+I4xDUQl4TBJvyTOmQEJiH9q4aj13cXa18TT41AMOd
/XDpPUMYy0Tj3MGGx3cG+bLio4F6Mv+HtHpZzbTA7mvwsnmf94pVwyV6lCiaZO7H
wZpToh29v2l43MQ90Cchoz6SbQKlyyeIouMwUGEH+fMT8XLW5/7zsptfomq/I6kc
8oUO0ryi2fDcPZalvYh4HQPm8H7fOGHvoO2N5fv5NwBnwOxhxmKO92wA/B34HwEP
smyf1Ox+5fnXDC/mhSuWUxyWAtPvZC9Gk31pZO7/xjffjsz8kf0WOvwiVymwEOjS
UAHMYjsAtGoQKPk+ChRSxXP79Ub9O96zM8H9VcJaxdicE9H1t8aws7E22U0Ygwo8
/qoRKeCUVc7rNXV9RiT2aCbDLk+2Z2EnBKf44TYi8DFX
=p14Y
-----END PGP MESSAGE-----

{% endhighlight %}

<p>And we're ready, that's how the message should look like. Now you're ready to send sensitive information without possibility that someone may read this.</p>

<p>I hope you enjoyed this topic, and it gave you some insight in how Diffie-Hellman works, how to create PGP keys, and how to encrypt messages using PGP. Always send sensitive information over PGP as it gives you certainty that no one will ever read this message. As always remember and... </p>
<p>~Stay safe!</p>
