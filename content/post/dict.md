---
title: "数据结构-字典"
date: 2022-11-07T22:55:36+08:00
lastmod: 2022-11-07T22:55:36+08:00
draft: false
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

假设数值对p的关键字为k，有一hash函数函数为f，那么理想情况下，p在hash表中的位置是f(k)。如果f(k)为nil，则说明没有记录

hash表可以将平均时间复杂度提快到O(1)，最快情况下为O(n)，不过如果是需要经常**按序**输出所有元素或者查找元素的话，跳表执行效率优于hash表


为什么说是理想情况呢？

我们按照这种定义来做个情景应用

假设现在有100个学生，但是他们的学生编号是`20220001-20229000`
`f=k-20220000`
那么这种情况下，关键字的范围有是`20220001`~`20229000`
f(k)的范围也是`1`~`9000`

那我们就需要创建一个长为9000的的散列表来容纳，而且实际使用可能只有100。这是十分不明智地，而且初始化一个如此长的散列表，所需要的时间也是很高的

所以当关键字范围很大时，就不采用理想方式


非理想情况下的hash表，它的位置数量（可以理解为链表中的节点数量，数组中的长度）小于关键字的范围，hash函数会将若干个不同的关键字映射到同一个位置，即可能发生`f(k1)`=`f(k2)`=`f(k3)`=...=`f（kn）`

这种非理想情况下，hash表的每一个位置被称为一个桶（bucket）,f(k)就是起始桶

	ps: 之所以叫起始桶而不是直接叫桶的地址之类的，是因为在开放寻址法的情况下，如果出现了冲突+溢出，实际使用的桶是要往后面寻找的，而不是算出的这个结果，所以被称为起始桶

这种情况下的散列函数有很多选择，最简单的就是*除法散列函数*

`f(k)=k%D `D指桶的数量

#### 冲突

多个关键字对应的起始桶相同时，就是冲突

#### 溢出

如果储存桶没有空间储存一个新的数对，就是溢出

当溢出发生时，有多种处理方法。最常见的是线性探查法，其余方法还有平法探查法，双重散列法

#### 均匀散列函数

使用该hash函数映射到每一个桶的关键字的数量大致相等，冲突和溢出的平均数最少

#### 良好散列函数

实际应用中性能表现好的均匀散列函数

#### 线性探查法

在发生溢出后，将数值对放入下一个桶中。

比如现在有 0-10 是十一个桶，使用hash函数为`f(k)=k%11`，每一个桶只能容纳一个键值对 

先插入 80，40，65.则分别放入了 3，7，10 三个桶

接着插入24，58。 24放入桶2，没有问题，58本因放入桶3，但是桶3已有80，故向后寻址，放入桶4。

接着放入32，本因放入桶10，但是桶10已有65。这种情况下可以将之视为一个循环队列。桶10之后是桶0，将它放入桶0中。

上面的文字描述便可以很好地解释线性探查法的寻找与插入过程。

寻找：

	先查看起始桶中有无对应键值对，若无，则向后寻找。如果一直找回初始解桶还是不存在，或者找到了一个未溢出的桶，则说明该键值对不存在


插入：

	先向起始桶插入，如果起始桶溢出，则向后一个插入。如果全部都溢出，则返回错误

删除：

	这种情况下的删除比较麻烦。不能只把桶中对应的键值对删除就了事。还需要从删除位置的下一个桶开始，逐个检查每一个桶，来判断每一个桶中的元素是否实在起始桶，是否需要向移动一次，直到抵达一个未溢出的桶或者回到删除位置为止。

#### 链式散列

如果桶可以容量无数个键值对，那就不需要纠结溢出的问题了。

所以我们将每个桶都配置一个线性表，自然就可以了。

#### 负载因子

容纳的键值对数量处于桶的数量便是负载因子。

一般情况下负载因子越高，hash表的性能就越低，因而当负载因子达到一定程度的时候便应该启用hash扩容。

在Go原生的map中，当负载因子达到6.5时，便会启动扩容

可见`runtime/map.go`:

``` go
const (
	// Maximum number of key/elem pairs a bucket can hold.
	bucketCntBits = 3
	bucketCnt     = 1 << bucketCntBits

	// Maximum average load of a bucket that triggers growth is 6.5.
	// Represent as loadFactorNum/loadFactorDen, to allow integer math.
	loadFactorNum = 13
	loadFactorDen = 2
）
```

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

### hash 表

