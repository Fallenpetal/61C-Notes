---
layout: default
title: Virtual Memory III
author: liuxing
math: mathjax2
parent: Virtual Memory
nav_order: 3
---

# Virtual Memory III
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}





## Multi-level Page Tables

### why need it?

对于一台32bit 4GB RAM, 4KB pages的机器

$$
虚拟内存空间大小=2^{32}byte\\
page\space table\space entry数量=\frac{VM Space Size}{PageSize}=\frac{2^{32}byte}{4KB}=\frac{2^{32}byte}{2^2*2^{10}byte}=2^{20}=1M\space条\\
PA=log_{2}4GB=log_{2}{2^2*2^{30}bytes}=32bit\\
offset=log_{2}PageSize=log_{2}4KB=log_{2}(2^2*2^{10})=12bit\\
PPN=PA-offset=32-12=20bit\\
每条PTE大约4bytes(20bits\space PPN+status\space bits)\\
所以Page\space Table\space Size = PTE数量\times PTE大小=1M\times 4bytes=4MB
$$

看样子不算太坏，但是请注意：每一个程序都有自己的page table，如果有100个程序正在运行，那么有400MB的空间开销用于Page table

为了减小开销，引入Multi-level Page Tables


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430190233285.png?raw=true)

1级Page Table 存储2级Page Table的PPN，2级Page Table存储我们想要的数据的索引(PPN)

需要注意的是：

**1级page table一定在RAM中，而2级page table有的可以在RAM中，有的可以在Disk上**

假设我们要求每页page都是4KB的大小，之前说了一条PTE大小大概是4byte(PPN + status bit)，那么一页page  table 有$$4KB/4byte=1K=1024条$$ ,对于1024条PTE, 需要$$log_{2}1024=10bit$$ 进行编址，same as 2级Page Table. 

需要10bit 为 1st level table 的1024条PTE进行编址，同时也需要10bit 为2nd level table 的1024条PTE进行编址

也就是说1级Page Table可以映射1024个2级Page Table,每个2级Page Table可以映射1024条data,我们可以将VPN划分为2个10bit index **请注意都是index,而不是page table里面的存值**



![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430191704897.png?raw=true)

例子：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430192656570.png?raw=true)

使用多级Page Table之后，内存开销下降了很多：

只有1级Page Table时的内存开销：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430193047326.png?raw=true)

多级Page Table下，1级Page Table 的内存开销：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430193120441.png?raw=true)

当然，还有2级Page Table 的开销没算进去，但是有些2级Page Table是存储在Disk上的，开销一定比不分级前小

### Some Q and A

**Q**{: .label .label-red}：在多级PageTable的情况下，当运行一个程序时，最少的page table数是多少？

**A**{: .label .label-green}: 2 。1级Table必须在RAM中，然后至少有一个2级Table，索引我们想要访问的数据



**Q**{: .label .label-red}:多级table影响TLB吗？

**A**{: .label .label-green}: 不影响。当TLB Hit时，直接访问RAM中的data,  当TLB Miss 时，我们从原先的只需要访问1级Page Table 变成了 先访问1级page table,再访问 2级Page Table，再访问data，内存访问次数变成了 **3次** ，但是减小了内存开销

---

## Caches and Virtual Memory

### Physically Indexed, Physically Tagged Cache


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430193836110.png?raw=true)

过程：

1. CPU想要访问某个VA下的data
1. 去TLB中查找
   1. 如果查找到VPN对应的PPN，Translation Hit
   1. 查找失败，Miss, 去RAM中的PageTable查找(多级PageTable),找到后更新TLB
1. 将PA拆成TIO, 去Cache中查找data
   1. Cache Hit, 返回data 至CPU
   1. Cache Miss ,访问RAM，更新Cache, 返回data至CPU



先介绍两个术语

 **Synonyms**{: .label .label-blue}


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430194601291.png?raw=true)

不同的虚拟地址映射到相同的物理地址



 **Homonyms**{: .label .label-blue}


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430194637855.png?raw=true)

相同的虚拟地址映射到不同的物理地址



**PIPT Cache**

- 多个进程能使用相同的cache吗？
  - 可以
- 缺点
  - 在访问cache之前必须将VA translate to PA

### Virtually Indexed, Virtually Tagged Caches


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430195004750.png?raw=true)

过程：

1. CPU想要访问某个VA下的data
1. 直接将VA拆成TIO形式，去访问Cache
   1. Cache Hit, 返回data
   1. Cache Miss, 不得不去RAM中寻找data
1. 由于RAM是PA编址，我们不得不把VA translate to PA, 访问TLB
   1. TLB hit, 得到PA, 去RAM中加载data，更新Cache并返回至CPU
   1. TLB miss, 去访问Page Table，得到PA, 更新TLB，加载data，返回

**VIVT Cache**

- 多个进程能使用相同的cache吗？
  - 不可以，因为直接采用VA 访问Cache，如果两个Process访问相同VA, 会造成覆盖(synonyms and homonyms)
- 在每次context switch 时都需要flush Cache



#### PIPT Vs VIVT


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430202742392.png?raw=true)

那么我们既不想在访问Cache 之前先Translate VA -->PA ,也不想受困于synonyms and homonyms

结合二者，有了VIPT Cache

### Virtually Indexed, Physically Tagged Caches

- 为避免synonyms and homonyms,使用PA 去访问Cache
- 并行执行TLB 和 Cache的浏览

Invariant

PA 与 VA 的invariant 是 page offset bit 都相同


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430200133363.png?raw=true)


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430200249762.png?raw=true)

执行过程：

1. CPU想要访问VA 处的数据，将VA拆分为VPN 和 page offset

1. 以下两件事同时发生

   1. 利用VPN去查看TLB
      1. TLB hit, 得到PA,得到PPN,也就是Tag
      1. TLB miss, 去访问Page Table,更新TLB
   1. 将page offset分为index 和 byte offset 去查看Cache

1. 将PPN 和 Cache tag对比，如果相等则Hit, 返回数据至CPU

   1. 否则Cache miss，访问RAM, 更新Cache, 返回

   



#### Cons

cache size 受限于page offset 的bit 数

**Q**{: .label .label-red} : page size 4KB, 一个direct-mapped cache 能存储多少bytes?

**A**{: .label .label-green} : 4KB, 我们只能用page offset bits 去索引 cache, $pageoffset = log_{2}4KB = 12bit$ 

因此我们只能编址12bit, 也就是4KB 的数据

**Q**{: .label .label-red} : 在相同page size 下， 如何让cache 更大？

**A**{: .label .label-green}: 增加cache 的 associativity

---

## Review

- RAM = Main Memory
- RAM 包含data和code
- 为了访问data 或 code，需要将其从disk 载入 RAM


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430202141663.png?raw=true)

为了在Disk 和 RAM 之间传输data更加容易，我们为move data设置一个标准的granularity

- The granularity that we use is called a page
- The size of a page is typically 4KB
- 这意味着，如果我想访问某个整数，我需要将该整数所在的Page 全部载入RAM


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220430202408745.png?raw=true)



为什么我们需要虚拟内存？

1. RAM 不足以一次性存储所有我们所需的data
1. 我们的程序认为自己的data是连续的
   - 当你写程序的时候，你并不知道它将会储存到内存的何处
   - 我们只需要认为内存是连续的，VM 会在背后为我们处理一切
1. 在进程之间起到保护作用