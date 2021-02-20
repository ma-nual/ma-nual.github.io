---
layout:     post
published:  true
title:      "struct，union占用字节数的计算"
subtitle:   "C++笔记"
date:       2021-2-20 15:00:00
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

## 字节对齐

在32位系统下，gcc的对齐方式为1,2,4，默认为4字节对齐

在64为系统下，gcc的对齐方式为1,2,4,8，默认为8字节对齐

字节对齐是为了能让计算机快速读写，是一种以空间换时间的存储策略

## struct与union介绍

### struct

结构体，相互关联的元素的集合，每个元素都有自己的内存空间，每个元素在内存中的存放是有先后顺序的，就是定义时候的顺序

### union

联合体，在一个联合体里可以定义多种不同的数据类型，这些数据共享一段内存，在不同的时间里保存不同的数据类型和长度的变量，以达到节省空间的目的，但同一时间只能存储其中一个成员变量的值

## struct占用字节数的计算

一个struct所占的总的内存大小，并不是各个元素所占空间之和，而是存在字节对齐问题

**struct占用的字节数 = 最后一个成员的偏移量 + 最后一个成员数据类型的大小 + 末尾填充字节数**

应该注意：

1. 每个成员的偏移量，即成员的实际地址离其结构的首地址的距离，要整除本身的大小，若不能整除，在其前的成员的后面进行字节填充
2. 最后的结构的大小要整除最大成员的大小，若不能整除，在最后的成员的后面进行字节填充

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
}test_struct_c; //4 + 4 + 24 + 8 = 40

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