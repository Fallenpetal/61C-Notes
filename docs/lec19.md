---
layout: default
title: Amdahl's Law, Thread-Level Parallelism, OpenMP Introduction
author: liuxing
math: mathjax2
nav_order: 19
---

# Amdahl's Law, Thread-Level Parallelism, OpenMP Introduction
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Amdahl's Law

**陷阱：在改进计算机的某个方面时期望总性能的提高与改进大小成正比**

Amdahl's law阐述了”对于特定改进的性能提升可能由所使用的改进特征的数量所限制“的规则。

Amdahl's law 简单来说就是：你只能加速系统中能被加速的部分，而不能被加速的部分无论如何也不能被加速。

可以使用Amdahl定律来计算程序改进后的执行时间:

$$
改进后的执行时间=\frac{受改进影响的执行时间}{改进量}+不受影响的执行时间
$$

举个简单的例子:
假设一个程序需要在一台计算机上运行100秒，其中有80秒的时间用于乘法操作，如果想要把该程序的运行速度提高5倍，乘法操作的速度应该改进多少？

解：

设改进量为n，其中80s是可改进的时间，20s是不可改进的时间，那么

$$
20=\frac{80}{n}+20 \\
\frac{80}{n}=0
$$

无论如何 $$\frac{80}{n}=0$$ 不成立，因此无论如何改进都不能使得该程序的运行速度提高五倍。

这种情况在日常生活中被称为边际收益递减定律，Amdahl定律则是该定律的量化版本。


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220409222116452.png?raw=true)

边际收益递减可简单理解为越靠近边界时，投入越大回报却越小

### Speedup 加速比

我们可以在加速比的方向上重新定义Amdahl's law:

$$
加速比=\frac{改进前的执行时间}{改进前的执行时间-受优化影响的执行时间+\frac{受优化影响的执行时间}{优化量}}
$$

假设改进前的执行时间在某个单位时间内为1，并且受优化影响的执行时间可以被视作与原始执行时间的比值，那么上述公式可以重写为:

$$
加速比=\frac{1}{1-受优化影响的执行时间比例+\frac{受优化影响的执行时间比例}{优化量}}
$$

记：

$$
F=受优化影响的执行时间比例=\frac{受优化影响的执行时间}{原始执行时间} \\
S=优化量/优化因子
$$

那么，用符号化语言表示为:

$$
Speedup=\frac{1}{(1-F)+\frac{F}{S}}
$$

举例:

假设某程序的一半执行时间能被因子2加速，求加速比？


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220409222038074.png?raw=true)

#### 加速比的挑战: 更大规模的问题

要在并行处理器上获得良好的加速比，同时保持问题的规模不变，比通过增加问题的规模获得良好的加速比更难。

Strong Scaling: 在保存问题规模不变的同时所测量的加速比

Weak Scaling: 问题的规模与处理器数量成比例增长时所测量的加速比

#### 加速比的挑战：Load balancing(负载均衡)

确保每一个处理器完成相同的工作量，如果其中一个处理器所分担的工作过多，那么先完成任务的处理器不得不等待而浪费时间，使性能大打折扣。

---

## Multiprocessor

简单的多核处理器示意图:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220409223019039.png?raw=true)

每一个处理器都有自己的PC和执行一个独立的指令流(MIMD)

不同的处理器访问相同的内存地址，这种模型称为 共享内存多处理器(SMP)

有两种方式去使用多核处理器:

- MIMD: 不同处理器执行不同的指令流和数据流，例如你的操作系统正在运行多个程序（工作级并行性）
- 使用多个处理器处理单个程序

#### 比较Parallelism的类型

- SIMD-type parallelism(Data Parallel)

  一个有利于SIMD 类型的问题很容易映射到MIMD类型

  - 更少的控制逻辑
  - 经典例子:与CPU相比，显卡是巨大的超级计算机：浮点数运算性能是CPU的500x--1000x

- MIMD-type parallelism(data-dependent Branches)

  一个有利于MIMD类型问题并不能轻松映射成SIMD类型

#### 并行性是提高性能的唯一方式


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220409225222944.png?raw=true)

难处:

- 顺序处理器的性能并不如像想象中的提高那么大
- 时钟频率很难再增长
- CPI趋于平缓

优点:

- SIMD 可以做到4-16 word的子字并行
- MIMD : 处理器核心数逐年上升

