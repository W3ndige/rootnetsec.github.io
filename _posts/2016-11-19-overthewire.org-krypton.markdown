---
layout:     post
title:      "Overthewire.org - Krypton"
subtitle:   "Write-Up"
date:       2016-11-19 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Overthewire
---

<p>Krypton is another wargame from OverTheWire, that is based on cryptography and cryptoanalysis. As I really love this topic I decided to give it a try, unfortunately level 6 was way harder than I thought and I couldn't complete this, but I'm still happy with the results. </p>

<h1>Krypton 0</h1>

<p>When we first access the Krypton game, we are greeted with this message. </p>
{% highlight text %}
Welcome to Krypton! The first level is easy. The following string encodes the password using Base64:
S1JZUFRPTklTR1JFQVQ=
Use this password to log in to krypton.labs.overthewire.org with username krypton1 using SSH. You can find the files for other levels in /krypton/
{% endhighlight %}

<p>As we know it's encoded using Base64, we can easily solve that in many ways - you can decode this string online, using built in Linux base64 module, or by writing tool in any programming language you want.  </p>

{% highlight bash %}
echo S1JZUFRPTklTR1JFQVQ= | base64 --decode
{% endhighlight %}


<h1>Krypton 1</h1>

<p>Firstly, let's find the files to work on. </p>

{% highlight bash %}
krypton1@melinda:~$ cd /krypton
krypton1@melinda:/krypton$ ls  
krypton1  krypton2  krypton3  krypton4  krypton5  krypton6
krypton1@melinda:/krypton$ cd krypton1
krypton1@melinda:/krypton/krypton1$ ls
README  krypton2
{% endhighlight %}

<p>From this one, we know that the string is encrypted using simple Rot13 encryption, meaning that each letter in the plaintext has been moved 13 times in the alphabet to produce the ciphertext. There are also many ways to decrypt this, but I'll show you this simple script that works really well. </p>

{% highlight bash %}
krypton1@melinda:/krypton/krypton1$ cat krypton2 | tr 'A-Za-z' 'N-ZA-Mn-za-m'
LEVEL TWO PASSWORD ******
{% endhighlight %}

<p>Got it! Let's move to the next one. </p>

<h1>Krypton 2</h1>

<p>As in the previous one, let's firstly move to the krypton directory. </p>
{% highlight bash %}
krypton2@melinda:/krypton/krypton2$ ls
README  encrypt  keyfile.dat  krypton3
{% endhighlight %}

{% highlight text %}
krypton2@melinda:/krypton/krypton2$ cat README
Krypton 2

ROT13 is a simple substitution cipher.

Substitution ciphers are a simple replacement algorithm.  In this example
of a substitution cipher, we will explore a 'monoalphebetic' cipher.
Monoalphebetic means, literally, "one alphabet" and you will see why.

This level contains an old form of cipher called a 'Caesar Cipher'.
A Caesar cipher shifts the alphabet by a set number.  For example:

plain:	a b c d e f g h i j k ...
cipher:	G H I J K L M N O P Q ...

In this example, the letter 'a' in plaintext is replaced by a 'G' in the
ciphertext so, for example, the plaintext 'bad' becomes 'HGJ' in ciphertext.

The password for level 3 is in the file krypton3.  It is in 5 letter
group ciphertext.  It is encrypted with a Caesar Cipher.  Without any
further information, this cipher text may be difficult to break.  You do
not have direct access to the key, however you do have access to a program
that will encrypt anything you wish to give it using the key.  
If you think logically, this is completely easy.

One shot can solve it!

Have fun.

Additional Information:

The `encrypt` binary will look for the keyfile in your current working
directory. Therefore, it might be best to create a working direcory in /tmp
and in there a link to the keyfile. As the `encrypt` binary runs setuid
`krypton3`, you also need to give `krypton3` access to your working directory.

Here is an example:

krypton2@melinda:~$ mktemp -d
/tmp/tmp.Wf2OnCpCDQ
krypton2@melinda:~$ cd /tmp/tmp.Wf2OnCpCDQ
krypton2@melinda:/tmp/tmp.Wf2OnCpCDQ$ ln -s /krypton/krypton2/keyfile.dat
krypton2@melinda:/tmp/tmp.Wf2OnCpCDQ$ ls
keyfile.dat
krypton2@melinda:/tmp/tmp.Wf2OnCpCDQ$ chmod 777 .
krypton2@melinda:/tmp/tmp.Wf2OnCpCDQ$ /krypton/krypton2/encrypt /etc/issue
krypton2@melinda:/tmp/tmp.Wf2OnCpCDQ$ ls
ciphertext  keyfile.dat
{% endhighlight %}

<p>This time instructions are a little bit more complicated, but whole task seems to be quite easy. Since I've previously made Python script to brute force the ciphertext, let's test it out in this situation. You can take a look at my script, but I strongly encourage you to try it by yourself! </p>

<a href="https://github.com/W3ndige/encryption-tools/blob/master/caesar-cipher.py">Caesar-cipher.py</a>

<p>And it worked, we've got the password to the next one: ************ !</p>

<h1>Krypton 3</h1>

<p>Wow, this seems harder than the previous ones, but that's the whole point of the challenge, right? </p>

{% highlight text %}
krypton3@melinda:~$ cd /krypton/krypton3
krypton3@melinda:/krypton/krypton3$ ls
HINT1  HINT2  README  found1  found2  found3  krypton4
krypton3@melinda:/krypton/krypton3$ cat README
Well done.  You've moved past an easy substitution cipher.

