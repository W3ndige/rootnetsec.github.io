---
layout:     post
title:      "SharifCTF 2018 - hidden"
date:       2018-02-03 2:00:00
author:     "W3ndige"
permalink: /:title/
category: 'Write-Ups'
---

Another challenge from [SharifCTF](http://ctf.certcc.ir/ctf8/) called ***hidden***.

- Category: Forensics
- Points: 100

{% highlight text %}
Find the hidden process.

The flag is SharifCTF{MD5(Process id)}.
{% endhighlight %}

### Solution

First inspections using the `file` command does not reveal a lot, but we seem to get what's the content of the dump with the `strings` tool.

{% highlight bash %}
main:~/projects/sharictf > file dump
dump: data
main:~/projects/sharictf > strings dump
...
Microsoft DH SChannel Cryptographic Provider
dhTl
vPrivateKeyInfoEncode
IPSec Policy agent endpoint
IPSec Policy agent endpoint
IPSec Policy agent endpoint
IPSec Policy agent endpoint
IPSec Policy agent endpoint
\PIPE\lsass
\\SECURE-CBE6C864
...
{% endhighlight %}

As we know that's it's the memory dump, I decided to run this awesome linux tool for memory dump analysis called ***Volatility***. Firstly I'll run it with `imageinfo` option that will show us basic information about the dump.

{% highlight bash %}
main:~/projects/sharictf > volatility imageinfo -f dump
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/w3ndige/projects/sharictf/dump)
                      PAE type : PAE
                           DTB : 0x359000L
                          KDBG : 0x80545c60L
          Number of Processors : 1
     Image Type (Service Pack) : 3
                KPCR for CPU 0 : 0xffdff000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2018-01-28 17:35:20 UTC+0000
     Image local date and time : 2018-01-28 21:05:20 +0330

{% endhighlight %}

From that information, the most essential one is the suggested profile `WinXPSP2x86`, we'll use it in later analysis. As we have to get the PID of the hidden process, I decided to look into the documentation of this tool and here it is - `psxview`.

{% highlight text %}
This plugin helps you detect hidden processes by comparing what PsActiveProcessHead contains with what is reported by various other sources of process listings. It compares the following:
{% endhighlight %}

Let's run the analysis.

{% highlight bash %}
main:~/projects/sharictf > volatility --profile=WinXPSP2x86 psxview -f dump
Volatility Foundation Volatility Framework 2.6
Offset(P)  Name                    PID pslist psscan thrdproc pspcid csrss session deskthrd ExitTime
---------- -------------------- ------ ------ ------ -------- ------ ----- ------- -------- --------
0x010eb4c0 rundll32.exe            396 True   True   False    True   True  True    True     
0x01c279c0 svchost.exe             900 True   True   False    True   True  True    True     
0x01e64350 vmtoolsd.exe            404 False  True   False    True   True  True    True     
0x025b7020 explorer.exe           1576 True   True   False    True   True  True    True     
0x01e6d608 winlogon.exe            644 True   True   False    True   True  True    True     
0x01ecd378 svchost.exe             988 True   True   False    True   True  True    True     
0x031b1cf0 spoolsv.exe            1508 True   True   False    True   True  True    True     
0x01fbe410 lsass.exe               700 True   True   False    True   True  True    True     
0x0096c0e8 wscntfy.exe             920 True   True   False    True   True  True    True     
0x039347a8 svchost.exe            1188 True   True   False    True   True  True    True     
0x0308d9f0 svchost.exe            1604 True   True   False    True   True  True    True     
0x01c58798 vmacthlp.exe            856 True   True   False    True   True  True    True     
0x01de4878 svchost.exe            1236 True   True   False    True   True  True    True     
0x01bbd488 services.exe            688 True   True   False    True   True  True    True     
0x01fbd6e0 svchost.exe            1024 True   True   False    True   True  True    True     
0x02e7eb20 svchost.exe            1692 True   True   False    True   True  True    True     
0x01209a00 System                    4 True   True   False    True   False False   False    
0x01bbd900 smss.exe                548 True   True   False    True   False False   False    
0x021a7da0 csrss.exe               620 True   True   False    True   False True    True     
0x02dbb448 wmiprvse.exe            908 False  True   False    False  False False   False    2018-01-28 17:34:22 UTC+0000
0x01ebe168 cmd.exe                1704 False  True   False    False  False False   False    2018-01-28 17:34:00 UTC+0000
{% endhighlight %}

Now we can see that the only process with false value is `vmtoolsd.exe` with PID `404`. The last step is to calculate MD5 hash out of that PID and submit the flag.

`SharifCTF{4f4adcbf8c6f66dcfc8a3282ac2bf10a}`

### Tools
[Volatility](https://github.com/volatilityfoundation/volatility)

### References
[https://github.com/volatilityfoundation/volatility/wiki/Command-Reference-Mal#psxview](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference-Mal#psxview)

[https://www.youtube.com/watch?v=ceXT9fBGJaI](https://www.youtube.com/watch?v=ceXT9fBGJaI)