关键挑战是如何写好并行程序，使得能够更好利用核心数的增长

---

## Threads

线程：执行某些任务的顺序指令流

每一个线程都包括 PC(程序计数器) , 寄存器状态，和栈，并共享进程(process)的地址空间，而进程则不共享，线程是一个轻量级的进程

每一个核都会提供1个或多个硬件线程(**hardware threads**)用于执行指令，例如常见的 Intel芯片提供 2 threads/core

操作系统会基于硬件线程复用多个软件线程(software threads)

#### Operating System Threads

- 通过在时间上复用多个硬件线程，给与你一种有许多线程在同时活跃的假象

**如何去实现？**

- 通过中断硬件线程的执行并将寄存器和PC 保存至内存，可以从某硬件线程上移除软件线程
  - 例如某个线程因等待网络连接或等待用户输入而阻塞，可以切换到其他线程去

- 通过将其寄存器载入硬件线程并跳转至其对应的PC，可以使软件线程重新活跃

#### Hardware Multithreading

核心思想：处理器的资源是如此的珍贵以至于我们不应该让它闲置，例如cache miss时的memory latency 影响很大

在等待cache miss 的时间里切换到另一个硬件线程从而减少损失，因为上下文切换(context switch)所花费的时间远小于cache miss

那么增加PC 和 Registers的硬件数量，我们可以在每次切换线程时不必保存上下文

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220409232847062.png?raw=true)

也就是同一个Core内复制两份PC 和 Registers File

#### 多线程 VS 多核

Multithreading:

只需1%更多的硬件，能获得1.1x以上的加速

共享整数加法器，浮点数运算单元，所有的Caches(L1 , L2, L3),以及main memory

Multicore:

≈50% more hardware, ≈2X better performance?

共享外部Caches(L2, 或者 只有L3),以及main memory

**现代处理器大小核的设计思想**

有时候你需要运行大型程序，你需要用到大核，但是当你处理小事情的时候，不需要用到大核，小核足够且更加节能。

---

## OpenMP

OpenMP是C 语言的扩展，一套支持共享内存多处理的API，包括:

- 编译器指示(Complier Directives)
- 运行时库(Runtime Library Routines)
- 环境变量(Environment Variables)

大多数C语言编译器现已支持OpenMP，使用OpenMP API的指令是


gcc -fopenmp source.c |


也就是加flag -fopenmp

使用OpenMP的核心思想是：

- 注意共享变量与私有变量
- 用于并行化、工作共享、同步化的OpenMP指令

#### 基于线程并行性的共享内存模型

共享内存环境中的多线程，明确的编程模型，程序员可以完全控制并行化的情况

优点:

- 利用共享内存，我们无需担心数据的处地
- 编译器指示(Complier Directives)使用起来很简单
- 遗留的串行代码不需要重写

缺点：

- 代码只能在共享内存环境中运行
- 编译器必须支持OpenMP, 例如gcc 4.2版本以上
- 存在边际收益递减(Amdahl's law), 也就是处理器的核数增加未来优化不明显

#### OpenMP 编程模型--Fork - Join Model


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220410102621266.png?raw=true)

OpenMP程序以单进程（主线程）开始，并按顺序执行，直到遇到第一个并行区域结构。

FORK：主线程创建一个并行线程队

- 程序中被并行区域所包围的代码块在各个线程之间并行执行

JOIN：当线程队完成并行区域中的代码块执行时，它们同步并终止，只留下主线程。

#### OPenMP 的使用

确保两点:

在头文件加入:

```c
#include<omp.h>
```

编译时加flag:

```
gcc -fopenmp
```

OpenMP使用 Pragmas 扩展 C 语言

Pragmas 是C 为语言扩展提供的预处理机制，相当于C 语言中的宏指令，如 #define，#include

如何构建Fork-Join模型中的并行区域？

基本语法如下:

```c
#pragma omp parallel
{
	/* code goes here */
}
```

