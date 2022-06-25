---
layout: default
title: Multi-Level Caches, Cache Questions
author: liuxing
math: mathjax2
nav_order: 17
---

# Multi-Level Caches, Cache Questions
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Review

### 内存编址


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407191125924.png?raw=true)

观察这张图，内存的编址明明是8bit，但是却以32byte进行编址，猜想是为了方便cache取值，按cache  line size 对内存地址进行重排，也就是说按照每32byte对内存进行编址

### Cache的另一种布局

我们一般看到的全关联的Cache布局一般是这样的:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407191736339.png?raw=true)

然后4路组关联的Cache 是这样的:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407191828298.png?raw=true)

也就是每一组Tag 和 Data 竖向排列，实际上也可以横向排列，横向排列的话如下图:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407192006070.png?raw=true)

### Recall: IMEM 和 DMEM



回想一下我们在单周期数据通路设计中，数据和指令都是存储在main memory中的，为了避免内存访问的冲突，将数据和指令分为 IMEM 和 DMEM，实际上则是两个Cache

---

## ArrayStrides 如何影响Cache

```c
int sum_array(int *my_arr, int size, int stride) {
	int sum = 0;
	for(int i = 0; i < size; i+= stride) {
		sum += my_arr[i];
	}
	return sum;
}
```

以上函数的目的是以stride 的步长去访问my_arr[] 中的元素，

假设有数组有32个元素，且为int类型（sizeof(int) = 4byte），全关联Cache，line size = 16 byte,也就是说每行可以存放4 个 int 数组元素，有16 lines, 现在我们想通过改变步长stride 来观察对Hit Rate 的影响。

- Stride = 1


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407193421549.png?raw=true)
当步长为1时，该函数连续访问a[0], a[1], a[2], a[3]......在第一次访问a[1]时发生compulsory miss，随后从内存中bring a[0], a[1], a[2], a[3] 4个元素到cache中，因此后面的访问 Hit,然后四个元素访问完毕后，从第五个开始cache miss，bring whole line from memory，然后继续这种pattern

- Stride = 2


  ![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407193737835.png?raw=true)

访问a[0], a[2] ,a[4]......

访问a[0]bring a[0], a[1], a[2], a[3] ,然后下一次只访问了a[2]，因此只有一次hit, a[1]和 a[3]属于unused

- Stride = 4


  ![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407193927852.png?raw=true)



可以发现，**当stride >= block size** 时，并未利用到cache 的空间局部性所带来的好处，因此注意stride 对 Hit Rate 的影响！

---

## 矩阵乘法的优化1

假设矩阵以行优先的顺序存储:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407194710877.png?raw=true)

两个矩阵A B 相乘的模式是:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407194759174.png?raw=true)
$$
C[0][0] = \sum_{*=0}^{n}A[0][*]\cdot B[*][0]
$$
回忆一下前面说的stride 对 hit rate 的影响，由于对 A 的第一行是连续访问的，stride = 1，而对B 的访问是跨行访问，stride = n,假设矩阵远远大于cache 的大小，结合上面（**当stride >= block size** 时）可知

- 对B 而言Hit Rate = 0%,
- 对 A 而言 Hit Rate = 1 - sizeof(type) / cache block size  (compulsory miss)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407195721191.png?raw=true)

注意此处slide 是错误的，对C 而言应该是 Miss, Hit, Hit, Hit

上面假设cache line size = 4 个矩阵元素大小

### 通过矩阵转置(Matrix Transpose)提高Hit Rate


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407200044147.png?raw=true)

转置即
$$
\begin{aligned}
A^T[x][y] = A[y][x]
\end{aligned}
$$
因为以行优先存储时，我们对矩阵B 的访问相当于访问 1 4 7

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407200219346.png?raw=true)

每次跨步3 个元素，那么只要将 $$ B 变成 B^T $$ ,在数组中的存储则变为

[1 4 7 2 5 8 3 6 9]

那么此时对1 4 7 的访问跨步为1，很好地利用了空间局部性。

