---
layout: default
title: Introduction to Synchronous Digital Systems (SDS)
author: liuxing
math: mathjax2
nav_order: 11
---

# Introduction to Synchronous Digital Systems (SDS)
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}



## Clock Signal

常用单位是Hz,常见的时钟频率是4 GHz,因为
$$
\begin{aligned}
Period = \frac{1}{frequencey}
\end{aligned}
$$
所以常见周期是 0.25 ns

Period(周期) ：从一次上升沿到下一次上升沿的时间

时钟信号分为上升沿触发和下降沿触发

---

## 时延的存在

考虑一个累加器，该累加器是做加法后再将结果反馈到输入端进行累加，假设累加器计算出结果需要5ns的时延，由于时延的影响，得到的结果与我们的期望不同


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314091215199.png?raw=true)

在上一次输入的结果还没有算出来之前，就已经有下一次输入了，因此我们要做的是，等到上一次计算完毕之后才进行新的输出(类似于向后端请求数据，拿到后才能进行下一步操作)

实现这样想法我们需要使用 D 翻转器(D-flip-flop)

---

## D Flip-Flop

- Synchronous(同步) ：取决于时钟信号
- Asynchronous(异步)：独立于时钟信号
- 一个翻转器是一个 state element
- State Element：一个能够存值的电路元件


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314091849422.png?raw=true)

R 和 S  异步控制端，与时钟信号无关，S=1，设置Q=1,R=1，设置Q=0

输入为 D ,输出为 Q ,下面输出  $ \bar{Q}$ 

D 触发器不改变输入，只是起到暂存的作用，输入D 是什么状态，输出Q 就是什么状态，特征方程为
$$
Q = D
$$
时钟信号又分为上升沿触发和下降沿触发

- Rising edge triggered
  - 当时钟信号从0到1的瞬间接受D 的输入，输出为 Q
- Falling edge triggered
  - 当时钟信号从1到0的瞬间接受D 的输入，输出为 Q


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314090805856.png?raw=true)

### Three Delays

由于寄存器设计本身的物理特性，有着3个时延：

Clock-to-Q delay ：从D输入一个状态到从Q输出所花费的时间

Set-up Time : 在时钟信号到来**之前**输入端需要维持状态保持稳定的时间

Hold Time ： 在时钟信号到来**之后**输入端需要维持状态保持稳定的时间


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314093222741.png?raw=true)

#### Example：上升沿触发


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314093409757.png?raw=true)

---

## Register

如何构建一个32bit 的寄存器？

将32个翻转器放在一起，则成为寄存器，寄存器也是 state element


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314093849844.png?raw=true)

### Write Enable


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314094013992.png?raw=true)

如图为寄存器的抽象符号表示，分为数据输入端，输出端，时钟信号，Write 使能端， R 端(R = 1,寄存器清0)

输入是32个 1 bit 数同时输入，组成32bit 数字，Write Enable 表示寄存器是否能被写入数据

- 当 Write enable = 0,寄存器维持原值
- 当 Write enable = 1,寄存器在时钟信号到来时更新数据

使用多路选择器MUX 实现该效果:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314094343640.png?raw=true)

---

## Max hold time & Min clock cycle time

Combinational logic delay : 数据通过组合逻辑电路传播所花费的时间

例如，通过一个反向器的时延是2ns，那么两个反向器的时延就是4ns



hold time 就是在时钟信号到来之后维持输入稳定的时间

如何定义 maximum hold time？

**对于某个寄存器Reg 来说，maximum hold time 是接收到时钟信号后，寄存器Reg 的输入发生变化所需的时间**

例如，对下图寄存器B 来说，maximum hold time 是?

- 时钟信号之后，从A 接收输入保持稳定直至B 接收输入的时间

$$
Max\space hold\space time = clk-to-q\space delay+Shortest \space CL \space time
$$


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314100044914.png?raw=true)



**minimum clock cycle time--最小时钟信号周期：**

因为时钟周期的定义是：从一个上升沿至另一个上升沿的时间(positive example)，因此最小时钟周期就是要**保证在下一个时钟信号到达时，寄存器刚好可以接受输入**，即从一个state element的输入至下一个state element的输入所需时间

例如下图中，从寄存器B 接受输入至寄存器 A 接受输入所需时间即为一个最小时钟周期

计算公式 是

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314101831350.png?raw=true)




![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314101916468.png?raw=true)

