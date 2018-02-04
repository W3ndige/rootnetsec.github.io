---
layout:     post
title:      "SharifCTF 2018 - client01"
date:       2018-02-03 1:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

Here we have a challenge from this year [SharifCTF](http://ctf.certcc.ir/ctf8/) called ***client01***.

- Category: Forensics
- Points: 75

{% highlight text %}
Attached file is the homepage of the client01. He knows the flag.
{% endhighlight %}

### Solution

As the only thing we get is the home page of some user, I decided to look in for some clues in different places.

{% highlight python %}
main:~/projects/sharictf/client01 > ls -la
razem 112
drwxr-xr-x 18 w3ndige users 4096 01-24 06:13 .
drwxr-xr-x  5 w3ndige users 4096 02-03 19:26 ..
-rw-r--r--  1 w3ndige users  220 01-21 08:20 .bash_logout
-rw-r--r--  1 w3ndige users 3526 01-21 08:20 .bashrc
drwx------  6 w3ndige users 4096 01-24 06:10 .cache
drwxr-xr-x  3 w3ndige users 4096 01-24 06:09 .cinnamon
drwx------  9 w3ndige users 4096 01-21 08:33 .config
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Desktop
-rw-r--r--  1 w3ndige users   55 01-21 08:33 .dmrc
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Documents
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Downloads
drwx------  2 w3ndige users 4096 01-21 08:33 .gconf
drwx------  3 w3ndige users 4096 01-21 08:33 .gnupg
-rw-------  1 w3ndige users  632 01-24 06:09 .ICEauthority
drwx------  3 w3ndige users 4096 01-21 08:33 .local
drwx------  4 w3ndige users 4096 01-24 06:10 .mozilla
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Music
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Pictures
-rw-r--r--  1 w3ndige users  675 01-21 08:20 .profile
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Public
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Templates
drwx------  4 w3ndige users 4096 01-21 08:35 .thunderbird
drwxr-xr-x  2 w3ndige users 4096 01-21 08:33 Videos
-rw-------  1 w3ndige users   51 01-24 06:09 .Xauthority
-rw-------  1 w3ndige users 7072 01-24 06:13 .xsession-errors
-rw-------  1 w3ndige users 7046 01-21 08:44 .xsession-errors.old
{% endhighlight %}

As there's nothing exciting in this process, let's skip to the moment where in `.thunderbird/5bd7jhog.default/ImapMail/imap.gmail.com/[Gmail].sbd/` there were still trashed messages talking about uploaded file on filehosting.org

Piece of message from `Kosz` or `Trash`.
{% highlight html %}
<html>
<head>
<title>filehosting.org | free easy unlimited filehosting</title>
</head>
<body style="background-color: #2a2a2a; font: 12px Arial, sans-serif; padding: 10px;">
    <div style="width: 500px; margin: 0 auto;">
        <div style="background-color: #4F4F4F; margin: 7px 0 0 7px;">
            <div style="background-color: #FFFFFF; padding: 10px; position: relative; left: -7px; top: -7px;">
                <h2 style="margin: 0 0 10px; font-weight: bold; font-size: 14px;"><a style="color: #00BB00; text-decoration: none;" href="http://www.filehosting.org">filehosting.org - Download</a></h2>
                <h3 style="margin: 0 0 10px; font-size: 12px; border-bottom: 1px dashed #CCCCCC;">Thank you for using our service.</h3>
                <dl style="line-height: 1.5em; margin: 0;">
                    <dt style="text-decoration: underline; margin: 0 0 5px;">Download</dt>
                    <dd style="margin: 10px 20px 10px; padding: 3px; background-color: #EEEEEE;">
                        Download link for file <b>file</b>:
                        <div style="padding: 5px 0;"><a style="color: #00BB00; text-decoration: none;" href="http://www.filehosting.org/file/details/720884/Ncemd1SxbOVaOrbW/file">http://www.filehosting.org/file/details/720884/Ncemd1SxbOVaOrbW/file</a></div>
                        This download link is valid only once. This file can't be reused after downloading !
                    </dd>
                </dl>
                <div style="border-top: 1px dashed #CCCCCC; margin-top: 20px; padding-top: 3px; font-size: 10px; text-align: center;">
                    filehosting.org - upload, send, share and backup your files completely free!<br>
                    <a style="color: #00BB00; text-decoration: none;" href="http://www.filehosting.org/">upload file</a> &bull; <a style="color: #00BB00; text-decoration: none;" href="http://www.filehosting.org/faq">FAQ</a> &bull; <a style="color: #00BB00; text-decoration: none;" href="http://www.filehosting.org/imprint">imprint</a>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
{% endhighlight %}

After visiting the link we have to provide an email address in order to download the file. Instructions are clear, on the provided email we'll get the download link.

{% highlight bash %}
main:~/projects/sharictf > file file
file: data
{% endhighlight %}

That doesn't tell much. Now I decided to take a look at the file using bless hex editor.

{% highlight text %}
89 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52 00 00 03 E8 00 00 00 C8 04 03 00 00 00 89 C9 D6 7C 00 00 00 1B 50 4C 54 45 00 00 00 FF FF FF 5F 5F 5F 9F 9F 9F BF BF BF DF DF DF 7F 7F 7F 3F 3F 3F 1F 1F 1F AD A0 D6 E1 00 00 00 09 70 48 59 73 00 00 0E C4 00 00 0E C4 01 95 2B 0E 1B 00 00 0E FC 49 44 41 54 78
{% endhighlight %}

If you look closely, you'll see slightly broken ***PNG*** header which is `89 50 4E 47 0D 0A 1A 0A`. Let's modify ours, by adding value `50`, copying the whole hexdump from bless and creating a new file with the copied content.

![Flag](/img/sharifctf/client01-flag.png){:class="img-responsive center-block"}

### Tools
[Bless Hex Editor](https://apps.ubuntu.com/cat/applications/precise/bless/)

### References
[https://www.filesignatures.net/index.php?page=search&search=PNG&mode=EXT](https://www.filesignatures.net/index.php?page=search&search=PNG&mode=EXT)
