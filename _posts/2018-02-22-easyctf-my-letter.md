---
layout:     post
title:      "EasyCTF 2018 - My Letter"
date:       2018-02-22 2:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

This time we're walking through the challenge called ***My Letter*** from the forenscics category.

- Category: Forensics
- Points: 80

{% highlight text %}
I got a letter in my email the other day... It makes me feel sad, but maybe it'll make you glad. :( file
{% endhighlight %}

You can find full problem info [here](https://github.com/EasyCTF/easyctf-iv-problems/tree/master/my_letter), together with the original file.

### Solution

Firstly, let's view the file with a docx viewer like `Libre Office Writer`.

![My Letter](/img/easyctf/my_letter.png){:class="img-responsive center-block"}

As you can see, some of the letters are bolded. Let's delete the rest and see what we'll get.

![Rick Roll](/img/easyctf/rick_roll.png){:class="img-responsive center-block"}

Just a troll :D

But let's remember that docx is actually made of a zip, so let's change the into `my_letter.zip` and unpack it. Maybe its content will show us the flag?

{% highlight bash %}
main:~/projects/easyctf/word > ls -la
razem 76
drwxr-xr-x  5 w3ndige users  4096 02-22 11:25  .
drwxr-xr-x 11 w3ndige users  4096 02-22 11:25  ..
-rw-r--r--  1 w3ndige users  1427 02-10 21:17 '[Content_Types].xml'
drwx------  2 w3ndige users  4096 02-22 11:25  docProps
-rw-r--r--  1 w3ndige users 36679 02-11 12:09  letter.zip
drwx------  2 w3ndige users  4096 02-22 11:25  _rels
-rw-r--r--  1 w3ndige users 15623 02-10 21:13  template.png
drwx------  5 w3ndige users  4096 02-22 11:25  word
{% endhighlight %}

An image? Let's take a look.

![Template](/img/easyctf/template.png){:class="img-responsive center-block"}

### Tools
`Libre Office Writer`

### References
[http://www.forensicswiki.org/wiki/Word_Document_(DOCX)](http://www.forensicswiki.org/wiki/Word_Document_(DOCX))
