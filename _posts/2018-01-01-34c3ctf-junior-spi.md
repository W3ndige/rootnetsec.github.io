---
layout:     post
title:      "34C3 Junior CTF - spi"
date:       2018-01-01 0:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

This time we're going to look at quite interesting challenge from ***34C3 Junior CTF***  called ***spi***, with a little bit of backstory.
- Category: misc
- Difficulty: easy
- Points: 70

We are provided only with a audio file called `spi_zahlensender`.

<audio controls="controls">
  Your browser does not support the <code>audio</code> element.
  <source src="../img/34c3ctf-junior/spi_zahlensender.ogg" type="audio/ogg">
</audio>
_Cryptic message_

Looking at the file firstly does not seem to tell us anything interesting. Just repeating numbers told by a woman. But in order to understand this task more, I decided to make quick DuckDuckGo search looking for something connected to the file name ***zahlensender***.

And there it is, a Wiki article about [Number Stations](https://en.wikipedia.org/wiki/Numbers_station). We can quickly learn that such stations transmited messages just like in our challenge. Another great article to read about Number Stations can be read on [Crypto Museum](http://www.cryptomuseum.com/spy/owvl/index.htm).

As I read more about this topic, I decided it's essential to write down those numbers. But who would want to manually write down numbers for more than 2 minutes. That's why I decided to use this [tool](https://speech-to-text-demo.ng.bluemix.net/), which made most of the work for me.

{% highlight text %}
Speaker 0:
    Seven eight three four eight one one six seven six one zero five five two one zero three seven six eight three four eight one one six seven six eight three five two one zero three seven six one zero five four eight one one seven seven six eight three six five one one six seven six eight three four eight one one six seven six one zero five six five one one six seven six one zero five four eight one one seven seven six eight three five two one zero three seven six eight three four eight one one six seven six eight three four eight one zero three seven six eight three five two one one seven seven six eight three six five one one seven seven six eight three five two one one seven seven three six seven four eight one one seven seven six eight three five two one one six seven six one zero five six five one one six seven six eight three four eight one one seven seven three six seven four eight one one six seven six eight three four eight one one six seven three six seven four eight one one seven seven six eight three five two one one six seven six one zero five six five one one six seven six eight three four eight one one six seven six eight three six five one one six seven six one.
Speaker 0:
    Five five two one one six seven three six seven five two one one six seven six one zero five five two seven eight six seven one zero three six one six one end of a message.
{% endhighlight %}

Now it was time to correct some mistakes and convert these numbers. Here's the final form.

{% highlight text %}
76 83 48 116 76 105 52 103 76 83 48 116 76 83 52 103 76 105 48 117 76 83 65 116 76 83 48 116 76 105 65 116 76 105 48 117 76 83 52 103 76 83 48 116 76 83 48 103 76 83 52 117 76 83 65 117 76 83 52 117 73 67 48 117 76 83 52 116 76 105 65 116 76 83 48 117 73 67 48 116 76 83 48 116 73 67 48 117 76 83 52 116 76 105 65 116 76 83 48 116 76 83 65 116 76 105 52 116 73 67 52 116 76 105 52 78 67 103 61 61 end of a message.
{% endhighlight %}

Most of the numbers were in ASCII range, so I decided to decode each of them.

{% highlight text %}
L S 0 t L i 4 g L S 0 t L S 4 g L i 0 u L S A t L S 0 t L i A t L i 0 u L S 4 g L S 0 t L S 0 g L S 4 u L S A u L S 4 u I C 0 u L S 4 t L i A t L S 0 u I C 0 t L S 0 t I C 0 u L S 4 t L i A t L S 0 t L S A t L i 4 t I C 4 t L i 4 N C g = =
{% endhighlight %}

Base64! Let's run another round of decoding.

{% highlight text %}
---.. ----. .-.- ----. -.-.-. ----- -..- .-.. -.-.-. ---. ----- -.-.-. ----- -..- .-..
{% endhighlight %}

Now we have Morse Code. But no online service could not read this, stating errors in the message. That's were I started to look for mistakes in writing down numbers from audio, ASCII conversion. But after a while, on Wiki page of [Morse Code](https://en.wikipedia.org/wiki/Morse_code), I found out that dots and dashes are swapped.

As we know the flag starts with `3`, so I looked for Morse Code representation of this number, which is `...--`. Now we can swap these characters.

{% highlight text %}
...-- ....- -.-. ....- .-.-.- ..... .--. -.-- .-.-.- ...- ..... .-.-.- ..... .--. -.--
{% endhighlight %}

Which after decoding shows the flag `34C4.5PY.V5.5PY`.
