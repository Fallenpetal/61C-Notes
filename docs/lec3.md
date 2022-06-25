---
layout: default
title: C Intro - Pointers, Arrays, Strings
author: liuxing
nav_order: 3
---

# C Intro - Pointers, Arrays, Strings
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Pointer and Array

C的指针大小取决于计算机的架构

32位的机器，则指针大小为32bit，

64位的机器，则指针大小为64bit，指针的大小为基本的word size



如果你做这样的事:

```c
int *foo = NULL;
*foo = 32;
y = *foo
```

当你尝试给*foo赋值或者dereference时，你的程序就会崩溃，因为空指针并不指向任何东西，如果你尝试写入或读取，则会遇到问题

对于指针运算需要格外小心，尽量少做

```c
int a[10];
int *p = a;
p++;
```

这样的操作，因为

![image-20220225165008282](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220225165008282.png?raw=trueg)

如果p本身就被分配在a的后一位的内存地址上时，执行p++相当于将指针指向p自己，这会造成十分困惑

### sizeof()

sizeof()不是函数，而是在complie time时执行的操作，进而替换值

![image-20220225170526710](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220225170526710.png?raw=trueg)

sizeof(array)会返回array中所有元素使用的memory space

sizeof(pointer)只会返回单一pointer的memeory amount



将数组作为参数传递入函数，则会丢失数组的大小！因为只是相当于传入数组的首地址

```c
int foo(int array[],unsigned int size){};
```

因此需要将数组的大小作为参数传递



C  的数组声明不能包含变量

```c
int ar[x]
```

如果x 是变量，那么会报错



C 的数组连他自己都不知道其数组长度，例如，当你写下

```c
	int a[5] = {0,1,2,3,4};
	a[6] = 7;
```

即使是数组越界了也能运行，谁知道内存中的某一个地址已经被赋值为7了呢？

### Use Defined Constants

当你对一个数组使用循环:

```c
int i, ar[10];
for(i = 0; i < 10; i++){ ... }
```

上述方式不如定义一个常数来代表数组的大小：

```c
const int ARRAY_SIZE = 10;
int i, a[ARRAY_SIZE];
for(i = 0; i < ARRAY_SIZE; i++){ ... }
```

这样做的好处是免去copy两次”10“，并且未来需要修改的时候，只需修改ARRAY_SIZE而不必修改两次”10“

---

## C Strings

C中没有String类型，字符串是用字符数组来表示

```c
	char c[] = "abc";
	printf("%d %d",sizeof(c),strlen(c));         // the output is 4,3
```

当你这样声明一个字符数组，会自动添加一个终止符'\0'，且占1 byte，长度只有3，不包含 null terminator, 但是大小是 4 byte 

---

## Unions

Unions类似于Struct，但是Union只分配其内最大成员变量的memory space，Union可以被看其成成员变量类型其中的任意一种，例如:

```c
union foo {
	int a;
	char b;
	union foo *c;
}

union foo f;
f.a = 5       //将f看作int
f.c = &f      //将f看作union指针
```

每次对union进行赋值时发生覆盖

---

## Endianness

```c
union confuzzle { int a; char b[4]; };
union confuzzle foo;
foo.a = 0x12345678; 
```

在32bit的架构上，f.b[0]是什么？

实际上与架构的endianness有关

Big endian : The first character is the most significant byte: 0x12

Little endian: The first character is the least significant byte: 0x78

---

## C Memory Management

程序的地址空间包含4个区域：

stack: 函数调用，向下增长

heap:动态内存空间的分配函数，包括calloc(),malloc(),realloc(),free()，向上增长

static data:全局变量

code:程序开始时加载

![image-20220225172101128](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220225172101128.png?raw=trueg)

如果一个变量是在函数内，那么它将被分配于stack，函数外的全局变量，将被分配于static data

**Note** : main()也是函数

### The Stack

- 每当有函数被调用时，一个新的'stack frame'被分配到stack上

- stack frame包括：

  返回地址(who called me?)

  参数

  函数内的局部变量的空间

stack frame使用连续的memory，且有一个stack pointer,当一个函数完成调用后，stack pointer上移，完成释放

![image-20220225172702745](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220225172702745.png?raw=trueg)



### Managing the Heap

C提供了一些管理Heap的函数:

- malloc()    分配一块未初始化的memory,返回void*,一般进行cast
- calloc()     分配一块memory，初始化为全0,返回void*，一般进行cast
- free()     释放先前分配的memory,不返回任何值
- realloc()   改变先前分配的memory大小,返回void*

可见malloc()的速度要优于calloc()，后者还需要将memory初始化为全0

---

---

## Discussion 2 solutions

如果对一个非指针变量dereference,则会将该变量当作指针，强行非法访问该变量所指向的内存区域

```c
int x = 0xff00;
*x     //意味着去到0xff00这个位置访问该地址上存储的值
```

如果对非指针变量进行free()，则会发生invalid free error   例如:

```c
int x;
free(x);
```

bitwise exclusive-or(按位异或)可以实现交换两个变量的值的功能:

```c
int x,y;
x=x ^ y;
y=x ^ y;
x=x ^ y;
```

这样就将x和y的值交换了，原理是:

- x = x ^ y         先将x和y不相交的部分赋给x(相交部分从集合中移除), x现在为集合(X or Y)-(X and Y) | 

- y = x ^ y         将x与y的相交部分又加入集合内了，但是把y所独立含有的部分移除了，即现在的状态y的值为集合X。|

- x = x ^ y         这时候将前面每一步的集合结果相交的部分拿掉最后得到的结果为Y|

对于字符数组，以'\0'作为终止符，例如以下函数:

```c
void increment(char *string, int n) {
	for (int i = 0; i < n; i++) {
		*(string + i)++;
	}
}
```

该函数想要实现将原来的string数组中的每一个元素都增大1，但是实际上n并不是数组的末尾，正确的写法应该是:

```c
void increment(char *string, int n) {
	for (int i = 0; string[i] != 0; i++) {
		string[i]++;
	}
}
```

---