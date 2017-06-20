---
layout:     post
title:      "Overthewire.org - Behemoth 0 -> 1"
subtitle:   "Write-Up"
date:       2017-03-12 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
permalink: /:title/
category: Overthewire
---
<h1>Introduction</h1>

<p>Today we're going to attack another reverse engineering challenge, this time from the <b>Behemoth</b> series. </p>

<p><i>This wargame deals with a lot of regular vulnerabilities found commonly 'out
in the wild'. While the game makes no attempts at emulating a real environment
it will teach you how to exploit several of the most common coding mistakes
including buffer overflows, race conditions and privilege escalation.</i></p>

<p>Sounds fun, so let's jump straight into the unknown! </p>

<h1>Challenge</h1>

<p>This time, we are given only a binary file, so no more viewing the source code. </p>

{% highlight bash %}
behemoth0@melinda:~$ cd /behemoth/
behemoth0@melinda:/behemoth$ ./behemoth0
Password: password
Access denied..
{% endhighlight %}

<p>We have to somehow get the password which may result in spawning a shell. Our best friend - <b>gdb</b> can help us with getting some better information about how we should approach this challenge.  </p>

{% highlight bash %}
behemoth0@melinda:/behemoth$ gdb ./behemoth0
(gdb) disas main
{% endhighlight %}

<p>This code looks really interesting. Most of the functions are known from C, but what is <b>memfrob</b>? </p>

{% highlight c %}
Dump of assembler code for function main:
   0x080485a2 <+0>:	push   %ebp
   0x080485a3 <+1>:	mov    %esp,%ebp
   0x080485a5 <+3>:	and    $0xfffffff0,%esp
   0x080485a8 <+6>:	sub    $0x70,%esp
   0x080485ab <+9>:	mov    %gs:0x14,%eax
   0x080485b1 <+15>:	mov    %eax,0x6c(%esp)
   0x080485b5 <+19>:	xor    %eax,%eax
   0x080485b7 <+21>:	movl   $0x475e4b4f,0x1f(%esp)
   0x080485bf <+29>:	movl   $0x45425953,0x23(%esp)
   0x080485c7 <+37>:	movl   $0x595e58,0x27(%esp)
   0x080485cf <+45>:	movl   $0x8048720,0x10(%esp)
   0x080485d7 <+53>:	movl   $0x8048738,0x14(%esp)
   0x080485df <+61>:	movl   $0x804874d,0x18(%esp)
   0x080485e7 <+69>:	movl   $0x8048761,(%esp)
   0x080485ee <+76>:	call   0x8048400 <printf@plt>
   0x080485f3 <+81>:	lea    0x2b(%esp),%eax
   0x080485f7 <+85>:	mov    %eax,0x4(%esp)
   0x080485fb <+89>:	movl   $0x804876c,(%esp)
   0x08048602 <+96>:	call   0x8048470 <__isoc99_scanf@plt>
   0x08048607 <+101>:	lea    0x1f(%esp),%eax
   0x0804860b <+105>:	mov    %eax,(%esp)
   0x0804860e <+108>:	call   0x8048440 <strlen@plt>
   0x08048613 <+113>:	mov    %eax,0x4(%esp)
   0x08048617 <+117>:	lea    0x1f(%esp),%eax
   0x0804861b <+121>:	mov    %eax,(%esp)
   0x0804861e <+124>:	call   0x804857d <memfrob>
   0x08048623 <+129>:	lea    0x1f(%esp),%eax
   0x08048627 <+133>:	mov    %eax,0x4(%esp)
   0x0804862b <+137>:	lea    0x2b(%esp),%eax
   0x0804862f <+141>:	mov    %eax,(%esp)
   0x08048632 <+144>:	call   0x80483f0 <strcmp@plt>
   0x08048637 <+149>:	test   %eax,%eax
   0x08048639 <+151>:	jne    0x8048665 <main+195>
   0x0804863b <+153>:	movl   $0x8048771,(%esp)
   0x08048642 <+160>:	call   0x8048420 <puts@plt>
   0x08048647 <+165>:	movl   $0x0,0x8(%esp)
---Type <return> to continue, or q <return> to quit---
   0x0804864f <+173>:	movl   $0x8048782,0x4(%esp)
   0x08048657 <+181>:	movl   $0x8048785,(%esp)
   0x0804865e <+188>:	call   0x8048460 <execl@plt>
   0x08048663 <+193>:	jmp    0x8048671 <main+207>
   0x08048665 <+195>:	movl   $0x804878d,(%esp)
   0x0804866c <+202>:	call   0x8048420 <puts@plt>
   0x08048671 <+207>:	mov    $0x0,%eax
   0x08048676 <+212>:	mov    0x6c(%esp),%edx
   0x0804867a <+216>:	xor    %gs:0x14,%edx
   0x08048681 <+223>:	je     0x8048688 <main+230>
   0x08048683 <+225>:	call   0x8048410 <__stack_chk_fail@plt>
   0x08048688 <+230>:	leave  
   0x08048689 <+231>:	ret    
End of assembler dump.
{% endhighlight s%}

<p>Let's take a look at the man pages description. </p>

<p><i>The  memfrob() function encrypts the first n bytes of the memory area s
by exclusive-ORing each character with the number 42.  The  effect  can
be reversed by using memfrob() on the encrypted memory area.</i></p>

<p><i>Note  that  this function is not a proper encryption routine as the XOR
constant is fixed, and is suitable only for hiding strings.</i></p>

<p>That looks like a good thing to exploit, but I have another idea. Firstly, we will have to set a breakpoint at the <b>strcmp</b> function. </p>

{% highlight bash %}
(gdb) break *0x08048632
Breakpoint 1 at 0x8048632
(gdb) run
Starting program: /games/behemoth/behemoth0
Password: AAAAAAA

Breakpoint 1, 0x08048632 in main ()

{% endhighlight %}

<p>Now let's take a look at the <b>0x0804862f</b> address. This line copies the value from <b>%eax</b> to the location in memory that <b>%esp</b> points to. Maybe we can view what <b>%eax</b> is storing inside? </p>

{% highlight bash %}
(gdb) x/s $eax
0xffffd67b:	"AAAAAAA"
{% endhighlight %}

<p>Wow, that's the password we have entered just seconds ago! So maybe <b>%esp</b> will be our desired password? </p>

{% highlight bash %}
(gdb) x/s $esp
0xffffd650:	"{\326\377\377o\326\377\377p\326\377\377\322\202\004\b \207\004\b8\207\004\bM\207\004\bFw\353eatmyshorts"
{% endhighlight %}

<p>This string at the end looks quite suspicious, maybe we should check that? </p>


{% highlight bash %}
behemoth0@melinda:/behemoth$ ./behemoth0
Password: eatmyshorts
Access granted..
$ cat /etc/behemoth_pass/behemoth1
**********
{% endhighlight %}

<p>It was the password! </p>

<h1>Conclusion</h1>
<p>I'm definitely going to continue Behemoth wargame, as from the beginning challenges are really fun and intriguing. Thanks <a href="http://overthewire.org/wargames/">OverTheWire</a>!</p>
