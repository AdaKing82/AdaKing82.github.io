---
layout:     post
title:      Is JVM platform independent or dependent?
subtitle:   Maybe we are all misleaded by some java books
date:       2025-12-16
author:     Brother King
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - JVM
---
# Java Virtual Machine (JVM) is platform independent? or dependent?

## The Conceptual Model of Computer

Actually, the computer is a device that implements the Turing model, which contains input parameters, a gadget that computes accroding to some rule, and the output result. 

![](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/turning.jpg)

Tape is used for inputing the data, program is the instructions that applies to compute the data, the cubes horizontal arranging is the current status of the machine.

At present, the digital computers are all the products implementing the abstract model with different phisycal forms.

For example, the addition algorithm below

```bash
c = a + b
```

a and b are both the input parameters, the c is the output of the computerization.

## How to implement the calculation?

If we design a computer, there is a CPU with summing unit inside it, how to implement the adding calculation and outputs the result?  Maybe, a scheme can be designed as below

- A memory
- A CPU
- Three instructions: add, read memory, write memory

![image-20251220130122098](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/concept_computer_eng.png)

How to implement the addtion program above? Maybe we can adopt the procedure below

内存：memory

高地址：high memory address

低地址：low memory address

执行加法： execute the adding instruction

![](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/stack_computer_adding.png)

Step 1: put a into the memory

Step 2: push the b into stack

Step 3:  retrieve the a and b from the memory into the CPU, and execute the adding program

Step 4: clear the  a and b in the memory, and then puts the c back into the memory

This type of computer is called stack computer. By this concept, the computer can not only execute summarization, but also be used to applying subtraction, multiplication, and division, etc.

the merits of this computer is the instructions defined by the designers are compact and streamlined, all the elements manipulated by the instructions are arranged on the stack.

However, there are some intrinsic defects, such as the low efficiency, reading and writing from the memory to CPU is time consuming, and the limitation of memory address accessement, etc.

## Improvement

For accelerating the computing procedures, a register, which residents in the CPU, can be used to always keep the value of top element of the stack. The process is below.

![](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/evolution.png)

As the register is internally located in CPU, the accessing speed is faster than the R/W from memory.

Following the development of the technology, the number of registers, which can be puts in the CPU, has become increased. Different manufacturers develop different numbers of registers for various purposes according to the original architecture usages. Taking x86 as an example, (the AMD and Intel CPU as we always hear from advertisements), there are 32 general pupose registers, and each of them is marked with a distinct number and alias. The table below describes the top 3 registers.

| No.  | Alias |
| ---- | ----- |
| 0    | rax   |
| 1    | rcx   |
| 2    | rdx   |


As a result, the form of CPU is developed to the form below.

![image-20251220132422231](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/von_eng.png)

## The architecture types

For the register implementations and memory-accessing patterns are various, Complex Instructions Set Computer (CISC) and Reduced Instruction Set Computer were evolved and be adopted by different companies.  For instance, the adding instruction, which is defined in x86 (one of the CISC), is below.

```assembly
ADD EAX, EBX
```

This instruction is to calculate the addition of the values in the `EAX` and `EBX`, and put the result to `EBX`


Comparing to expression above, the classic ARM instruction (one of the RISC instructions) implements the addition is below.

```bash
ADD X0, X1, X2
```

the procedure is to add the value from `x1` and `x2`, and exports the result to `x0`.

Actuall, no only are the form of expressions different, but the machine code translated from the compiler are also various. For x86, the machine code of adding is below.

```bash
0x01C3
```

However, the machine code compiled in ARM is obviously different.

```bash
0x8B000000
```

## The dependence in platform

Apparently, the machine code from x86 and `ARM` is totally different, despite they do the same summing calculation.  In that case, if a program is planed to calculate an addition, maybe the final instruction executed in the CPU is quite various.

Take an adding calculation by C language as an example.

```C
int a,b,c;
a = b + c;
```

After the source code compiled by the gcc, which is one of the C compilers, the source file is generated into an executable file containing the binary codes only recognized by the computer. If you dig into it, the instruction for adding is different. Therefore, although the content of the high-level language is identical, the executable program becomes platform independent during the final step of execution. 

## Why Java is platform independent?

What we know about the platform independence from Java refers to the content of .class file generated from Java Virtual Machien (JVM) is same. In other words, no matter what the compiling computer is, the content of the target file is same. For example, the .class file generated by the `x86`, can be executed in `ARM`. Consequently, any computer, just installed  JVM, can normally executes the Java program according to the .class file. It is markedly different to the C/C++ file compiled by the gcc since the binary code file must be directly fullfill the requirement of the platform.

### Reason

Java virtual machine (JVM) firstly reads the .class files at beginning, and then translate them and loads the content to various parts to run. Now, why the JVM is called virtual machine? Because the internal mechanism of the JVM is a software implmented computer by a stack calculating form. the computer is called **hotspot**, which is almost written by C++. A shell written in Java(JDK) is wrapped around hotspot, providing a great number of interfaces to developers to call. 

The <font color="brown">orange part</font> is related to platform-independent.

The <font color="green">green part</font> is linked to platform-dependent.

![JVM_Arch](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/VM_Arch.png)

In essence, for the software aspect, all the Java programs are executed by a custom stack computer, yet, the actual computation on a stack-based machine is ultimately executed through platform's native instructions.

That is the reason why there are many different folders with architectural name in the source code.

![1744879997625](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/cpus.png)

Moreover, to implment "**once compiled, executed any where**", the JVM provides some distinct variables and functions accoriding to the operating system(OS), even with its CPU characteristics.

![1744880101657](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-16/os_cpus.png)

## Summary

Java virtual machine stems from its execution via software-implemented, stack-based virtual machine. Its platform dependence, however, is rooted in the fact that JVM own operations rely on platform-specific instructions and the underlying operations.