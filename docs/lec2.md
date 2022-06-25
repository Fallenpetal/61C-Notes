---
layout: default
title: C Intro - Basics
author: liuxing
nav_order: 2
---

# C Intro - Basics
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Compilation:Overview

C compliers将 C程序 直接映射为特定架构的机器码(一系列的0和1)

the processor直接执行机器码

- 与Java不同，Java是由虚拟机(Virtual machine)解释(interpret)，将程序转换为独立于架构的字节码(bytecode),或者由just-in-time(JIT)转换为机器码(machine code)
- 与Python不同，Python是运行时转换为字节码(bytecode)

对于C,一般来说是两个处理步骤，一是编译器将source files(.c后缀)编译为object files(.o后缀)，然后Linker将 .o files链接成为可执行文件

![image-1](https://raw.githubusercontent.com/Fallenpetal/Pictures-Library/master/image-20220223210055062.png)

### 优点

**优秀的运行时间表现** : 一般来说比Python或Java快得多

**合理的编译时间 : **只有被修改的文件才会被重新编译

### 缺点:

- 已编译的文件，包括可执行文件，是适用于特定的体系结构，依赖处理器的类型和操作系统
- 每在新的系统上，可执行文件需要进行重新构建

------

## C Pre-Processor(CPP)

C source files(源文件)在交给编译器之前，是先由CPP进行预处理的

![image-20220223211203652](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223211203652.png?raw=true)

具体进行的处理为：

- #include "file.h"  将整个file.h文件引入
- #include<stdio.h> （不太理解）Looks for file in standard location
- define PI 3.14  进行文本替换，PI全部替换为3.14
- 程序的注释被全部替换为空格
- 添加命令option -save-temps 可以查看预处理结果

更多可见http://gcc.gnu.org/onlinedocs/cpp/

-----

## C 整数和特定bit长度的数字

![image-20220223212859497](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223212859497.png?raw=true)

![image-20220223212926624](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223212926624.png?raw=trueg)

---

## C的Memory模型

我们视C的Memory为一个单一的巨型数组

![image-20220223213144828](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223213144828.png?raw=true)

![image-20220223213201965](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223213201965.png?raw=true)

---

## Pointer 语法

指针是一个变量，储存特定类型的另一个变量的地址，例如x的值是23，但是x的地址是104，则指针p=104

![image-20220223213441982](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223213441982.png?raw=true)



![image-20220223213258778](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220223213258778.png?raw=true)

### undefined behavior(未定义的行为)

- 声明一个指针只会分配一个存储指针的空间，并不会给指针分配将要指向的地址空间
- C中声明一个变量并不会初始化，谁也不知道它里面的值会随机成为what

```c
void f() 
{ 
 int *ptr; 
 *ptr = 5; 
}
```

例如上面，声明了一个指针ptr，并未初始化，因此ptr随意指向程序中某一个变量的地址，然后将该变量赋值为5

谁也不能保证这是否对程序有所影响，该指针随机指向的地址所对应的值被5覆盖，如果该值本身就是5，那还好，否则可能成为使程序崩溃的致命因素