---

## Blocking 的应用1

**Cache Blocking**

- 通过重排数据的访问顺序使得更好利用Cache 的时空局部性
- 阻止重复地驱逐和从内存中获取相同的数据(由于cache size 有限)

### 通过Blocking 对 矩阵转置进行优化

假设我们要对 A 进行转置:

![img](https://inst.eecs.berkeley.edu/~cs61c/sp22/labs/lab07/matTnorm.png)

A 原来的行变为列（上图为naive transpose的示意图，与下面介绍无关），同样假设矩阵远大于Cache size

代码:

```c
    for (int x = 0; x < n; x++) {
        for (int y = 0; y < n; y++) {
            A transpose[y + x * n] = A[x + y * n];
        }
    }
```

可以发现分别访问了 A  和 A 的转置，

最开始执行 A[0] [0] = A[0] [0]

那么访问A[0] [0] ，会from memory bring A[0] [1] , A[0] [2], A[0] [3] to cache

- 下一步是 A[0] [1] = A[1] [0] , A[0] [1] Cache Hit, 但是 A[1] [0] Miss, 带来A[1] [1], A[1] [2]...未使用
- 下一步是 A[0] [2] = A[2] [0] , A[0] [2] Cache Hit, 但是 A[2] [0] Miss，带来A[2] [1], A[2] [2]...未使用
- 下一步是 A[0] [3] = A[3] [0] , A[0] [3] Cache Hit, 但是 A[3] [0] Miss，带来A[3] [1], A[3] [2]...未使用

因此对于 A 来说，access pattern 是 Miss Hit Hit Hit

对于 A 的转置来说，access pattern 是 Miss unused unused unused

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407202502443.png?raw=true)



那么接下来我们想要改善这种情况，因为cache size 太小了，所以整个过程根本无法存储所有的矩阵的元素，使用分块(Blocking) ,将原矩阵分块处理，进而提高时间局部性和空间局部性

也就是对矩阵转置的过程分成很多小块，对小块矩阵进行转置:

![img](https://inst.eecs.berkeley.edu/~cs61c/sp22/labs/lab07/matTblock.png)

需要注意的是$$ A_{12} 的转置并非在原位置的 A_{12}^{T} , 而是A_{21}^{T} $$

过程演示:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407203347633.png?raw=true)

对每个小块进行转置的过程分析:

cache足以容纳分块后每一小块所有元素,假设分块后的每个子矩阵是2x2大小

$$
\begin{aligned}
A_{11}^{\top}[0][0]=A_{11}[0][0] \\
A_{11}^{\top}[1][0]=A_{11}[0][1] \\
A_{11}^{\top}[0][1]=A_{11}[1][0] \\
A_{11}^{\top}[1][1]=A_{11}[1][1] 
\end{aligned}
$$

对 A 和 A 转置的访问将 四个元素全部存入 cache中之后便能够很好地利用时间局部性
代码:

```c
    for (int i = 0; i < n; i += blocksize) {
      for (int j = 0; j < n; j += blocksize) {
        /* B x B size matrix */
          for (int x = i; (x < i + blocksize) && (x < n); x++) {
            for (int y = j; (y < j + blocksize) && (y < n); y++) {
                A transpose[y + x * n] = A[x + y * n];
            }
          }
      }
    }
```

也就是


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407205608514.png?raw=true)

----

## 矩阵乘法优化2

接下来的内容是补充自CSAPP(15-213) 的 Lecture10 ,是Berkeley 没有讲的内容，但是lab7中有提到。

对于矩阵乘法，一般想到的是三重循环 i j k进行计算:

```c
/* ijk */
for (i=0; i<n; i++)  {
  for (j=0; j<n; j++) {
    sum = 0.0;
    for (k=0; k<n; k++) 
      sum += a[i][k] * b[k][j];
    c[i][j] = sum;
  }
} 
```

但是实际上除了i j k ,还可以有

i k j ,j i k, j k i, k i j ,k j i

