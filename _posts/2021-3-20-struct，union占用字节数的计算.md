---
layout:     post
published:  true
title:      "struct，union占用字节数的计算"
subtitle:   "C++笔记"
date:       2021-3-20 15:00:00
author:     "Manual"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
---

> 本文介绍了数据类型，struct，union占用字节数的计算方法

## 存储空间

常用数据类型占用额存储空间

| 数据类型        | 32位编译器 | 64位编译器 |
| :-------------- | :--------- | :--------- |
| char            | 1byte      | 1byte      |
| int             | 4byte      | 4byte      |
| float           | 4byte      | 4byte      |
| double          | 8byte      | 8byte      |
| short           | 2byte      | 2byte      |
| unsigned int    | 4byte      | 4byte      |
| long            | 4byte      | 8byte      |
| unsigned long   | 4byte      | 8byte      |
| long long       | 8byte      | 8byte      |
| 指针            | 4byte      | 8byte      |
| enum(占用同int) | 4byte      | 4byte      |

## 内存字节对齐

在32位系统下，gcc的对齐方式为1,2,4，默认为4字节对齐

在64为系统下，gcc的对齐方式为1,2,4,8，默认为8字节对齐

`#pragma pack(n)` 表示的是设置n字节对齐，windows默认是8，linux是4

```c++
struct A{
    char a;
    int b;
    short c;
};
```

char占一个字节，起始偏移为零，int占四个字节，min(8,4)=4；所以应该偏移量为4，所以应该在char后面加上三个字节，不存放任何东西，short占两个字节，min(8,2)=2;所以偏移量是2的倍数，而short偏移量是8，是2的倍数，所以无需添加任何字节，所以第一个规则对齐之后内存状态为`0xxx|0000|00`

此时一共占了10个字节，但是还有结构体本身的对齐，min(8,4)=4；所以总体应该是4的倍数，所以还需要添加两个字节在最后面，所以内存存储状态变为了 `0xxx|0000|00xx`，一共占据了12个字节

**需要对齐的原因**

- 平台原因（移植原因）：不是所有的硬件平台都能访问任意地址上的任意数据，某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常
- 硬件原因：经过内存对齐之后，CPU的内存访问速度大大提升，访问未对齐的内存，处理器要访问两次（数据先读高位，再读低位），访问对齐的内存，处理器只要访问一次，为了能让计算机快速读写，提高处理器读取数据的效率，我们使用内存对齐

## struct与union介绍

### struct

结构体，相互关联的元素的集合，每个元素都有自己的内存空间，每个元素在内存中的存放是有先后顺序的，就是定义时候的顺序

### union

联合体，在一个联合体里可以定义多种不同的数据类型，这些数据共享一段内存，在不同的时间里保存不同的数据类型和长度的变量，以达到节省空间的目的，但同一时间只能存储其中一个成员变量的值

## struct占用字节数的计算

一个struct所占的总的内存大小，并不是各个元素所占空间之和，而是存在字节对齐问题

**struct占用的字节数 = 最后一个成员的偏移量 + 最后一个成员数据类型的大小 + 末尾填充字节数**

应该注意：

1. 每个成员的偏移量，即成员的实际地址离其结构的首地址的距离，要整除`min(#pragma pack()指定的数,这个数据成员的自身长度)`，若不能整除，在其前的成员的后面进行字节填充
2. 最后的结构的大小要整除`min(#pragram pack() , 长度最长的数据成员)`，若不能整除，在最后的成员的后面进行字节填充

```c++
#include <cstdio>
#include <iostream>
using namespace std;

struct E1 {
	int a; char b; char c;
}e1; //4 + 1 + 1 + 2 = 8

struct E2 {
	char b; int a; char c;
}e2; //1 + 3 + 4 + 1 + 3 = 12

struct B{
    char a;
    double b;
    int c;
}test_struct_b; //1 + 7 + 8 + 4 + 4 = 24

struct E{

}test_struct_e; //空结构体占用字节数为1，编译器默认分配了一个字节，以便标记可能初始化的类实例，同时使空类占用的空间也最少（即1字节）

int main() {
	cout << sizeof(E1) << endl; //8
	
	cout << sizeof(E2) << endl; //12
  
  cout << sizeof(test_struct_b) << endl; //24
  
  cout << sizeof(test_struct_e) << endl; //1
  
	return 0;
}
```

## union占用字节数的计算

联合体的所有的成员相对于基地址的偏移量都为0，他的结构空间要大到足够容纳最“宽”的成员，并且对齐方式要适合于所有公用体中所有类型的成员

**union占用的字节数 = max(成员的偏移量) + 某位填充字节**

应该注意：

max(成员的偏移量)要整除各个成员，若不能整除，在最后的成员的后面字节填充

```c++
#include <cstdio>
#include <iostream>
using namespace std;

union A{
    int a[5];
    char b;
    double c;
}; //20 + 4 = 24

int main() {
  cout << sizeof(A) << endl; //24
	return 0;
}
```

## 混合结构体占用字节数的计算

```c++
#include <cstdio>
#include <iostream>
using namespace std;
#define LL long long 

struct E5 {
	char a1,a2,a3,a4,a5,a6;
}e5; //1 + 1 + 1 + 1 + 1 + 1 = 6
struct E6 {
	char a1,a2,a3;
}e6; //1 + 1 + 1 = 3
struct E7 {
	struct E5 elem5;
	struct E6 elem6;
	LL a;
}e7; //6 + 3 + 7 + 8 = 24

struct E8 {
	char a[9];
}e8; //9
struct E9 {
	struct E8 elem8;
	LL a;
}e9; //9 + 7 + 8 = 24

typedef union{
    long i;
    int k[5];
    char c;
}UDATA; //4 * 5 + 4 = 24

struct C{
    int cat;
    UDATA cow;
    double dog;
}test_struct_c; //4 + 24 + 4 + 8 = 40

UDATA temp;

int main() {
	cout << sizeof(E5) << endl; //6
  
	cout << sizeof(E6) << endl; //3
  
	cout << sizeof(E7) << endl; //24
	
	cout << sizeof(E8) << endl; //9
  
	cout << sizeof(E9) << endl; //24
  
  cout << sizeof(test_struct_c) + sizeof(temp) << endl; //40 + 24 = 64
	return 0;
}
```

首先，作为成员变量的结构体的偏移量必须是自己最大成员类型字节长度的整数倍，其次，整体的大小应该是结构体中最大基本类型成员的整数倍，结构体中字节数最大的基本数据类型，应该包括内部结构体的成员变量，在64位机器上，结果如上程序

