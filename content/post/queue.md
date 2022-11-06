---
title: "数据结构-队列"
date: 2022-10-31T21:58:57+08:00
lastmod: 2022-10-31T21:58:57+08:00
draft: false
keywords: ["go","data structure"]
description: "完成对队列的定义，实现与应用"
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

# 数据结构 队列

## 定义

 队列：遵循先进先出原则的线性表。其插入与删除操作分别在表的两端进行。进行插入操作的是队尾(back/rear)，删除操作的是队首(front)

 常见方法：
 ```go
type Queue interface {
	Empty() bool
	Size() int
	// 返回队首值
	Front() (interface{}, bool)
	// 返回队尾值
	Back() (interface{}, bool)
	// 返回并删除队首元素
	Pop() (interface{}, bool)
	// 将元素插入队尾
	Push(interface{}) bool
}
 ```

 ## 实现

 基于定义，编写数组与链表实现的队列

### 数组实现

在编写数组实现之前，我们首先需要思考一个问题。
front、back、size的关系应该是什么。

基于定义，首先想到

 `front = a[0]`,`back=a[size-1]`

 也就是

 `location(i)=i`

如果是这样的话，我们每次进行插入操作，只需要直接将`a[size]`写入值即可，也就是一个 O(1) 操作

但是删除的话，我们就需要将全部元素向前移动一位，这样的话就是O(n)

那如果更改一下呢

`back = a[front+size]`

即 `location(i)=location(front+i)`

用图像表示的话如下：

![图1](/static/queue/queue-1.png)

手绘，字丑，意会即可

如此操作的话，无论是pop还是push，都只是移动指针而不是元素，也就都是O(1)，代价就是这个队列的空间会不断被浪费

那么就继续优化

`back = a[front+size]%arryLen`

即 `location(i)=location(front+i)%arrLen`

很明显，如果这样的话，我们就可以将数组看出一个环了。进站出站都是围着这个环来做，这也就不是普通的队列了，是循环队列了

可见图（这里我们对front的定义做了点改动，不再是删除的节点，而是该节点的前一个节点）