<font color='red'>请注意：左大括弧{   必须与#pragma语句单独隔一行，也即是二者不能在同一行</font>

那么每个线程都会复制一份并行区域内的代码并运行，在这里，线程调度(Thread scheduling)是非决定性的



OpenMP默认共享变量，为了使用私有变量，需要声明:

```
#pragma omp parallel private(x)
```

#### OpenMP与线程

- OpenMP 线程是操作系统线程(software threads), 操作系统将会在硬件线程上复用发出请求的
- OpenMP线程，并希望每个OpenMP线程都能在真实的硬件线程上运行，因此并不是操作系统级时间复用(OS-level time-multiplexing.)
- 但是在计算机上的其他任务也能使用硬件线程
  - 如果你有大量的I/O, 你可能想要比硬件线程更多的线程，这样子在等待I/O的时候能让其他线程运行
- Be careful when timing results!

#### 使用OpenMP创建线程

设置你想要OpenMP使用的线程数量，最大等于 物理上的核数×每核支持线程数

```
omp_set_num_threads(x);
```

查看当前线程数:

```
num_th = omp_get_num_threads();
```

每个线程都有一个唯一的ID, 查询线程ID:

```
th_ID = omp_get_thread_num();
```



#### 并行执行Hello World


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220410110459471.png?raw=true)

---

## Lock Synchronization

#### 数据竞争(Data Races)

​	如果来自两个不同的线程的访存请求访问同一个位置，且至少有一个是写，且连续出现，那么这两次存储访问形成了数据竞争

​	当任务之间相互独立时，并行执行更为容易，但通常任务之间需要协作。协作意味着一些任务正在写入其他任务必须读取的值。需要知道任务何时完成写入以便其他任务安全地读出，因此任务之间需要同步。如果它们不同步，则存在数据竞争，程序结果会根据事件发生次序而改变。

**例子：买牛奶**{: .label}

冰箱里没有牛奶了，你准备去买牛奶，当你去买牛奶的时候你室友回来也看到冰箱里没牛奶了，于是他也去买牛奶，结果你们两个都买了牛奶，导致多于实际所需牛奶

解决方法是什么？去买牛奶之前留一张便签

#### Hardware Synchronization

在多处理器中实现同步所需的关键是一组硬件原语(hardware primitives)，能够提供以 **原子**的方式读取和修改内存。

原子操作：在内存单元的读取和写入之间不能插入任何其他操作

如何在软件中实现？

两种方法：

单指令: 寄存器和内存之间的原子交换(Atomic swap)

一对指令: 一个用于读，一个用于写

首先来看单指令的实现

#### Option1 :Read/Write Pairs

**第二条指令**返回一个值，该值表示该**指令对**(instruction pair)是否被原子执行。如果任何处理器执行的所有其他操作都发生在该对指令之前或之后，则该指令对实际上是原子的。因此，当指令对实际上是原子操作时，没有其他处理器可以在指令对之间改变值。

在RISC-V 中的实现为:

- Load reserved : 

  ```yaml
  lr rd, rs
  ```

  从rs指向内存处加载值到rd，并添加**保护**(reservation)

- Store conditional:

  ```yaml
  sc rd, rs1, rs2
  ```

  将rs2中的值写入rs1所指向的内存处，只有在lr 指令的保护有效时才会写入成功，并设置rd的值

  - 如果成功设置rd为0，失败设置rd为非0值

这对指令按序使用：

​	如果lr 指令中rs所指向的内存中的值，在sc指令执行到同一地址之前发生了改变，那么sc指令执行失败，且不会将值写入内存，返回非0值到rd

因此保证了lr 和 sc 指令执行之间不会有来自其他线程的lr sc指令执行冲突，实现原子交换:

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220410223006820.png?raw=true)

#### Option2: single instruction

0 表示lock is free / open / unlocked / lock of

1 表示 lock is set / closed / locked / lock on

处理器尝试通过将寄存器中的1与该锁变量对应的内存地址的值进行交换来设置加锁。

- 如果某个其他的处理器已声明访问该锁变量，则交换指令返回值为1，表面该锁已经被占用
- 否则为0，表示加锁成功，并把锁变量设为1，防止其他处理器也加锁成功

如果两个处理器尝试同时进行交换操作，这种竞争会被阻止，交换是不可分割的，硬件将对两个同时发生的交换进行排序。

上述操作也被称为Test-and-Set模式:

- Test : 查看锁变量的值是否为1,如果为1，锁变量被占用，try again
- Set : 如果为0，那么将其设为1，否则意味着被抢先加锁，加锁失败，try again


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220410225823393.png?raw=true)

