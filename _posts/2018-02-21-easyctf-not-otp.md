---
layout:     post
title:      "EasyCTF 2018 - Not OTP"
date:       2018-02-21 2:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

Today we're going through cryptography challenge from EasyCTF called ***Not OTP***.

- Category: Crypto
- Points: 100

{% highlight text %}
It seems we've intercepted 2 strings that were both encrypted with what looks like OTP! Is it possible to decrypt them?

c1 = 38445d4e5311544249005351535f005d5d0c575b5e4f481155504e495740145f4c505c5c0e196044454817564d4e12515a5f4f12465c4a45431245430050154b4d4d415c560c4f54144440415f595845494c125953575513454e11525e484550424941595b5a4b
c2 = 3343464b415550424b415551454b00405b4553135e5f00455f540c535750464954154a5852505a4b00455f5458004b5f430c575b58550c4e5444545e0056405d5f53101055404155145d5f0053565f59524c54574f46416c5854416e525e11506f485206554e51
{% endhighlight %}

In addition we get a hint telling us that there's something about the ***cribs***.

### Solution

As the hint is about cribs, I decided to firstly use an attack called ***crib dragging***, which works only when two messages are encrypted (xored as in OTP) using the same key. AS you probably know, OTP is only safe when each message has it's own, unique key.

But back to the attack. To make sure the attack will work, we have to know some of the plaintext from one of the messages. It may be part of the flag, most common English words etc. Firstly, we xor two messages together. Then we xor the suspected plaintext with the xored messages, but at each position of the messages. If this operation produces some readable English text, we may suspect that the plaintext was a part of the message.

Now as we know the basics, let's get into the task.

To make solution easier and the process of attacking faster, I decided to use a ready tool called [CribDrag](https://github.com/SpiderLabs/cribdrag) made by Daniel Crowley. Thanks!

From now we're ready to attack both ciphertexts. Firstly, we'll have to xor them with each other but luckily, CribDrag has separate script to do this.

{% highlight bash %}
main:~/projects/easyctf/cribdrag > python2 xorstrings.py 38445d4e5311544249005351535f005d5d0c575b5e4f481155504e495740145f4c505c5c0e196044454817564d4e12515a5f4f12465c4a45431245430050154b4d4d415c560c4f54144440415f595845494c125953575513454e11525e484550424941595b5a4b 3343464b415550424b415551454b00405b4553135e5f00455f540c535750464954154a5852505a4b00455f5458004b5f430c575b58550c4e5444545e0056405d5f53101055404155145d5f0053565f59524c54574f46416c5854416e525e11506f485206554e51
0b071b0512440400024106001614001d06490448001048540a04421a00105216184516045c493a0f450d4802154e590e195318491e09460b1756111d00065516121e514c034c0e0100191f410c0f071c1b00460e1c11147f1d1a503c0c1654002d01135f0e141
{% endhighlight %}

Now let's the output and run it in cribdrag.

{% highlight bash %}
main:~/projects/easyctf/cribdrag > python2 cribdrag.py 0b071b0512440400024106001614001d06490448001048540a04421a00105216184516045c493a0f450d4802154e590e195318491e09460b1756111d00065516121e514c034c0e0100191f410c0f071c1b00460e1c11147f1d1a503c0c1654002d01135f0e141a
Your message is currently:
0	________________________________________
40	________________________________________
80	_______________________
Your key is currently:
0	________________________________________
40	________________________________________
80	_______________________
Please enter your crib:
{% endhighlight %}

Now the guessing part! Firstly, we know that the message will contain `easyctf{` part as it's part of the flag.

{% highlight bash %}
74: "z vdh}{"
75: "$m|~of="
*** 76: "intext u"
*** 77: "jfobc2hg"
78: "b}hy%zzj"
*** 79: "yzs?mhwo"
80: "~a5wer"
{% endhighlight %}


In one of this parts we see text that highly looks like part of the English sentence. Let's take it's position and place in the mesage.

{% highlight bash %}
Enter the correct position, 'none' for no match, or 'end' to quit: 76
Is this crib part of the message or key? Please enter 'message' or 'key': message
Your message is currently:
0	________________________________________
40	____________________________________easy
80	ctf{___________________
Your key is currently:
0	________________________________________
40	____________________________________inte
80	xt u___________________
{% endhighlight %}

My next guess was word `flag`. Will it be a good one?

{% highlight bash %}
67: "*o-i"
*** 68: "e of"
69: "*b`g"
70: "hma~"
Enter the correct position, 'none' for no match, or 'end' to quit: 68
Is this crib part of the message or key? Please enter 'message' or 'key': message
Your message is currently:
0	________________________________________
40	____________________________flag____easy
80	ctf{___________________
Your key is currently:
0	________________________________________
40	____________________________e of____inte
80	xt u___________________
{% endhighlight %}

Now we can see that it's getting more readable. After that we're ready to add `is` , but to make more readable strings, I decided to input whole `flag is easyctf{`.

{% highlight bash %}

66: "7 b+.hs9z vdh}{"
67: "*o-i!ij?$m|~of="
*** 68: "e of plaintext u"
69: "*b`g9v2,jfobc2hg"
70: "hma~?(/b}hy%zzj"
Enter the correct position, 'none' for no match, or 'end' to quit: 68
Is this crib part of the message or key? Please enter 'message' or 'key': message
Your message is currently:
0	________________________________________
40	____________________________flag is easy
80	ctf{___________________
Your key is currently:
0	________________________________________
40	____________________________e of plainte
80	xt u___________________
{% endhighlight %}

After that part I was stuck for a while. I had no clue what to put there next, trying some random different characters until I noticed that there should be a space also before the `flag` word. This simple operation will add another letter into the key, which may help us find the right crib.

{% highlight bash %}
Enter the correct position, 'none' for no match, or 'end' to quit: 67
Is this crib part of the message or key? Please enter 'message' or 'key': message
Your message is currently:
0	________________________________________
40	___________________________ flag is easy
80	ctf{___________________
Your key is currently:
0	________________________________________
40	___________________________le of plainte
80	xt u___________________
{% endhighlight %}


But no luck for me. After that part, my team mate proposed a word `sample`, which gave pretty good results.

{% highlight bash %}
Enter the correct position, 'none' for no match, or 'end' to quit: 62
Is this crib part of the message or key? Please enter 'message' or 'key': key
Your message is currently:
0	________________________________________
40	______________________uess! flag is easy
80	ctf{___________________
Your key is currently:
0	________________________________________
40	______________________ sample of plainte
80	xt u___________________
Please enter your crib:

{% endhighlight %}

Lastly, after quite some time we were able to slowly reveal some parts of the message and key.

![Not OTP Solution](/img/easyctf/not_otp.png){:class="img-responsive center-block"}

### Tools
[https://github.com/SpiderLabs/cribdrag](https://github.com/SpiderLabs/cribdrag1)

### References
[https://travisdazell.blogspot.com/2012/11/many-time-pad-attack-crib-drag.html](https://travisdazell.blogspot.com/2012/11/many-time-pad-attack-crib-drag.html)

[https://en.wikipedia.org/wiki/Known-plaintext_attack](https://en.wikipedia.org/wiki/Known-plaintext_attack)
