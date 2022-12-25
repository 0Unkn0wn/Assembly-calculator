# Assembly calculator<!-- omit in toc -->

![logo](logo.jpg)

In this tutorial I will cover the basics of assembly on Linux x86-64 and how to create a simple calculator app.

- [Getting started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installing the build tools](#installing-the-build-tools)
  - [Installing NASM](#installing-nasm)
  - [Installing SASM](#installing-sasm)
- [Assembly basics](#assembly-basics)
  - [Basic instructions](#basic-instructions)
  - [System V calling convention](#system-v-calling-convention)
  - [Hello World](#hello-world)
- [Simple Assembly Calculator](#simple-assembly-calculator)
  - [Calculator functions](#calculator-functions)
    - [Readoperator](#readoperator)
    - [Compute](#compute)

## Getting started

### Prerequisites

For this project Ubuntu or any other Linux distribution should be installed on a VM or directly on the system. This tutorial will focus on Ubuntu set up but most packages should be the same regardless of the distribution.

### Installing the build tools

The quickest way to get started with assembly is to get the essential development and build tools.

So, open a terminal and install the build-essential package :

```bash
 sudo apt install build-essential
```

Now to verify that everything installed correctly we can check by running the following commands one at a time :

```bash
 make --version
GNU Make 4.3
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later http://gnu.org/licenses/gpl.html
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

```bash
 gcc --version
gcc (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

```bash
 g++ --version
g++ (Ubuntu 11.3.0-1ubuntu1~22.04) 11.3.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### Installing NASM

Netwide Assembler (NASM) is an assembler for the x86 CPU architecture that works on every modern platform. We are going to use it in our programs because of its stability. YASM assembler is a newer alternative that accepts NASM syntax.

To install NASM run the following command :

```bash
 sudo apt install nasm
```

After that we need to verify the installation. We could do so with the following command :

```bash
 nasm --version
NASM version 2.15.05
```

### Installing SASM

While we could compile code using the terminal, using an ide could help speed up the process by quite a bit. That`s what SASM is, an ide for NASM, MASM, GAS, FASM assembly languages. SASM has syntax highlighting and debugger which will help us during our assembly journey.

To install it run the following command :

```bash
 sudo apt install sasm
```

Or it could be downloaded from : <https://dman95.github.io/SASM/english.html>.

## Assembly basics

### Basic instructions

These instruction are the most important and used ones.

1. `mov destination, source` to copy contents
2. `add destination, operand, sub destination, operand` to add/subtract values
3. `lea destination, [address]` to load address of a memory location
4. `call function, ret` to call a function or return from one
5. `cmp op1, op2` to compare two values
6. `jmp label` to jump unconditionally
7. `jCC label` to jump conditionally depending on `CC` :
                `jnz` is jump if not zero
                `jz` is jump if zero
                `jg` is jump if greater
                `jl` is jump if less
8. Assembly code is case insensitive so you can use either form: `RAX` or `rax`, `CMP` or `cmp`, etc
9. `[variable]` refers to the value of that variable (dereferences the pointer named variable)

A complete list of all available instructions can be found at : <https://www.felixcloutier.com/x86/>.

There are 5 main types of variables :

```text
res/db – reserve/define byte (8 bits)
res/dw – reserve/define word (16 bits)
res/dd – reserve/define doubleword (32 bits)
res/dq – reserve/define quadword (64 bits)
res/dt - reserve/define tenword (80 bits)
```

### System V calling convention

The registers `RAX, RDI, RSI, RDX, RCX, R8, R9, R10, R11` are volatile.

The registers `RBX, RSP, RBP, R12, R13, R14, R15` are nonvolatile.

The registers `XMM0-XMM7` are used for floating point numbers.

No need for any shadow area unlike in MASM (Microsoft Macro Assembler) just aligning the stack to 16 bits is required.

Parameters to functions are passed in the registers `rdi, rsi, rdx, rcx, r8, r9`, and further values are passed on the stack in reverse order.

The return value is stored in the `rax` register, or if it is a 128-bit value, then the higher 64-bits go in `rdx`.

More info can be found at : <https://wiki.osdev.org/System_V_ABI>.

### Hello World

This is how a classic Hello World program would look like in assembly using NASM syntax :

```nasm
default rel

global  main

extern  printf

section .data
    format      db 'Hello world!', 0xA, 0

section .bss

section .text
    main:
        sub     rsp, 8

        lea     rdi, [format]
        mov     al, 0
        call    printf wrt ..plt

        add     rsp, 8
        xor     rax, rax
        ret
```

Above everything is where we declare external functions and the global main label

The program is structured in 3 parts :

1. Section .data : this is where initialized goes
2. Section .bss : this is where uninitialized data goes
3. Section .text : this is where the code goes

When starting to write a program we assume that the stack is aligned to 16 bits. Before calling a function we need to make sure the stack will be aligned we do it by writing `sub rsp, 8`. After the function we write `add rsp, 8` so the stack is aligned again. Note that if we use `push/pop` prior to calling a function it is possible that the stack might be already aligned.

`default rel` instructs to use relative addressing

 In NASM if you write call `printf wrt ..plt`, it means that the call will actually jump to PLT (Procedure Linkage Table)

`xor     rax, rax` for return 0

## Simple Assembly Calculator

The complete code can be found in the github repository : <www.pune-cacatu-aici.com>

We begin by printing a message in the terminal with all available operations and how the program works.

```asm
    lea rdi , [message]
    sub     rsp, 8
    call puts
```

We continue by reading the first number using `scanf` and store it in the variable called `firstnr`. We pass the format to `scanf` as well so `scanf` knows what type of data to read.

```asm
    lea     rsi, [firstnr]     
    lea     rdi, [number_format]             
    mov     al, 0              
    call    scanf
```

Then we call the function called `readoperator`. The return value will be in `rax` which is a volatile register so we store it in `rbx` which is protected. The next function call will override the value in `rax`.

```asm
    call getchar
    call readoperator
    mov rbx,rax
```

Then we are going to read the second number using `scanf` and store it in the variable called `secondnr`.

```asm
    lea     rsi, [secondnr]    
    lea     rdi, [number_format]         
    mov     al, 0                       
    call    scanf
```

We then use a function named `compute` to calculate the result. This function takes the numbers in `xmm` registers which are specific registers used for floating point numbers and also takes the operand from `rbx`.

```asm
    movsd xmm0,[firstnr]
    movsd xmm1,[secondnr]
    mov rax,rbx
    mov rdi,rax
    call compute    
```

At the end we simply print the result from `rax`.

```asm
    lea     rdi, [number_format]      
    mov     al, 1                  
    call    printf 
```

The whole main is wrapped in a while loop so that multiple calculation can be done one after the other. Pressing q will quit the program, any other input will simply continue the program. This is printed after every calculation.

```asm
    main:
        @while_input:
        ...
        lea rdi,[quit_message]
        call puts
        call getchar;;to clear the buffer
        call getchar
        add rsp,8
        cmp rax,'q'
        je @end_while_input
        jmp @while_input
        @end_while_input:
                
        xor rax,rax        
    ret
```

### Calculator functions

There are 2 function in the program : `compute` and `readoperand`.

#### Readoperator

A number from 1 to 5 has been assigned for every operand currently implemented. The number 0 refers to a incorrect operand. The result of the function will be saved in rax. Other operations can be easily added using the same style.

```asm
readoperator:
    ;function that reads the operator and stores in rax a number which can be interpreted : 
    ;0->invalid operand
    ;1-> + operand
    ;2-> - operand
    ;3-> * operand
    ;4-> / operand
    ;5-> ^ operand
        sub rsp,8
        call getchar;getchar returns in al
        add rsp,8
        cmp al,'+'
            jne @noplus
        mov rax,1
        ret
        @noplus:
        cmp al,'-'
            jne @nominus
        mov rax,2
        ret
        @nominus:
        cmp al,'*'
            jne @nomul
        mov rax,3
        ret
        @nomul:
        cmp al,'/'
            jne @nodiv
        mov rax,4
        ret
        @nodiv:
        cmp al,'^'
            jne @nopower
        mov rax,5
        ret
        @nopower:
        ;other operations can be added here
        ;if the code reaches here it means that an invalid operation has been detected
        mov rax,0;loading 0 in rax to show invalid operator
     ret
```

#### Compute

Based on what number the readoperator returns the function chooses the correct operation. For the basic ones : addition, subtraction, multiplication and division there are already instructions defined for floating point numbers. For the exponentiation operation a custom instruction was needed.

```asm
compute:
        cmp rdi,1
            je @plus
        cmp rdi,2
            je @minus
        cmp rdi,3
            je @multiplication
        cmp rdi,4
            je @division  
        cmp rdi,5
            je @power  
             
        cmp rdi,0
            je @bad
             
        @plus:
            addsd xmm0,xmm1
            ret
        @minus:
            subsd xmm0,xmm1
            ret
        @multiplication:
            mulsd xmm0,xmm1
            ret
        @division:
            divsd xmm0,xmm1
            ret
         @power:
            movsd xmm2,xmm0
            xor r12,r12
            cvtsd2si r12,xmm1
            @while_power:
                cmp r12,1
                    jbe @endpower
                mulsd xmm0,xmm2
                dec r12
                jmp @while_power
            @endpower:
            ret
        @bad:
            xor rax,rax
      ret
```
