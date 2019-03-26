---
layout:     post
title:      'Securinets Prequals 2019 - AutomateMe'
date:       2019-03-24 0:00:00
author:     'W3ndige'
permalink: /:title/
category: 'Securinets Prequals 2019'
---

Altough I'm not sure if that was part of intended solutions, I've decided to write up cool little challenge from **Securinets Prequals**, called **AutomateMe**. Let's take a look at how some tools can speed up your process of reverse engineering. 

In this task you're given a binary containg a huge amount of compare statements. 

```text
w3ndige@main ~/D/c/s/automate-me> rabin2 -I bin
arch     x86
baddr    0x0
binsz    170368
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
pic      true
relocs   true
relro    full
rpath    NONE
static   false
stripped false
subsys   linux
va       true
```

We can analyze it with `radare2` to get a better overview of how these compares work. Let's take a look at this disassembled sample. 

```text
| 0x00000786      3c68           cmp al, 0x68                          ; 'h'
| ; DATA XREF from main (+0x173d3)
| 0x00000788      7416           je 0x7a0
| ----------- true: 0x000007a0  false: 0x0000078a
| ; DATA XREF from main (+0x1742f)
| 0x0000078a      488d3de57c02.  lea rdi, str.nope_:                   ; 0x28476 ; "nope :( "
| ; DATA XREF from main (+0x1758d)
| 0x00000791      b800000000     mov eax, 0
| ; DATA XREF from main (+0x17681)
| 0x00000796      e815feffff     call sym.imp.printf                   ; int printf(const char *format)
| ; DATA XREF from main (+0x1776e)
| 0x0000079b      e9007c0200     jmp 0x283a0
````

At address `0x00000786` we have one of these instructions, which will compare a byte from the user input with the value `0x68`, which in ASCII is represented by letter `H`. If the value is equal, we move to the next compare block. It may seem like we can just extract the values from all `cmp` used in `main` to get the full message that will pass the comparisions. 

```text
| ----------- true: 0x000283a0
| ; CODE XREF from main (0x788)
| ; DATA XREF from main (+0x17869)
| 0x000007a0      488b45e0       mov rax, qword [var_20h]
| ; DATA XREF from main (+0x17928)
| 0x000007a4      4883c008       add rax, 8
| ; DATA XREF from main (+0x179e7)
| 0x000007a8      488b00         mov rax, qword [rax]
| ; DATA XREF from main (+0x17a86)
| 0x000007ab      0fb64002       movzx eax, byte [rax + 2]             ; [0x2:1]=76
| ; DATA XREF from main (+0x17b4c)
| 0x000007af      8845ff         mov byte [var_1h], al
| ; DATA XREF from main (+0x17bdd)
| 0x000007b2      8075ffeb       xor byte [var_1h], 0xeb
| ; DATA XREF from main (+0x17ca3)
| 0x000007b6      807dff8e       cmp byte [var_1h], 0x8e
| ; DATA XREF from main (+0x17d70)
| 0x000007ba      7416           je 0x7d2
```

But if you look at the next compare, you can see that there things get a little bit more complicated. The value from `al` register is moved into `[var_1h]` variable, then the variable is xored with `0xeb` and after that, we get another `cmp` of our variable and another immediate value. Of course it does not mean that we cannot recover the value of character which will pass the `cmp`, we can  xor vale of `0xeb` and `0x8e` to get the desired character. In this example, it would be an letter `e`. 

```python
>>> chr(0xeb ^ 0x8e)
'e'
``` 

At that moment I've started to think about writing a `Python` script that would operate on disassembly and gather all the values that would pass the comparisions. But I've kept in my mind that I haven't used newly released `ghidra` tool yet, so I've quickly jumped into it. 

![Ghidra View](/img/securinets-prequals/ghidra-view.png){:class="img-responsive center-block"}

If you look at the decompiler window, you can see that `ghidra` easily decompiled all of these into a nice pseudocode, containing already calculated values of every character.

Let's copy all of that output into a text file, write a nice regexp and extract the values from withing `''` characters - `\'(.*?)\'`. After that we just need the delete leftover characters, replace `\n` with blank characters and here we have an input. All of this can be done with `VS Code` in one minute. 

```
w3ndige@main ~/D/c/s/automate-me> cat flag.txt 
There are many reasons for performing reverse engineering in various fields. Reverse engineering has its origins in the analysis of hardware for commercial or military advantage. However, the reverse engineering process in itself is not concerned with creating a copy or changing the artifact in some way; it is only an analysis in order to deduce design features from products with little or no additional knowledge about the procedures involved in their original production. In some cases, the goal of the reverse engineering process can simply be a redocumentation of legacy systems. Even when the product reverse engineered is that of a competitor, the goal may not be to copy them, but to perform competitor analysis. Reverse engineering may also be used to create interoperable products and despite some narrowly tailored United States and European Union legislation, the legality of using specific reverse engineering techniques for this purpose has been hotly contested in courts worldwide for more than two decades.before getting into important things ; here is you flag securinets{automating_everything_is_the_new_future}.First of all, I feel it is important to point out that Reverse Engineering Automation saves investigation time but does not replace t\the rest of the process, and secondly, for reasons including \"Repeated actions\" and \"Dynamic code fragments that are detected at a later stage of the program (Crypters, Packers).\" and \"Bypassing software protections, such as SSDT.\" and \"Allocating memory of the software running within Ollydbg\" therefore we made this task to urge people to think about automation and be creative ;Software reverse engineering can help to improve the understanding of the underlying source code for the maintenance and improvement of the software, relevant information can be extracted in order to make a decision for software development and graphical representations of the code can provide alternate views regarding the source code, which can help to detect and fix a software bug or vulnerability. Frequently, as some software develops, its design information and improvements are often lost over time, but this lost information can usually be recovered with reverse engineering. This process can also help to cut down the time required to understand the source code, reducing the overall cost of the software development. Reverse engineering can also help to detect and eliminate a malicious code written to the software with better code detectors. Reversing a source code can be used to find alternate uses of the source code, such as to detect unauthorized replication of the source code where it wasn\t intended to be used, or to reveal how a competitors product was built. This process is commonly used for \"cracking\" software and media to remove their copy protection, or to create a (possibly improved) copy or even a knockoff, which is usually the goal of a competitor or a hacker.Malware developers often use reverse engineering techniques to find vulnerabilities in an operating system (OS), in order build a computer virus that can exploit the system vulnerabilities. Reverse engineering is also being used in cryptanalysis in order to find vulnerabilities in substitution cipher, symmetric-key algorithm or public-key cryptography.
```

Flag is withing this block of text.

```
securinets{automating_everything_is_the_new_future}
```