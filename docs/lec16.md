---
layout: default
title: Direct-Mapped & Set-Associative Caches, Cache Performance
author: liuxing
math: mathjax2
nav_order: 16
---

# Direct-Mapped & Set-Associative Caches, Cache Performance
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Recall

### Cache Lines

当你从main memory 中将数据携带到 cache 中时，带过来的是一整行cache line

一般的cache line size 是 64 bytes

tack advantage of spatial locality

举例：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330154019370.png?raw=true)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330154105738.png?raw=true)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330154120781.png?raw=true)



### Tag

在cache 中，每一行都有一个 tag 帮助鉴别我们正在访问的memory address 是否在cache 中存在

检测是否匹配的方法:

valid bit = 1 且 tags 相同 -- 相当于用 与门(and gate) 连接两个输入 valid bit 和 comparator 

of tags的结果



### Some Terminology

**Hit Rate**

- cache hit 的次数 / 总的访问次数

**Miss Rate**

- 1 - hit rate

**Hit Time**

- 访问某address并 cache hit 所花费的时间

**Miss penalty**

- cache miss 时，访问下一层级的硬件最终找的data 所花费的时间



### Write-allocate / No write-allocate

首先 write 相当于往内存中写入数据，对应的是 store 指令，先复习上节课的两种ways:

**write-through**{: .label .label-yellow }

- 同时向cache 和 memory 中写入，后者耗时更长，但是该方法实现简单

**write -back**{: .label .label-yellow }

- 先只在cache中写入，并设置dirty bit = 1,那么相当于cache 是 more update than memory (cache领先memory一个版本，相当于git:本地仓库持有最新版本，远程库版本稍微旧一些)， 当未来该行将被驱逐出cache时，我们才把该行写入cache以保留数据

在介绍另外两种之前，先搞清楚 write hit 和 write miss

**<font color='red'>write hit</font>** : 当你想向内存中某地址写入数据，如果地址映射到cache中存在，就叫write hit，那么你就可以反复修改cache中的数据，最后才写入memory(对于write back 策略)



**<font color='red'>write miss</font>**:  当你想向内存中某地址写入数据，如果地址映射到cache中不存在，或是被其他地址的映射占据，就叫write miss，可以类比 read miss ：想要读取的data 在 cache中不存在，只能去内存搬运

[一个参考回答](https://cs.stackexchange.com/questions/133352/what-is-a-cache-write-miss)

**write-allocate**{: .label}

- 发生write miss 时，从内存中把block 搬至 cache 并更新

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330164023834.png?raw=true)

**No write-allocate**{: .label}

- 发生write miss 时， 只更新memory ，不把block 搬入 cache

现在，这四种方式有两种常见的组合：

Write through, no write-allocate

- write hit 时，同时更新内存和cache
- write miss 时，只更新内存



 Write back, write-allocate

- write hit 时，不更新内存，只把 cache 中对应行的dirty bit 设为1 ，等到将被驱逐时才更新内存
- write miss时，同时更新内存和 cache

值得注意的是，以上两种策略在发生read miss 时均需要更新 cache

---

## Approximate LRU(Clock Algorithm)

该小节我们将实现近似LRU算法:

- 每行cache line 都有 1位 <font color='red'>referenced bit</font> ，初始化为0
- 每当某一行被访问 (Access line) 时，只将referenced bit 设为1,（不需要移动分针）
- 当 cache 满了的时候，即所有entry的valid bit = 1时，我们需要开始驱逐，假设有一个**分针**指向当前行：
  - 从cache 的第一行开始，检查该行的 referenced bit（简写为ref bit）:
    - 如果ref bit = 1, 将其设置为0，并将分针移动至下一行
    - 如果ref bit = 0, 将该行驱逐，并把新的cache line插入该行替代
- 当下一次我们需要驱逐某行时，从分针最后停留的地方开始

具体演示过程在slide p24 开始，动画效果比笔记更好，因此不再赘述。

该算法执行的动画过程就好似时钟的滴答滴答，分针随着时间移动，因此叫clock algorithm

因为cache 初始化的时候所有valid bit all = 0，因此要遍历一遍cache 之后才会开始驱逐，事实发现驱逐行的顺序确实符合LRU(Least Rencently Use)



**The Drawback for LRU**

在某些情况下，LRU 的 hit rate 竟然是 0% !,例如下面这个例子:

cache line 有 4 行，最开始valid bit all = 0, 现在你打算循环访问内存中的某数组 [A B C D E]：

- 访问 A ,cache miss，将 A 存入 line1
- 访问 B ,cache miss，将 B 存入 line2
- 访问 C ,cache miss，将 C 存入 line3
- 访问 D ,cache miss，将 D 存入 line4
- 访问 E ,cache miss，此时cache 已满，根据LRU ，将A 驱逐，将 E 存入 line1
- 一次循环结束，下一次循环开始
- 访问 A, 由于 A 刚刚被驱逐，此时line1上是 E ，因此 cache miss ，根据 LRU 将 B 驱逐，把 A存入 line2
- 访问 B ，B 刚刚被驱逐，cache miss ......
- 由此不断循环下去，总是cache miss ,因此 hit rate = 0%



**More Eviction Policies**

除了 LRU, 介绍其他两种:

Most Recently Used (MRU)

- 如其名

Random

- 随机选择一行驱逐，好处是cache中不需要其他的数据位来追踪(ex. LRU)

对于刚才的例子，采用这两种算法的话，MRU's hit rate = 75%, Random's hit rate != 0

---

## Direct Mapped Caches

