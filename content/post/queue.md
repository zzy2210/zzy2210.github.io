---
title: "数据结构-队列"
date: 2022-10-31T21:58:57+08:00
lastmod: 2022-10-31T21:58:57+08:00
draft: true
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

```