![图-2](/static//queue//queue-2.png)

这样的话，我们也可以通过 `front = back` 来判断队列为空

不过有个问题，就是大家想一下就会发现，如果我们给队列完全填满了，他也会是`front = back`

这是不行的，所以我们需要在加一条限制，队列的最大容量是 arryLen - 1 每次push前做一次检测，如果发现当前容量已经是最大容量了，需要进行一次扩容(arryLen = arryLen*2) 才可以进行push

明确了这些后，就可以开始编码了。

```go
type ArrayQueue struct {
	QueueFront int
	QueueBack  int
	Array      []interface{}
	QueueSize  int
}

func New() *ArrayQueue {
	array := make([]interface{}, 4)
	return &ArrayQueue{
		QueueFront: 0,
		QueueBack:  0,
		Array:      array,
		QueueSize:  0,
	}
}

func (q *ArrayQueue) Empty() bool {
	// return q.queueFront == q.queueBack
	return q.QueueSize == 0
}

func (q *ArrayQueue) Size() int {
	return q.QueueSize
}

func (q *ArrayQueue) Front() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	return q.Array[q.QueueFront+1], true
}

func (q *ArrayQueue) Back() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	return q.Array[q.QueueBack], true
}

func (q *ArrayQueue) Pop() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	front := (q.QueueFront + 1) % len(q.Array)
	val := q.Array[front]
	q.Array[front] = nil
	q.QueueFront = front
	q.QueueSize--
	return val, true
}

func (q *ArrayQueue) Push(val interface{}) bool {
	if q.Size() == len(q.Array)-1 {
		q.Array = append(q.Array, make([]interface{}, len(q.Array))...)
	}
	back := (q.QueueBack + 1) % len(q.Array)
	q.Array[back] = val
	q.QueueBack = back
	q.QueueSize++
	return true
}

```

### 链表实现

基于单链表实现同样要区分一下哪里是队列头，哪里是队列尾

首先我们需要两个指针，front与back 分别指向头与尾

如果是插入操作的话，无论哪一端是队尾都是很简单的

如果头节点是队尾：

```go
queue= &node{
	Val:val,
	Next:head,
}
```

如果链表结尾是队尾：

``` go
back.Next :=  &node{
Val:val,
Next:nil,
}
back = back.Next
```

可见他们都是O(1)操作，但是删除操作就不一样了。


假设是头节点为队尾，那么删除操作需要先遍历到队尾前一个节点，随后断开链。

而链尾是队尾的情况只需要让 `front = front.Next` 就可以轻松断开

所以我们定义这个链表实现的队列，头节点处为队首，尾节点处为队尾

```go
type LinkedQueue struct {
	// 指向链首
	QueueFront *chainNode
	QueueBack  *chainNode
	QueueSize  int
}

type chainNode struct {
	Val  interface{}
	Next *chainNode
}

func (q *LinkedQueue) Empty() bool {
	/*
		if q.QueueBack == q.QueueBack && q.QueueFront == nil {
			return true
		}
		return false
	*/
	return q.QueueSize == 0
}

func (q *LinkedQueue) Size() int {
	return q.QueueSize
}

func (q *LinkedQueue) Front() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	return q.QueueFront.Val, true
}

func (q *LinkedQueue) Back() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	return q.QueueBack.Val, true
}

func (q *LinkedQueue) Pop() (interface{}, bool) {
	if q.Empty() {
		return nil, false
	}
	val := q.QueueFront.Val
	q.QueueFront = q.QueueFront.Next
	q.QueueSize--
	return val, true
}

func (q *LinkedQueue) Push(val interface{}) bool {
	node := &chainNode{
		Val: val,
	}
	if q.Empty() {
		q.QueueFront = node
	} else {
		q.QueueBack.Next = node
	}
	q.QueueBack = node
	q.QueueSize++
	return true
}

```

## 应用

应用方面仍然使用leetcode题目作为演示

### 字符串中的第一个唯一字符

[leetcode](https://leetcode.cn/problems/first-unique-character-in-a-string/)

送一个长度为n的字符串（只有小写字母），要求返回第一个无重复字符串的索引。

这里解法是先写一个长度为26的数组。从a[0]到a[25]分别代表a-z
将它们的值置为n,代表还未检索到。

创建一个队列，之后开始检索字符串。每检索到一个字符串便将对应的a[x]的值置为该字符串的索引，并且将索引与字节的结构体送入队列。
如果发现已经有索引了，就开始清空队列，一直清到队首的结构体没有被检索。

一直到将字符串检索完，如果这是队列非空，则队首的值是唯一的，如果为空则说明无唯一字符，返回-1

代码实现：

``` go 
type pair struct {
	ch  byte
	pos int
}

func firstUniqChar(s string) int {
	n := len(s)
	alpbabet := make([]int, 26)
	for i, _ := range alpbabet {
		alpbabet[i] = n
	}
	q := []*pair{}
	for i := range s {
		b := s[i]
		index := b - 'a'
		if alpbabet[index] == n {
			q = append(q, &pair{
				ch:  b,
				pos: i,
			})
			alpbabet[index] = i
		} else {
			alpbabet[index] = n + 1
			for len(q) != 0 {
				index := q[0].ch - 'a'
				if alpbabet[index] == n+1 {
					q = q[1:]
				} else {
					break
				}
			}
		}
	}
	if len(q) > 0 {
		return q[0].pos
	}
	return -1 
}

```

## 总结

本文完成了对队列的定义与代码实现，并且就队列完成了一道leetcode题目

其实最开始想引用参考书籍里的例子，但多为应用题或之前章节题目的部分的改进，叙述过多，所以并未使用。

这次学习与博文撰写期间，周末公司外出团建，耽搁时间略多，虽然回来后还是有些晕车想吐，所幸还是写出了这篇文章，没有在计划开启的第二个星期就失败