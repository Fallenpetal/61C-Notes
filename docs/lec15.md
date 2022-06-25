---
layout: default
title: Memory Hierarchy, Fully Associative Caches
author: liuxing
math: mathjax2
nav_order: 15
---

# Memory Hierarchy, Fully Associative Caches
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Introduce to Cache

这张图是现目前我们的Components of a Computer（计算机组成?）:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329160206607.png?raw=true)

可以看到Processor 和 Memory之间是通过 接口进行访问

​	我们知道main Memory 是非常巨大的，想象一下，如果我们把内存比喻成一个巨大的图书馆，每次去访问内存的过程看作是： 需要去寻找一本书用于写论文

​	那么根据实际的经验可以得知，去库存丰富的图书馆找一本心仪的书是非常慢的过程，况且当你用完之后，需要将它还回到书架上，等你下次又需要这本书时，还得重新按照上述步骤去找它。

现在有一个比较好的解决方法：

准备一个小的书架，将借来的好几本书都放在书架上，那么下次使用时只需从书架上拿书而不必跑去图书馆，不需要时也可以暂时存放在书架上。

​	通过这个例子来类比Processor 和 Memory 之间的访问，可知是非常缓慢的，因此诞生了中间件 Cache

​	我觉得Cache 是 Greate Idea 之 “加速经常性事件”

一些我们需要多次访问的数据将其存在Cache上，比起直接去main memory中访问要便捷很多

加入Cache 之后的Components of Computer:


 ![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329161736393.png?raw=true)

---

## Memory Hierarchy

根据处理器的访问速度和大小，有一个Memory 金字塔:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329162020390.png?raw=true)

在硬件上距离Processor 越近的，Processor对其的访问速度越快，金字塔从上往下对应从快到慢，并且硬件规模上，寄存器最小，Disk最大，并且越上层的硬件资源，由于材料的选择，更加昂贵！

实际上，Cache相当于 Main memory 的一个数据子集, Cache 中存放主存的数据的副本，由于Cache 更小，其检索数据的速度也相对越快。

---

## Locality

Cache 的工作原理，有两点:

### **Temoral locality**

如果某个内存地址我们现在引用它，那么将来很有可能会再次使用

- 使最近访问的数据更接近处理器

### **Spatial Locality**

如果某个内存地址我们现在引用它，那么将来很有可能会引用其相邻的内存地址

- 移动整块连续的words更接近处理器

这很容易理解，举例来说，例如

- 循环:
  -  相当于对某一块指令堆重复执行

- 数组：
  - 连续访问数组中的元素，a[0], a[1], a[2]......
- 调用栈:
  - prologue 与 epilogue


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329163536818.png?raw=true)

---

## Work Process of Cache

当没有Cache 时，执行指令:

```yaml
lw t0, 0(t1)
```

t1 存放地址 0x12F0 ，Memory[0x12F0] = 99

1. 处理器向内存发出地址0x12F0
2. 内存读取 0x12F0处的值--99
3. 内存将 99 送回处理器
4. 处理器将99存入 t0



当有 Cache 时，上述过程变为：

1. 处理器向 Cache 发出地址 0x12F0
2. Cache 检索是否包含 Memory[0x12F0]处数据的副本（99）：
   1. 如果有(Cache Hit) : 将 99 送至处理器
   2. 如果没有(Cache Miss)：Cache 将 地址0x12F0发送至 Memory，然后Memory将值送回Cache, Cache copy 并送回至处理器
3. 处理器将99存入 t0



上述有两个术语:

### Cache Hit

- 你寻找的数据正好在Cache中存在
- Cache检索数据并送回至处理器

### Cache Miss

- Cache中不存在你寻找的数据
- Cache 去 Main Memory 中寻找所需数据，Copy 一份后送至处理器

---

## Fully Associative

有三种方法将data存入cache中:

- Fully Associative
- Direct Mapped
- Set-Associative

本节课只介绍 Fully Associative

Fully Associative 的特点是：可以存放在Cache 中的任何位置（接下来可以看到其他方法是固定位置)

Fully Associative就是将想要访问的地址分成两部分： Tag 和 byteoffset


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329165910245.png?raw=true)

然后根据Tag 和 byteoffset 在 Cache 中寻找数据，(Cache 长这样)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329165846382.png?raw=true)

需要注意的是，当开始运行一个新的程序时，Cache 中实际上包含的是 garbage , 对我们的程序来说是属于无效信息，因此还需要添加一个 Valid Bit 以鉴别是否该cache中的数据对程序有效，也就是说，如果

valid bit = 0，即使tag对应上，依然属于cache miss

那么在添加了valid bit 之后，Cache 是这样子的：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329170626005.png?raw=true)

接下来介绍一些术语(Terminology)

