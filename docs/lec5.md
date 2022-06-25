---
layout: default
title: Floating Point
author: liuxing
math: mathjax2
nav_order: 5
---

# Floating Point
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Shift Operation

给定n bit 大小的数，shift操作后会发生截断，例如以下 4 bit 数

Left Shift : a << b  代表 a 左移 b 位，相当于乘以 $$2^b$$ ,后面补0

``` 
0b1001 << 2 = 0b0100     //最高位1被截断
```

Right Shift : 需要区分是算术右移还是逻辑右移，

如果是逻辑右移:

那么和左移操作一致，

如果是算术右移，右移后需要扩展符号位:

```
0b0101 >> 2 = 0b0001   //符号位 = 最高位 = 0
```

```
0b1101 >> 2 = 0b1111    //符号位 = 1，需要扩展1
```

与左移操作不同的是，由于舍入的原因，右移 b 位并不能看作是除以 $2^b$ 

---

## Scientific Notation(Normalized Form)

回忆高中学习的科学计数法，光速 $$ c = 3.0 * 10^8 m/s $$ ，分别解释一下科学计数法的各部分

![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302190739527.png?raw=true)

小数点后面的称为 **Significand/Mantissa** ,例如上图Mantissa是0.2318，10进制的 Base 是10，次方是 Exponent，从个位向左，权值分别是 $$10^0 , 10^1 , 10^2 , 10^3......$$ ，从个位向右，权值分别是 $$ 10^{-1},10^{-2},10^{-3}...... $$ 

那么同样的方法，对于二进制数，Base 是 2，各位之间的权值是

![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302191142848.png?raw=true)

与十进制类似地，二进制的科学计数法表示如下

![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302191317226.png?raw=true)

让我们考虑一下如何将上图中的浮点数用二进制存储下来

**Define Normalized form**

- 二进制的规格化形式与科学计数法定义类似，小数点左边必须是非0值，由于二进制的取值只有0和1，因此小数点左边必须为1，这样的数称为 Noramlized Number

- 由于小数点左边的值一定为1，因此在存储浮点数的时候，可以省略对1的存储，在将二进制数转换为人类可读的十进制数时再将1还原，只需保存 0.0101
- 显然我们在二进制下进行存储，那么对于 Base = 2 也不用去存

最后我们需要保存的只剩下 Mantissa 和 Exponent ，其实还有一个表示正负的符号位 sign



---

## Floating point diagram

单精度浮点数使用 4 byte进行存储，根据上面的分析，将一个 32bit 的空间分为以下几部分


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302191712404.png?raw=true)

- 最高位的1bit S 位用于存放符号位 ，0表示正数，1表示负数
- 中间的8bits 用于存放Exponent，范围是[0,255]
- 最后23位用于存放 Mantissa，也就是尾数，如果尾数没有23位，后面补0就行了，1.0和1.0000无区别

根据 IEEE754 标准，在二进制转换为十进制的时候，还需要将 Exponent 减去一个 Bias，
$$
Bias=2^N-1
$$
N 是位数，因此这里 $$Bias = 2^8-1=127$$ ,所以最终进行十进制转换时，需要 Exponent - 127

---

## Normalized Number

对于规格化数字，Exponent的范围是[1,254], 0和255表示其他含义，因此根据上面的分析，二进制转10进制的公式是


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302193843487.png?raw=true)

转换方法说明:

- 对于二进制转10进制，将32bit 划分为1 bit,8 bit,23 bit，然后带入上面的公式即可
- 对于10进制转为二进制，对小数点左边和右边分别做二进制转换，然后使用科学计数法表示二进制，将小数点右边的作为mantissa，指数还需要**加127** (2转10时减去127，此时要加回来)作为Exponent 即可


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302194702551.png?raw=true)


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302194727119.png?raw=true)

---

## Rounding

在做计算或者转换的时候，容易发生舍入，比如：

- 双精度浮点数转换为单精度浮点数  Double --> Float
- 浮点数转换为整数 Float --> Integer

一共有四种舍入规则:

1. Round to Nearest

向最近的数舍入，如果刚好位于二者中间时，则向偶数舍入

```
2.4 -> 2   2.5 -> 2
```

2. Round toward 0

向0舍入，也就是向数轴上接近原点的方向舍入

```
2.001 -> 2    -2.999 -> -2
```

3. Round toward $$ +\infty $$ 
4. Round toward $$-\infty$$ 

### How to Represent 0 ?

- Sign = 0 or 1 ,也就是 +0 和 -0
- Exponent = all zeros (0)
- Mantissa = all zeros

### How to Represent Infinity ?

- Sign = 0 or 1，也就是 $$ +\infty 和 -\infty $$ 

- Exponent = all ones (255)
- Mantissa = all zeros

也就是最开始说的，Exp[0,254]表示规格化数字，0和255表示其他

### NaN ( Not a Number )

对于一些不是数字的值，例如 $$ \sqrt{-1} $$ 等等，计算机如何去表示

- Sign = 0 or 1
- Exponent = all ones(255)
- Mantissa = 非0值

### The Problem of Precison of Representation of Normalized Number

对于规格化数字所能表示的最小值是


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302200442860.png?raw=true)

最大值是

![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302200559889.png?raw=true)

因此规格化数字，所能表示的正数范围是 $$ [2^{-126},(2-2^{-23}) \cdot 2^{127}] $$   ,负数范围是$$ [-(2-2^{-23}) \cdot 2^{127},2^{-126}] $$ ，只有符号位不同的差别


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201031391.png?raw=true)

**Overflow 与 Underflow**


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201125438.png?raw=true)

**Floating Point Step Size**

规格化浮点数的步长公式 : 
$$
2^{x-150}
$$
x是unbiased的值，也就是 Exponent [1,254]，相当于x每增加1，步长增加2倍


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201542113.png?raw=true)


---


## Denormalized Numbers

由于规格化数字精度不够，因此对于十分接近0的数，使用非规格化的数字表示


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201711611.png?raw=true)

这里要求小数点最左边的数是0，而不是规格化表示的1，因此左移1位，相当于乘以2，而exponent = 0,所以变成乘以 $$2^{-126}$$

**Denorm Range**


![image-20220302190739527](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302190739527.png?raw=true)

**Denorm Step Size**

![image-20220302201848975](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201848975.png?raw=true)



非规格化数字的步长是常数，一直都是 $$ 2^{-149} $$

---

## Important Table

![image-20220302201935098](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220302201935098.png?raw=true)



**Note**:

- 浮点数不满足加法结合律，Big + Big + Small != Big + Small + Big |

---

## A Tips For Convert

一个小例子，如何去表示 0.3?

```
0.3 x 2 = 0.6 ---0
0.6 x 2 = 1.2 ---1
0.2 x 2 = 0.4 ---0
0.4 x 2 = 0.8 ---0
0.8 x 2 = 1.6 ---1
0.6 x 2 = 1.2 ---1
```

出现两次0.6，找到循环结 因此循环是1001 1001 1001
所以Mantissa = 01001 1001 1001 .... 由于尾数只有23位，因此后面会发生截断，也就是说计算机无法精确表示0.3，只能表示大约值

上面的方法步骤为：

- 每次乘2，标记小数点左边的数（0或1）
- 如果出现大于1，那么继续用大于1的部分（截去1）做乘以2的操作
- 直至出现循环