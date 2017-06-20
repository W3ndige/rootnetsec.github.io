---
layout:     post
title:      "Overthewire.org - Bandit"
subtitle:   "Write-Up"
date:       2016-07-20 12:00:00
author:     "W3ndige"
header-img: "img/overthewire-header.png"
permalink: /:title/
category: Overthewire
---

<h1>Introduction</h1>

<p>Bandit wargame is ideal for begginers - it let's you get to know the basics of Linux operating system, and is a great start into the beautiful word of CTF's.</p>
<p>Let's get started!</p>

<b>0 --> 1</b>
<p>In the first level, we get to know the <i>ls</i> command which is used to view files in the specified directory and the <i>cat</i> command that is used to view files.</p>
{% highlight bash %}
bandit0@melissa:~$ ls
readme
bandit0@melissa:~$ cat readme
{% endhighlight %}

<b>1 --> 2</b>
<p>This time we have to use a simple trick, because the file containing the password is called <i>-</i>, and it won't be doable as in the previous task. It would be too easy :)</p>
{% highlight bash %}
bandit1@melissa:~$ ls
-
bandit1@melissa:~$ cat ./-
{% endhighlight %}

<b>2 --> 3</b>
<p>Files with the spaces in the names can be viewed using quotes.</p>
{% highlight bash %}
bandit2@melissa:~$ ls
spaces in this filename
bandit2@melissa:~$ cat "spaces in this filename"
{% endhighlight %}

<b>3 --> 4</b>
<p><i>Ls</i> is an powerfull command, with <i>-la</i> option we can view the hidden files and directories.</p>
{% highlight bash %}
bandit3@melissa:~$ ls
inhere
bandit3@melissa:~$ cd inhere
bandit3@melissa:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 2012-05-10 23:51 .
drwxr-xr-x 3 root    root    4096 2012-05-10 23:51 ..
-rw-r----- 1 bandit4 bandit3   33 2012-05-10 23:51 .hidden
bandit3@melissa:~/inhere$ cat .hidden
{% endhighlight %}

<b>4 --> 5</b>
<p>This time we've got a hint that it's only human readable file. We can make use of a <i>file</i> command that will let us see the information about each one.</p>
{% highlight bash %}
bandit4@melissa:~$ ls
inhere
bandit4@melissa:~$ cd inhere
bandit4@melissa:~/inhere$ ls -la
total 48
drwxr-xr-x 2 root    root    4096 2012-05-10 23:51 .
drwxr-xr-x 3 root    root    4096 2012-05-10 23:51 ..
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file00
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file01
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file02
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file03
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file04
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file05
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file06
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file07
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file08
-rw-r----- 1 bandit5 bandit4   33 2012-05-10 23:51 -file09
bandit4@melissa:~/inhere$ file ./-*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
bandit4@melissa:~/inhere$ cat ./-file07
{% endhighlight %}

<b>5 --> 6</b>
<p>Now we've got repeat from the previous level, but in addition it is file that is 1033 bytes in size and is not executable. <i>Find</i> command may be useful.</p>
{% highlight bash %}
bandit5@melissa:~$ ls
inhere
bandit5@melissa:~$ cd inhere
bandit5@melissa:~/inhere$ ls -la
total 88
drwxr-x--- 22 root bandit5 4096 2012-05-10 23:51 .
drwxr-xr-x  3 root root    4096 2012-05-10 23:51 ..
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere00
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere01
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere02
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere03
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere04
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere05
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere06
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere07
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere08
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere09
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere10
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere11
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere12
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere13
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere14
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere15
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere16
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere17
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere18
drwxr-x---  2 root bandit5 4096 2012-05-10 23:51 maybehere19
bandit5@melissa:~/inhere$ find ./ -size 1033c
./maybehere07/.file2
bandit5@melissa:~/inhere$ cat ./maybehere07/.file2
{% endhighlight %}

<b>6 --> 7</b>
<p>This one is a little bit trickier since it has properties: -owned by user bandit7 -owned by group bandit6 -33 bytes in size.</p>
{% highlight bash %}
bandit6@melissa:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@melissa:~$ cat /var/lib/dpkg/info/bandit7.password
{% endhighlight %}

