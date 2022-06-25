---
layout: default
title: RISC-V, RISC-V Functions
author: liuxing
math: mathjax2
nav_order: 7
---
# RISC-V, RISC-V Functions
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}



## Program Counter(PC)

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306142218026.png?raw=true)

PC 是在处理器内部一个特殊的寄存器，在Registers File(x0 - x31) 32个寄存器之外，用于保存下一条将要执行的指令地址(instruction address)

指令从内存中获取，然后control unit使用data path  和  memory system 执行指令，并更新PC

对于RISC-V 32bit 来说，每条指令为 4 byte ，因此默认按顺序的下一条指令地址是

```
PC = PC + 4 byte
```

---

## Instruction 1.  Jump and Link --- JAL

前面讲了 

```
j label
```

可以跳转到 label 的位置执行指令，设想一下，如果我们想要执行的 label 是一个函数(function) ，那么跳转到该函数的 address , 并执行完毕之后，需要返回到调用处，以表示此次函数调用已执行完毕

使用 j label 完成上面的工作的话，需要满足:

- 存储函数的返回地址 (store the return address)
- 更新PC 的值

因此有一条新的RISC语句可以完成这件事，Jump and link

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306151747011.png?raw=true)

```yaml
jal rd, Label
```

详细手册说明如下:

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306144010842.png?raw=true) 

也就是说，

rd 存储返回地址，返回地址 return address 总是下一条指令的执行地址(sequential)，也就是 

```yaml
x[rd] = pc + 4

pc = pc + label = pc + offset
```

这里我们想要跳转至的label 被 assembler 转译为 **20 bit 长度**的 offset 

We’ll learn about why it’s 20 bits later

### Using Register x1 to store return address

按照RISC 标准，选择寄存器x1 保存 返回地址

x1 重命名为 ra 

```
jal ra, L1
```

---

### Using Register x0 to avoid to store return address

如果一个函数执行完毕后需要返回到调用处，使用 jal ,返回地址则存储在 rd  register ，假如我们希望跳转至label 后不再返回，如何做到？例如上节课介绍的branch 分支，loop,  if-else 分支跳转后则不再返回:

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306144825180.png?raw=true) 

使用 Register x0可以做到，还记得x0 永远存储 0 吗？这样就不会store return address

```
jal x0, label
```

**jal x0, L1** 相当于伪指令 **j L1** 

---

## Instruction 2. Jump and Link Register --- JALR

上面说到，一次jar 跳转的offset 只有20 bits 大小，然而这并不足以让我们能够到达内存的任何地方，因此我们有一条另外的指令可以做到 : jump and link register --- jalr

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306150133772.png?raw=true) 

```yaml
jalr rd, rs, imm
```

与 JAL 不同的点是：

- 不再是 PC = PC + imm , 而是 PC = rs + imm
- Two main uses
  - Returning from functions (which were called using Jump and Link)
  - Calling pointers to function

详细手册说明:

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306150052949.png?raw=true)

简单来说就是 rd 依然存储返回地址，由于并没有指定跳转的 label , 跳转地址变成 rs + imm，

rs register 存储基地址(base address)[任意合法地址均可]，imm是立即数

```
x[rd] = pc + 4      //按顺序的下一条指令
```

```
pc = x[rs] + immediate = x[rs] + offset
```

对于不想要返回的jump, 同样地, 用 x0 寄存器 存储 return address 即可

### Jump Register Pseudo Instruction

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306152604852.png?raw=true)
了解了那么多，下面我们来看一个函数调用的例子，这是一个函数调用另一个函数的例子

### Function call Example

```yaml
caller function:
	# do something
	jal x1, callee
	
callee function:
	# do something
	jal x0, x1
```

---

## Calling Convention

###  RISC-V Convention

汇编语言与高级语言是非常不同的，汇编语言没有参数检查，在汇编中everything is the result of "convention".

当谈到convention时，你可能想到在高级语言中如 函数命名规范，每行代码多长等等

但是本质上对 function 的功能无影响，然而整个汇编都基于 convention ,如果你不按照 convention 写汇编代码，那么你的代码很可能无法正常运行，除非你自己开创一门新的汇编语言~

RISC-V convention 由三个重要的部分组成:

- registers
- function calls
- prologue / epilogue

###  Registers

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307102022872.png?raw=true)

介绍一下RISC-V 的部分寄存器:

**ra** ，ra是存储函数的返回地址，考虑以下函数调用
{: .label .label-blue }

```python
def foo():
	x = 1
	bar()
	z = 2
def bar():
	y = 7
```

在foo()中调用bar()，当bar()执行完毕 y = 7之后，将会回到 z = 2，因此 ra 储存的就是 bar() 的返回地址，在此处就是 z = 2的位置

**sp** ,sp是存储栈顶指针的寄存器，下面再介绍
{: .label .label-yellow }

**a** a0-a7是参数寄存器，a0和a1既可以存储函数返回值，也可以存储函数参数，a2 - a7存储函数参数，当参数多于8个，则存储在stack 上
{: .label .label-green }

**saved register / temoporary register**  二者都可以用来存值，temoporary register 与 saved register相比更具易失性

**caller-saved and callee-saved**: 

在调用者(caller)的角度看来，函数**总承诺**在调用另一个函数后，受保护的(preserved)寄存器值不变，例如

```
addi s0, x0, 5     # s0 contains 5
jal ra, func       # call a function
addi s0, s0, 0     # s0 still contains 5 here
```

