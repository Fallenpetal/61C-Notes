---
layout: default
title: Compiler, Assembler, Linker, Loader
author: liuxing
math: mathjax2
nav_order: 9
---

# Compiler, Assembler, Linker, Loader
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Outline

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/QQ%E6%88%AA%E5%9B%BE20220624224853.png?raw=true)

## Compiler 

编译器将高级语言(例如C) 转换为汇编语言(例如RISC-V)，文件后缀发生的变化:

```yaml
foo.c --> foo.s
```

输出的foo.s 文件可以包含伪指令，由下一阶段的assembler 处理

简单介绍一下Complier的流程，61c没细说，应该是在CS164里面详细讲解。

- Lexer
  - Turns the input into "tokens", recognizes problems with the tokens
- Parser
  - Turns the tokens into an "Abstract Syntax Tree", recognizes problems in the  program structure
-  Semantic Analysis and Optimization
  - Checks for semantic errors, may reorganize the code to make it better
- Code generation
  - Output the assembly code

本课介绍的Complier只有这些

---

## Assembler

将 汇编语言(例如foo.s)转换为二进制，真正意义上的Machine Language(foo.o)

```yaml
foo.s --> foo.o
```

这一过程将替换掉伪指令，生成 Object File

### Assembler Directives

经过complier的过程会生成一些assembler directives, 汇编器指令(assembler directives)是指示assembler采取某些操作或更改设置的指令。汇编器指令不代表指令(instructions)，也不会翻译成机器代码。

- **.text**{: .label .label-blue } --  .text directive 告诉 assembler 后面的信息是program text(assembly instructions)，翻译的机器码(machine code)会被写入内存的 text segment

- **.data**{: .label .label-blue } -- .data directive 告诉 assembler 后面的信息是 program data, .data 指令后面的信息将是data value，并将存储在data segment中。

- **.globl symbol**{: .label .label-yellow } -- 声明 symbol 是全局且能够从其他文件引用(referenced)

- **string str**{: .label .label-green } -- 将字符串 str 加上终止符"\0" (null-terminator)存入内存

- **words w1....wn**{: .label .label-red } --  在内存中直接写入w1....wn 

  - `.word`本质上是在内存中创建一些空间来保存数据。它后面的每个项目都是 4 个byte大小。类似于创建数组：

  - assembly:

  - ```yaml
    array:
    .word 0x12121212, 0x23232323, 0x34343434, 0x4, 0x5
    ```

    C 语言:

    ```c
    int array[] = {0x12121212, 0x23232323, 0x34343434, 0x4, 0x5};
    ```

    [see .word .data .text](https://stackoverflow.com/questions/60403872/what-is-word-data-and-text-in-assembly)


[see more detail 1](https://eng.libretexts.org/Bookshelves/Electrical_Engineering/Electronics/Implementing_a_One_Address_CPU_in_Logisim_(Kann)/02%3A_Assembly_Language/2.03%3A_Assembler_Directives#:~:text=Assembler%20directives%20are%20directions%20to,not%20translated%20into%20machine%20code.)

### Producing Machine Language

- Simple Case

对于 算术，逻辑，移位运算的instruction ，所有信息均已包含，直接转换为 machine code 即可，(上次课的 R ,S , I Format等等)

- Branches

因为Branches 是 PC-Relative ，当伪指令被替换为真正的指令时，我们必须知道将要跳转的分支要经过多少条指令

#### Forward Reference

考虑一下:

```yaml
L1: 
	slt t0, x0, a1
 	beq t0, x0, L2
 	addi a1, a1, -1
 	jal x0, L1
L2: add t1, a0, a1
```

我们可以看到，在L1中，对 L2的 beq 指令属于 forward reference,例如在c中:

```c
c = 7;
some code 
...
int c;
```

变量c在未定义之前便使用了，这就相当于forward reference,

在C 中这么做肯定会报错，但是 RISC-V 里面不会，因为assembler 会做 2 pass

#### 2 passes

assembler 会遍历 program 两次，

第一次是找到 labels 的位置，也就是offset，

第二次再将准确的offset 翻译成 machine code

#### Two Problems

- jumps (j and jal)

在文件内jump 是 PC-Relative ，我们可以轻易算出

然而我们不能jump to other files

- references to static data

一般情况下我们使用 `la` 和 `li` 加载地址，二者分别是(lui addi)和(auipc addi)的伪指令

 These will require the full 32-bit address of the data: 

**auipc**{: .label .label-yellow }  - when we include into a relocatable block 

**lui**{: .label .label-yellow }  - when we have an absolute address

为了解决这些problems,we create two tables

### Symbol Table


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220311201811012.png?raw=true)

用于匹配Label 和 .data 以及可能使用的变量 所在内存地址的表

### Relocation Table

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220311201834763.png?raw=true)