- **Cache line/block**{: .label .label-yellow }
  - 如上图中黑框部分称为Cache line
  - 在main memory 和 cache 中传输的最小单元
  
- **Line size/block size**{: .label .label-yellow }
  - cache line 的大小，如上图黑框部分的大小为 4 byte ,真实的Cache 要大得多，例如64byte ,此处只是用作演示

- **Tag**{: .label .label-yelllow }
  - 鉴别存储在给定的cache line 处的data 是否为我们所需
- **Valid bit**{: .label .label-green }
  - Cache line中的数据是否对程序有效

- **Capacity**{: .label }
  - Cache 中能够存储的总的 data bytes 数量
    - 对于full associative cache, capacity = the number of lines * line size
    - 如上图中 4 cache line x 4 bytes = 16 bytes = capacity



那么，如何确定Tag 和 Byte offset?
$$
\begin{aligned}
byte \space offset \space bits = log_{2}(line \space size)
\end{aligned}
$$
例如cache line 的 size 是 4 bytes ，意味着我们需要用2 bit offset对其编址:

- 00 01 10 11

来表示4个byte
$$
\begin{aligned}
tag \space bits = address \space bits - offset \space bits
\end{aligned}
$$
剩下的则都是 tag bits

---

## Example

接下来的例子 cache line size均为4byte ，这意味着 byte offset 是 2bit，最初状态valid bit 均为0

例如我们要向内存地址0x43F 处 load a byte:

```yaml
load byte at 0x43F
```

将 0x43F 写成二进制的形式:

```
0100 0011 1111
```

2 bits for byte offset:

byte offset = 11

剩下的10 bits for Tag:

Tag = 0x10F


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329192454800.png?raw=true)

由于最开始时Cache 中全是 garbage, 因此总是 Cache Miss,  此时Cache 将0x43F 送至Main Memory，Memory 读取 0x43F 并返回整块 对齐(aligned)的内存，即使此处你只是读取 单个字节，但是别忘了Spatial locality。

这里你只是想读取 offset = 11处的 byte ,但是实际从Memory 返回至Cache的是 00 01 10 11 4byte的内存

如果你是load word ，那么刚好返回一栏 cache line（因为此处cache line size 刚好和word 大小相同为4 byte, 如果cache line size=64 byte 的话，从内存中load 回来的就是 64byte， 也就是说load 刚好cache line size 大小的内存块，填满一栏 cache line）

后面还有好几个例子，在slide 上，这里就不演示了。

随着处理器对内存的访问，越来越多的数据被保存至Cache中，那么当 Cache 满了，此时又需要有新的数据从Memory 中过来时怎么办呢?例如下图：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329193041204.png?raw=true)

此时Cache 已满，现在我再次执行一条新的load word 指令:

```yaml
load byte at 0x972
```

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329194455972.png?raw=true)

此处Tag = 0x25C ，依然是Cache Miss，但是Cache 已经满了，该怎么办？

---

## Eviction Policies

当Cache 满了，此时又需要load 一条新的cache line 时，应该 evict 哪条原本在Cache 中的cache line呢?

### Least Recently Used (LRU)

- 硬件追踪访问历史
- 将最长一段时间未被使用的entry 替换掉

最长一段时间内数据未被使用意味着很可能未来不再使用了，因此可以作为eviction policies 

接着刚才的例子，添加一位 LRU ，把各个entry 都标记上LRU 的情况：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329195135778.png?raw=true)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329195227107.png?raw=true)

LRU指数越大说明该entry 最近很长一段时间未被使用，LRU = 0 意味着“刚刚被使用”

因此，将LRU = 3替换掉，然后 **更新LRU **


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329195450673.png?raw=true)

记得每次访问之后都要更新 LRU

---

## Handling Stores

上面我们举的例子都是 load 指令，那么对于 store指令如何处理呢？

我们知道store 指令是需要向内存中写入数据，现在有两种策略:

1. **Write-through**{: .label}

   同时向cache 和 main memory 中写入，后者需要花费更多时间

2. **Write-back**{: .label}

​		新增 1 bit 数字(dirty bit)以作为写入标记，只向cache 中写入，并将dirty bit = 1，当未来该行要被驱逐出cache时，我们才将该行写入memory

很明显write back 更好，这样做能降低内存的流量，因为你很有可能对某个地址执行多次写入，如果直接访问内存的话速度显然要慢很多

例子:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329200323069.png?raw=true)

---

## Fully Associative Drawback

当我们需要读取某个内存地址，然后去去访问Cache 时，对于 Tag 的鉴别是**并行**的，也就是同时进行，因此Cache 的每行都需要一个comparator来比较 Tag,比较费硬件资源


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220329225016532.png?raw=true)

下节课将看到Directed Mapped Cache 如何节省comparator资源
