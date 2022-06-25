---
layout: default
title: Virtual Memory I
author: liuxing
math: mathjax2
parent: Virtual Memory
nav_order: 1
---

# Virtual Memory I
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Probelms with Memory

为什么要引入虚拟内存，先从三个问题说起

### Not Enough Space

RISC-V 32 的版本用32bit 对地址空间进行编址，那么可访问多大的内存呢？

- $$ 2^{32} bytes =  2^{2}*2^{30} bytes = 4 GB $$

如果电脑的物理内存RAM 只有1GB 大小的话，如下图一一映射访问地址，会发现超过1GB 以后的RAM没有对应的地址了

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430150420679.png?raw=true)



### Holes in Address Space

假设我们的RAM是4GB 大小


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430150517839.png?raw=true)

现在有三个程序，所需RAM空间分别为1GB, 2GB, 2GB

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430150637972.png?raw=true)

假设我们先顺序运行程序1 ，程序2


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430150727334.png?raw=true)

然后再退出程序1


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430150748122.png?raw=true)

现在RAM剩下2 GB 的空间，而程序3刚好也是2GB ，但是由于程序运行是需要连续的地址空间，RAM的2GB是被分段的，一个1GB 在上面，一个1GB在下面，所以就算RAM空间足够容纳程序3，也运行不了(Memory Fragmentation)

### Ensuring Protection from Other Programs

第三个问题是要保护进程间的孤立性，我们之前说了每个程序都有自己的地址空间，**在每个程序自己看来，它们能访问任何32bit 的地址**

例如你写了2个程序，程序1 需要访问address 1024处的值，程序2也可能有段代码是需要访问address 1024处的值，那么它们会发生相互覆盖

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430151349594.png?raw=true)



这也就是当前存在的三个问题，下面我们将通过引入虚拟内存的方式解决问题

---

## Solving 3 Problems with Virtual Memory

虚拟内存就是假想中的内存，真实并不存在，也就是让每个程序都认为自己有一个可访问的32bit 地址空间，虚拟内存与真实物理内存之间通过某种映射一一对应。

### Solving Problem #1: Not Enough Memory


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430151943476.png?raw=true)

当真实的RAM空间不够时，也就是当虚拟内存大于物理内存时，多于的部分可以映射到磁盘(disk)上，相当于一个指向磁盘的指针

当需要的数据在磁盘上时，且RAM 已满时，从磁盘将数据载入RAM ，并选择一个RAM address替换

#### VM Performance

当需要的数据在磁盘上不在RAM 里时，会造成程序性能下降，因为访问磁盘是极其昂贵的，这也就是为什么买一台RAM 足够大的电脑性能更佳

### Solving Problem #2: Holes in Address Space

当出现Memory Fragmentation时


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430152656361.png?raw=true)

我们可以对虚拟内存与物理内存之间建立一种映射关系：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430152804576.png?raw=true)

将程序3上部分映射到RAM 上方1GB 空间，下部分映射到RAM 下方1GB 空间

在程序3 看来，在它的虚拟内存中，地址空间仍然是连续的，通过映射解决Memory Fragmentation

### Solving Problem #3: Keeping Programs Secure

程序1 和程序2 访问相同的address 1024，在它们看来它们能够访问32bit 地址空间的任意位置，我们只需将它们相同的两个虚拟地址 映射到 真实物理内存的不同地址


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430153242644.png?raw=true)

当然，如果两个程序共享一些library(例如两个程序都有<stdio>库，那么它们也可以映射到相同的物理地址（如下图黑色线条）


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430153406637.png?raw=true)