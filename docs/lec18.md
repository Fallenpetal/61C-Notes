---
layout: default
title: Flynn's Taxonomy, Data-Level Parallelism
author: liuxing
math: mathjax2
nav_order: 18
---

# Flynn's Taxonomy, Data-Level Parallelism
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Parallel Processing

由于技术和经济的挑战，以及昂贵的能量花费，CPU Clock Rates 不再增长

并行处理是唯一提高速度的方式

有两种基本的并行方式:

- Multiprogramming 

  在多个处理器上并行地执行各个独立的程序

  - easy

- Parallel computing

  在多个处理器上并行执行一个程序

  - hard

作为程序员我们当然希望以Parallel computing 的方式运行单个程序，That make faster

---

## Flynn's Taxonomy

根据指令流和数据流的数量，可以分为**SISD**(Single-Instruction/Single-Data Stream), **SIMD**(Single-Instruction/Multiple-Data Stream), **MISD**(Multiple-Instruction/Single-Data Stream) 和 **MIMD**(Multiple-Instruction/Multiple-Data Streams)

也就是弗林分类法


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408170712586.png?raw=true)

### SISD

指并未在数据流和指令流中利用并行性的顺序计算机，SISD体系架构的例子是传统的单处理器计算机
![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408194254140.png?raw=true)

### SIMD

SIMD计算机使用单指令流处理多数据流，例如 Intel SIMD 指令扩展 或 NVIDIA GPU


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408194428056.png?raw=true)

### MIMD

多个自主处理器同时对不同数据执行不同指令, 例如多核计算机和仓储级(Warehouse-Scale)计算机


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408194650766.png?raw=true)

### MISD

MISD计算机在单一数据流上执行多个指令，no example

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408194810844.png?raw=true)


在体系结构中最常见的并行性使用是SIMD 和 MIMD

尽管可以在MIMD的计算机上编写运行在不同处理器上的独立程序，但是，为了实现更宏大的目标，程序员通常会编写一个运行在MIMD计算机中所有处理器上的程序，不同的处理器通过条件语句执行不同的代码段。这种变成风格称为**单程序多数据流**(Single Program Multiple Data )(“SPMD”)，它是在MIMD计算机上变成的常用方法(------摘自P & H)

---

## X86中的SIMD

**parallelism and computing : subword parallelism**

SIMD 对数据向量进行操作，模式为:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408200225562.png?raw=true)

例如，通过将128bit 的加法器内划分进位链，处理器可以同时对16个8bit 操作数，或者8个16bit 操作数，4个32bit 操作数，2个64bit 操作数进行**并行操作**

将这种在一个宽字内进行的并行操作称为子字并行(subword parallelism)，更通用的名称是数据级并行(data level parallelism)

**AVX SIMD Registers**


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408200328983.png?raw=true)

**SIMD Data Types**


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408200410682.png?raw=true)

---

## SIMD x86 Intrinsics



**Intrinsics** : 从C 直接访问汇编（避开了C 经过 Compiler 变成汇编语言）

### x86 Intrinsics AVX Data Types

- __m256  将256bit 当作八个单精度浮点数，代表YMM寄存器或内存位置 |
- __m256d  将256bit当作4个双精度浮点数，代表YMM寄存器或内存位置 |
- __m256i   将256bit 当作integers，bytes , words等等 |
- __m128  4个单精度浮点数 |
- __m128d  2个双精度浮点数 |

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408202508284.png?raw=true)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408202353426.png?raw=true)

具体的intrinsics代表的x86指令和并行性，可以参考[Intel Intrinsics Guide][https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#techs=AVX&ig_expand=4936]

各种说明如下:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408202719772.png?raw=true)

---

## 优化DGEMM矩阵乘法

### python版本

接下来让我们优化双精度浮点数通用矩阵乘法(dgemm)

该乘法在python中的代码为:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408203003231.png?raw=true)

假设  1 MFLOP = 每秒一百万次浮点操作

那么在python中的MFLOP效率为:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408203111025.png?raw=true)

