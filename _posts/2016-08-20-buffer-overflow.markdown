---
layout:     post
title:      "Buffer Overflow"
subtitle:   "Introduction to Reverse Engineering"
date:       2016-08-20 10:00:00
author:     "W3ndige"
header-img: "img/buffer-overflow-header.jpg"
permalink: /:title/
category: "Reverse Engineering"
---
<h1><a name="introduction">Introductions</a></h1>
<p>Exploiting a buffer overflow vulnerability is very creative and a bit difficult to understand as it takes many different parts of computer technology knowledge to understand and pull off an attack. But after mastering, it's such a powerfull skill, as there are still programs with that kind of vulnerability. In addtion it lets you better understand how computers and programs work.</p>
<p>Let's explore this topic together!</p>
<p><b>Content</b></p>
<ol>
<li><a href="#introduction">Introduction</a></li>
<li><a href="#memory">Memory</a></li>
<li><a href="#registers">Registers</a></li>
<li><a href="#buffer-overflow">Buffer Overflow</a></li>
</ol>


<h1><a name="memory">How the memory is organized?</a></h1>
<p>We're gonna start by briefly examining the memory layout of a C program, especially the stack since this is a place where most of the buffer overflows occurs. </p>
![address-space](/img/buffer-overflow/address-space.png){:class="img-responsive"}


<p><b>Kernel</b> - Section storing command line parameters that we pass to the program and environment variables. </p>
<p><b>Stack</b> - Stack holds local variables,  function parameters and return addresses for each of your functions. We will come back to it in more detail just in a minute. </p>
<p><b>Heap</b> - Dynamically allocated memory for large chunks of data. The heap grows upwards in memory (from lower to higher memory addresses) as more and more memory is required.</p>
<p><b>Data</b> - Place for initialized and unitialized variables.</p>
<p><b>Text</b> - This is the section where the code of the executable stored. It is often read only cause you don't want to be messing around with that ;)</p>


<h1><a name="registers">What about registers?</a></h1>
<p><b>%ESP</b> - Extended Stack Pointer register which purpose is to let you know where on the stack you are. As the stack grows downward in memory (from higher address values to lower address values) the %ESP register points to the lowest memory address.</p>
<p><b>%EBP</b> - Extended Base Stack Pointer pointing to the base address of stack. Local variables are accessed by subtracting offsets from %EBP and function parameters are accessed by adding offsets to it.</p>
<p><b>%EIP</b> - Extended Instruction Pointer contains the address of the next instruction to read on the program. It points always to the <i>Text</i> segment.</p>

<p>Here you can take a look how the stack with few parameters can look like, together with the registers. </p>
![stack](/img/buffer-overflow/stack.png){:class="img-responsive"}
<p>Now let's look at this simple code.</p>
{% highlight c %}
void func(int param_1, int param_2)
{
    int param_3;
    int param_4;
    // some code
}
void main()
{
    func(param_1, param_2);
    // next instruction
}
{% endhighlight %}

<p>Assume that our <i>%EIP</i> is pointing to the <i>func</i> call in <i>main</i>. Now the following steps would be taken: </p>
<ol>
<li>Firstly a function call is found, push parameters on the stack from right to left (<i>param_2</i> first, then <i>param_1</i>)</li>
<li>Now we need to know where to return after <i>func</i> is completed, so push the address of the next instruction on the stack.</li>
<li>Find the address of <i>func</i> and set <i>%EIP</i> to that value. Next we're moving to the <i>func()</i></li>
<li>As we are in the function we need to update <i>%EBP</i>. But before that, we save it onto the stack, so we can later come back to <i>main()</i> </li>
<li>Set <i>%EBP</i> to be equal to <i>%ESP</i>. <i>%EBP</i> now points to current stack pointer.</li>
<li>Push local variables onto the stack. <i>%ESP</i> is being changed in this step.</li>
<li>After func gets over we need to reset the previous stack frame. So set <i>%ESP</i> back to <i>%EBP</i>. Then pop the earlier <i>%EBP</i> from stack, store it back in <i>%EBP</i>. That way pointer register points back to where it pointed in <i>main()</i>.</li>
<li>Pop the return address from stack and set <i>%EIP</i> to it. The control flow comes back to <i>main</i>, just after the func function call.</li>

</ol>

<h1><a name="buffer-overflow">Buffer Overflow Vulnerability</a></h1>
<p>Now as we know the concept behind the memory of running program, let's take a look at vulnerability called buffer overflow. </p>
<p>A <b>buffer overflow</b> happens when you assign more data than can fit into the buffer and overwriting the code beyond memory address resulting in program crash.However the problem is that somewhere beyond your buffer is the return address and if you manage to load byte code of your program you may be able to execute it. It may result with privilage escalation, where you byte code is intended to open the shell, as well as an ability to run functions that were not intended to run at that moment.  </p>
<p>Now let's look at this simple example: </p>
{% highlight c %}
#include <stdio.h>

#include <string.h>


int main(void)
{
    char buff[15];
    int pass = 0;

    printf("\n Enter the password : \n");
    gets(buff);

    if(strcmp(buff, "strongpassword"))
    {
        printf ("\n Wrong Password \n");
    }
    else
    {
        printf ("\n Correct Password \n");
        pass = 1;
    }

    if(pass)
    {
        printf ("\n I love cookies! \n");
    }

    return 0;
}
{% endhighlight %}
<p>We've got this simple C program that asks for a password from user and if the password is correct then it provides you with a a secret.</p>

<p>To compile it we can use this built in gcc compiler. </p>
{% highlight bash %}
gcc vuln.c -o pass -fno-stack-protector
{% endhighlight %}
<p>With -fno-stack-protector argument that will disable the stack protection. </p>
<p>Version to compile 32 bit binaries: </p>
{% highlight bash %}
gcc vuln.c -o pass -fno-stack-protector -m32
{% endhighlight %}
<p>You may need to install 32 bit utilities in order to compile 32 bit binaries. This command worked for me:</p>
{% highlight bash %}
apt-get install gcc-multilib
{% endhighlight %}

<p>Now you can run it by typing this command. As you see it asks us for a password and then if the password is correct it provides us with a secret. </p>
{% highlight bash %}
$ ./pass

 Enter the password :
strongpassword

 Correct Password

 I love cookies

{% endhighlight %}


<p>But what if we enter a string that is more than 15 characters long? Let's see. </p>

{% highlight bash %}
$ ./pass

 Enter the password :
1111111111111111111111111111111

 Wrong Password

 I love cookies
Segmentation Fault

{% endhighlight %}
<p>Even though we entered wrong password it gave us a secret. This is because  attacker supplied an input of length greater than what buffer can hold and it overwrote the memory of integer ‘pass’. So despite of a wrong password, the value of ‘pass’ became non zero and since then the secret was given. </p>

<p>If you're a C programmer and want to avoid buffer overflows there are few tips that can help your work: </p>
<p>[*] Use <b>fgets()</b> instead of gets()</p>
<p>[*] Use <b>strncmp()</b> instead of strcmp(), <b>strncpy()</b> instead of strcpy() and so on.</p>
<p>[*] Make sure that the <b>memory auditing</b> is done properly</p>

<p>That's the logic behind basic buffer overflows, as you may know there are more advanced cases like injecting machine code that can provide us with root shell and so on. We will explore this together in the next part of reverse engineering series. </p>
<p>As always I hope you enjoyed exploring this topic with me and keep tuned for the next part about shellcodes!</p>