对于Direct Mapped Cache, 我们把地址分为三个部分:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330204724512.png?raw=true)

与full associate cache 不同的是，多了index 部分，该部分对应cache 中的特定的index，

full associate 是可以将memory 中带来的line 存在任意的位置，但是 direct mapped 则要求内存中的某地址映射到cache中的特定index 行

计算TIO 的各部分对应的bit:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330205415050.png?raw=true)

那么让我们来看一个memory 映射到 cache 中的例子：


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330205657966.png?raw=true)

假设Cache 的 cache line 大小是 1 byte ，内存的地址编址是 0,1,2,3....即每个address存 1 byte

根据 **TIO** 计算 tag, index 和 offset：
$$
offset = log_{2}(Block\space size)=log_{2}(1)=0\newline
index = log_{2}(the \space number\space of\space line)=log_{2}(4)=2
$$
因此，4bit 的 内存地址分解为 2 bit Tag, 2 bit index, 0bit offset(None)

那么0b0000处的 index 是0b00

0b0100处的index 是 0b00

0b1000处的index 是 0b00

......

可以发现只要后2 bit 是00，那么都隶属于index = 00 ,也就是说只要是 0000,0100,1000,这样的地址，都会映射到 index00，刚好是 4 字节对齐的，对应十六进制: 0 4 8 C

因此在上图中刚好内存中所有蓝色的块，映射到cache 中的第0行(index=00)

**结论是**: 映射到cache 中的某一行的块，在内存中都是对齐的，具体对齐字节数与将address分解为 TIO 后的I 和 O有关

例如下图中，当O 是1时，对齐的情况又不一样了，是 0 8 0 8...8字节对齐:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330211540591.png?raw=true)

### Example

以下例子采用 Write back 策略：

最开始的时候Cache 全是 garbage ，all valid bit = 0


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330212954226.png?raw=true)

1. Read byte at 0xFE2


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330213045879.png?raw=true)

1. Read byte at 0xFE8
![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330213328204.png?raw=true)

1. Read byte at 0xFE9

   ![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330213441095.png?raw=true)



### **关于Direct Mapped Cache 的驱逐策略**

每个地址只能被存在对应映射的index 处，如果当我们向cache 中 bring a new line 时发现该处已经被存有其他的address 时，必须驱逐它。

但凡index处冲突了那就把原来的覆盖，因此不存在LRU 策略

例如对于如下memory 到 cache 的映射:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330214051917.png?raw=true)

如果最开始 cache 中第一行存入的是 0x0的block , 当我们再存入 0x4处的block时，就把原来 0x0的 block

驱逐了。

接上面的例子，继续 Read byte at 0xDF9


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330214310468.png?raw=true)

把原来的 index = 1处的block给驱逐了。

### **direct mapped 在硬件上的实现**


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330214448537.png?raw=true)

上图中的cache line size 是16 byte ，按理说需要4 bit （$log_{2}16$ ) 编址 offset，但是只使用了 2 bit 进行offset编址，为什么呢？

原因是我们刚才说的，映射到cache中同一行的地址在内存中是对齐的，例如 0000， 0100，1000全部对应index = 00，然后最后两bit 都是00，可以忽略对他们的编址，也就是说对00，01，10进行编址就行了，所以只需要2 bit 的偏移。

**Direct mapped cache的优点**

可以发现你想访问的数据存在于特定的index 处，因此不必向fully associative一样并行检查所有行，direct mapped 只需要 1 个比较器

**Direct mapped cache的缺点**

当特定行满时，驱逐策略是直接替换该行，原来的数据则丢失了，利用率不高

因此接下来我们将看到在Direct mapped 的基础上进行改进的Set-associative caches

---

## Set Associative Caches

set associative 就是，原来的index 只能插入1行数据，那么我在原来的index 上增加多个插槽(slot)，这样就可以插入更多行而不必直接将原来的行给驱逐

那么随之而来的时，同一个index下有了多行，当同一个index满了的时候，依然采取LRU 踢出行

```
the number of slot = the number of way
```

那么2个插槽的index 叫做 2-way set associate


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330220302507.png?raw=true)

这里的LRU 记录在同一个 index 下对行进行编号 ，**LRU = 最长时间未访问的行编号**

例子:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330220458702.png?raw=true)
上图中，在同一个index 下，可以随意选择一行插入，那么另一行的编号就是LRU bit，如上图我们插入第0行，那么第一行的编号1则是LRU bit

其他例子看slide吧,下图是 P & H 中一个4-way Set Associative Cache的例子

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330221007644.png?raw=true)

---

## Other applement

### Type of Misses

**Compulsory Miss**

- 首次访问从未在cache中出现过的block引起的
- 在硬件上使用pre-fetcher 消除这种miss(prefetcher 是一种特殊的电路，它尝试猜测你下一步想要访问的blocks)

**Capacity Miss**

- 当Cache不能包含程序执行过程中所需要的所有block时引起的
  - 例如前面的例子，Cache只有4行的容量，但是我要读取的数组为[A B C D E]五个，如果你装满了前四个，当你装入第五个时，前四个之一又被驱逐了，然后当你又访问以前的四个之一时，其中一个刚刚被驱逐，你又需要从内存中带回cache，如此往复。
- 尽可能增大cache capacity 来消除这种miss

**Conflict Miss**

- 在set-associative 和 direct mapped中出现，也就是内存中多个block争夺有限个index中的slot
- 在fully associative 中不会出现，因为fully associative 理论上可以和内存建立一一对应的全映射
- 使用fully associative 消除这种miss

### 三种方式的比较

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220330221754173.png?raw=true)