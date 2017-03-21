---
layout:     post
title:      "Vulnhub.com - Mr.Robot 1"
subtitle:   "Write-Up"
date:       2016-07-30 12:00:00
author:     "W3ndige"
header-img: "img/mr-robot-1-header.jpg"
category: Vulnhub
---
<h1>Introduction</h1>
<p>This VM is a great way to celebrate the upcoming of a season 2 of Mr. Robot. As it's designed for beginners-intermediate it will be fun challenge and opportunity to learn something new. Our task is to find three different keys located somewhere in the machine. </p>
<p>Let's get started!</p>
<h1>How to join the #fsociety?</h1>
<p>Firstly, let's find out what IP address is assigned to this VM using nmap, in addition we can scan for the OS version of this machine, and are there any open ports.</p>
{% highlight bash %}
nmap -Pn -A 10.0.2.0/24
{% endhighlight %}
<p>-Pn – will scan hosts even if they ignore us</p>
<p>-A – will determine operating system of the host</p>
![mr-robot-nmap](/img/mr-robot-1/mr-robot-nmap.png){:class="img-responsive"}
<p>From what we can see, IP of the machine is 10.0.2.4 and has two open ports: port 80 (http) and port 443 (https). What that means, is that it's hosting web site, using Apache web server. Let's check what it is!</p>
<p>After typing the IP address to the web browser, we can see well known interactive commercial for the new season.</p>
![mr-robot-view](/img/mr-robot-1/mr-robot-web-view.png){:class="img-responsive"}
<p>Always remember to check the robots.txt file as it may be hiding some useful information, which may come handy for us.</p>
<p>In this one we've got a rule dissalowing web crawlers from indexing 2 files: key-1-of-3.txt and fsocity.dic.</p>
![mr-robot-robotstxt](/img/mr-robot-1/mr-robot-robotstxt.png){:class="img-responsive"}
<p>We can get these files by using wget commands (very useful one!).</p>
{% highlight bash %}
wget 10.0.2.4/key-1-of-3.txt
{% endhighlight %}
{% highlight bash %}
wget 10.0.2.4/fsocity.dic
{% endhighlight %}
<p>Yeah! We've got the first key, two more to go!</p>
<p>Second file is a dictionary, and I assume we will have to use it in the future brute force attack.</p>
![mr-robot-filesfromrobots](/img/mr-robot-1/mr-robot-filesfromrobots.png){:class="img-responsive"}
<p>Now let's use nmap script - http-enum which may reveal some more useful information.</p>
![mr-robot-httpenum](/img/mr-robot-1/mr-robot-http-enum.png){:class="img-responsive"}
<p>Script showed us a lot more that we've asked for. It's based on WordPress, and we may use this information to preform many attacks. Now let's focus on the readme file and the wp-login.php site.</p>
<p>Readme won't help us :(</p>
![mr-robot-readme](/img/mr-robot-1/mr-robot-readme.png){:class="img-responsive"}
<p>But will login page help us?</p>
<p>Most common login and passwords pairs like admin:admin haven't worked but remeber that we have the dictionary, to perform an attack. Now, we have to find the username. </p>
<p>And actually, elliot is correct username! I tried it, by simply typing the names of the characters in this show :)</p>
![mr-robot-elliot](/img/mr-robot-1/mr-robot-elliot.png){:class="img-responsive"}
<p>Our next task is to brute force this page, using provided dictionary and the wpscan tool.</p>
{% highlight bash %}
wpscan -u 10.0.2.4 --wordlist ~/fsocity.dic --username elliot
{% endhighlight %}
<p>And after roughly 5 hours, password is ER28-0652 - one of the last ones in the word list.</p>
![mr-robot-werein](/img/mr-robot-1/mr-robot-werein.png){:class="img-responsive"}
<p>But what to do next?</p>
<p>We can try to upload a reverse shell to gain access to the server.</p>
<p>Simplest way would be to edit some .php file in order to get shell. I’ll try with page.php and use the code from:</p> [PenTestMonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell "PenTestMonkey")
<p>Then let’s create blank page.</p>
<p>Last essential step would be setting up listener and after that just the visit our blank page.</p>
{% highlight bash %}
nc -lvp 3344
{% endhighlight %}
<p>And we’re in the server, as elliot user. Now let’s see if there’s anything interesting in the home directory.Most promising folder was ‘robot’ containing 2 files: key-2-of-2.txt and password.raw-md5</p>
<p>Unfortunately we don’t have permission to cat the .txt file but let’s look at the m5 file which gives us: robot:<b>c3fcd3d76192e4007dfb496cca67e13b</b> – probable login and password pair . Cracking it with CrackStation, gives us <b>abcdefghijklmnopqrstuvwxyz</b>. In the next step let's try to login in Mr.Robot vulnerable machine.</p>
![mr-robot-robot](/img/mr-robot-1/mr-robot-robot.png){:class="img-responsive"}
<p>After that we're able to view the second key, one more to go!</p>
![mr-robot-key2of2](/img/mr-robot-1/mr-robot-key-2-of-2.png){:class="img-responsive"}
<p><b>822c73956184f694993bede3eb39f959</b></p>
<p>One last part - privilage escalation - we have to get to the root account.</p>
<p>We can find any misconfigured executable files that can provide us what we want.</p>
{% highlight bash %}
find / -perm -u=s -type f 2>/dev/null
{% endhighlight %}
![mr-robot-nmapstart](/img/mr-robot-1/mr-robot-nmapstart.png){:class="img-responsive"}
<p>Which gives information that nmap is installed on this server with root privileges. Let’s try to exploit nmap –interactive.</p>
![mr-robot-key3of3](/img/mr-robot-1/mr-robot-key-3-of-3.png){:class="img-responsive"}
<p>And yes, we've got the last key!</p>
<p>It was great challenge to fulfill my knowledge hungry brain :). Thanks for Jason for creating such a great boot2root machine!</p>
[Jason](https://www.vulnhub.com/author/jason,292/ "Jason")

<p>~Stay safe!</p>
