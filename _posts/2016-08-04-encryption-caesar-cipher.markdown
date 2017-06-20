---
layout:     post
title:      "Let's Talk Encryption"
subtitle:   "Part 1 - Caesar Cipher"
date:       2016-08-04 12:00:00
author:     "W3ndige"
header-img: "img/encryption-caesar-header.jpg"
permalink: /:title/
category: Cryptography
---
<h1>How the Ceasar cipher works?</h1>
<p><b>The Caesar cipher</b>, also called <b>Caesar's code</b>, <b>shift cipher</b> or <b>Caesar shift</b>, is one of the simplest and most known forms of encryption. It is a <b>substitution cipher</b> where each letter in the <b>plaintext</b> (original message) is replaced with a letter corresponding to a certain number of letters up or down in the alphabet creating a <b>cipher text</b>. </p>

![caesar-shift](/img/encryption-caesar/Caesar-cipher-shift.png){:class="img-responsive"}

<p>Caesar cipher, due to the fact that the key is very small is vulnerable to the brute force attack, where attacker has to try only 25 combinations in order to decrypt the message. In addition, the original structure of the plaintext remains intact, so a person trying to decrypt the message can perform a frequency analysis - method to decrypt by analysing the frequency of each letter and comparing it with the frequency of each letter in certain alphabet. For example in English alphabet letter <i>e</i> is the most common one, and by comparing the most common letter in the cipher with letter <i>e</i> we can try to discover the shift that was used. Read more at: </p>
[Letter-Frequency](https://en.wikipedia.org/wiki/Letter_frequency)


<h1>History behind Caesar Cipher</h1>
<p>This cipher is named after Julius Caesar, who used it to protect his communication with generals during military campaigns. According to Suetonius, who wrote a book about life of the Roman emperor, Julius used the substituation cipher to a shift of three, meaning shifted each letter 3 places through the alphabet where D becomes A and so on.</p>

<p><blockquote>If he had anything confidential to say, he wrote it in cipher, that is, by so changing the order of the letters of the alphabet, that not a word could be made out. If anyone wishes to decipher these, and get at their meaning, he must substitute the fourth letter of the alphabet, namely D, for A, and so with the others.
<p>Suetonius, Life of Julius Caesar 56</p>
</blockquote></p>

<h1>Programming Caesar cipher tools</h1>
<p>Great way to learn how this cipher works is to create your own tools to encrypt, decrypt or even brute force the message. You can do this in any programming language you know, but I'm gonna stick with Python as it's my favourite one.</p>
<p>Firsty let's create a function to encrypt the message with a certain shift.</p>
{% highlight python %}
#!/usr/bin/python

key = 'abcdefghijklmnopqrstuvwxyz'

def encrypt(plaintext, n):
    result = ''

    for letter in plaintext.lower():
        try:
            i = (key.index(letter) + n) % 26
            result += key[i]
        except ValueError:
            result += letter

    return result.lower()
{% endhighlight %}
<p>Firstly we declare a variable key that is English aplhabet. Adding the algorith to check if the character exists in our alphabet will allow us to use punctuation, spaces without encrypting them.</p>
<p>Now let's add to this <i>main</i> function that will use function <i>encrypt</i> and will display the results.</p>
{% highlight python %}
def main(plaintext, n):
    encrypted = encrypt(plaintext, n)
    print('[*] Shift: %s' % n)
    print('[*] Plaintext: %s' % plaintext)
    print('[*] Ciphertext: %s' % encrypted)
{% endhighlight %}
<p>And the last step: providing input for the message and the key.</p>
{% highlight python %}
if __name__ == '__main__':
    plaintext = input('Please enter your message: ')
    shift = int(input('Enter the shift: '))
    main(plaintext, shift)
{% endhighlight %}

<p>Great! It works and the message is encrypted. </p>

{% highlight bash %}
$ python3 caesar-encode.py
Please enter your message: No one will ever read this!
Enter the shift: 15
[*] Shift: 15
[*] Plaintext: No one will ever read this!
[*] Ciphertext: cd dct lxaa tktg gtps iwxh!
{% endhighlight %}

<p>But what if we get the encrypted message from a friend, with a provided key? </p>
<p>Let's create a decrypt script!</p>

{% highlight python %}
#!/usr/bin/python

key = 'abcdefghijklmnopqrstuvwxyz'

def decrypt(n, ciphertext):
    result = ''

    for letter in ciphertext:
        try:
            i = (key.index(letter) - n) % 26
            result += key[i]
        except ValueError:
            result += letter

    return result

def main(encrypted, n):
    decrypted = decrypt(n, encrypted)
    print('[*] Shift: %s' % n)
    print('[*] Encrypted: %s' % encrypted)
    print('[*] Decrypted: %s' % decrypted)


if __name__ == '__main__':
    encrypted = input('Please enter your encrypted message: ')
    shift = int(input('Enter the shift: '))
    main(encrypted, shift)

{% endhighlight %}


<p>As you can see it's mostly the same as the encrypt script. Let's test this!</p>

{% highlight bash %}
$ python3 caesar-decode.py
Please enter your encrypted message: cd dct lxaa tktg gtps iwxh!
Enter the shift: 15
[*] Shift: 15
[*] Encrypted: cd dct lxaa tktg gtps iwxh!
[*] Decrypted: no one will ever read this!
{% endhighlight %}

<p>Great! We decrypted the secret message! But what if we don't have a key but our curiosity won't let us leave this message unread. We can try to analyse frequency of letters but it may take some time. So why not to try and brute force it?</p>

{% highlight python %}
#!/usr/bin/python

import sys

ciphertext = input("[*] Enter your ciphertext: ").upper()
alphabet = list("ABCDEFGHIJKLMNOPQRSTUVWXYZ")
plaintext = ""
shift = 1

while shift <= 26:
 for letter in ciphertext:
  if letter in alphabet:
   plaintext += alphabet[(alphabet.index(letter)+shift)%(len(alphabet))]
 print("Shift: " + str(shift))
 print("Ciphertext: " + ciphertext)
 print("Plaintext: " + plaintext)
 shift = shift + 1
 plaintext = ""
{% endhighlight %}

<p>This script is not as advanced as the previous ones becouse as you'll see it 'eats' the spaces but it is capable of trying all combinations so let's give it a try.</p>

{% highlight bash %}
$ python3 caesar-brute.py
[*] Enter your ciphertext: cd dct lxaa tktg gtps iwxh!
Shift: 1
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: DEEDUMYBBULUHHUQTJXYI
Shift: 2
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: EFFEVNZCCVMVIIVRUKYZJ
Shift: 3
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: FGGFWOADDWNWJJWSVLZAK
Shift: 4
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: GHHGXPBEEXOXKKXTWMABL
Shift: 5
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: HIIHYQCFFYPYLLYUXNBCM
Shift: 6
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: IJJIZRDGGZQZMMZVYOCDN
Shift: 7
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: JKKJASEHHARANNAWZPDEO
Shift: 8
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: KLLKBTFIIBSBOOBXAQEFP
Shift: 9
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: LMMLCUGJJCTCPPCYBRFGQ
Shift: 10
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: MNNMDVHKKDUDQQDZCSGHR
Shift: 11
Ciphertext: CD DCT LXAA TKTG GTPS IWXH!
Plaintext: NOONEWILLEVERREADTHIS

{% endhighlight %}


<p>As always I hoped you enjoyed exploring this topic with me and it brough you some clarity about how the ciphers worked back then, how to write encoding, decoding and brute forcing scripts using shift cipher. In the next part of <i>Let's Talk Encryption</i> series we're going to explore more present ciphers.</p>

<p><b>UPDATE!</b></p>
<p>Check out my newest Caesar tool at Github!</p>
[Github](https://github.com/W3ndige/encryption-tools "Github")
