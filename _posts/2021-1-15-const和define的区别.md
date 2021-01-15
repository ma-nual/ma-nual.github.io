---
layout:     post
published:  true
title:      "const和define的联系和区别"
subtitle:   "C++笔记"
date:       2021-1-15 15:00:00
author:     "Manual"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
---

> 本文介绍了const和define的区别

## 联系

它们都是定义常量的一种方法。

## 区别

* define定义的常量**没有类型**，const定义的常量是**有类型**的。
* define定义的常量**没有分配内存空间**，只是进行了简单的**替换**，const定义的常量**要分配内存空间**，存放在**静态存储区**。
* define定义的常量可能会有**多个拷贝**，占用的内存空间大，const定义的常量只有**一个拷贝**，占用的内存空间小。
* define定义的常量是在**预处理阶段**进行替换，而const在**编译阶段**确定它的值。
* define**不会进行类型安全检查**，而const**会进行类型安全检查**，安全性更高。
* define**可以定义函数**，而const**不可以**。
* define**不受定义域限制**，而const**有定义域限制**。
* define可以通过`#undef`来**使之前的define失效**，const常量定义后**将在定义域内永久有效**。
* define是**不能进行调试**的，因为在预编译阶段就已经替换掉了，const常量是**能进行调试**的。