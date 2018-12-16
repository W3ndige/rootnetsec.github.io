---
layout:     post
title:      "PicoCTF 2018 - Safe RSA"
date:       2018-10-12 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2018
---

Now that you know about RSA can you help us decrypt this ciphertext? We don't have the decryption key but something about those values looks funky.. 

### Solution

{% highlight text %}
N: 374159235470172130988938196520880526947952521620932362050308663243595788308583992120881359365258949723819911758198013202644666489247987314025169670926273213367237020188587742716017314320191350666762541039238241984934473188656610615918474673963331992408750047451253205158436452814354564283003696666945950908549197175404580533132142111356931324330631843602412540295482841975783884766801266552337129105407869020730226041538750535628619717708838029286366761470986056335230171148734027536820544543251801093230809186222940806718221638845816521738601843083746103374974120575519418797642878012234163709518203946599836959811
e: 3

ciphertext (c): 2205316413931134031046440767620541984801091216351222789180582564557328762455422721368029531360076729972211412236072921577317264715424950823091382203435489460522094689149595951010342662368347987862878338851038892082799389023900415351164773
{% endhighlight %}

### Solution

We are presented with ciphertext, public modulus and eponent, which is very small comparable to the ones used in present cryptography (0x10001). As we may remember (or just Google), this scenario would be amazing for large number of messages, using attack called `Håstad's broadcast attack`.

But in this challenge, we only have one message so we have to dig deeper. If the plaintex was very short, unpadded message we could just take the cubic root of the ciphertext in order to recover the message.

Luckily, I've found this awesome [write-up](http://eindbazen.net/2012/05/plaidctf-2012-rsa/), containing the write up for very similar challenge. Read there for more details and modify the script with variables with ours.

{% highlight python %}
#!/bin/python 
import gmpy

n = 0x2f6c2f0f6266f297f890f9246c0b189a702db378d4b021339d995e0c0f03507477cf5ab319206ac6b8141bf3c32071ef0a822018d12f307c9222dff07c0f3556f89327b87e843b29c4567faaea1e6253cd6a647d2ab6679b322a6b32f4bdbb3523c325e027707f6728deaca6914b6cf2456ace3bf848014a511de272c9145f1e042db27380e6dfb823d9eb6a635c885f073ae83b3d19ab7eb4a545cc4e05e336cf8e3d3811d0d501b3fd622a366b52649d66265bb097735e66ac5eef7f1e77aeedf70c58b6f3d1ddcdbc0560177464d8a7750d3f535250e500c3cd7ee03da6851eaee27d5911ec16fc7742b8e15d7f32b137a208ffd05bb9f5275c0f4e64443
e = 0x3
cipher_str = 0x15acbd1c515cb91085f155a56aef8af99cef3c2b32c24beed9048a80216e60b13505f09596e71b1c3f517c407133a492e42c1b974196b95105a48f6f3de240a8ff4de8a847c93e90f46c6bfd93d0ac5fe97d844e95b85fa65c981f8fc79655b9518f65

 
gs = gmpy.mpz(cipher_str)
gm = gmpy.mpz(n)
g3 = gmpy.mpz(3)
 
mask = gmpy.mpz(0x8080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808080808000)
test = 0
while True:
  if test == 0:
   gs = gs
  else:
   gs += gm
  root,exact = gs.root(g3)
  if (root & mask).bit_length() < 8:
    print root
    break

print '\n',hex(int(root))[2:-1].decode('hex')
{% endhighlight %}

Now run the script.

{% highlight text %}
[w3ndige@main]$ python2 solve_safersa.py 
13016382529449106065839070830454998857466392684017754632233906857023684751222397

picoCTF{e_w4y_t00_sm411_81b6559f}
{% endhighlight %}