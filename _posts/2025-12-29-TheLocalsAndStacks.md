---
layout:     post
title:      The memory layout of locals and stacks in JVM
subtitle:   Explanation of local variables and stacks
date:       2025-12-29
author:     Brother King
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - JVM
---

# The memory layout of locals and stacks in JVM

## Locals and Stacks

Here is a java source code.

```java
public class Add_Sample{
    public int add(int i, int j){
        int k = 100;
        int result = i + j + k;
        return result;
    }
    public static void main(String[] args){
        int result = new Add_Sample().add(10,20);
        System.out.println(result);
    }
}
```

Compile it by javac.

```bash
javac Add_Sample.java
```

Then, inspect the bytecodes refering to by `javap`.

```bash
javap -v Add_Sample
```

Here are the outputs.

```
public int add(int, int);
  Code:
   Stack=2, Locals=5, Args_size=3
   0:   bipush  100
   2:   istore_3
   3:   iload_1
   4:   iload_2
   5:   iadd
   6:   iload_3
   7:   iadd
   8:   istore  4
   10:  iload   4
   12:  ireturn
  LineNumberTable:
   line 3: 0
   line 4: 3
   line 5: 10


public static void main(java.lang.String[]);
  Code:
   Stack=3, Locals=2, Args_size=1
   0:   new     #2; //class Add_Sample
   3:   dup
   4:   invokespecial   #3; //Method "<init>":()V
   7:   bipush  10
   9:   bipush  20
   11:  invokevirtual   #4; //Method add:(II)I
   14:  istore_1
   15:  getstatic       #5; //Field java/lang/System.out:Ljava/io/PrintStream;
   18:  iload_1
   19:  invokevirtual   #6; //Method java/io/PrintStream.println:(I)V
   22:  return
```

## How to find out the depth of the locals and (operand) stacks

The value of `Code` can be seen as below, taking the `add` function as an example.

```
public int add(int, int);
  Code:
   Stack=2, Locals=5, Args_size=3
```

We can see the `Stack=2`, which means the depth of the operand stacks is 2.

`Locals=5` represents the number of the local variables is 5.

`Args_size=3` means the arguments count is 3. Compared to the formal parameters of the function, there is an additional parameter, which is the `this` pointer pointing to the `add` function itself.

![locals_stacks](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-29/locals_stacks_eng.png)

## How to execute the procedure by locals and stacks

Go back the the bytecodes of `add` function, main function transfers the 10 and 20 to the `add` function as parameters. Hence, the top 3 elements of the locals are filled with  values like this.

```assembly
0:   bipush  100
2:   istore_3
3:   iload_1
4:   iload_2
5:   iadd
6:   iload_3
7:   iadd
8:   istore  4
10:  iload   4
```

The execution steps of the bytecodes is below.

![locals_stacks_procedure1](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-29/locals_stacks_procedure1.png)

![locals_stacks_procedure1](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/2025-12-29/locals_stacks_procedure2.png)

At last, you can see the result of the calculation at final step in stacks, which is 130.

## The real memory layout of  Locals and (Operand) Stacks

In `openjdk`, the `rlocals` is arranged at the top of interpreter stack, and the operand stacks position is under the the interpreter stack.

![image-20251229111655030](https://cdn.jsdelivr.net/gh/adaking82/AdaKing82.github.io/images/locals_stacks_real_layout_eng.png)

If the procedure needs the second parameter, what the JVM has to do is getting the value from an address with specific offset by `rlocals` pointing to.

For example, the implementation of `istore` in `x64` is below.

```assembly
locals_index(rbx);//put the index of the bytecodes into the rbx register
__ movl(iaddress(rbx), rax);//assgin the value in rbx into the rax. In x86, the rax is considered as the top of stack by default
```

If a pushing operation is to be executed, what needs to do is the  `sp` decrement by a specific offset and puts the value to the location that the `sp` points to. I do not give an example here. Because the implementation is somewhat complicated with a mechnism called **TOSCA** (top-of-stack caching).

## Summary

Locals (or local variables) includes temporary variables and arguments, and `this` pointer of the current function.

The operating stack completes the calculation by stack in Java virtual machine.

In the `openjdk` interpreter's stack frame layout, local variables are stored at the top. They are accessed by combining `rlocals` register with specific offset. Directly below this area lies the operand stack, where immediate computational values are held. The `sp` register is used to manage the overall growth and shrinkage of this combined stack structure.

