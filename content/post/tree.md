---
title: "数据结构-二叉树与其他树"
date: 2022-11-14T20:46:54+08:00
lastmod: 2022-11-14T20:46:54+08:00
draft: false
keywords: ["go","data structure"]
description: "完成对二叉树的遍历讲解与实现"
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
# 数据结构-二叉树与树

## 前言

本文主要内容为二叉树，树只是一概而过。

其实在去年的时候有写过一篇二叉树的博客，当时是以leetcode题目为主，不过最后因为没有做完目标题目也没有把文章发出来。

现在相关知识点已经忘了很多了，文章草稿也丢失不见了，所以也是从头开始吧

## 定义

### 引入树结构的原因

前面已经有了线性数据结构与表结构，但是这些结构都不能很好地表达层级关系，所以引入了新的数据结构，树

### 树

一个非空的有限元素集合便是树（部分资料中并不强调树是非空的）

其中一个元素是他的根，其余元素组陈它的子树

#### 高度/深度

一棵树的层级个数便是它的高度与深度

#### 元素的度 degree of an element

指一个元素的孩子个数，叶子节点的度为0

#### 树的度 degree of a tree

指一颗树的所有元素的度的最大值

### 二叉树

每个元素只有两个子树，并且有序（区分左右子树）的树就是二叉树

## 实现

因为二叉树的插入可能插入的地方并不是固定的，无法做到一个指令完成插入。而二叉树的主要知识点就是四种遍历。

所以这里主要谈及二叉树的四种遍历，所以也不给出接口了。



后续在搜索树等文章可能会提出接口。

```go

type BinaryTree struct {
	size int
 	node *Node
}

type Node struct {
	val interface{}
	left *Node
	right *Node
}

```

#### 前序遍历

前序遍历，便是先访问一个节点，再访问它的左子树，随后访问它的右子树

比如一棵树是
```
                 1
        2               3
4           5       6          7  
```

那么它的前序遍历的结果就是 1 2 4 5 3 6 7

```go
func (t *BinaryTree) PreOrder() {
	preOrder(t.Node)
}

func preOrder(node *Node) {
	if node != nil {
		fmt.Println(node.Val)
		preOrder(node.Left)
		preOrder(node.Right)
	}
}
```

#### 中序遍历

中序遍历就是先访问节点的左子树，然后访问它本身，再访问右子树

依旧是上面的树

```
                 1
        2               3
4           5       6          7  
```
它的结果是 4 2 5 1 6 3 7

```go
func (t *BinaryTree) InOrder() {
	inOrder(t.Node)
}

func inOrder(node *Node) {
	if node != nil {
		inOrder(node.Left)
		fmt.Println(node)
		inOrder(node.Right)
	}
}
```

#### 后序遍历

后续遍历则是先访问左右子树，最后访问它本身

```
                 1
        2               3
4           5       6          7  
```

它的结果是 4 5 2 6 7 3 1

```go
func (t *BinaryTree) PostOrder() {
	postOrder(t.Node)
}

func postOrder(node *Node) {
	if node != nil {
		postOrder(node.Left)
		postOrder(node.Right)
		fmt.Println(node.Val)
	}
}

```

#### 层次遍历

层次遍历就与前面三种完全不同了，前面三种看着就很简单，递归就完事了，层次遍历则不同

如果大家写过leetcode相关的题目的话就会注意到，中序、后序遍历是简单题，而层次遍历是中等

层次遍历便是按层访问，在同一层中按从左往右的顺序访问

如上面的

```
                 1
        2               3
4           5       6          7  
```
它的输出结果就是 1 2 3 4 5 6 7

这里的思路是创建一个队列，一颗树非空的情况下，先加入它自己与他的左右节点。最后加入它左右节点的左右节点，如此遍历

```go
func (t *BinaryTree) LevelOrder() {
	queue := queue.LinkedQueue{}
	node := t.Node

	for node != nil {
		fmt.Println(node.Val)
		if node.Left != nil {
			queue.Push(node.Left)
		}
		if node.Right != nil {
			queue.Push(node.Right)
		}
		queueNode, _ := queue.Pop()
		node = queueNode.(*Node)
	}
}
```

## 结语

这篇文章，咕了一两个星期，有点抱歉。

看到文章的变化应该就能感受到我的纠结，我一开始想要做到像之前的文章一样写出一个完整的实现，但是想不到该如何抽象。最终还是只写了四种遍历。

应用方面一时半会也想不到应该用什么。层次遍历甚至算是leetcode里面的中等题目。。。

其次就是，中途又被朋友拉去打游戏了，有点颓废。

现在收拾收拾，继续前进。