<b>7 --> 8</b>
<p>Now we have file called data.txt, where password is stored next to word 'millionth'. <i>Cat</i> and <i>grep</i> commands will help us a lot!</p>
{% highlight bash %}
bandit7@melissa:~$ ls
data.txt
bandit7@melissa:~$ cat data.txt | grep millionth
millionth       ********************
{% endhighlight %}

<b>8 --> 9</b>
<p>We have to find a string that only occurs once in the data.txt file. Let's pipe <i>cat</i>, <i>sort</i> and <i>uniq</i> commands.</p>
{% highlight bash %}
bandit8@melissa:~$ cat data.txt | sort | uniq -u
{% endhighlight %}

<b>9 --> 10</b>
<p>Differently from the last one, password is stored somewhere among the few lines of '='</p>
{% highlight bash %}
bandit9@melissa:~$ ls
data.txt
bandit9@melissa:~$ strings data.txt | grep '='
========== the
R=ev2,
NF=!^
M5Q=
========== password
TuI@=
========== iss
c       =$
w=RO
eD=p
jR=JlB
G========== ****************
:=1p
KA=%
{% endhighlight %}

<b>10 --> 11</b>
<p>In this one, password is encoded with base64 algortihm. We can use built in linux tool <i>base64</i></p>
{% highlight bash %}
bandit10@melissa:~$ base64 -d data.txt
The password is
{% endhighlight %}

<b>11 --> 12</b>
<p>Rot13 encryption can be really fun programming project, which can be implemented in many, many languages but in this level, let's use shell script </p>
{% highlight bash %}
bandit11@melissa:~$ cat data.txt | tr a-zA-Z n-za-mN-ZA-M
The password is
{% endhighlight %}


<b>12 --> 13</b>
<p><b>Really iritating level</b></p>
<p>We are provided with a hexdump of a file that has been repeatedly compressed. Useful thing would be to create a work directory in /tmp.</p>
{% highlight bash %}
bandit11@melinda:~$ mkdir /tmp/w3nditor/
bandit11@melinda:~$ cd /tmp/w3nditor/
bandit11@melinda:/tmp/w3nditor$ xxd -r ~/data.txt dane
bandit12@melinda:/tmp/w3nditor$ file dane  
dane: gzip compressed data, was "data2.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
bandit12@melinda:/tmp/w3nditor$ file dane
dane: bzip2 compressed data, block size = 900k
bandit12@melinda:/tmp/w3nditor$ bunzip2 dane
bunzip2: Cant guess original name for dane -- using dane.out
bandit12@melinda:/tmp/w3nditor$ file dane.out
dane.out: gzip compressed data, was "data4.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
bandit12@melinda:/tmp/w3nditor$ mv dane.out dane.gz
bandit12@melinda:/tmp/w3nditor$ gunzip dane.gz
bandit12@melinda:/tmp/w3nditor$ file dane
dane: POSIX tar archive (GNU)
bandit12@melinda:/tmp/w3nditor$ tar xvf dane
data5.bin
bandit12@melinda:/tmp/w3nditor$ file data5.bin  
data5.bin: POSIX tar archive (GNU)
bandit12@melinda:/tmp/w3nditor$ tar xvf data5.bin
data6.bin
bandit12@melinda:/tmp/w3nditor$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@melinda:/tmp/w3nditor$ mv data6.bin data6.bz
bandit12@melinda:/tmp/w3nditor$ bunzip2 data6.bz
bandit12@melinda:/tmp/w3nditor$ file data6
data6: POSIX tar archive (GNU)
bandit12@melinda:/tmp/w3nditor$ tar xvf data6
data8.bin
bandit12@melinda:/tmp/w3nditor$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
bandit12@melinda:/tmp/w3nditor$ mv data8.bin data8.gz
bandit12@melinda:/tmp/w3nditor$ gunzip data8.gz
bandit12@melinda:/tmp/w3nditor$ file data8
data8: ASCII text
bandit12@melinda:/tmp/w3nditor$ cat data8
The password is
{% endhighlight %}

