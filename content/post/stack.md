---
title: "Stack"
date: 2022-10-30T12:33:35+08:00
lastmod: 2022-10-30T12:33:35+08:00
draft: false
keywords: ["go","data structure"]
description: "完成对栈的定义，实现与应用"
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

# 数据结构 栈

## 前言

久违的博客。保持学习，回顾以前曾经学过的数据结构。工作了一年多基本全是数组/哈希表解决一切，基本没想过其他实现，也没想过底层或者优化。

结果前几天做写一个地方，写了一个O(n^2)的代码，在思考能不能降到O(n)甚至O(1)的时候，突然发现自己对于这块的知识已经模糊了，于是重新开始学习数据结构与算法。

这可能会是一个系列，我希望我能做到一周一篇。

## 定义

栈，最常见的定义就是一个遵守先进后出，并且只在一端进行存取操作的线性表。

如一打碟子，我们总是会先拿最上面的，而最底下的那个往往是最先放到桌子上的。这便是一个形象的栈结构。

它的常见方法有：

``` go
type Stacks interface {
	// 判断栈是否为空
	Empty() bool
	// 输出栈大小
	Size() int
	// 返回栈顶元素，但不推出
	Top() (val interface{}, ok bool)
	// 推出栈顶元素
	Pop() (val interface{}, ok bool)
	// 向栈顶压入元素
	Push(val interface{}) (ok bool)
}
```
我不是狠喜欢给值定义为interface，不过为了表示普适性，还是这样写了。

## 实现

基于上面的定义，我们就可以很轻松地做写栈的实现了。我们用两个基础线性表：数组、链表 分别实现

个人愚见，所谓的用链表或者数组实现，其实只是将链表或者数组再次抽象

### 数组实现

```go

type ArrayStack struct {
	Array []interface{}
}

func (s *ArrayStack) Empty() bool {
	return len(s.Array) == 0
}

func (s *ArrayStack) Size() int {
	return len(s.Array)
}

func (s *ArrayStack) Top() (val interface{}, ok bool) {
	if s.Empty() {
		return nil, false
	}
	val = s.Array[s.Size()]
	return val, true
}

func (s *ArrayStack) Pop() (val interface{}, ok bool) {
	if s.Empty() {
		return nil, false
	}
	val = s.Array[s.Size()]
	s.Array = s.Array[:s.Size()-1]
	return val, true
}

func (s *ArrayStack) Push(val interface{}) (ok bool) {
	if s.Empty() {
		s.Array = []interface{}{}
	}
	s.Array[s.Size()] = val
	return true
}

```

### 链表实现

链表实现有一个问题需要思考，那就是栈顶选在头节点还是末尾

如果选在末尾的话，我们使用Top、Pop、Push 需要调用的链表方法对应 `Get（Size（）-1）`,`insert(Size(),val)`,`Remove(Size()-1)`
这样的话，时间复杂度是 O(n)

而如果选在头节点的话,就都是`Get（0）`,`insert(0,val)`,`Remove(0)`
时间复杂度为O（1）

因而选择用头节点做栈顶
``` go
type LinkedStack struct {
	ChainNode *chainNode
	StackSize int
}

type chainNode struct {
	Val  interface{}
	Next *chainNode
}

func (s *LinkedStack) Empty() bool {
	return s.StackSize == 0
}

func (s *LinkedStack) Size() int {
	return s.StackSize
}

func (s *LinkedStack) Top() (val interface{}, ok bool) {
	if s.Empty() {
		return nil, false
	}
	return s.ChainNode.Val, true
}

func (s *LinkedStack) Pop() (val interface{}, ok bool) {
	if s.Empty() {
		return nil, false
	}
	val = s.ChainNode.Val
	s.ChainNode = s.ChainNode.Next
	s.StackSize--
	return val, true
}

func (s *LinkedStack) Push(val interface{}) (ok bool) {
	s.ChainNode = &chainNode{
		Val:  val,
		Next: s.ChainNode,
	}
	s.StackSize++
	return true
}
```

## 应用

来到应用这一步了，使用leetcode题目作为演示

一道经典的栈应用题目： 括号匹配

### 括号匹配
(leetcode)[https://leetcode.cn/problems/valid-parentheses/]

简单题，题目要求就是给定一个字符串，仅由'()[]{}'组成，判断字符串能否闭合

思路简单，string进来后入栈

如 `[{()}]`

那么先入一半 `[{(`

后面的每一个元素必定与栈顶元素抵消

那如果是 `[](){}`呢

那就每入第一个元素，之后的元素可以与栈顶元素抵消

同时可以在一开始下先判断一次奇偶，为奇必错，直接返回

代码：

```go
func isValid(s string) bool {
	n := len(s)
	if n%2 == 1 {
		return false
	}
	// 使用Map快速配对
	pairs := map[byte]byte{
		')': '(',
		']': '[',
		'}': '{',
	}
	stack := []byte{}
	for i := 0; i < n; i++ {
		v, ok := pairs[s[i]]
		if ok {
			// 拿到了右括号，但是没有与其抵消的左括号在栈顶，说明无配对括号，错误
			if len(stack) == 0 || stack[len(stack)-1] != v {
				return false
			}
			// 抵消
			stack = stack[:len(stack)-1]
		} else {
			stack = append(stack, s[i])
		}
	}
	return len(stack) == 0
}

```


## 结语

 本文使用go语言实现了栈的两种实现方式，对应的代码与单元测试可以详见 
 [个人github](https://github.com/zzy2210/data_structures)

日后会尽力保证每周更新博客，后续数据结构计划为：
- 队列
- 跳表与散列
- 二叉树与其他树
- 优先级队列
- 竞赛树
- 搜索树
- 平衡搜索树
- 图