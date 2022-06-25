---
layout: default
title: Virtual Memory II
author: liuxing
math: mathjax2
parent: Virtual Memory
nav_order: 2
---

# Virtual Memory II
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}



## How Does VM Work?: Translation

例如一个RISC-V 指令

```
lw t1, 1024(x0)
```

是把虚拟地址1024处的值加载至t1寄存器，需要通过映射，把VA 1024(VA is Virtual Address)转换为PA(Physical Address)，因为我们的RAM 是以PA 进行编址的

然后通过PA 找到想要的数据，这里有两种情况：

- VA 对应的PA 在RAM中，在RAM找到数据
- VA 对应的PA 在Disk中，将数据加载至RAM中，更新Translation Map

最后返回至t1寄存器

VA 转 PA 的过程则成为translation


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430154313036.png?raw=true)

---

## Page Tables

定义：从虚拟地址 到 物理地址的 Translation Map 称为 Page Tables

Page Table Entry: 对于每条VA-->PA 的条目称为PTE

假设我们对每一个虚拟地址(1 word)都对应一个物理地址的映射的话，Page Tables需要多少条目?

$$
32bit编址有2^{32} byte的地址空间\\
1word=4byte=2^2byte\\
那么PageTable有条目数：\frac{2^{32}}{2^2}=2^{30}条
$$

Q：如何去减少Page Table 中的条目数？

A：可以不细分到对每条VA 进行映射，而是对某个范围的VA 进行映射


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430155404505.png?raw=true)

例如将虚拟地址0-4095 作为一个entry，那么条目数下降为：

$$
\frac{2^{32}byte}{4KB}=\frac{2^{32}byte}{2^2*2^{10}byte}=2^{20}条
$$

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430155744815.png?raw=true)

**一个entry 和 一页page其实是同一个东西，一个完整的Page Table 有多少page 就相当于有多少 Entry**

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430155805880.png?raw=true)

与Cache 相同，为了更好的利用时空局部性，现在当我们从磁盘上加载数据时，不再是加载单个word，而是加载一整页page(entry)：

例如要从磁盘加载物理地址 9120, OS 会将bytes 8192 - 12287 从磁盘载入RAM

---

## Address Translation

将虚拟内存和物理内存分页，通过Page Table映射进行对应：

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430160641591.png?raw=true)

那么Translation的过程需要两点:

- Page Number
  - 索引，第几页
  - 所需编址的bit数 = $$log_{2}PageCount$$
- Page offset
  - 索引，在该页的哪个位置
  - 所需编址的bit 数= $$log_{2}PageSize$$

一些术语：

**VPN**{: .label}: Virtual Page Number，虚拟内存的分页数量

**PPN**{: .label}: Physical Page Number, 物理内存的分页数量

那么Virtual Address可以拆分为 VPN + Page offset , Physical Address 可以拆分为 PPN + Page offset

**需要注意的是，VA 的 Page offset 与 PA 的Page offset 对应相同，无需Translation**

例子：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430161420441.png?raw=true)

- VA size : 也就是对VM进行编址所需bit数，在多少bit的机器上就是多少bit，如图32bit
- PA size : 对PM 进行编址所需bit数，等于$log_{2}RAMSize$ 
- Page offset: 在一页Page 内，对所有VA 的索引数编址所需bit 数，等于$log_{2}PageSize$ 

这里page的大小是4KB，$$Page offset=log_{2}4KB=log_{2}(2^{2}*2^{10})=12bit$$ 


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430161957632.png?raw=true)

以下是Fine Grain Page Table 的例子，**而且事实上Page Table中只存储PPN，VPN是索引，不需要存储**，就像是一个数组，不用考虑数组索引的存储


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430162155618.png?raw=true)

---

## Page Faults

VA 和 PA 通过Page Table 进行映射，假如某VA在Page Table 中映射的结果为Disk,也就是对应的PA不在RAM中时，称为Page Faults

出现Page Faults时会发生什么？

1. CPU生成page faults exception
1. 硬件将控制权交给OS ,OS  执行page fault handler 程序去修复
1. 如果RAM 没有足够的空间了，OS 会选择某一page驱逐
1. 在驱逐之前，如果该page is dirty，需要先写回disk，然后OS更新相应的Page Table Entry,指向Disk,表明已将该page放入Disk
1. OS 从Disk 中将我们想要的page 载入RAM中，并为新的Page 更新page table entry
1. OS 将控制权还给硬件，从造成page fault exception的指令处继续执行，但是由于已经被修复，这次不会再造成exception

最后想说的是，处理page faults的发生真的是一件非常非常慢的事，This is why having more RAM will make your computer faster

---

## Page Replacement Policies

### First in, First out (FIFO)

与队列类似，驱逐最旧的(最先访问的)page,只有当所访问page 不在RAM中时才需要更新


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430164212100.png?raw=true)

### Least Recently Used (LRU)

驱逐最长时间未使用的Page,每次访问都需要更新Page Table


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430164306928.png?raw=true)

### Random

随机选择某Page 进行驱逐



### Cons

可以构造一种访问模式，使得LRU和 FIFO都失效：

如果RAM 只能同时容下4 Page,依次访问0，1，2，3，4，0，1，2，3

---

## Additional Page Table Metadata

在Page table中，除了PPN 外，还有其他bit ，如V,D,R,W,X

V: Valid ,1表示page in RAM 且 映射有效，0表示page is not in RAM |

D: Dirty, 1表示page on RAM is **more up to date** than page on disk, 0表示page in RAM 和 page in Disk 相同 |

R: Read permission |

W: Write permission |

X: Execute permission |

R W X 是保护位，阻止其他process访问无权访问的page，如果它们无权访问，会造成memory protection fault 或 segmentation fault

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430182915532.png?raw=true)

---

## Translation Lookaside Buffer -- TLB

Q: 有了虚拟内存之后，当你每次想要访问某项数据，需要发生多少次内存访问？

A：2次

- 读Page Table ,得到translated physical address, page table 在 RAM中
- 使用physical address 在RAM中访问所需数据

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430183354600.png?raw=true)

如何使其更快，也就是说就减少内存访问次数，我们知道对内存的访问是很慢的。

类比学过的Cache，这里对于Page table也需要一个Cache,称为Translation Lookaside Buffer(TLB)


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430183607335.png?raw=true)

过程如上：

- 处理器发出虚拟地址(VA)，先去TLB中找，看看有没有对应的物理地址(PA)
- 如果有，Hit,直接在RAM中访问PA，将数据返回，只存在一次Memory Access
- 如果没有，Miss, 去访问RAM中的page table，并更新TLB，执行第二步

那么同样地，TLB 也有额外的meta


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430183930596.png?raw=true)

TLB比Page table要更小，使用Fully Associative，一般有32 - 128 entries,但是每个entry可以映射到较大的Page，TLB的驱逐策略一般是：Random / FIFO



### TLB Flush

对于多核处理器而言，每一个Core都有自己的TLB，对于某个Core上正在活跃的Process来说，TLB与之相关，所以在发生context switch 的时候，也就是切换Process后，TLB需要被Flush,也就是之前与上一个活跃的进程有关的entry都无效了



---