从寄存器B 接受输入开始算起，经过一个Clk-to-q 时延，经过两个 inverter 时延，经过寄存器A的Set-up时延，此时寄存器A 可以接受输入，刚好可以开启时钟信号

---

## Design PC register


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314102607737.png?raw=true)

例如指令 **beq rs1, rs2 ,target-address** 将指令和寄存器的值做&,

我们知道，如果不同时满足当前指令instr == beq 且 rs1==rs2，PC不会跳转到新的branch，而是继续下一条指令的执行，一条指令是4byte ，因此下一条指令pc = pc + 4

使用多路选择器MUX 实现，S=1表示branch to new address，S=0表示继续执行下一条指令，不跳转

---

## Combinational vs Sequential Logic

组合逻辑:

- 只要有输入，输出就可以里面被计算
- 输出只取决于输入

时序逻辑：

- 与时钟信号同步
- 输出取决于先前状态和输入

---

## Metal-Oxide Semiconductor Field Effect Transistor

### MOS FET

- Source 源极：输入端
- Gate 栅极：控制开关的打开/关闭
- Drain 漏极：输出端

分类:

NMOS 与 PMOS

#### NMOS


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314103522122.png?raw=true)

Gate = 1，开关闭合，线路允许信号通过，输出 D = S

Gate = 0,开关断开，线路不允许信号通过，输出未知

由于物理特性，NMOS不擅长传输信号”1“，因此我们将S 接”0“

#### PMOS


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314103837053.png?raw=true)

正好与 NMOS 相反

Gate = 0，开关闭合，线路允许信号通过，输出 D = S

Gate = 1,开关断开，线路不允许信号通过，输出未知

由于物理特性，PMOS不擅长传输信号”0“，因此我们将S 接”1“

#### 两个符号，Vdd 与 Ground

Vdd 表示高电平(逻辑1)

Grund表示接地(逻辑0)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314104124096.png?raw=true)

---

## Building Gates with NMOS / PMOS Transistors

### 构建非门

写出反向器的真值表:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314104646194.png?raw=true)

利用 NMOS 的特性，将栅极Gate 作为 A ，S端(源极) 接0：

**A = 1，开关闭合，输出为0**

A = 0，开关打开，输出未知

利用 PMOS 的特性，将栅极Gate 作为 A ，S端(源极) 接1：

**A = 0，开关闭合，输出为1**

A = 1，开关打开，输出未知



由于NMOS 和 PMOS 只在一种情况下有输出，而另一种情况输出未知，但是二者正好互补，因此可以结合二者的特性完成反向器

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314105014989.png?raw=true)

这种NMOS 与 PMOS 互补(Complementary)的结果称为 CMOS，PMOS接入高电平，称为Pull-up network,NMOS 接入低电平，称为 Pull-down network

#### CMOS :

- 使用NMOS 与 PMOS 互补完成逻辑功能
- 由Pull-up network 和 Pull-down network构成

---

### 构建与非门

与非表达式
$$
\overline{A B}
$$
写出真值表：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314105938352.png?raw=true)

可以发现，当A  B 同时为1，输出为0，可以使用两个开关(1表示开关关闭)串联接0实现，对应串联的NMOS


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110033906.png?raw=true)

当 A 或 B 至少一个为0时(0表示开关关闭)，输出为1，可以用两个开关并联接1实现，对应并联的PMOS


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110155795.png?raw=true)

合成上拉网络和下拉网络:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110339526.png?raw=true)

---

### 构建与门

由于上面我们构建了非门与与非门，那么与门:
$$
AB=\overline{\overline{A B}}
$$
只需将与非门的输出当作非门的输入即可


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110554005.png?raw=true)

同理可先构建或非门，再构建或门(slide上有例子，就不赘述了，不懂时看slide即可)

---

### 构建或非门

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110807353.png?raw=true)

### 构建或门


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110830173.png?raw=true)

---

## Reduce Transistors

统计一下，可以发现


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314110923898.png?raw=true)

带”非“的门使用的晶体管数量比不带"非"的门少

考虑DeMorgan's Law，二次取反

把与门 和 或门变成与非/或非的形式
$$
A B=\overline{\overline{A B}}=\overline{\bar{A}+\bar{B}}
$$

$$
A+B=\overline{\overline{A+B}}=\overline{\bar{A} \cdot \bar{B}}
$$

例子:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220314111630814.png?raw=true)