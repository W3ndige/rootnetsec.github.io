---
layout:     post
title:      "Let's Talk Encryption"
subtitle:   "Part 2 - Vigenère cipher"
date:       2016-08-09 12:00:00
author:     "W3ndige"
header-img: "img/encryption-vigenere-header.jpg"
permalink: /:title/
category: Cryptography
---
<h1>Vigenère cipher - how it works?</h1>
<p>Vigenère cipher is an method of encrypting message by using series of different Caesar ciphers based on the letters of a keyword. Basically to encrypt a message using the Vigenère cipher you first need to choose a keyword. After that repeat this keyword over and over until it is the same length as your secret message. </p>
<p>Now for each plaintext letter, you find the letter on the left-vertical row of the tabula recta. Then you take the corresponding letter from your keyword, and find it at the top-horizontal row of the table. Where these two lines cross in the table is the ciphertext letter you use.</p>
![tabula-recta](/img/encryption-vigenere/tabula-recta.png){:class="img-responsive"}
<p><center><i>Tabula Recta</i></center></p>
<p>Let's encrypt message - <b>hello</b> with a keyword <b>boom</b>.</p>
<ul>
<li>Since our message is 5 letters and keyword is only 4 we have to repeat it until it's same length as message - in this example keyword will be <b>boomb</b>.</li>
<li>Now we have to take the first letter from the message and find it on the left-vertical row of the table.</li>
<li>Next get the first letter of the keyword and also get its position, now on the upper-horizontal row.</li>
<li>After that find where lines, in which were those letters cross and write down the letter you get.</li>
<li>Repeat for each letter.</li>
</ul>
<p>When you finish you should get the encrypted message: <b>iszxp</b> </p>
<p>As you now may suspect - decrypting is just reversing the process on encryption - but knowing the ciphertext (places where lines cross) and keyword - upper row of the table.</p>
<ul>
<li>Get the keyword and repeat it until it's same length (boomb) as ciphertext (iszxp).</li>
<li>Get the first letter of a keyword from the upper-horizontal row of the table</li>
<li>Get the first letter of a ciphertext and find it in the same row that the letter from the previous step.</li>
<li>Finally find the letter from the left-vertical row that is in the same line as the letter from the previous step. </li>
<li>Repeat for each letter.</li>
</ul>
<p>And that's it! We can easilly encrypt and decrypt the messages with Vigenère cipher.</p>
<h1>History behind Vigenère cipher </h1>
![tabula-recta](/img/encryption-vigenere/cipher-disk.png){:class="img-responsive"}
<p><center><i>A reproduction of the Confederacy's cipher disk</i></center></p>
<p>This cipher was originally invented by Giovan Battista Bellaso in 1553, in book called <i>La cifra del. Sig. Giovan Battista Bellaso</i>, but is called after Blaise de Vigenère. Blaise published the stronger version of a cipher in 1586 which was later in 19th century misattributed to Vigenère. This cipher gained the reputation of being strong, even some mathematicians called it 'unbreakable' but in 19th Friedrich Kasiski published a method of deciphering a Vigenère cipher.</p>
<p>Nowadays there are couple of ways of deciphering the cipher.</p>
<ul>
<li>Kasiski examination</li>
<li>Friedman test</li>
<li>Frequency analysis</li>
<li>Key elimination</li>
</ul>
[Read More](http://practicalcryptography.com/cryptanalysis/stochastic-searching/cryptanalysis-vigenere-cipher/ "Read More")

<h1>Programming the Vigenère cipher</h1>
<p>As always, best way to learn ciphers is by programming one. I'm using Python, with itertools and functools.</p>
{% highlight python %}
#!/usr/bin/python3
from itertools import cycle
import functools
{% endhighlight %}
<p>Then let's see how the encrypt function works.</p>

{% highlight python %}
alphabet = 'abcdefghijklmnopqrstuvwxyz'

def encrypt(key, plaintext):
    pairs = zip(plaintext, cycle(key))
    ciphertext = ''

    for pair in pairs:
        total = functools.reduce(lambda x, y: alphabet.index(x) + alphabet.index(y), pair)
        ciphertext += alphabet[total % 26]

    return ciphertext.lower()
{% endhighlight %}
<p>Firstly I define the alphabet that will help me getting character indexes. Then I'm getting the plaintext and the key, and I'm building a list of tuples using <b>zip()</b> function. Next I use <b>cycle()</b> to repeat the letters of my key for the entire length of the plaintext.</p>
<p>After that I use functools.reduce (reduce in python2) so I can get the index of characters to add them. I finally get the new letter after a modulo of 26 and append that to our resulting ciphertext string.</p>
{% highlight python %}
def decrypt(key, ciphertext):
    pairs = zip(ciphertext, cycle(key))
    plaintext = ''

    for pair in pairs:
        total = functools.reduce(lambda x, y: alphabet.index(x) - alphabet.index(y), pair)
        plaintext += alphabet[total % 26]

    return plaintext

{% endhighlight %}

<p>The decryption function is exactly the same but instead of adding numbers I'm subtracting them.</p>
{% highlight python %}
def main():
    print("[*] Welcome to Vigenere Cipher tool")
    print("[*] Would you like to encrypt [E] or [D] decrypt the message?")
    mode = input("[*] Enter your chooice: ").upper()
    if mode == "E":
        plaintext = input("[*] Please enter your message: ")
        key = input("[*] Now enter your key: ")
        encrypted = encrypt(key, plaintext)
        print("[*] Encrypting...")
        print('[*] Encrytped: %s' % encrypted)
    if mode == "D":
        ciphertext = input("[*] Please enter your message: ")
        key = input("[*] Now enter your key: ")
        decrypted = decrypt(key, ciphertext)
        print("[*] Decrypting...")
        print('[*] Decrypted: %s' % decrypted)
    else:
      	print("[*] Error")

if __name__ == '__main__':
    main()
{% endhighlight %}
<p>Main function is created to get the messages and keywords from the user, as well as a option to encrypt or decrypt the text.</p>
<p>Now let's check how this works!</p>
<p>Firstly encrypt mode, (after entering 'e' or 'E' letter). I'm gonna stick with the word hello and the boom keyword as we may check whether or not it works ;)</p>
{% highlight bash %}
$ python3 viegenare.py
[*] Welcome to Vigenere Cipher tool
[*] Would you like to encrypt [E] or [D] decrypt the message?
[*] Enter your chooice: E
[*] Please enter your message: hello
[*] Now enter your key: boom
[*] Encrypting...
[*] Encrytped: iszxp

{% endhighlight %}
<p>Great! Now the decrypt mode.</p>

{% highlight bash %}
$ python3 viegenare.py
[*] Welcome to Vigenere Cipher tool
[*] Would you like to encrypt [E] or [D] decrypt the message?
[*] Enter your chooice: d
[*] Please enter your message: iszxp
[*] Now enter your key: boom
[*] Decrypting...
[*] Decrypted: hello
{% endhighlight %}
<p>Now we have fully functioning Vigenère cipher tool. I hoped you enjoyed this topic as a part of the <i>Let's talk encryption</i> series. Stay tuned for more updates about encryption as well as other infosec topics. </p>
<p><b>Part of Encryption Tools Project</B></p>
[Github](https://github.com/W3ndige/encryption-tools "Github")
