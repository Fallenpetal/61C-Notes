---
layout: default
title: RISC-V Datapath, Single-Cycle Control Intro
author: liuxing
math: mathjax2
nav_order: 12
---

# RISC-V Datapath, Single-Cycle Control Intro
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Nasty Realities

### Delays in CMOS circuits

晶体管并不是完美的开关，它们存在:

- 开关断开时依然可能漏电
- 开关闭合时有电阻存在
- 存在电容

因此对于翻转器，有着clk-to-q 延迟和set-up time 延迟

$$
\begin{aligned}
T \geqslant t_{clk \rightarrow Q}+t_{CL}+t_{setup}
\end{aligned}
$$
如何减少延迟T?

### CMOS circuits use electrical energy

Energy（能量）是做功的度量

Power（功率）是单位时间内能量的消耗量

对于小型可持电子设备来说

- Energy Efficiency -- 限制电池寿命
- Power -- 限制高温

对于基础设施和服务器来说

- Energy Efficiency -- 决定了运营成本
- Power -- 除热导致成本提高

平衡Energy Efficiency 和 Power成为重要决定因素

每一次logic transition 都会消耗能量
$$
\begin{aligned}
E=\frac{1}{2} CV_{d d}^{2}
\end{aligned}
$$

$$
\begin{aligned}
P = \frac{dE}{dt} = \frac{1}{2} \alpha CV^2_{dd}F
\end{aligned}
$$

功率正比于F,我们可以减少频率来减少功率，但是并不能提高能量效率

能量效率:
$$
\begin{aligned}
E \propto V_{d d}^{2}
\end{aligned}
$$
但是
$$
\begin{aligned}
T_{logic}\propto V_{dd}
\end{aligned}
$$
因此我们能够通过降低电压提高能量效率，且使用并行性弥补性能的不足

---

## Building Single-Cycle RISC-V RV32I Datapath

假设:

指令的读取是异步的(asynchronous)，而写入是同步的(synchronous)，也就是一次写入只能发生在时钟周期沿 到来之时，state elements 的更新只发生在上升沿(assume positive) ,称为单周期处理器

datapath 主要由六个部分组成：

- PC : 程序计数器，存储当前指令的地址
- IMEM : 存储指令的memory
- Registers File : 寄存器文件，RV32拥有32个寄存器，编号x0--x31 
- ALU ： 算术逻辑运算单元，执行算术运算和逻辑运算
- DMEM : 存储数据的memory
- IMM Gen : 立即数单元，取立即数并做符号扩展



将Memory分为IMEM 和 DMEM ，数据和指令分开存放，之后的课我们会使用caches 替换

在PC 的寻路下，指令从IMEM中被读出(假设IMEM read-only)，Load/Store 指令将会访问 DMEM

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316170231446.png?raw=true)

接下来我们说说如何分别实现 R I S B J Format的指令的通用方法。

**具体的方法就是**：

- 按照指令格式，确定哪些格式使用rd,rs1,rs2寄存器，和imm字段

  - 也可以写出指令例子，例如 add rd, rs1, rs2

- 写出该指令的作用，也就是硬件发生的状态改变，例如 add就是

  - ```yaml
    rd = rs1 + rs2
    pc = pc + 4
    ```

- 根据作用，在五部分的基础上去连接线路，发生选择时使用MUX（多路选择器）

- 与Control Flow 连接

---

## Implement R Format inst

### add

R指令格式


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314103837053.png?raw=true)

例子: 

```yaml
add rd, rs1, rs2
```

指令的作用:

```yaml
rd = rs1 + rs2
pc = pc + 4
```

连接Datapath


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316171358242.png?raw=true)

解释:

1. PC 读取IMEM 取出指令，PC 另一条线路与ALU 加 4 然后写回PC ，表示 PC = PC+4
2. 指令的三个字段inst[11:7],inst[19:15],inst[24:20]分别对应rd, rs1,rs2
3. 读取三个寄存器，将读出的数据Data A 和 Data B 送入 ALU 做加法运算
4. 运算的结果写回 rd 寄存器, 控制流中RegWEn 是写使能，等于1表示能够写入

### sub

与加法一模一样，唯一的不同是，数据送入ALU后执行的不再是加法操作，而是减法，具体则是在Control Logic 里面添加一个 ALUSel ，表示可以执行的不同算术逻辑运算：

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316172343002.png?raw=true)

---

## Implement I Format inst

I-format 用于需要立即数操作的指令，因此指令格式有一个12bit的立即数字段，该字段经过 **Imm Gen** 之后，做符号扩展至32bit 输出

### addi

指令例子：

```yaml
addi rd, rs1, imm
```

指令发生的作用:

