---
layout: default
title: Assembly, RISC-V Intro
author: liuxing
math: mathjax2
nav_order: 6
---

# Assembly, RISC-V Intro
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Great Idea : Levels of Representation

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/hira.png?raw=true)


CS61C是自顶向下介绍计算机组成，此时我们位于第二层 汇编语言

---

## Assembly Language

**ISA : Instruction Set Architecture** 

**CPU : Central Processing Unit**

不同的CPU执行不同的指令集，特定的CPU执行的指令集被称为ISA，例如 ARM,Intel x86, MIPS,RISC-V......

**RISC : Reduced Instruction Set Computer**

- 一条单一只能一次只能执行一项操作
- 开发该指令集的初衷是尽可能使指令的数目简单化，轻松构建更快的硬件
- RISC-V是开源的，支持32 bit,64 bit 和128 bit ,61C使用的是32bit的版本

---

## Registers

与高级语言不同的是，汇编语言使用的不是变量，而是寄存器，汇编语言在寄存器上执行，寄存器位于Processor内，小而速度快，与处理器外的内存相比，寄存器要快上50-500倍

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303144851343.png?raw=true)

在61C的RISC-V 在中，拥有32个寄存器，单个寄存器能存储 32bit ，也就是 1 Word，因为我们使用的是RISC-V 32Bits 的版本

- 32个寄存器被命名为 x0 --> x31，与高级语言不同，这些命名是固定的，我们不能随意命名寄存器
- x0寄存器具有特殊的意义，它的存值永远为0
- 寄存器没有变量类型一说，向其中存储Char ,Int ,Double都可以

---

## RISC-V Instruction Operation

#### Addition
![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303145524159.png?raw=true)

第一个寄存器是Destination register ，后两个是 Source register，

上述语句的作用是 : 取出 x2, x3 寄存器中的值相加后存入 x1

汇编语言一次指令只支持使用两个 Source register ,如果你要做类似

```
x1 = x2 + x3 + x4
```

的操作，一条指令无法完成

---

#### Subtraction

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303145759613.png?raw=true)

**Example** 

```c
compute a = b + c + d -e
       x10 x11 x12 x13 x14
```

将上面的C语句翻译为汇编语言

```yaml
add x10, x11, x12      # a = b + c
add x10, x10, x13      # a = a + d
sub x10, x10, x14      # a = a - e
```

---

#### Assignment

在实现变量赋值时，寄存器 x0 非常有用，例如C中，实现变量赋值

```c
f = g
x3  x4    
```

可以利用 寄存器 x0 完成

```yaml
add x3, x4, x0       # f = g + 0 = g
```

---

#### Immediates

在计算过程中常数的使用是非常频繁的，因此提供"立即数",可直接在汇编语言中使用数字，只需要在对应的指令后面添加 **i** :

```
add --> addi
```

例如

```c
f = g - 10
x3  x4     
```

在RISC-V中:

```yaml
addi x3, x4, -10
```

没有减法的立即数，也就是说没有 subi ，因为任何减法可以用加法操作执行，RISC-V尽可能地限制指令的数量，但凡能够被分解为更简单的操作的指令都不会被包含(这也是它设计的初衷)

- 立即数被限制大小为12bit
- 当你使用立即数时，它会做对应的符号扩展 (Sign Extension)

**Why Sign Extension?**

符号扩展的目的是保证数表示的值不变，例如 

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303151815551.png?raw=true)

寄存器内是32位，则将该数存入时按0扩展至32位：

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303151828744.png?raw=true)

结果仍然是3，

对于负数表示也是一样的，例如-3以补码(Two's Component)的形式表示为:

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303151909477.png?raw=true)

转换为十进制:取反+1

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303152105716.png?raw=true)

在32bit 的寄存器中，以符号位1进行扩展后得到

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303152148525.png?raw=true)

取反加一后结果仍然一样：

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303152215727.png?raw=true)

---

### Loading Data from Memory into Processor Registers

##### Load word --- lw

寄存器 x15 存放一个指向数组的指针，该数组存储在内存里面，如何将内存中的数组的 index [3]的值取出 存入寄存器 x10 ？

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303153126697.png?raw=true)

```yaml
lw Register1, (offset)Register2
```

从Register2中的 Base 地址偏移 offset 个 byte 后的地址取出值存放在 Register1中，需要手算偏移量offset，int 大小为4 byte，从a[0] 到 a[3] 有12byte

- **Note** : offset必须是常数，不能是寄存器 |

---

### Storing Data from Processor Register into Memory

##### Store word --- sw

**Note**: 这里需要注意一下，前面我们讲的 add ,sub, lw 操作，都有Destination Register,但是对于Store ,是从寄存器向内存中存值，因此没有 Destination Register，只有提供data  的 Source Register

寄存器 x15 存放一个指向数组的指针，该数组存储在内存里面，如何将寄存器 x10 的值存入内存中数组index[3]中？

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303154012300.png?raw=true)

---

#### Loading and Storing Bytes

lw 和 sw 是分别向内存取 word 和 存 word 大小的数据，我们也可以存取 byte

**Load Byte --- lb**

**Store  Byte -- sb**

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303191021174.png?raw=true)

##### Loading Bytes

当你从内存中 load byte 时，它被放置于寄存器的低8位，并做**符号扩展(sign extended)**

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/20220303191244946.png?raw=true)

如果你不想要它做符号扩展，可以加 **u** 以0扩展：

```yaml
lbu x10, 3(x15)
```

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303191505753.png?raw=true)

##### Storing bytes

当你将Register 中的值存入 Memory 时，只需将低8位copy，不需要符号扩展（原本就是8bit，值保持不变）

---

### RISC-V Logical Instructions

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303192117609.png?raw=true)