Hopefully you just encrypted the alphabet a plaintext
to fully expose the key in one swoop.

The main weakness of a simple substitution cipher is
repeated use of a simple key.  In the previous exercise
you were able to introduce arbitrary plaintext to expose
the key.  In this example, the cipher mechanism is not
available to you, the attacker.

However, you have been lucky.  You have intercepted more
than one message.  The password to the next level is found
in the file 'krypton4'.  You have also found 3 other files.
(found1, found2, found3)

You know the following important details:

- The message plaintexts are in English (*** very important)
- They were produced from the same key (*** even better!)


Enjoy.
krypton3@melinda:/krypton/krypton3$ cat HINT1
Some letters are more prevalent in English than others.
krypton3@melinda:/krypton/krypton3$ cat HINT2
"Frequency Analysis" is your friend.
{% endhighlight %}

<p>I'm going to use tools from this <a href="http://www.richkni.co.uk/php/crypta/index.php">website</a> as they really make process of analyzing the ciphertext faster. As all messages were encrypted with the same key, I have copied them into the tools to have bigger material, and to get better and more accurate samples. What will come really helpful, is the frequency of letters in English. </p>

![letter-frequency](/img/overthewire-krypton/letter-frequency.jpg){:class="img-responsive"}

<p>Now let's slowly work out our way to the solution. Since I know that the most frequent letter that occurs is 's', I can assume that in plaintext it will be the letter 'e'. In addition 3 letter sequence - 'jds' can be matched as 'the'. </p>
<p>After hardly an hour of work I finally managed to find out all of the letter. Here's the encryption key: </p>
{% highlight text %}
Plaintext:   a   b   c   d   e   f   g   h   i   j   k   l   m
Ciphertext:  b   o   i   h   g   k   n   q   v   t   w   y   u
Plaintext:   n   o   p   q   r   s   t   u   v   w   x   y   z
Ciphertext:  r   x   z   a   j   e   m   s   l   d   f   p   c
{% endhighlight %}
<p>Now the last step is to change the letters from the ciphertext: KSVVW BGSJD SVSIS VXBMN YQUUK BNWCU ANMJS. </p>

<p>Got it! Let's move to the next one ;)</p>

<h1>Krypton 4</h1>
<p>This time we will work on Viegenere Cipher. </p>

{% highlight text %}
Good job!

You more than likely used frequency analysis and some common sense
to solve that one.

So far we have worked with simple substitution ciphers.  They have
also been 'monoalphabetic', meaning using a fixed key, and
giving a one to one mapping of plaintext (P) to ciphertext (C).
Another type of substitution cipher is referred to as 'polyalphabetic',
where one character of P may map to many, or all, possible ciphertext
characters.

An example of a polyalphabetic cipher is called a Vigen�re Cipher.  It works
like this:

If we use the key(K)  'GOLD', and P = PROCEED MEETING AS AGREED, then "add"
P to K, we get C.  When adding, if we exceed 25, then we roll to 0 (modulo 26).


P     P R O C E   E D M E E   T I N G A   S A G R E   E D
K     G O L D G   O L D G O   L D G O L   D G O L D   G O

becomes:

P     15 17 14 2  4  4  3 12  4 4  19  8 13 6  0  18 0  6 17 4 4   3
K     6  14 11 3  6 14 11  3  6 14 11  3  6 14 11  3 6 14 11 3 6  14
C     21 5  25 5 10 18 14 15 10 18  4 11 19 20 11 21 6 20  2 8 10 17

So, we get a ciphertext of:

VFZFK SOPKS ELTUL VGUCH KR

This level is a Vigen�re Cipher.  You have intercepted two longer, english
language messages.  You also have a key piece of information.  You know the
key length!

For this exercise, the key length is 6.  The password to level five is in the usual
place, encrypted with the 6 letter key.

Have fun!
{% endhighlight %}

<p>You can read more about this polyalphabetic cipher in my previous post <a href="https://www.rootnetsec.com/2016/08/09/encryption-vigenere-cipher/">Let's Talk Encryption</a> dedicated to Vigenère cipher. </p>

<p>But to finish this challenge, we can use online tools, that will automate process of analyzing frequency which as you know, or may suspect, is very time consuming. I will use this <a href="http://www.mygeocachingprofile.com/codebreaker.vigenerecipher.aspx"> website</a>.</p>

<p>Firstly I will analyze intercepted messages to try and find the matching key. Great, key from that message seems to be <b>frekey</b>. Now, let's see decrypt our ciphertext HCIKV RJOX resulting in the keyphrase to the next level! </p>

<h1>Krypton 5</h1>

<p>So once again we have to try frequency analysis. </p>

{% highlight text %}
Frequency analysis can break a known key length as well.  Lets try one
last polyalphabetic cipher, but this time the key length is unknown.

Enjoy.
{% endhighlight %}

<p>As in the previous one, I automate the process of analysing, with result of <b>keylength</b> as the suspect key. Let's try whether or not it works. BELOSZ decrypted using key KEYLENGTH results in the password to the next one! </p>

<hr>

<p>It was great fun to complete these challenges, Krypton 6 was unfortunately to hard to complete, but I'll definitely come back to it after gaining some more knowledge about cryptography. </p>