<b>13 --> 14</b>
<p>Password stored in /etc/bandit_pass/bandit14 and only readable by bandit14. What can we do about it? Maybe let's use the SSH key stored in the home directory</p>
{% highlight bash %}
bandit13@melinda:~$ ls
sshkey.private
bandit13@melinda:~$ cat sshkey.private
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAxkkOE83W2cOT7IWhFc9aPaaQmQDdgzuXCv+ppZHa++buSkN+
gg0tcr7Fw8NLGa5+Uzec2rEg0WmeevB13AIoYp0MZyETq46t+jk9puNwZwIt9XgB
ZufGtZEwWbFWw/vVLNwOXBe4UWStGRWzgPpEeSv5Tb1VjLZIBdGphTIK22Amz6Zb
ThMsiMnyJafEwJ/T8PQO3myS91vUHEuoOMAzoUID4kN0MEZ3+XahyK0HJVq68KsV
ObefXG1vvA3GAJ29kxJaqvRfgYnqZryWN7w3CHjNU4c/2Jkp+n8L0SnxaNA+WYA7
jiPyTF0is8uzMlYQ4l1Lzh/8/MpvhCQF8r22dwIDAQABAoIBAQC6dWBjhyEOzjeA
J3j/RWmap9M5zfJ/wb2bfidNpwbB8rsJ4sZIDZQ7XuIh4LfygoAQSS+bBw3RXvzE
pvJt3SmU8hIDuLsCjL1VnBY5pY7Bju8g8aR/3FyjyNAqx/TLfzlLYfOu7i9Jet67
xAh0tONG/u8FB5I3LAI2Vp6OviwvdWeC4nOxCthldpuPKNLA8rmMMVRTKQ+7T2VS
nXmwYckKUcUgzoVSpiNZaS0zUDypdpy2+tRH3MQa5kqN1YKjvF8RC47woOYCktsD
o3FFpGNFec9Taa3Msy+DfQQhHKZFKIL3bJDONtmrVvtYK40/yeU4aZ/HA2DQzwhe
ol1AfiEhAoGBAOnVjosBkm7sblK+n4IEwPxs8sOmhPnTDUy5WGrpSCrXOmsVIBUf
laL3ZGLx3xCIwtCnEucB9DvN2HZkupc/h6hTKUYLqXuyLD8njTrbRhLgbC9QrKrS
M1F2fSTxVqPtZDlDMwjNR04xHA/fKh8bXXyTMqOHNJTHHNhbh3McdURjAoGBANkU
1hqfnw7+aXncJ9bjysr1ZWbqOE5Nd8AFgfwaKuGTTVX2NsUQnCMWdOp+wFak40JH
PKWkJNdBG+ex0H9JNQsTK3X5PBMAS8AfX0GrKeuwKWA6erytVTqjOfLYcdp5+z9s
8DtVCxDuVsM+i4X8UqIGOlvGbtKEVokHPFXP1q/dAoGAcHg5YX7WEehCgCYTzpO+
xysX8ScM2qS6xuZ3MqUWAxUWkh7NGZvhe0sGy9iOdANzwKw7mUUFViaCMR/t54W1
GC83sOs3D7n5Mj8x3NdO8xFit7dT9a245TvaoYQ7KgmqpSg/ScKCw4c3eiLava+J
3btnJeSIU+8ZXq9XjPRpKwUCgYA7z6LiOQKxNeXH3qHXcnHok855maUj5fJNpPbY
iDkyZ8ySF8GlcFsky8Yw6fWCqfG3zDrohJ5l9JmEsBh7SadkwsZhvecQcS9t4vby
9/8X4jS0P8ibfcKS4nBP+dT81kkkg5Z5MohXBORA7VWx+ACohcDEkprsQ+w32xeD
qT1EvQKBgQDKm8ws2ByvSUVs9GjTilCajFqLJ0eVYzRPaY6f++Gv/UVfAPV4c+S0
kAWpXbv5tbkkzbS0eaLPTKgLzavXtQoTtKwrjpolHKIHUz6Wu+n4abfAIRFubOdN
/+aLoRQ0yBDRbdXMsZN/jvY44eM+xRLdRVyMmdPtP8belRi2E2aEzA==
-----END RSA PRIVATE KEY-----
bandit13@melinda:~$ ssh -i sshkey.private bandit14@localhost
Could not create directory '/home/bandit13/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' cant be established.
ECDSA key fingerprint is 05:3a:1c:25:35:0a:ed:2f:cd:87:1c:f6:fe:69:e4:f6.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit13/.ssh/known_hosts).

This is the OverTheWire game server. More information on http://www.overthewire.org/wargames
bandit14@melinda:~$ cat /etc/bandit_pass/bandit14

{% endhighlight %}