本来这里打算写一下线性表的dic实现，但是仔细想了想，线性表无非是之前的interface改成一个结构体里面两个interface，没有什么变动，就决定跳过线性表实现，直接开始写链式实现的hash

不过这里的hash map 是很不可靠的，比如没有扩容机制等，只能作为一个加强对链表hash理解的学习代码

```go
package hash

type HashDict struct {
	Buckets []*chainNode
	size    int
}

type chainNode struct {
	Key  interface{}
	Val  interface{}
	Next *chainNode
}

func NewHashDict() *HashDict {
	return &HashDict{
		Buckets: make([]*chainNode, 11),
	}
}

func (d *HashDict) Empty() bool {
	return d.size == 0
}

func (d *HashDict) Size() int {
	return d.size
}

// 这里使用hash函数为f(k)=k%11
func (d *HashDict) Get(key interface{}) (interface{}, bool) {
	var bucket int
	switch key.(type) {
	case int:
		bucket = key.(int) % 11
	case string:
		var ascii rune
		for _, v := range key.(string) {
			ascii += v
		}
		bucket = int(ascii) % 11
	default:
		// 个人能力有限，仅提供对string与int类型
		return nil, false
	}
	for node := d.Buckets[bucket]; node != nil; node = node.Next {
		if node.Key == key {
			return node.Val, true
		}
	}
	return nil, false
}

func (d *HashDict) Insert(key, val interface{}) bool {
	var bucket int
	switch key.(type) {
	case int:
		bucket = key.(int) % 11
	case string:
		var ascii rune
		for _, v := range key.(string) {
			ascii += v
		}
		bucket = int(ascii) % 11
	default:
		// 个人能力有限，仅提供对string与int类型
		return false
	}
	if d.Buckets[bucket] == nil {
		d.Buckets[bucket] = &chainNode{
			Key: key,
			Val: val,
		}
		d.size++
		return true
	}
	for node := d.Buckets[bucket]; node != nil; node = node.Next {
		if node.Key == key {
			node.Val = val
		}
		if node.Next == nil {
			node.Next = &chainNode{
				Key: key,
				Val: val,
			}
			d.size++
		}
	}
	return true
}

func (d *HashDict) Erase(key interface{}) bool {
	var bucket int
	switch key.(type) {
	case int:
		bucket = key.(int) % 11
	case string:
		var ascii rune
		for _, v := range key.(string) {
			ascii += v
		}
		bucket = int(ascii) % 11
	default:
		// 个人能力有限，仅提供对string与int类型
		return false
	}
	if d.Buckets[bucket] == nil {
		return true
	} else if d.Buckets[bucket].Key == key {
		d.Buckets[bucket] = d.Buckets[bucket].Next
		d.size--
	}
	for node := d.Buckets[bucket]; node.Next != nil; node = node.Next {
		if node.Next.Key == key {
			node.Next = node.Next.Next
			d.size--
			return true
		}
	}
	return true
}

```

## 应用

字典（hash实现）的优点就再与基于关键字存取。 

本系列开篇文章说过，我个人是因为某次优化突然脑子转不过来了，于是打算回顾一下相关知识，重开了博客

这里就假设一个大概相似的情景

现在从外界传入了一串字符串，我们需要将之与黑名单对比，并且删去黑名单内字符，输出余下的内容

看上去很简单吧

我当时脑子一抽，写出了第一个版本

``` go 
func demo(val []string) []string {
	black := []string{"a", "b", "c"}
	for i, b := range val {
		for _, v := range black {
			if v == b {
				val = append(val[:i], val[i:]...)
			}
		}
	}
	return val
}
```
先不论代码是否有点问题，单是这两个for循环一套，直接来了个O(n^2)，就应该感到了深深的难受

那怎么办呢， 用 就用字典（map）吧

``` go
func demo2(val []string) []string {
	black := []string{"a", "b", "c"}
	dict := make(map[string]bool)
	for _, v := range val {
		dict[v] = true
	}
	for _, v := range black {
		_, ok := dict[v]
		if ok {
			dict[v] = false
		}
	}
	var result []string
	for k, v := range dict {
		if v == true {
			result = append(result, k)
		}
	}
	return result
}
```
明显可见，没有O(n^2)现象

## 总结

这篇文章，写的有点急了。最后在写应用的时候，有点迷茫不知道该怎么说应用了。leetcode上面看了看hash标签的题目，好像也不是很好体现

最后干脆心一横，拿之前脑抽的事情，大概描述一下写出来了。
