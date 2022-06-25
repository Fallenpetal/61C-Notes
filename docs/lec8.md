---
layout: default
title: RISC-V Instruction Formats
author: liuxing
math: mathjax2
nav_order: 8
---
# RISC-V Instruction Formats
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}



## 前言

前面已经学习了 High Level Language 和 Assembly Language ，现在来到 Machine Language，介绍一些 RISC-V 的 instruction format
{: .fs-6 .fw-300 }

##  R-Format

###  Layout


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307210817887.png?raw=true)

接下来介绍的几种指令格式都有一些相同的布局，以R-format为例，介绍:

- **opcode**{: .label .label-yellow } : 用来确定你使用的是哪一种指令
- **funct7 + funct3**{: .label .label-blue }  : 用来确定所执行的操作，例如add, sub 等等
- **rs2**{: .label .label-green }  : source register 2
- **rs1**{: .label .label-green } : source register 1
- **rd**{: .label .label-yellow } : destination register

我们知道在RISC-V 32bit 中拥有32个寄存器，编号为 x0 -- x31，对于单个寄存器使用5 bit ($$ 2^5 $$) 大小的空间即可存储，因此寄存器的字段大小为5 bit 

###  All RV32 R-format instructions


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307211630770.png?raw=true)

可以看出 R format 的 opcode 是 **0110011** (对于所有的算术运算和逻辑运算都是相同的opcode)

加法的 funct7 与 funct3 全0

图左funct7中蓝圈中的0 1用于区分逻辑操作和算术操作，(SRL/SRA)

**Example**

执行如下指令:

```yaml
add x18, x19, x10
```

在R-format 下为:


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307212022498.png?raw=true)

解析一下，

rs1是 x19 ,19对应二进制 10011,rs2是x10, 10对应二进制01010，rd是x18，18对应10010

R-format的opcode是0110011

add操作的funct7 + funct3是 全0

**Note** : **R-format的特点是 两个 source register ,一个 destination register**

---

## I-Format

当我们只有一个source register ，而另一个操作数是immediate 时，上述的R-Format不再适用，因此有了I-Format ，针对立即数(imm)

### Layout


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307212716175.png?raw=true)

因为我们不需要rs2，但是rs2仅能带来 5 bit 的空间，这对于立即数的表示是不够的，因此将 funct7和rs2字段全部用来表示imm，imm的空间为12bit

(这也就解释了前面的章节我们说立即数只有12 bit，布局如此)

**12 bit 的 imm所表示值的范围是$$ [-2048_{10}, +2047_{10}] $$** 

在进行算术运算前，由于 寄存器的空间是32bit ，imm会做sign-extended(符号扩展) 到32bit

对于大于12bit 的 imm 如何存放？稍后会作解释

### All RV32 I-format Arithmetic/Logical Instructions


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307213345048.png?raw=true)

I-Format 的 opcode 是 **0010011** (对于所有的算术运算和逻辑运算都是相同的opcode)

funct3 与 R-Format 相同

对于Shift 移位运算 (SLLI, SRLI, SRAI) 因为RISC-V 32 的寄存器只有32 bit 的大小，因此我们做移位时最多移动31位(超过31位，所有的值已经被截断了，只是不断填0，没意义)

将12bit 的 imm区域分为 7bit 和 5bit的 **shamt** ，蓝圈中的0 1用于区分逻辑右移和算术右移

###  Example

```yaml
addi x15, x18, -50
```

rs1是x18 ,对应 10010

rd是x15, 对应 01111

-50以补码(2's complement) 的形式存储在imm字段


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307214330205.png?raw=true)

**Note** : I-format 的特点是 一个 resource register , 一个 destination register ,一个 imm |

###  Load Instructions

回忆一下Load instructions ，我们有

- Load word
- Load byte
- Load byte unsigned

补充:

- Load Halfword
- Load Halfword unsigned

unsigned： 如果不加unsigned ，那么数据会做sign-extened，加unsigned以0扩展

```yaml
lw rd, offset(rs1)
```

可以发现 Load 指令是I-Format 类型的, 因为我们有一个 rd , 一个 rs, 一个imm


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307215248398.png?raw=true)

####  All RV32 Load Instructions 


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307215332721.png?raw=true)

解释一下,

opcode 是 **0000011** (上面的I-format 的 opcode 0010011是针对算术和逻辑运算的)

funct3字段被分为 最高位 和 低2位，MSB 表示是否做符号扩展, 

0 = sign-extended

1 = unsigned

低2位表示加载数据大小，因为我们有word, byte,half-word

00 ：1 byte

01： 2 bytes = half word

10： 4 bytes = word

#### Example

```yaml
lw x5, 4(x19)
```


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307215826208.png?raw=true)

---

##  S-Format

回忆一下Store instructions，例如


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307220535633.png?raw=true)


sw 有2个 source register，没有 destination register

而 I-Format 有1个 source register ，1个destination register，因此 I-Format不适合于 Store instruction，由此有了 S-Format

### Layout

因为 R-Format 同样是有两个source register ，layout可参考 R-format(相同的字段在相同的位置上，这样设计cpu容易处理)


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307220752119.png?raw=true)

store指令没有 rd reg，将 rd 和 funct7的字段用于存储imm, 两段在空间位置上分裂，但是不影响

imm的大小是12bit ,范围与 I-Format 一致

###  Store Instructions

与load 类似:

