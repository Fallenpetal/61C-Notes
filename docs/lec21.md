---
layout: default
title: Operating Systems
author: liuxing
math: mathjax2
nav_order: 21
---

# Operating Systems
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## What is an Operating System

生活中常见的操作系统有：

- Unix
  - Berkeley Software Distribution
  - macOS
- Linux distribution (Linux 发行版，开源)
  - Debian
  - Ubuntu
  - Red Hat
- Microsoft Windows



那么什么是操作系统呢？

操作系统是“魔术师”：为硬件资源提供干净，使用简便的抽象


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427201710339.png?raw=true)

如图，

Processor 抽象成 Threads

Memory/Cache 抽象成 Address Spaces

Disk 抽象成File

网络设备抽象成 Sockets

已编译并运行的程序 抽象成 Process





进程(Process)是程序的一次执行过程，也就是说程序运行起来了，加载到了内存中，并占用了cpu的资源。这是一个动态的过程：有自身的产生、存在和消亡的过程，这也是进程的生命周期。

进程是系统资源分配的单位，系统在运行时会为每个进程(每个Compiled Program)分配不同的内存区域。

每个Process 有自己独立的Threads,Address Spaces,Files 和 Sockets

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427202421203.png?raw=true)



**线程（thread）**

进程可进一步细化为线程，是一个程序内部的执行路径。

若一个进程同一时间并行执行多个线程，那么这个进程就是支持多线程的。

线程是cpu调度和执行的单位，每个线程拥有独立的运行栈和程序计数器（pc），线程切换的开销小。

一个进程中的多个线程共享相同的内存单元/内存地址空间。

### Context Switch

上下文切换：从一个正在执行的程序切换到另一个

允许多个Process 在同一个 Processor 上允许，由OS 来决定何时进行context switch

进行context switch 的过程是：

1. OS从当前的进程中接手CPU的控制权
2. OS将当前进程的状态保存
3. OS加载下一个进程的状态
4. OS 将CPU的控制权交给下一个进程



操作系统也是”裁判“：管理 保护，孤立，和共享资源：

每个Process有操作系统为之分配好的Address Spaces，如果另一个Process非法访问了不属于它的地址空间，会发生 Segment Fault:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427203131849.png?raw=true)


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427203142788.png?raw=true)



这也就是 Protection:

OS 将自己 ，以及其他各个Processes 相互孤立，好像各进程之间不知道有其他进程存在似的，尽管它们实际上运行在相同的硬件上

---

## Dual Mode Operation

硬件至少提供两种模式：

1. Kernel Mode(supervisor mode)
2. User Mode

在User Mode 下的某些操作是被禁止的，例如：与硬件直接交互，向kernel memory直接写入

在绝大多数情况下，OS在 User Mode 下运行，不过某些操作不得不切换至Kernel Mode：

System calls, interrupts, exceptions



在User Mode下，如果我们的程序因为某些原因需要在kernel mode 下运行，则将不得不把控制权转交给OS，然后OS执行用户想要的，并将控制权还给用户，因此事实上用户永远不能亲自在kernel mode 下进行操作

### System Calls(syscall)

允许程序向操作系统请求服务：

- 新建/删除文件
- 读/写文件
- 访问外部设备，如扫描仪，打印机
- RISC-V 中 的ecall 指令

与函数调用类似，只不过由内核(Kernel)执行



例如，user mode 下，第$ i_{s} $ 条指令是系统调用，那么则会跳转到kernel mode下执行一系列指令，然后再回到 第 $ i_{s+1} $ 条指令


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427204529419.png?raw=true)

---

## Interrupts vs Exceptions

### Interrupts

对于当前正在运行的程序而言，由外部事件引起中断

例如：Youtube 播放视频时，按下空格键暂停

对当前程序来说是异步的:

- 不需要被立即处理，但是需要稍后处理，例如流水线中，等某条指令完全执行完毕再进行中断



### Exceptions

在当前程序正在执行期间，由某event引起的

例如：非法指令，除以0

同步：必须立即被处理

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427205233416.png?raw=true)

例如有 PC 地址异常，非法Opcode, 数据地址异常 等等

### 如何处理中断和异常？

Trap Handler: 专门处理中断和异常的程序

从程序的角度来看，要保证看起来”什么都没有发生“

Traps:

- 在错误指令之前的所有指令必须被完成
- 在错误指令之后的指令必须被冲刷(flushed)
- 该条错误指令必须被冲刷(flushed)
- 开始执行trap handler



Trap Handler 是怎么做的？

1. 保存当前程序的状态（所有的寄存器，包括PC)
2. 寻找造成中断或异常的原因
3. 处理中断或异常
   1. 如果处理好了，恢复程序的状态，将控制权返还给程序
   2. 否则终止程序，释放资源，并调度新的程序



例子：

$$i_{a}$$ 是造成interrupts/exceptions的指令


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427205859169.png?raw=true)

一般来说，能够继续执行的程序是

- interrupts
- Certain memory exceptions

需要终止的程序是

- Illegal instruction
- Certain illegal memory accesses

从该程序的角度来看，似乎什么都没有发生：

因为trap handler处理完毕后还原程序状态，唯一的变化可能是

- 当前指令与下一条指令的gap 变大了
- Cache 可能被销毁了
  - Because something else was using them

---

## Other OS Knowledge

### Kernel


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427210503912.png?raw=true)

### Shell


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220427210525970.png?raw=true)

### Boot

当你的电脑开机的时候会发生什么？

1. BIOS(Basic Input/Output System) 运行
   - Power-on-self-test(POST)
     - 类似于检测你的键盘是否正确连接，这是最基础的输入设备
   - 执行bootloader
2. bootloader 加载部分操作系统
3. 操作系统初始化服务，驱动等等
4. 启动进程

Bootstrapping: 一系列调用链，在每一阶段，先加载一个小而简单的程序，下一阶段加载更大更复杂的程序

### 你如何开始执行一个程序？

Loader: 将程序载入内存

1.loader 将程序载入内存

2.loader 设置 **argc ** 和 **argv**

3.OS 跳转到 main 函数，并将控制权转交给process



### 总结：What is an Operating System?

- 在不同运行中的Processes间提供 孤立
  - 每个程序在自己的世界运行
- 提供与外部世界交互的手段
  - 例如鼠标，显示器，网络
- 将物理资源变成干净，易用的抽象层
  - Masks limitations(next Lecture: Virtual Memory)
  - 更高阶的对象：files,sockets
- 管理 保护，孤立，和共享资源
  - 资源的分配和交流