##### Left Shifting

- 左移，共 32bit 溢出部分截断

- 左移相当于乘以$ 2^n$ 
- RISC 指令是 Shift Left Logical --- sll
- Shift Left arithmetic 与 Shift Left Logical 效果相同，因此不必引入

可以对寄存器进行移位操作:

```yaml
sll x10, x11, x12  # x10 = x11 << x12  
```

也可以对常数(immediates)进行移位：

```yaml
slli x10, x11, 2  # x10 = x11 << 2
```

##### Right Shifting

- Shift right logical --  **srl**

与逻辑左移类似，右移后最右边溢出部分截断，左边补0

- Shift right arithmetic -- **sra**

右移后左边不再补0，以符号位扩展(sign extended)

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303193229195.png?raw=true)

右移正数和偶数相当于除以 $ 2^n$ ，但是小数点后面被截断，例如上图第二栏的例子

右移负的奇数相当于除以 $ 2^n $ ,但是四舍五入到负无穷，例如上图第四栏例子

- 由此可见这与C语言的Rounding 不同，C 语言是向0舍入(Round toward 0)

---

#### Shifting Can be used to compute the address

假如说某个寄存器里面存储了一个index，该index我们目前无法得知其值，且需要向内存内某数组的该index存值，使用shift 来做 乘以 $2^n$ 的操作，以此计算 offset

Eg .  Register x15 存储指向 Integer 数组的指针，该数组位于Memory内，如何将 x10内的值存储至数组下标 index = x11 处？

```yaml
slli x11, x11, 2     #x11 = x11 << 2 = x11*4  
add x12, x15, x11   #x12 = x15 + x11
sw x10, (0)x12      #array[x12] = x10
```

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303194544290.png?raw=true)

---

#### Branch instructions

分支指令改变程序的control flow ，也就是说程序不再按顺序地执行指令，而是可以从一个分支跳转到另一个分支，例如对 if-else 语句很有帮助

##### Types of branch instructions

- Conditional branch
- Unconditional branch

##### Labels

```
 Labels are used to give control flow instructions places to go
• When you are writing code, you don’t know the memory
address of the location you want to go to
• You can place a label in the assembly at the place that you
want to branch to and then specify that label in your code
```

---

##### Branch if equal -- beq

```
beq register1, register2, L1
```

如果 reg1 = reg2 ，指令执行跳转至L1标签处，否则继续按顺序执行指令

例子:

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303195410934.png?raw=true)

```yaml
beq x10,x11,Exit
add x14,x13,x12
Exit:
```

如果x10=x11那么就退出，否则(a != b)执行 e = c + d, 标签Exit**后要加冒号**

```c
if(a != b) 
  e = c + d; 
```

##### Branch if not equal -- bnq

```yaml
bnq reg1, reg2 L1
```

如果reg1 != reg2，则跳转到标签L1的位置执行指令，否则继续按顺序执行

例子:

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303195830962.png?raw=true)

```yaml
bne x10,x11,Exit
add x14,x13,x12
Exit:
```

如果x10 != x11就退出，否则执行 e = c + d

从上述例子可以看出与C 有点反着的感觉，当C语言条件是==，汇编使用 bnq, 当C条件是 !=，汇编使用beq，**但是逻辑上是融洽的**，因为 bnq 和 beq 在满足条件后都会跳转到另一个 label ，但是C 满足 if 的条件是继续按顺序执行if 下面的代码，因此为了保证与 C 效果一致所以使用 "反方向"

##### Branch on less than -- blt

```
blt reg1, reg2, L1
```

If reg1 < reg2, jump to code at the location of label L1, otherwise continue
executing the code in sequence

 ![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303200639029.png?raw=true)

##### Branch on greater than or equal -- bge

```
bge reg1, reg2, L1
```

If reg1 >= reg2, jump to code at the location of label L1, otherwise continue
executing the code in sequence

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220303200657428.png?raw=true)

blt 和 bge 都是针对有符号数的表示，如果想比较无符号数，加后缀 **u**

```
bltu     
```

```
bgeu
```

---

##### Pseudo Instructions

RISC-V 只有 小于 和 大于等于 ，没有表示 大于(branch if greater than)或 小于等于(branch if less than or equal)的指令，因为你可以转换：

•A > B is equivalent to B < A
•A <= B is equivalent to B >= A

但是有伪指令，比如当你执行 bgt (branch if greater than) 大于 ：

```yaml
bgt x2, x3, foo  # if x2 > x3
```

assembler 会帮你转换成

```yaml
blt x3, x2, foo  # if x3 < x2
```

---

#### Unconditional Branches 

##### Jump

```
j label
```

直接跳转到 label 处

##### If - else 的例子

```c
if (a == b)
	e = c + d
else
	e = c - d
```

```
x10 = a
x11 = b
x12 = c
x13 = d
x14 = e
```

RISC-V实现:

```yaml
	bnq x10, x11, else
	add x14, x12, x13
	j done
else:
	sub x14, x12, x13
done:
```

##### Loop 的例子

假设 寄存器 x8 存储 内存中array 的地址

将 C 转换为 RISC-V

```c
int A[20];
int sum = 0;
for (int i=0; i < 20; i++)
sum += A[i];
```

RISC-V:

```yaml
    add x9,x8,x0 		# x9=&A[0]
    add x10,x0,x0 		# sum=0
    add x11,x0,x0 		# i=0
    addi x13,x0,20 		# x13=20
Loop: 
    bge x11,x13,Done
    lw x12,0(x9) 		# x12=A[i]
    add x10,x10,x12 	# sum+=A[i]
    addi x9,x9,4 		# x9=&A[i+1]
    addi x11,x11,1		 # i++
    j Loop
Done:
```