<b>14 --> 15</b>
<p>In this one, our task is to submit current password to a localhost at port 30000.</p>
{% highlight bash %}
bandit14@melinda:~$ telnet localhost 3000
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection refused
bandit14@melinda:~$ telnet localhost 30000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
***************************
Correct!


Connection closed by foreign host.
{% endhighlight %}

<b>15 --> 16</b>
<p>Same as previous one, we have to only add SSL.</p>
{% highlight bash %}
bandit15@melissa:~$ openssl s_client -connect localhost:30001
CONNECTED(00000003)
depth=0 /CN=melissa.labs.overthewire.org
verify error:num=18:self signed certificate
verify return:1
depth=0 /CN=melissa.labs.overthewire.org
verify return:1
---
Certificate chain
0 s:/CN=melissa.labs.overthewire.org
i:/CN=melissa.labs.overthewire.org
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICyjCCAbICCQDE6DxysXt56TANBgkqhkiG9w0BAQUFADAnMSUwIwYDVQQDExxt
ZWxpc3NhLmxhYnMub3ZlcnRoZXdpcmUub3JnMB4XDTEyMDUxMDIxMzYzOVoXDTIy
MDUwODIxMzYzOVowJzElMCMGA1UEAxMcbWVsaXNzYS5sYWJzLm92ZXJ0aGV3aXJl
Lm9yZzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAL85VFz7tV/45RID
5x804dSKyvmZH62lOjAg0NhW7Kbc9L6mmq3EVd4As/kupXYs0d7hCiMjJri0X2e8
GTM+nysxZLTR1qa2j/KOzQ7FgQ4vp4R4JQZP6ofhNPvBybh6BwYE5hFzRARK9Y3x
+dr3ZefeAE7Ea1k6NzH7p6HAtpkG36SD6GbhLV9HFhwOCwBWGPnXPfXA/2XBdZzY
/h6FWrxZPqdALjy8dCeRlNPqG7dD8CIWK4dpBGudxfyXiki5YfwOirotEWjI1E/C
JK2/jWT7tYLIrVKzOF0dwDWYxNMRnwn5+S2F2/AERSRBlwrtMb6jJf+g2pU27eAe
3xvtJs8CAwEAATANBgkqhkiG9w0BAQUFAAOCAQEAtDKEX9gWmEyKqkhPN1L+wjEi
M2HH/XMgDxHrqWgy0Xl9gznuvM0pkOEXUOKWkfKDQfskk8cbgqn0hEvaX7AKrNL4
Nbm1JD+hUSSFtW3sxmv+aHkdEz6H70oUp712wP2Hu3DF7paVSPC5yB1vqoNYmHX/
J9CwqptVj+dLaDeY+ayzEwOuaEcd+cpP4OTbMLy0SuKLONr1+NaA5IPaVE/XOmlE
wW7zNRcJ3kxnvsHrqF4ZeYPBLNmhDT3ZD4qso+JiL9lme5YbP7+dCQo5Oa1AT7Dz
UmKZhWQTLsnI6Eyl8NwLnxiSkIOUigN6WF8bnd1F9FVKfmjQDSjBJHGqTE4Trg==
-----END CERTIFICATE-----
subject=/CN=melissa.labs.overthewire.org
issuer=/CN=melissa.labs.overthewire.org
---
No client certificate CA names sent
---
SSL handshake has read 1436 bytes and written 229 bytes
---
New, TLSv1/SSLv3, Cipher is DHE-RSA-AES256-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: zlib compression
Expansion: zlib compression
SSL-Session:
Protocol  : TLSv1
Cipher    : DHE-RSA-AES256-SHA
Session-ID: 5AED820CF694E077E4F590C9089FC77A050DC3A3BBCE7F383B811CBD4937DAC9
Session-ID-ctx:
Master-Key: 201FB305DD48B3D4746DA988FD88B0EF939A766A393DFED1D9184DA6BD41B28F4ABDF06AE23DA7B0DEFF0329C69499E8
Key-Arg   : None
TLS session ticket:
0000 - b4 b5 f0 bf 88 14 bc 85-59 9b e6 22 ea f3 7f a1   ........Y......
0010 - 61 8a 25 48 a4 08 cb 5c-f0 2d 8a 97 b1 78 c3 eb   a.%H...\.-...x..
0020 - 14 6b 41 99 71 5e 62 6b-bf 6a 17 18 82 cc 69 1a   .kA.q^bk.j....i.
0030 - d5 a3 fd 08 97 8c b8 3a-d7 52 7c 01 31 eb c8 be   .......:.R|.1...
0040 - 09 a0 fd 58 cc aa d9 98-51 53 71 98 7d 8f 92 78   ...X....QSq.}..x
0050 - 00 8c d3 1d b0 57 df 70-0a af 92 44 6c b8 5e 85   .....W.p...Dl.^.
0060 - 1f e1 87 fd c6 da db bd-35 da 89 a0 b9 da fe 37   ........5......7
0070 - 0f 5b 4e d9 96 16 3b 7e-6b fb 0f 42 51 67 5f d9   .[N...;~k..BQg_.
0080 - 11 9a 8d a3 95 2a 9b d1-f6 9b ce 2c 55 62 92 4b   ..........,Ub.K
0090 - a8 89 b1 9f 8a b8 f7 6b-b7 65 2d e4 7e 52 6b 6c   .......k.e-.~Rkl

Compression: 1 (zlib compression)
Start Time: 1363810708
Timeout   : 300 (sec)
Verify return code: 18 (self signed certificate)
---
*****************************
Correct!
*****************************

read:errno=0
{% endhighlight %}

<b>16 --> 17</b>
<p>Now we've got a range of ports that we have to check, using <i>nmap</i> tool, then we have to check which one is using SSL.</p>
{% highlight bash %}
bandit16@melinda:~$ nmap -p31000-32000 localhost -sV

Starting Nmap 5.21 ( http://nmap.org ) at 2014-07-31 20:13 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00076s latency).
Not shown: 995 closed ports
PORT      STATE SERVICE VERSION
31000/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (protocol 2.0)
31046/tcp open  echo
31518/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31691/tcp open  echo
31790/tcp open  msdtc   Microsoft Distributed Transaction Coordinator (error)
31960/tcp open  echo
Service Info: OSs: Linux, Windows

Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.19 seconds
{% endhighlight %}
<p>From the scan we are able to say that ports 31000, 31046, 31691 and 31960 are out. Let's check other ones, and yes! Port 31790 is the desired one!</p>
{% highlight bash %}
bandit16@melissa:~$ openssl s_client -connect localhost:31790
CONNECTED(00000003)
depth=0 /CN=melissa.labs.overthewire.org
verify error:num=18:self signed certificate
verify return:1
depth=0 /CN=melissa.labs.overthewire.org
verify return:1
---
Certificate chain
0 s:/CN=melissa.labs.overthewire.org
i:/CN=melissa.labs.overthewire.org
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICyjCCAbICCQDE6DxysXt56TANBgkqhkiG9w0BAQUFADAnMSUwIwYDVQQDExxt
ZWxpc3NhLmxhYnMub3ZlcnRoZXdpcmUub3JnMB4XDTEyMDUxMDIxMzYzOVoXDTIy
MDUwODIxMzYzOVowJzElMCMGA1UEAxMcbWVsaXNzYS5sYWJzLm92ZXJ0aGV3aXJl
Lm9yZzCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAL85VFz7tV/45RID
5x804dSKyvmZH62lOjAg0NhW7Kbc9L6mmq3EVd4As/kupXYs0d7hCiMjJri0X2e8
GTM+nysxZLTR1qa2j/KOzQ7FgQ4vp4R4JQZP6ofhNPvBybh6BwYE5hFzRARK9Y3x
+dr3ZefeAE7Ea1k6NzH7p6HAtpkG36SD6GbhLV9HFhwOCwBWGPnXPfXA/2XBdZzY
/h6FWrxZPqdALjy8dCeRlNPqG7dD8CIWK4dpBGudxfyXiki5YfwOirotEWjI1E/C
JK2/jWT7tYLIrVKzOF0dwDWYxNMRnwn5+S2F2/AERSRBlwrtMb6jJf+g2pU27eAe
3xvtJs8CAwEAATANBgkqhkiG9w0BAQUFAAOCAQEAtDKEX9gWmEyKqkhPN1L+wjEi
M2HH/XMgDxHrqWgy0Xl9gznuvM0pkOEXUOKWkfKDQfskk8cbgqn0hEvaX7AKrNL4
Nbm1JD+hUSSFtW3sxmv+aHkdEz6H70oUp712wP2Hu3DF7paVSPC5yB1vqoNYmHX/
J9CwqptVj+dLaDeY+ayzEwOuaEcd+cpP4OTbMLy0SuKLONr1+NaA5IPaVE/XOmlE
wW7zNRcJ3kxnvsHrqF4ZeYPBLNmhDT3ZD4qso+JiL9lme5YbP7+dCQo5Oa1AT7Dz
UmKZhWQTLsnI6Eyl8NwLnxiSkIOUigN6WF8bnd1F9FVKfmjQDSjBJHGqTE4Trg==
-----END CERTIFICATE-----
subject=/CN=melissa.labs.overthewire.org
issuer=/CN=melissa.labs.overthewire.org
---
No client certificate CA names sent
---
SSL handshake has read 1436 bytes and written 229 bytes
---
New, TLSv1/SSLv3, Cipher is DHE-RSA-AES256-SHA
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: zlib compression
Expansion: zlib compression
SSL-Session:
Protocol  : TLSv1
Cipher    : DHE-RSA-AES256-SHA
Session-ID: 913086AEE63018EFB254F2105A4597FC5CB419BFFBCE5B1FAF10EC7967668530
Session-ID-ctx:
Master-Key: 155AF180C1B8BB81BDE85105A05F1C6D5E7B8511B6C9E83B257EC2012B102170522C965E114B233D108A838C7520DED6
Key-Arg   : None
TLS session ticket:
0000 - 95 66 61 4b c2 a6 3c 36-50 d2 8d fd 58 fb 03 30   .faK..<6P...X..0
0010 - 2c 38 20 12 84 02 08 68-d0 f3 5d 47 1a 8a 86 b2   ,8 ....h..]G....
0020 - 01 19 4f cb 46 85 e8 a2-36 e2 ac fd 9f 2e 66 1e   ..O.F...6.....f.
0030 - ab 99 67 49 93 f0 82 0e-56 60 0f 4b c2 28 b6 7b   ..gI....V.K.(.{
0040 - 7f 55 f9 cf 9d d9 07 0a-4f 40 a6 7d cc 89 4b b4   .U......O@.}..K.
0050 - f3 2a 0e b8 35 ac e9 e3-04 b7 3b e8 b5 32 8b 7a   ...5.....;..2.z
0060 - a8 05 c9 e9 89 74 c4 fc-40 8d 3b 49 2e de 63 be   .....t..@.;I..c.
0070 - 25 b5 f5 05 85 17 82 92-8b 95 5c 8b e6 e9 f3 e7   %..............
0080 - 21 49 9b a6 b8 82 fc d8-6e 67 54 31 ad a3 75 ee   !I......ngT1..u.
0090 - 43 07 47 54 bb fa d9 9a-2a ef 20 85 28 2d 2b 63   C.GT..... .(-+c

Compression: 1 (zlib compression)
Start Time: 1363811727
Timeout   : 300 (sec)
Verify return code: 18 (self signed certificate)
---

Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

read:errno=0



bandit16@melissa:~$ mkdir /tmp/ssss
bandit16@melissa:~$ cd /tmp/ssss
bandit16@melissa:/tmp/ssss$ touch sshkey.private
bandit16@melissa:/tmp/ssss$ vi sshkey.private
bandit16@melissa:/tmp/ssss$ chmod 600 sshkey.private
bandit16@melissa:/tmp/ssss$ ssh -i sshkey.private bandit17@localhost
Could not create directory '/home/bandit16/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' can't be established.
RSA key fingerprint is 9d:09:d9:46:84:df:f9:dd:cc:7c:dc:49:a0:95:b2:10.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit16/.ssh/known_hosts).
bandit17@melissa:~$
{% endhighlight %}

<b>17 --> 18</b>
<p>Result of comparing the differences of two files will be our password.</p>
{% highlight bash %}
bandit17@melissa:~$ ls
passwords.new  passwords.old
bandit17@melissa:~$ diff passwords.new  passwords.old
42c42
< ******************** --- > ********************
{% endhighlight %}

<b>18 --> 19</b>
<p>Password is stored in the home directory but each time we try to connect using SSH, we are immediately disconnected.</p>
{% highlight bash %}
~ $ ssh bandit18@bandit.labs.overthewire.org
Byebye !
Connection to bandit.labs.overthewire.org closed.

~ $ ssh bandit18@bandit.labs.overthewire.org cat readme

This is the OverTheWire game server. More information on http://www.overthewire.org/wargames.

Note: at this moment, blacksun is not available.

bandit18@bandit.labs.overthewire.org's password:
{% endhighlight %}

<b>19 --> 20</b>
<p>Here they give us a SUID binary. We can view the usage, and luckily it runs commands as other user. Let's check it out!</p>
{% highlight bash %}
bandit19@melinda:~$ ls
bandit20-do
bandit19@melinda:~$ ./bandit20-do
Run a command as another user.
Example: ./bandit20-do id

bandit19@melinda:~$ ./bandit20-do cat /etc/bandit_pass/bandit20

{% endhighlight %}

<b>20 --> 21</b>
<p>After logging in, we can see that there is a file  called. ‘suconnect’. It makes a connection to localhost on the port you specify as a argument. After that it reads a line and compares it to the password in the bandit20. If the password is correct, it will transmit the password for the next level.</p>
<p>In first shell:</p>
{% highlight bash %}
bandit20@melinda:~$ nc -l 55555
{% endhighlight %}
<p>Then in the second one let’s use our suconnect binary:</p>
{% highlight bash %}
bandit20@melinda:~$ ./suconnect 55555
{% endhighlight %}
<p>After entering correct password in the nc shell, we will get a password for the next level.</p>
{% highlight bash %}
********************************
********************************
{% endhighlight %}

<b>21 --> 22</b>
<p>There is a cron job, running certain scripts in different intervals. We can view the cron directory, where <i>cronjob_bandit22 looks like what we need.</i></p>
{% highlight bash %}
bandit21@melissa:~$ cd /etc/cron.d/
bandit21@melissa:/etc/cron.d$ ls -la
total 100
drwxr-xr-x  2 root root 4096 2013-01-03 16:39 .
drwxr-xr-x 94 root root 4096 2013-03-18 15:53 ..
-rw-r--r--  1 root root   54 2013-01-03 16:39 boobiesbot-check
-rw-r--r--  1 root root   61 2012-07-05 09:34 cronjob_bandit22
-rw-r--r--  1 root root   61 2012-07-05 09:34 cronjob_bandit23
-rw-r--r--  1 root root   61 2012-07-05 09:35 cronjob_bandit24
-rw-r--r--  1 root root   35 2012-03-29 14:16 eloi0
-rw-r--r--  1 root root   35 2012-04-07 18:33 eloi1
-rw-r--r--  1 root root   51 2012-12-23 00:06 hintbot-check
-rw-------  1 root root  233 2012-09-14 13:16 manpage3_resetpw_job
-rw-r--r--  1 root root  506 2012-06-19 03:06 php5
-rw-r--r--  1 root root  102 2011-01-05 11:23 .placeholder
-rw-r--r--  1 root root   58 2011-11-13 23:08 semtex0-32
-rw-r--r--  1 root root   58 2011-11-13 23:08 semtex0-64
-rw-r--r--  1 root root   59 2011-11-13 23:08 semtex0-ppc
-rw-r--r--  1 root root   36 2011-11-25 14:00 semtex10
-rw-r--r--  1 root root  143 2011-11-13 23:08 semtex12
-rw-r--r--  1 root root   35 2011-11-13 23:08 semtex5
-rw-r--r--  1 root root   29 2011-11-13 23:08 semtex6
-rw-r--r--  1 root root   96 2011-11-25 14:00 semtex8
-rw-r--r--  1 root root  134 2011-11-13 23:14 semtex9
-rw-r--r--  1 root root   29 2011-11-13 23:07 vortex0
-rw-r--r--  1 root root   30 2012-03-24 21:00 vortex20
-rw-r--r--  1 root root   52 2012-12-23 00:06 vulnbot0-check
-rw-r--r--  1 root root   52 2012-12-23 00:06 vulnbot1-check
bandit21@melissa:/etc/cron.d$ cat cronjob_bandit22
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
bandit21@melissa:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
bandit21@melissa:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
********************************
{% endhighlight %}

<b>22 --> 23</b>
<p>Another cron job. This time we weill have to modify the script (whoami parameter) to see where we have to look for the password.</p>
{% highlight bash %}
andit22@melissa:~$ cd /etc/cron.d
bandit22@melissa:/etc/cron.d$ ls
boobiesbot-check  eloi1                 semtex0-64   semtex6   vulnbot0-check
cronjob_bandit22  hintbot-check         semtex0-ppc  semtex8   vulnbot1-check
cronjob_bandit23  manpage3_resetpw_job  semtex10     semtex9
cronjob_bandit24  php5                  semtex12     vortex0
eloi0             semtex0-32            semtex5      vortex20
bandit22@melissa:/etc/cron.d$ cat cronjob_bandit23
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null
bandit22@melissa:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget

#Run the script:
bandit22@melissa:/etc/cron.d$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
********************************

#Check contents of that file in /tmp:
bandit22@melissa:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
********************************
{% endhighlight %}

<b>23 --> 24</b>
<p>Fistly let’s edit the script by adding following lines of code</p>
{% highlight bash %}
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/w3ndige
{% endhighlight %}
<p>to the cronjob_bandit24.sh</p>
{% highlight bash %}
bandit23@melinda:~$ cd /etc/cron.d
bandit23@melinda:/etc/cron.d$ ls
behemoth4_cleanup  cronjob_bandit24       melinda-stats          natas26_cleanup  semtex0-64   vortex0
cron-apt           cronjob_bandit24_root  natas-session-toucher  natas27_cleanup  semtex0-ppc  vortex20
cronjob_bandit22   leviathan5_cleanup     natas-stats            php5             semtex5
cronjob_bandit23   manpage3_resetpw_job   natas25_cleanup        semtex0-32       sysstat
bandit23@melinda:/etc/cron.d$ cat cronjob_bandit24
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        timeout -s 9 60 "./$i"
        rm -f "./$i"
    fi
done


bandit23@melinda:/etc/cron.d$ cd /tmp
bandit23@melinda:/tmp$ vim task.sh
bandit23@melinda:/tmp$ cat tash.sh
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/w3ndige
bandit23@melinda:/tmp$ ls -l tash.sh
-rw-rw-r-- 1 bandit23 bandit23 53 May 12 22:00 tash.sh
bandit23@melinda:/tmp$ chmod 777 tash.sh
bandit23@melinda:/tmp$ cp tash.sh /var/spool/bandit24/
bandit23@melinda:/tmp$ cat w3ndige
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ
{% endhighlight %}

<b>24 --> 25</b>
<p>More fun! We have to create a brute forcing script that uses numeral pins from 0 to 10000. I'm gonna use Python but you can use any script you can!</p>
{% highlight python %}
import socket
pin = 0
bandit = 'UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ'
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 30002))
client.recv(1024)
while pin in range(0, 10000):
   message = bandit + ' ' + ' ' + str(pin) + '\n'
   print 'Trying: ' + message
   client.sendall(message)
   data = client.recv(1024)
   print data
   pin += 1
{% endhighlight %}
<p>After executing it goes through all combinations, where  pin 5669 is the correct one.</p>
{% highlight bash %}