```yaml
li t2, 1
Try: lr t1, s1
    bne t1, x0, Try
    sc t0, s1, t2
    bne t0, x0, Try
Locked:
    # critical section
Unlock:
    sw x0,0(s1)
```

#### Option3: Atomic Memory Operations

```yaml
AMOSWAP rd, rs2, (rs1)
AMOADD rd, rs2, (rs1)
```

除此之外还有And,or,xor,max,min都支持：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411083229804.png?raw=true)

---

## OpenMP： for循环并行

例如现在要计算

```c
for（i=0; i<max; i++)
	zero[i]=0;
```

将循环内容分为多块，分配到不同的线程：

例如，如果max=100，可以把0-49分配给线程1，50-99分配到线程2

使用Fork-Join 模型并行执行循环:

```c
#pragma omp parallel for
for (i=0; i<max; i++)
	zero[i]=0;
```


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411083827819.png?raw=true)

主线程创建额外的线程，每个线程都有独立的上下文(execution context)

在循环体外面声明的变量默认是共享变量，循环index（例如i)对于每个线程来说默认是私有的

每个线程将自动分配区域：

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411084109491.png?raw=true)

---

## 最终练习: 计算π


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408170712586.png?raw=true)

将[0,1]分为多块，在线程队并行计算每一块的值，然后最终在主线程做总和：

其中num_steps是分块数，step相当于dx:

```c
#include <omp.h>
#define NUM_THREADS 4
static long num_steps = 100000; double step;
void main () {
    int i; double x, pi, sum[NUM_THREADS];
    step = 1.0/(double) num_steps;
#pragma omp parallel private ( i, x )
{
    int id = omp_get_thread_num();
    for (i=id, sum[id]=0.0; i<num_steps; i=i+NUM_THREADS)
    {
        x = (i+0.5)*step;
        sum[id] += 4.0/(1.0+x*x);
    }
}
    for(i=1; i<NUM_THREADS; i++)
        sum[0] += sum[i]; pi = sum[0];
    printf ("pi = %6.12f\n", pi);
}
```

计算结果:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411085950017.png?raw=true)

这里的sum变量是共享变量，是最终在主线程上做累加sum[i]的操作，那么我们能不能让他并行计算sum呢？

有一个想法是：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411090225710.png?raw=true)

这样做对不对呢？

答案是错误的，因为这个操作相当于:

```c
pi = pi + sum[Thread-id];
```

如果当前有1个以上的线程同时在读取pi，那么相当于在执行：

```
pi = pi + sum[Thread1];
pi = pi + sum[Thread2];
......
```

发生了数据竞争，*A “race” --> result is not deterministic but if we locked this we'd lose almost all speedup.*

#### Solution: OpenMP Reduction

将sum设为私有变量

```c
#pragma omp parallel for private (sum)
```

然后这样写：

```c
double avg, sum=0.0, A[MAX]; int i;
#pragma omp parallel for private ( sum )
for (i = 0; i <= MAX ; i++) {sum += A[i];}
avg = sum/MAX; 
```

对不对呢？

答案是错误，因为当sum私有之后，每个线程都有一个sum，上述操作相当于并行执行:

```c
sum = A[0];
sum = A[1];
sum = A[2];
...
```

虽然这次不再有数据竞争，但是最后结束并行时，剩下的sum只是主线程的sum，也就是

```c
sum = A[0];
```

最终被保留，那么 avg = sum/MAX自然就错了。



解决方法是执行 **归约(Reduction)**

归约: 将每个并行线程的一个或多个私有变量在并行区域结束时做某种操作，相当于聚合函数(Aggregation functions)

语法:

```
reduction(operation: var)
```

- Operation:

  执行的操作，如+，*，-，min，max，&，&&，|，||，^

- Var:

  执行操作的一个或多个变量

使用归约之后:

```c
double avg, sum=0.0, A[MAX]; int i;
#pragma omp for reduction(+ : sum)
for (i = 0; i <= MAX ; i++) {sum += A[i];}
avg = sum/MAX;
```

相当于在并行结束的时候，

```c
sum = A[0]+A[1]+A[2]+......
```

把所有线程的结果汇总到主线程，然后取平均值。

<font color="red">NOTE : 这里似乎是另外的例子了，因为计算π我们并没有用到A[i],如果按照前面说的分块，最后累加即可，并不需要取平均值</font> 

回到计算π的例子，执行归约之后:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220411092815616.png?raw=true)

