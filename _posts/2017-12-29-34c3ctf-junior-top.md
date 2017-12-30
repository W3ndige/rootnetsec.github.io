---
layout:     post
title:      "34C3 Junior CTF - top"
date:       2017-12-29 21:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

Let's look at another challenge from ***34C3 Junior CTF***  called ***top***.
- Category: crypto
- Difficulty: easy
- Points: 49

{% highlight text %}
Perfectly secure. That's for sure! Or can break it and reveal my secret?
{% endhighlight %}

Let's take look at the source code provided in the challenge.

{% highlight python %}
import random
import sys
import time

cur_time = str(time.time()).encode('ASCII')
random.seed(cur_time)

msg = input('Your message: ').encode('ASCII')
key = [random.randrange(256) for _ in msg]
c = [m ^ k for (m,k ) in zip(msg + cur_time, key + [0x88]*len(cur_time))]

with open(sys.argv[1], "wb") as f:
    f.write(bytes(c))
{% endhighlight %}

Firstly, I'd like to analyze the code by recreating it with other variables. Let's use python console for that.

{% highlight python %}
>>> import time
>>> import random
>>> cur_time = str(time.time()).encode('ASCII')
>>> random.seed(cur_time)
>>> msg = input('Your message: ').encode('ASCII')
Your message: 34C3_This_i5_our_Flag_forma7_:)
{% endhighlight %}

I'm using one of the flags provided in the ***faq*** section. Now we can check what happens during the encryption process. Firstly, `cur_time` variable is appended to the msg, which is our flag.

{% highlight python %}
>>> msg + cur_time
b'34C3_This_i5_our_Flag_forma7_:)1514571090.8714993'
{% endhighlight %}

Then `0x88` value is being multiplied by the length of `cur_time` variable.

{% highlight bash %}
>>> [0x88]*len(cur_time)
[136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136, 136]
{% endhighlight %}

After that we're ready to show the actual encryption process.

{% highlight bash %}
>>> list(zip(msg + cur_time, key + [0x88]*len(cur_time)))
[(51, 180), (52, 38), (67, 206), (51, 125), (95, 113), (84, 121), (104, 104), (105, 88), (115, 30), (95, 188), (105, 31), (53, 32), (95, 177), (111, 24), (117, 18), (114, 186), (95, 102), (70, 123), (108, 225), (97, 68), (103, 173), (95, 140), (102, 135), (111, 173), (114, 229), (109, 232), (97, 25), (55, 101), (95, 52), (58, 125), (41, 216), (49, 136), (53, 136), (49, 136), (52, 136), (53, 136), (55, 136), (49, 136), (48, 136), (57, 136), (48, 136), (46, 136), (56, 136), (55, 136), (49, 136), (52, 136), (57, 136), (57, 136), (51, 136)]

>>> [m ^ k for (m,k ) in zip(msg + cur_time, key + [0x88]*len(cur_time))]
[135, 18, 141, 78, 46, 45, 0, 49, 109, 227, 118, 21, 238, 119, 103, 200, 57, 61, 141, 37, 202, 211, 225, 194, 151, 133, 120, 82, 107, 71, 241, 185, 189, 185, 188, 189, 191, 185, 184, 177, 184, 166, 176, 191, 185, 188, 177, 177, 187]
{% endhighlight %}

The most important flaw of this script is that we are able to get the `cur_time` variable by ***xoring*** last ***n*** elements of the encrypted message. We know the key used with this, which is `0x88` so let's reverse this scheme.

{% highlight text %}
B9 BD B9 BB BF B9 B1 B9 BB BB A6 B0 BF BA B0 BF BD BA
{% endhighlight %}

These is encrypted part of a message that should represent `cur_time` variable.

{% highlight python %}
>>> chr(0xb9 ^ 136)
'1'
>>> chr(0xbd ^ 136)
'5'
{% endhighlight %}

After completing the process for whole message, we get the `cur_time` - which is `1513719133.8728752`. Now as we know whis value, and random number generator is seeded with this value, we can generate the same numbers as during the generation of the key for the challenge.

{% highlight python %}
>>> cur_time = str(1513719133.8728752).encode('ASCII')
>>> random.seed(cur_time)
>>> [random.randrange(256) for _ in range(57)]
[65, 3, 202, 248, 12, 208, 169, 70, 229, 55, 247, 44, 255, 145, 248, 97, 22, 126, 149, 11, 79, 248, 71, 22, 222, 13, 107, 217, 225, 117, 38, 42, 195, 170, 52, 194, 56, 223, 84, 154, 242, 141, 140, 125, 97, 80, 203, 171, 195, 119, 95, 9, 237, 196, 122, 154, 228]
{% endhighlight %}

This is our key. Now let's write a script that will ***xor*** values of the message and the key.

{% highlight python %}
>>> msg = [0x09, 0x66, 0xB8, 0x9D, 0x2C, 0xB9, 0xDA, 0x66, 0x9C, 0x58, 0x82, 0x5E, 0xDF, 0xF7, 0x94, 0x00, 0x71, 0x44, 0xB5, 0x38, 0x7B, 0xBB, 0x74, 0x49, 0xB1, 0x79, 0x1B, 0x86, 0x95, 0x1A, 0x56, 0x75, 0xB3, 0xDE, 0x5B, 0x9D, 0x48, 0xB0, 0x20, 0xC5, 0x86, 0xFD, 0xE3, 0x22, 0x0E, 0x20, 0xBF, 0xF4, 0xB4, 0x1F, 0x6F, 0x56, 0x8E, 0xA5, 0x08, 0xA9, 0x97]

>>> key = [65, 3, 202, 248, 12, 208, 169, 70, 229, 55, 247, 44, 255, 145, 248, 97, 22, 126, 149, 11, 79, 248, 71, 22, 222, 13, 107, 217, 225, 117, 38, 42, 195, 170, 52, 194, 56, 223, 84, 154, 242, 141, 140, 125, 97, 80, 203, 171, 195, 119, 95, 9, 237, 196, 122, 154, 228]

>>> for i in range(57):
...     c = msg[i] ^ key[i]
...     print(chr(c))
...
{% endhighlight %}

And here's our flag.

{% highlight text %}
34C3_otp_top_pto_pot_tpo_opt_wh0_car3s
{% endhighlight %}