• Any external label jumped to: **jal** 

​	• external (including **lib files**) 

• Any piece of **data in static** section 

​	• such as the **la** instruction

以上两个表将在 Linker 使用

### Object File Format

经过assembler 之后，生成 Object File,

- `object file header` : 描述了目标文件的其他部分的大小和位置。
- `text segment` : 包含机器语言代码。
- `data segment` : 包含在程序生命周期内分配的数据的二进制表示
- `relocation information` : 标记了在程序加载到内存时依赖于绝对地址的指令和数据。
- `symbol table` : 包含能被引用的labels 和 static data
- `debugging information` : 包含有关如何编译目标模块的简明描述，以便调试器可以将机器指令与C源文件相关联并使数据结构可读。
- [A standard format is ELF (except Microsoft)]

---

## Linker

### Agenda

Linker的作用 :

- 输入 Object File (例如 foo.o libc.o 等等多个.o文件)和 information tables
- 输出可执行文件(executable code,eg  a.out)
- 将多个.o 文件结合成单个可执行文件
- 因此可以做到separate compilation 对于单个文件的改变无需编译整个program，只需重新生成对应的.o文件，然后Link

### Process of Link


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220311172709071.png?raw=true)


- Step  1 

  从每个.o 文件取出text segment并将它们放在一起

- Step 2

  从每个.o 文件取出data segment并将它们放在一起  and concatenate this onto end of text segments

- Step 3 

  Resolve references, Go through Relocation Table;  handle each entry，fill in all absolute addresses

### Three Types of Addresses

- PC-Relative Addressing(**beq, bne, jal**)  PC 相对寻址

​	由于是PC-Relative，可以计算得出offset，<font color='red'>never relocate</font>

- External Function Reference(usually **jal**)  外部函数引用

  always relocate

- Static Data Reference(often `auipc`  and `addi`)  静态数据引用

  always relocate

  - RISC-V 经常使用 `auipc` 而不是 `lui`   so that a big block of stuff can be  further relocated as long as it is fixed relative to the pc

#### Which instructions need relocation editing?

- jump and link : Only for external jumps
- Loads and stores to variables in static area, relative to the **global pointer**

### Resolve References

Linker 假设 第一个text segment 的第一个word起始地址是 **0x04000000**

Linker 知道每一个 text 和 data segment 的长度，以及 text 和 data segment 的顺序

Linker 计算每一个被跳转至的label 和 每一份被引用的数据的绝对地址

**To resolve references**

在 symbol tables 里面搜索 data 或 label 的references

- 如果没找到，搜索library files(例如 <font color='blue'>printf</font>)
- once absolute address is determined, fill in the machine code  appropriately
- Output : 包含 text ，data 和 header 的可执行文件

---

## Loader

将 存储在disk上的可执行文件加载到内存中运行

在真实生活中，loader 是操作系统(OS)

- loading是操作系统的任务之一
- 事实上，Linker只是部分链接，依然有一些 external references 未解析



### What Loader do?

1.读取可执行文件header以确定text segment和data segment的大小 |

2.为text和data创建足够大的地址空间。|

3.将可执行文件中的指令和数据复制到内存中。|

4.将主程序的参数（如果有)复制到栈顶。|

5.初始化寄存器并将栈指针指向第一个空闲位置。|

6.跳转到启动例程，将参数复制到argument reg中并调用程序的主例程。当主例程返回时,启动例程通过exit系统调用终止程序。|