### C语言版本

使用C语言写dgemm，代码如下:

```c
• c = a * b
• a, b, c are N x N matrices

void dgemm_scalar(int N, double *a, double *b, double *c){
 int i,j,k; double cij;
 for(i = 0; i < N; ++i){
     for(j = 0; j < N; ++j){
         cij = 0
         for(k = 0; k < N; ++k){
             cij += a[i+k*N] * b[k+j*N];
         }
         c[i+j*N] = cij;
 }
 }
}
```

通过在执行前后两次调用clock()函数，可以得出处理时间


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407191125924.png?raw=true)

与python相比，C的执行效率:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408203427613.png?raw=true)

现在让我们使用intrinsics来写dgemm

也就是写一个可以并行处理的程序，而非原来的顺序处理的版本

### 向量化dgemm

矩阵乘法中所需的乘法操作，在intrinsics 中为:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408204109025.png?raw=true)

为了利用__mm256_mul_pd 的 4 并行性，也就是同一时刻可以执行4次for()循环，因此将迭代下标i++改为 i+=4

向量化之后的dgemm:

```c
void dgemm_avx(int N, double *a, double *b, double *c){
 int i,j,k; __m256d c0;
 for(i = 0; i < N; i += 4){
     for(j = 0; j < N; ++j){
         c0 = {0,0,0,0}
         for(k = 0; k < N; ++k){
             c0 = __mm256_add_pd(
             c0,
             __mm256_mull_pd(
             __mm256_load_pd(a+i+k*N),
             __mm256_broadcast_sd(b+k+j*N)));
             }
         _mm256_store_pd(c+i+j*N, c0);
         }
     }
}
```

性能提升:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408204608521.png?raw=true)

当前还存在哪些东西在阻碍着优化？

- Amdahl's Law:
  - 只能加速能加速的事件，not floating-ALU无法被加速

- 存在Hazards
- Caches

**向量化矩阵存在的流水线冒险**

查看内部循环内部循环:

```c
for(k = 0; k < N; ++k){
     c0 = __mm256_add_pd(
         c0,
         __mm256_mull_pd(
             __mm256_load_pd(a+i+k*N),
             __mm256_broadcast_sd(b+k+j*N)));
 }
```

add_pd 依赖于 mult_pd的结果，mult_pd 依赖于 load_pd 的结果，存在数据冒险(data hazard)

那么使用循环展开，可以合并不相关操作消除冒险



### 循环展开(Loop Unrolling)

循环展开是一种针对数组访问循环体的提高程序性能的技术。它将循环体展开多遍，从不同循环内寻找可以重叠执行的指令来挖掘更多的指令级并行性

- 对向量指令(SIMD)或超标量多发射指令暴露数据级并行性
- 将不相关操作混合流水线以减少hazards
- 减少循环"overhead"

例如:

```c
for(i=0; i<N; i++)
	x[i] = x[i] + s;
```

循环展开后变成:

```c
for (i=0; i<N; i+=4) {
    x[i] = x[i] + s; 
    x[i+1] = x[i+1] + s; 
    x[i+2] = x[i+2] + s; 
    x[i+3] = x[i+3] + s;
}
```

对于SIMD 的dgemm:

展开前的操作:
```yaml

load--> mul-->add

load--> mul-->add

load--> mul-->add

load--> mul-->add
```

展开后的操作:
```yaml

load-->load-->load-->load

mul-->mul-->mul-->mul

add-->add-->add-->add
```

注意i+=4变成了i+=UNROLL*4


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408212447071.png?raw=true)

性能表现:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408212534838.png?raw=true)

随之而来的问题是，当矩阵的规模N 超过480之后，为何性能急剧下降?

原因是超过了Cache 的大小，增加了Miss Rate

### Blocking

上节课已经提到过了分块思想，也补充了15-213的笔记，因此不再赘述


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408214315076.png?raw=true)

代码:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408214426950.png?raw=true)

最终性能表现:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220408214457414.png?raw=true)