Trying: ******************************** 5668

Wrong! Please enter the correct pincode. Try again.

Trying: ******************************** 5669

Correct!
The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG


Trying: ******************************** 5670

Exiting.

Trying: ******************************** 5671

Traceback (most recent call last):
File "brute.py", line 11, in &amp;lt;module&amp;gt;
data = client.recv(1024)
socket.error: [Errno 104] Connection reset by peer
bandit24@melinda:/tmp$
{% endhighlight %}

<b>25 --> 26</b>
<p>Here is the last one! We are going to log in using sshkey like in the bandit14 or bandit 17.  By resizing the window to the minimum we are able to view the text, the program will not stop working.</p>
{% highlight bash %}
bandit25@melinda:~$ ls
bandit26.sshkey
bandit25@melinda:~$ ssh -i bandit26.sshkey bandit26@localhost

Connection to localhost closed.

bandit25@melinda:~$ cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
bandit25@melinda:~$ cat /usr/bin/showtext
#!/bin/sh
more ~/text.txt
exit 0

bandit25@melinda:~$ ssh -i bandit26.sshkey bandit26@localhost
{% endhighlight %}
<p>Now before executing the command, window should be small enough to fit about 5 lines of code.</p>
<p>Next ssh to the localhost.</p>
<p>Type ‘v’ to enter vim.</p>
<p>Type ‘:r /etc/bandit_pass/bandit26’ to view the password.</p>

<p>It was the great challenge, useful for learing some Linux commands, simple socket networking and SETUID binaries.</p>
<p>Thanks for the Overthewire team for creating this one!</p>