```yaml
rd = rs1 + imm
pc = pc + 4
```

连接Datapath:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316180815239.png?raw=true)

解释:

- PC 存储当前指令的地址，一条线路由地址在IMEM中取指令，另一条线路与ALU做+4运算后写回PC
- 取出的指令不同字段对应不同的寄存器或Imm
- 由于R 指令是rs1和 rs2做运算，而 I 指令是 rs1 和 imm 做运算，因此在rs2 和 imm 之间添加MUX ，选择rs1是与 rs2做运算还是与 imm 做运算

**Q** ： 为什么不直接去掉Reg2 的输出线路，直接让 Reg1 与 imm 输入至 ALU？

**A** :	 保证通用性，设计一个Datapath 使之适合所有格式的指令，而不是针对每一个不同的指令就需要重新改变其物理硬件的连接方式



### load word

lw 是 I format 的指令:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316181935968.png?raw=true)

例子：

```yaml
lw rd, imm(rs1)
```

作用:

```yaml
1.将imm 与 rs1做运算
2.访问DMEM，取出数据
3.写回 rd
4.pc = pc + 4
```

根据作用连接Datapath:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316182248071.png?raw=true)

如果想要 load halfword , load byte 需要额外的circuits 去支持从内存中加载数据

---

## Implement S Format inst

### Store word

S format 指令格式与 I format 的区别:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316182740574.png?raw=true)

S format 的 imm 字段被分为了 imm[11:5]和 imm[4:0] ,

将 I format 的 imm字段imm[11:0]的低5位划分一下：

只需要 5bit 的 MUX 在指令中的低5位做选择即可，其他的imm字段被连接到指令的固定位置

sw 的例子：

```yaml
sw rs2, imm(rs1)
```

作用:

```yaml
1.将rs1 和 imm 做运算
2.读取 rs2
3.写入 DMEM 
4.PC = PC + 4
```

连接对应的Datapath


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316183738529.png?raw=true)

图中蓝线表示将 rs2 寄存器的数据写入DMEM中

---

## Implement B Format inst

B format 与 S format 最为相似，B format 有两个source reg rs1 和 rs2, 以及一个12bit 的Immediate字段

由于最后一位不存储的缘故， B format 能跳转的值是 -4096 to 4096 ($ 2^{12} $ ) bytes

branch 指令例子:

```yaml
beq rs1, rs2, label
```

作用:

```yaml
1.比较 rs1 与 rs2 ，得出结果
2.要么 PC = PC + 4
3.要么 PC = PC + imm
(imm = label - PC)
```

可以看到需要一个比较器Comparator ，PC 与 imm 做运算，需要用到 ALU

连接 Datapath


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316184541103.png?raw=true)

解释:

- 根据PC 从 IMEM 取出指令，取出字段 rs1 rs2 和imm 
- rs1 与 rs2 需要比较，新增一个 Branch Comp单元，结果输出到Control logic 
- BrUn = 1表示无符号数比较，BrEq=1 表示 rs1 = rs2, BrLT = 1 表示 rs1 < rs2
- 不再是rs1 进入ALU 做运算，而是PC 和 imm
- 在PC 和 rs1 之间添加一个MUX 表示选择 PC 做运算而不是 rs1
- 经由ALU 的结果与 PC+4做选择，添加一个MUX ,表示条件满足则跳转PC ，不满足则继续执行下一条指令

---

## Implement J Format inst

### jalr (I format)

jalr实际上是I format 的指令格式

例子:

```yaml
jalr rd, rs1, imm
```

作用:

```yaml
1.rd = PC + 4
将 PC + 4后写回rd
2.PC = rs1 + imm
将 rs1 与 imm 做运算(ALU)后写回PC
```

连接Datapath


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316185924871.png?raw=true)

现在 WBSelect MUX 变成了 0 1 2三个，分别表示 rd 的值可能来自：

ALU : 例如 R format 的rd = rs1 + rs2 的结果写回rd

DMEM : 例如 lw ,rd = DMEM 里面的data

PC+4： 如jalr

PC = x[rs1] + imm 的结果也会与 PC+4做个MUX ，兼容其他指令

### jal

J format 指令:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316190626494.png?raw=true)

例子:

```yaml
jal rd, label
```

作用:

```yaml
1.rd = pc + 4
将 pc + 4的结果写回rd
2.pc = pc + imm (where imm = label - Currenr_PC)
PC 与 imm 做运算(ALU)，结果写回PC 
```

连接Datapath


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316191037146.png?raw=true)

---

## In Conclusion

笔记只是简写，更多内容参考 slides lecture12

设计好的完整的单周期DataPath ，兼容 R I S B J Format

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220316191243062.png?raw=true)