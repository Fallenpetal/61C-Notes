---
layout: default
title: Cache Coherence, OpenMP Sharing Issues, Performance
author: liuxing
math: mathjax2
nav_order: 20
---

# Cache Coherence, OpenMP Sharing Issues, Performance
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## 回忆：Locks

Lock的出现：

来自不同线程的两次访存请求访问相同的内存位置，如果至少其中一个是 写入 ，并且它们相继出现，那么会形成 数据竞争

解决数据竞争的方法就是同步读写

计算机使用locks来控制对共享资源的访问，在大多数面向对象的编程语言中，locks就是一个你可以调用的object，而背后的原理则是一个整数的原子更新：

- 0 == unlocked
- 1 == locked

两个操作：

- lock.acquire()   //阻塞该线程直到 lock 解锁
- lock.release()   //解锁lock

OpenMP 也有锁：

```c
#pragma omp critical
{
... Only one thread is running at a time here
}
```

### 死锁--Deadlock

由于everything都被锁定等待something而无法取得进展的系统状态

---

## Multicore Multiprocessor

SMP : (Shared Memory) Symmetric Multiprocessor

- 有两个以上相同的CPUs/Cores
- 单个共享一致性内存


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426200520942.png?raw=true)

#### Q1 多核如何共享数据？

所有核共享单一物理地址空间(Main Memory)

#### Q2 多核如何协作？

通过内存中的共享变量

- 共享数据的使用 通过同步原语(locks)来协作，允许同一时间下只有一个处理器访问数据
- 现今的多核计算机都是SMP:
  - 唯一区别在于现代SMP使用 network  去构建 shared bus,以及拥有一个所有核共享的L3 Cache

#### Q3 可以支持多少核？

取决于工作量，大多数系统运行最好的单核，或者最节能的核心



### Multiprocessor Caches

由于对内存的访问是非常慢的，所以诞生了caches,那么在多核处理器中，每一个核都有一个本地的cache，只有当发生cache miss 的时候才会去访问main memory:

![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426202159822.png?raw=true)

---

## Cache Coherency

一个例子说明为什么需要cache coherent:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426201805001.png?raw=true)

CPU1 想要访问 0xC0FFEE 处的数据：

发生cache miss

cache 向main memory 请求数据，带回data 到cache，并送至 CPU1

假设0xC0FFEE处的值是False,绿色方框表示CPU1已拥有


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426202416651.png?raw=true)



那么现在，CPU2也想访问0xC0FFEE处的值：

发生cache miss

cache 向main memory 请求数据，带回data 到cache，并送至 CPU2


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426202557703.png?raw=true)



此时，假如CPU1执行 Write 操作，将0xC0FFEE处的False 改为了 True:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426202705466.png?raw=true)



那么现在的问题是：0xC0FFEE处的值在两个CPU中不相同！

现在CPU1持有0xC0FFEE的最新值，而CPU2 和 Memory 中的值都是旧值

因此为了解决数据不一致的问题，我们有：cache coherency 



#### Idea

当任何processor 发生cache read miss/write miss 的时候，通过内部网络通知其他processors

- 如果只是 read, 其他processor 可以拥有副本
- 如果一个processor做了 write，那么需要将其他processor 的副本无效化 

当一个processor发生write transactions时，其他caches 监听(snoop) 并检查它们持有的tags

- 如果tags 与正在执行write 操作的caches 相同，那么使之无效

其他说明：

- 使用write-back 策略
- 通过传播requests来通知其他processor
- 从相邻processor的cache中取得数据要比到memory快

---

## MOESI Protocol

原来的cache有了valid , dirty 位，现在再加一个shared 位

五种状态:

#### **Shared**{: .label}

- 当前cache拥有最新的data, 其他caches 也可以拥有副本，valid , shared 位设置为1

#### **Modified**{: .label .label-yellow}

- 当前cache 拥有最新的data，其他caches没有副本，且正在发生写入，内存的值是out-of-data(过期)，valid, dirty 位设置为1

#### **Exclusive**{: .label .label-green}