- Store word
- Store byte
- Store Halfword

####  All RV32 Store Instructions

![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307221347542.png?raw=true)

opcode是 **0100011** (与load instruction不同)

同样的 funct3 的低2位是表示

00 ：1 byte

01： 2 bytes = half word

10： 4 bytes = word

#### Example


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307221700918.png?raw=true)

---

## B-Format

回忆一下条件分支，if-else, 循环loop, Branch Instruction 有

beq, bne, blt, bltu, bge, bgeu

```yaml
Format: {comparison} {reg1} {reg2} {label}
```

label实际上是一个指令的地址， label = PC + immediate

imm的大小是12bit ，能从PC 跳转到 $$ ±2^{11} $$ bytes 远的地址 

因为每条指令的大小是 4byte ，所以我们可以跳转
$$$$
\frac{2^{11}}{4} = 2^9
$$$$
$$ ±2^{9} $$ 条指令

### Instruction Addressing

指令是存储在Memory里面的，每条指令是4 byte的大小


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307222952659.png?raw=true)

且每条指令的地址（指令地址对应imm字段）的最后两位总是00，那么我们可以不去存储这两个0（回忆一下浮点数的表示，规格化数字用科学计数法表示后小数点左边永远是1，可以忽略1的存储，节省空间）

这样我们多了2 bit 的空间，例如


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307223201628.png?raw=true)

但是事实上，RISC-V支持16 bit 的版本，在16 bit 的版本下，仅有最后一位永远是0


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307223257624.png?raw=true)

为了规范统一，32bit 的版本只忽略最后一位的存储，多了1 bit 的空间，可以跳转

$$ ±2^{10} $$ 条指令

### Layout


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307223501988.png?raw=true)

### Example

```yaml
Loop:   beq x19,x10,End
        add x18,x18,x10          1
        addi x19,x19,-1          2
        j Loop                   3
End: # target instruction        4
```

End Label实际上的offset (imm)  =  (4 instructions x 4 bytes) = 16 bytes

编码是0b 0000 0001 0000

最后一位0不存


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307223956171.png?raw=true)

### All RISC-V Branch Instructions


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220307224514915.png?raw=true)

---

##  J-Foramt

branch instruction能跳转的指令条数只有 $$±2^{10} $$ ，如果我们想要跳的更远，使用 jump instruction 

回忆一下 

```yaml
jal rd, label
```

还记得我们说label 被 assembler 转移成 20-bit 的 offset 吗？

原因是j-format 的布局如此


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308160014605.png?raw=true)

imm 字段拥有20 bit ，与 b-format 类似，我们可以省略对imm的最后一位的存储，这样下来就多出一位 bit, 21bit 可以跳转$$± 2^{20} $$ bytes ，也就是 $$ ± 2^{18}  $$ 条instruction

比 branch 指令跳转的多得多

### JALR is an I-type Instruction

除了 jal ，还有一条指令是 jalr(jump and link register) ，这条指令实际上是I-Format 而不是 J-Format


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308160457250.png?raw=true)

一个imm ,一个 rd, 一个 rs ,显然符合 I-format

但是 I-format 只有 11bit 的imm字段 ，跳转指令数为 $$ ± 2^{8}  $$ 条，


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308160832955.png?raw=true)

**Note** : 与 JAL 不同，imm 的最后一位0必须包含，因为这是 I-Format ，而不是 J-Format 或 B-Format  |

---

## PC-Relative Address vs Absolute Address

 **PC Relative Address**

将要跳转的地址与 PC 有关，

```yaml
relative address = PC + offset
```

offset只是一个相对偏移量

 **Absolute Address**

是一个full address

###  Questions on PC-addressing

每次运行同一个程序都会随机被加载到内存的不同位置，那么当位置改变时是否需要重新调整 branch 指令 里面 b-format 的 imm字段的值？

**ans** : 不用，因为是相对地址，（举个例子，你和你爸差20岁，十年后你们俩仍然差20岁，与当前年份无关）

---

##  U-Format

###  Load Upper Immediate(LUI)


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308161712782.png?raw=true)

前面我们说到对于大于12bit 的立即数如何处理，使用该指令，使立即数左移12位

与 addi 连用可以生成long immediates

```yaml
LUI x10, 0x87654       # x10 = 0x87654000
ADDI x10, x10, 0x321   # x10 = 0x87654321
```

伪指令 LI
{: .label .label-red }


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308161956659.png?raw=true)

###  One Corner Case

如果与一个负数做 addi 的操作，由于addi 之前 imm会做sign-extended，例如：


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308162130558.png?raw=true)

导致后20bit比真实值小1

**解决方法** 

- 因为与负数相加，后20bit会比真实结果少1，那么先将后20bit + 1后再做 addi 操作，可抵消
- 使用伪指令 **li** 会自动做solution

###  Add Upper Immediate PC (AUIPC)


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308162516123.png?raw=true)

###  Layout


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308162548144.png?raw=true)
###  LUI 和 AUIPC 与 JARL 联用call function


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308232317148.png?raw=true)

伪指令 : la
{: .label .label-red }

```yaml
la rd,symbol
```

该伪指令相当于

```yaml
auipc rd,symbol[32:12]
addi rd,symbol[11:0]
```

其作用是 加载地址(Load address)

```yaml
x[rd] = &symbol
```



---

## Summary of RISC-V Instruction Formats


![image-hira](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220308162624695.png?raw=true)