但是**不保证** 未受保护(unpreserved) 的寄存器不发生改变，也就是被调函数 可以**随意改变**未受保护的寄存器的值，如果调用者想要这些未受保护的寄存器(unpreserved register)的值也不发生变化，那么就需要将这些寄存器存入栈中，that's why it called caller-saved

在被调用者(callee)的角度看来，同样承诺受保护的寄存器(preserved register)不发生改变，那么如果被调用者想要使用这些寄存器，需要先存入栈备份，that's called callee-saved

因此，受保护的寄存器是callee-saved，不受保护的寄存器是caller-saved

**<font color = "red">更多查看此处，写的非常详尽</font>**

https://cs61c.org/sp22/projects/proj2/calling-convention/

###  Prologue / Epilogue

最后一步达成calling convention 是介绍 prologue 和 epilogue,关于如何保证value 不变：

- **sp** 在函数调用开始和结束时的 value 相同
- 所有的 **s reg** 在函数开始和结束时 value 相同
- 函数的return address 存储在 **ra**


```yaml
def prologue():

​		decrement **sp** by num **s registers** + local varibale space
​		store any **saved registers** used
​		store **ra** if a function call is made

def epilogue():

​		reload any **saved registers used**

​		reload **ra** if necessary

​		increment **sp** back to previous value

​		jump back to return address

```

---

## RISC-V 32 bit Memory Allocation

 C has two storage classes: automatic and static

<font color='blue'>automatic</font> :  automatic 变量是函数内的局部变量，当函数退出时被丢弃

<font color='blue'> static </font> :  static 变量存在于整个程序从开始运行至结束的时间内

Use stack for automatic (local) variables that aren’t in registers

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306170036404.png?raw=true) 

从上面的memory layout 可以看出，stack  posh是地址减小方向，stack pop是地址增加方向

有一个 stack pointer 指向栈顶，按照RISC标准，使用寄存器x2 存储 stack pointer ,x2重命名为 **sp**

### How to move the stack pointer?

栈顶指针下移,预留空间:

```
addi sp, sp, -x
```

栈顶指针上移，压缩空间:

```
addi sp, sp, x
```

### How to Store a Value on the Stack?

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306162012017.png?raw=true)
---

## RISC Registers Rename Convention 1

上面提到了一些寄存器特定的存储值:

x0 : 永远存0

x1 : 存储return address  ，命名为ra

x2 : 存储 stack pointer, 命名为 sp

以及temporary register 和 saved register的各自作用

按照 RISC 的标准，我们不再简单地使用 x0 - x31寄存器，Rename :



![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306163222486.png?raw=true)

---

## Argument and return value registers

一个函数通常拥有参数和返回值，所以也需要一些寄存器来存值，我们将使用 寄存器 x10 - x17作为参数寄存器(arguments register) ，并命名为 a0 -- a7

a0 , a1 同时也作为(return value register)

- **Note** :  我们有8个寄存器用于保存参数，2个寄存器用于保存函数返回值，如果一个函数的参数多于上述，那么多余部分将保存在stack 上 |

### RISC Registers Rename Convention 2

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306163802612.png?raw=true) 

---

## Example - (a "leaf" function - it calls nothing)

```c
int Leaf(int g, int h, int i, int j) 
{ 
 int f; 
 f = (g + h) – (i + j); 
 return f; 
} 
```

- 参数 g , h, i, j 存储在参数寄存器 a0, a1, a2 ,a3
- 假定使用 saved register s0 ，s1 计算f (在真实情况下我们可能使用的是temporary register t0, t1, t2,why? we will see later)
- 由于s0 和 s1是 callee-saved，那么需要在使用寄存器前将value 全部保存在stack 上，并使用结束后restore

```yaml
Leaf: addi sp, sp, -8       #栈顶指针下移，腾出两个空间以供保存Register
	  sw s0, 4(sp)
	  sw s1, 0(sp)
	  add s0, a0, a1        #s0 = g + h
	  add s1, a2, a3		#s1 = i + j
	  sub a0, s0, s1		#a0 = (g + h) - (i + j)，算出f保存在a0(return reg)
	  lw s1, 0(sp)
	  lw s0, 4(sp)			#从stack 取值restore,注意先进后出FILO
	  addi sp, sp, 8		#stack pointer复原
	  ret					#返回，或写成 jr ra (jalr x0 ra 0)
	 	  
```

### Observation

实际上我们发现Leaf()函数并没有内部调用其他函数，我们并不需要保存 ra (因为leaf()并没有调用其他函数，ra 从未改变)

那么只需使用temporary registers(caller-saved register)，不再使用 s0, s1

So we could have just as easily used t0 and t1 instead…

```yaml
leaf: 
    add t0,a0,a1 	# t0 = g + h 
    add t1,a2,a3 	# t1 = i + j 
    sub a0,t0,t1 	# return value (g + h) – (i + j) 
    ret 			# ret is shorthand for jalr x0 ra
```

---

## What If a Function Calls a Function?

- 包括函数递归调用自身
- **a0 - a7 和 ra**的值将会被覆盖(argument and return address)

例子:

```c
int sumSquare(int x, int y) {
    return mult(x,x)+ y;
}
```

sumSquare()内部调用mult(),原来的ra 寄存器存储的sumSquare()的return address将会被 mult()的return address 覆盖(overwrite)，因此在覆盖前使用stack 存储 sumSquare()的返回值

### Solution

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306170559807.png?raw=true)
![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220306170642074.png?raw=true) 
---

## Six Fundamental Steps in Calling a Function

  Put parameters in argument registers |
 use **jump and link** to jump to a function |
 Make room for local variables on stack to store |
 Perform desired task of the function |
 Put result value in **a0 - a1** register |
 use **ret** to return control to point of origin |