### External references -- static/dynamic linkng libraray files

#### static linking

传统的方法是静态链接，虽然这种静态方法是调用library routine的最快方法，但是有一些缺点:

- library routine 成为可执行代码的一部分，如果发布了新的library ，那么静态链接的程序依然使用旧版本，除非全部重新编译整个库文件
- 它会加载所有可能调用library 的所有routine，即使一些例程不一定会用到，相对程序而言，库可能很大

#### dynamically linked libraries

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220311203713889.png?raw=true)


---

## To Summarize whole call


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220311203958721.png?raw=true)


---

## Supplementary

### Tail Recursion

当函数最后一次执行的语句是调用自身时，则称为尾递归

例如：

```c
int foo(int x){ ...
     lots of code
     return foo(y);
}
```

以及

```c
// An example of tail recursive function
void print(int n)
{
	if (n < 0)
	return;
	cout << " " << n;

	// The last executed statement is recursive call
	print(n-1);
}

```

尾递归函数被认为比非尾递归函数更好，因为尾递归可以由编译器优化。

函数的调用会将一些 information 存储在stack 上，例如ra reg, argument reg, local variables (返回地址，参数，局部变量)，而对于非尾递归的递归函数，每次开启一轮新的调用时，都会进行 Prologue ，也就是不断生成新的stack frame，压栈，[**尾递归优化的思路是**，当递归调用是函数执行的最后一条语句时，调用后函数就没什么可做的了，不必再保存父函数的栈帧](https://www.geeksforgeeks.org/tail-call-elimination/) *引用geeksforgeeks*，相当于:

```yaml
label:
	do some code
	......
	jal x0, label
```

尾递归调用时，跳过Prologue 部分，重新将下一次调用时使用的参数放入a0 -- a7 reg，覆盖使用 ra reg ，saved reg 等等(其实 ra 是不变的，因为一直位于函数末尾，地址没变)

因此当foo()返回时，可以直接返回到第一层调用时的ra

- 不需要从调用foo()的地方不断回溯层层返回，也不需要对saved registers 过多restore

简言之则是 ： 假设一个stack frame需要 O(1) 即恒定的内存空间，那么对于**N**递归调用所需的内存将是 O(N)。 

尾调递归则将递归的空间复杂度从 O(N) 降低到 O(1)。



##### 补充两条伪指令 pseudo instruction

```yaml
call label  # call offset,call a function
```

相当于

```yaml
auipc x6, offset[31:12]
jalr x1, x6, offset[11:0]
```

```yaml
tail label  # tail offset
```

相当于

```yaml
auipc x6, offset[31:12]
jalr x0, x6, offset[11:0]
```

call 保存返回地址， tail 不报存返回地址，对尾递归使用 **j 或 tail**

---

### Integer Multiplication / Division

#### Multiplication

两条指令：

```yaml
mulh s1, s2, s3
```

将 s3 * s2 的结果高32位存入 s1

```yaml
mul s0, s2, s3
```

将 s2 * s3 的结果低32位存入 s0

由于n位数与m位数相乘会得到 n*m位数，因此两个32bit 的数相乘得到一个64 bit的数，我们知道 RISC-V的32bit版本寄存器大小只有32bit ，因此使用上述两条指令相乘的结果可以分别用两个寄存器 s1和 s0 存储

**Note**{: .label .label-red } 

如果你同时使用

```yaml
mulh s1, s2, s3
mul s0, s2, s3
```

那么architecture知道你想要两个结果，此处architecture会偷懒,只计算一次然后将结果分为两部分，因此如果你同时需要两个值时，将两条指令写在一起可以减少一次乘法计算(乘法很昂贵)

#### Division

因为

$$
\begin{aligned}
Dividend = Quotient * Divisor + Remainder
\end{aligned}
$$

所以两条指令：

```yaml
div rd, rs1, rs2
```

计算除法的结果，商

```yaml
rem rd, rs1, rs2
```

计算余数

同样地，如果你同时需要商和余数，将两条指令写在一起减少一次计算

