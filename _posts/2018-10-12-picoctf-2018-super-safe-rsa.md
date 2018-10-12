---
layout:     post
title:      "PicoCTF 2018 - Super Safe RSA"
date:       2018-10-12 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2018
---

{% highlight text %}
 Dr. Xernon made the mistake of rolling his own crypto.. Can you find the bug and decrypt the message? Connect with nc 2018shell2.picoctf.com 59208. 
{% endhighlight %}

### Solution

```
[w3ndige@main ~]$ nc 2018shell2.picoctf.com 59208
c: 7030597768026774310122104133160676217061526490101791811961111564793541177307659
n: 15275860861038824209873749378641369526079415895246568701858306343944764452502111
e: 65537
```

First thing that comes to my mind in RSA challenges, especially with these variables is to try and factorize the public modulus `n`. We can simply do that with [factordb](https://factordb.com/), that stores huge database of factorized numbers. 

Here we can find that `n` is a product of 2 prime numbers. 

{% highlight text %}
1527586086...11 = 138873400617639818142290700050114985179 · 109998464739103330461289147649384134274509
{% endhighlight %}

Now we know that both these numbes are our `p` and `q`. With that information we can easily calculate `d` and then decrypt the ciphertext. 

```python
>>> p = 138873400617639818142290700050114985179
>>> q = 109998464739103330461289147649384134274509
>>> phi = (p - 1) * (q - 1)
>>> def egcd(a, b):
...     if a == 0:
...         return (b, 0, 1)
...     g, y, x = egcd(b%a,a)
...     return (g, x - (b//a) * y, y)
... 
>>> def modinv(a, m):
...     g, x, y = egcd(a, m)
...     if g != 1:
...         raise Exception('No modular inverse')
...     return x%m
... 
>>> e = 65537
>>> d = modinv(e, phi)
```

As we have the `d`, just raise the `c` to the power of `d` modulo `n`. 

```python
>>> c = 7030597768026774310122104133160676217061526490101791811961111564793541177307659
>>> n = 15275860861038824209873749378641369526079415895246568701858306343944764452502111
>>> hex(pow(c, d, n))
'0x7069636f4354467b7573335f6c40726733725f7072316d33245f323436317d'
>>> txt = '7069636f4354467b7573335f6c40726733725f7072316d33245f323436317d'
>>> ''.join([chr(int(''.join(c), 16)) for c in zip(txt[0::2],txt[1::2])])
'picoCTF{us3_l@rg3r_pr1m3$_2461}'
```

There we have our flag!