- 当前cache 拥有最新的data，其他caches没有副本，内存也是最新版本，往往是某一个core 第一次发生read miss ,然后从memory 中 load data to local cache，此时仅有该core和memory拥有该data的最新值，因此是“独占”，只有valid 位为1。

- 如果其他cache 也读取(read)了这份data，那么状态从**Exclusive** 变成 **Shared**

- 如果对**Exclusive** 下的cache 执行 Write ，那么状态变为 **Modified**, 但是不会告知其他Caches，由于我们采用Write Back 策略，当this line 被驱逐时写回memory

#### **Owned**{: .label .label-blue}

- 例如对 **Exclusive** 状态下的cache 执行 Write 后，变成了 **Modified** ,此时如果有其他processor 访问已修改的数据，那么该cache 的状态从 **Modified** 变成 **Owned**

- **Owned**状态下，cache拥有最新的data，而内存中的data 是 过时的，如果有其他processor 访问最新data, 直接由该cache 提供data，而不是让其他processor 去访问内存

- 如果此时再次对Owned 状态下的cache 进行写入，状态变成 **Modified** 那么通知其他所有processor,使它们拥有的副本无效

- valid 位，dirty 位，shared 位设置为1

#### **Invalid**{: .label .label-red}

- 无效态，cache 中持有的是“垃圾”，不属于该程序，或是已过时的值



状态转换图：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426210340663.png?raw=true)

（**Probe** 表示其他core 对本core cache的访问，不带 **Probe** 表示，本core 对自己的cache 的访问)



valid , dirty , shared bit分布图：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426210443331.png?raw=true)

**Note**

Exclusive/Modified 状态下，写入操作不需要通知其他processor

Shared/Owned 状态下，当你要执行 写入 时，需要告知其他processor，使它们的副本无效

**也就是如果Shared bit为1 的话，表示其他processor 可能拥有该副本，因此一定要通知其他processor，使它们的copy 无效**



回到刚才的例子，最开始的时候，Cache 中不含 0xC0FFEE处的值，Cache 是 **Invalid**状态， 发生compulsory miss:


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426211153865.png?raw=true)



去内存取值成功后，由于此时只有CPU1 和 Memory 拥有该data 的最新版本，因此Cache1 的状态是 **Exclusive**


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426211346418.png?raw=true)



当CPU2 也对0xC0FFEE处的值发起访问请求时，Cache1 的状态则从 **Exclusive** 变成 **Shared** ：


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426211516507.png?raw=true)



此时，CPU1 对 0xC0FFEE执行 Write 操作，状态从 **Shared** 变成 **Modified** ,并通知其他processor，让它们在0xC0FFEE处的副本失效


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426211650020.png?raw=true)



如果CPU2 再次访问0xC0FFEE，那么data直接由CPU1提供，此时CPU1 的状态从 **Modified** 变成 **Owned**


![o](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220426211809501.png?raw=true)



### Coherency Miss

如果你在两个core 之间共享数据，每次当一个core 在执行 write 的时候，另一个core 会在下次 read/write 操作上发生 cache miss(被设为了invalid)

由此诞生了一种新的Miss type，截止到目前为止的4中cache miss type:

compulsory miss, capacity miss, conflict miss, coherency miss



---

## Golang Introduction

如果两个硬件线程共享相同的物理上的core，对相同数据的读写不会发生coherence miss,但是对不同数据的读写会发生很多cache miss

这也就是最开始为什么硬件多线程是loser

但是一个可选的选项是 Communicating Sequential Processes

- OpenMP 有非常有限的并行性
  - 仅仅适用于parallelizing loops和reduction
- Raw threads 很容易出错
  - 死锁
- 一门好的 CSP 语言： golang
  - 对于共享内存的多核处理器而言真的很好



- [The Go website](https://golang.org/)

- [Especially useful: Effective Go](https://golang.org/doc/effective_go.html)
- When searching Google, ask for golang, not go
-The language may be Go, but golang refers to the language too
- If I'm starting new code and I need to care about performance,
scalability, or maintainability, I use Go
- Only exception is if the problem can't stand a garbage-collector pause (such as an OS,
low level drone flight control, etc...), in that case I'll have to learn Rust



然后Go 这里我就没学了hhh