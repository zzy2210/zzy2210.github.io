---
title: "数据结构-字典"
date: 2022-11-07T22:55:36+08:00
lastmod: 2022-11-07T22:55:36+08:00
draft: true
keywords: ["go","data structure"]
description: "完成对跳表的说明，对散列的定义，实现与应用"
tags: ["data structure"]
categories: ["go","data structure"]
author: "y1nhui"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false

# Uncomment to add to the homepage's dropdown menu; weight = order of article
# menu:
#   main:
#     parent: "docs"
#     weight: 1
---

<!--more-->
# 数据结构 字典

## 前言

这一章的撰写有些纠结。因为我个人是根据 《数据结构、算法与应用 C++ 实现》这本书来进行学习与博客编写的。
这一章的章节名是跳表与散列，但是个人的粗略翻阅，感觉跳表并不是一个主要的点，甚至他的实现都只是个可选内容。
再考虑到平时翻阅资料似乎也很少看到跳表，更多的是哈希表，所以本文的核心内容还是放在了哈希表上面。
同时考虑到线性表、跳表、哈希表这些都是为了实现字典，所以本篇博客名为字典，而非跳表与散列之类。

## 定义

### 跳表

在一个长为n的有序数组上通过折半查找找到某个数据，需要的时间为O(logn)，但是在有序链表上确是O(n)

为了优化链表上的查找，我们可以随机地创建一些新的指针来指向链表中的某些节点。这样就不需要从头遍历到尾来查询，而是直接使用某些指针来定位

这样的链表便叫做跳表，额外在家指针的节点，以及增加多少个额外指针都是通过随机算法来确定。

跳表查找、插入、删除的平均时间复杂度为O(logn)，最坏情况为O(n)

### 散列表/哈希表


hash表可以将平均时间复杂度提快到O(1)，最快情况下为O(n)，不过如果是需要经常**按序**输出所有元素或者查找元素的话，跳表执行效率优于hash表



### 字典

dict与map理应没有区别，是由一组K-v 键值对组成的集合

多重字典：k-n*v 的一对多键值对组成的集合，当然，暂时不在我们的讨论范围内

定义：
``` go
type Dict interface {
	Empty() bool
	Size() int
	// 根据k返回v
	Get(key interface{}) (interface{}, bool)
	// 插入键值对
	Insert(key, val interface{})
	// 根据k删除键值对
	Erase(key interface{})
}
```

一般字典地Get除了通过关键字来随机地查找键值对外，还进行顺序存取，即利用迭代器按照关键字地升值顺序来逐渐查个查键值对


## 实现

### 线性表

没错，除了我们常用的hash表，或者本章提了一嘴的跳表外，我们甚至可以使用线性表来实现字典