总共6种顺序，也就是 $$ A_{3}^{3} $$ 

那么不同的顺序访问实际上对Cache的Hit Rate 是不同的，假设以行优先的顺序存储矩阵:

**i j k 的顺序(**我们最容易想到的顺序)

根据循环i j k的下标来看，因为函数是从最内层开始执行，我们只需看最内层的循环对Cache的使用情况

```c
sum += a[i][k] * b[k][j]
```

k从0到n ，因此标记为 * ,对于最内层的循环而言，i j 固定


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407210423070.png?raw=true)

由于是以行优先进行访问的，这里对矩阵A 的访问相当于步长为1:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407210528980.png?raw=true)

pattern : Miss Hit Hit Hit

对B 的访问相当于步长为N:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407210640811.png?raw=true)

pattern: Miss Unused Unused Unused

所以 A Miss Rate = 0.25 , B Miss Rate = 1.00

**k i j 的顺序**


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407211209150.png?raw=true)

最内层是j 在迭代，相当于i k 固定，j 对应某一行元素从0 到 n的变化，相当于扫描某一行

以i j k 的顺序进行矩阵乘法时的情况如下:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407212451821.png?raw=true)

那么i j k的顺序中:

- 一次循环则计算了 a1 x a1 ,a2 x a4, a3 x a7并得到c1

- 经过三次循环计算出c1 c2 c3

  

那么在k i j 的顺序中，矩阵乘法实际过程是:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407213453013.png?raw=true)



第一次循环计算 a1 x a1, a1 x a2, a1 x a3,这相当于把i j k 顺序下的三次循环中与a1有关的计算汇总到一步

然后同样的操作执行三次，最后累加，一下子可以得出一列的结果。



虽然i j k和 k i j都是以O(n^3)的时间复杂度计算矩阵乘法，但是二者对Cache 的利用率不同，在k i j的顺序下，Miss Rate 最低

**j k i 的顺序(the worst case)**

由于我们是将矩阵以行优先的顺序存入一维数组，j k i导致Cache Hit Rate = 0 (Miss Rate = 1)


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407213912083.png?raw=true)



还有三种顺序就不列举了，下图是Core i7对不同顺序的矩阵乘法的性能:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407214051510.png?raw=true)

---

## Blocking 的应用2

假设对于Cache 来说 ，要计算的矩阵非常大，例如8 wide 的Cache ,在执行一次循环后:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407214312112.png?raw=true)

也就相当于上面Berkeley说的:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407214531199.png?raw=true)

这种情况，标红处为Cache Size大小，由于stride太大，导致即使从内存中带来一些值，但是却暂时用不上

我们需要做的就是将矩阵分块(blocking),缩小每次处理的规模让Cache 足以容纳子矩阵的大小，从而阻止一些从内存中带来的元素Unused的发生

**最大分块大小**

由于Cache需要保证同时容纳正在计算的A , B 矩阵和输出矩阵 C, 因此blocksize满足:

![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407215037478.png?raw=true)

分块后，可见Cache 利用率非常高


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407215124084.png?raw=true)

代码:

```c
/* Multiply n x n matrices a and b  */
void mmm(double *a, double *b, double *c, int n) {
    int i, j, k;
    for (i = 0; i < n; i+=B)
	for (j = 0; j < n; j+=B)
             for (k = 0; k < n; k+=B)
		 /* B x B mini matrix multiplications */
                  for (i1 = i; i1 < i+B; i1++)
                      for (j1 = j; j1 < j+B; j1++)
                          for (k1 = k; k1 < k+B; k1++)
	                      c[i1*n+j1] += a[i1*n + k1]*b[k1*n + j1];
}

```

---

## Average Memory Access Time(AMAT)

公式:


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407215550869.png?raw=true)

例子看slide

### 影响AMAT的因素

 Associativity， block(entry) 的数量, block size

看slide

### Local Hit Rate / Global Hit Rate


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407220053674.png?raw=true)

### AMAT with 2-level Cache


![outline](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220407220135605.png?raw=true)