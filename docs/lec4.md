---
layout: default
title: C Memory Management
author: liuxing
nav_order: 4
---

# C Memory Management
{: .no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## Heap Managing

```c
void *realloc(void *p, size_t size)
```

重新分配之前分配的内存块大小,返回指向内存块第一个字节的新地址

- 如果 p = NULL，则相当于malloc()
- 如果 size = 0,则相当于free()

Note : realloc()可能会copy data，例如:

![image-20220226102213264](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220226102213264.png?raw=true)

假如当前位置的内存足够，则可以连续添加，相当于数组翻倍，

否则会去寻找合适的大小，将原来的数据copy到新的memeory space，再返回地址p,并free原来的内存块

### constant strings

```c
char *foo = "this is a constant";
char *bar = "this is a constant";
```

则事实上foo和bar指向同一个字符串，如果我对常量字符串进行写入:

```c
bar[3] = 'x';
```

属于undefined behavior， 程序会立即crash

当编译器看到

```c
char *foo = "abc";
```

编译器将其解释为：在 static memory中有常量字符串 "abc"，并且有一个foo指针指向它

当编译器看到

```c
char foo[] = "abc";
```

编译器将其解释为: 在 stack 上创建一个 4 字符 数组，初始化为"abc"

### Structure Layout In Memeory & Default Alignment Rules

假设考虑32 bit - architecture 的机器上:

- char : 1 byte,no alignment needed

- short: 2 bytes, 1/2 world aligned (also called half-words)，即2 byte 的 align

  ​     因此short的起始地址是 0,2,4,6....

- int : 4 bytes, one word

  ​     int的起始地址是0.4,8,12....相当于mod 4 == 0

-  pointer : same as int, 4 bytes, one word  

举例:

```
struct foo {
	int a;      //At 0
	char b;		//At 4
    short c;	//At  6
	char *d;	//At 8
	char e;		//At 12
}
```

![image-20220226124210921](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220226124210921.png?raw=true)

sizeof(foo) = 16 ,需要在最后再加3 byte作为padding，因为如果我们allocate 两个struct 的话，它们也需要对齐(aligned)



### How are Malloc / Free Implemented?

底层操作系统允许 malloc library 在 Heap 中使用 a large blocks of memory

例如 Unix **sbrk()** 

C standard malloc library 会追踪未使用的 free memeory space

![image-20220226150845602](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220226150845602.png?raw=true)

**The Fragmentation Problems**

计算机总喜欢使用连续的内存空间，例如当你

```c
int a[10];
for (int i = 0; i <= 10;i++) {
	a[i] = 1;
}
```

write off the array，则会去上图 Heap中 空闲的空间中任意开辟一块供你写入，造成内存分配不再连续，长此以往地积累最终导致程序crash

**faster malloc implementations**

为不同大小的 objects 保留不同的 blocks
- Buddy Allocators "总是四舍五入到2次方大小的 
blocks，以简化寻找正确的 size 和合并 
相邻的blocks。
- 然后可以使用一个简单的位图来知道哪些内存空间是空闲的，哪些是被占用的。

![image-20220226151532949](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220226151532949.png?raw=true)



*Malloc Implementations*

- 均提供相同的 library 接口，但是有着截然不同的实现
- 在已分配的 blocks 和未分配的 blocks 开头使用 header 去存储 malloc 内部的数据结构

---

## Common Memory Problems

### 1. Memory Leak

```c
int *pi;
void foo() {
	pi = malloc(8*sizeof(int));
	...
	free(pi);
}

void main() {
	pi = malloc(4*sizeof(int));
	foo();
}
```

在main()函数中首先给指针 pi 分配 4 int 的内存空间，然后调用foo()的过程中，重新给 pi 分配 8 int 的内存空间，之前的 pi 指向 4 int 内存空间，但是随后被覆盖，指针丢失

memory leak 也会导致 fragmentation

---

### 2. Writing off the end of arrays

```c
int *foo = (int *) malloc(sizeof(int) * 100);
    int i;
    ....
    for(i = 0; i <= 100; ++i){
    	foo[i] = 0;
}
```

Likewise，向 Heap free space中 随机一块地址写入，造成 fragmentation

---

### 3. Return Pointer into the Stack

```c
char *append(const char* s1, const char *s2) {
    const int MAXSIZE = 128;
    char result[MAXSIZE];
    int i=0, j=0;
    for (j=0; i<MAXSIZE-1 && j<strlen(s1); i++,j++) {
    result[i] = s1[j];
    }
    for (j=0; i<MAXSIZE-1 && j<strlen(s2); i++,j++) {
    result[i] = s2[j];
    }
    result[++i] = '\0';
    return result;
}
```

例如这个append()函数，返回一个指向stack-allocated memroy 的指针，可能会出现问题，回忆一下关于Stack：

![image-20220226160923804](https://github.com/Fallenpetal/Pictures-Library/blob/master/image-20220226160923804.png?raw=true)

- **Note** : 每次有函数调用都会压入一个函数栈帧(stack frame)，并有一个指向栈顶的指针，函数调用完毕后，栈顶指针上移，栈指针恢复的时候，原来栈帧的内容并没有被清0，因为result是局部变量，存储在栈帧上的。当下一次调用另一个函数时，栈帧就会被覆盖，result的内容就变成未知的了，You Lost your data!（如果你没有在函数外用寄存器来存储这个返回值）|

---

### 4. Use After Free

```c
 struct foo *f
....
f = malloc(sizeof(struct foo));
....
free(f)
....
bar(f->a); 
```

当你已经free(f)之后，你再去访问 f 可能会错误

当其他东西占据了那块内存，你的程序可能得到错误信息，此时写入则会破坏掉其他数据

---

### 5. Forget Ralloc() Can Move Data

```c
struct foo *f = malloc(sizeof(struct foo) * 10);
...
struct foo *g = f;
....
f = realloc(sizeof(struct foo) * 20); 
```

将 f 和 g 指向同一块内存，但是又重新分配了一块新的内存并将新指针返回给 f，此时 g 指向的原内存可能已被移走(与当前是否足够分配连续内存有关，如果不能会去其他地方开辟)，则此时对 g 读写可能会出现错误

---

### 6. Free Wrong Stuff

```c
struct foo *f = malloc(sizeof(struct foo) * 10)
...
f++;
...
free(f)
```

现在f 并非指向malloc()返回的地址，free()只知道malloc()返回的指针所指向的内存块大小，如果该指针非malloc()返回，造成迷惑

---

### 7. Double Free

```c
struct foo *f = (struct foo *) malloc(sizeof(struct foo) * 10);
...
free(f);
...
free(f);
```

在free(f)之后，假设其他的 malloc() 恰好分配到了刚刚free()的内存块，则会将这块内存释放

### 8. Consider null pointer case

```c
int findLastNodeValue(Node* head) {
 while (head->next != NULL) { 
 head = head->next;
 }
 return head->val;
 }
```

如果 head 为 null，则会出现空指针