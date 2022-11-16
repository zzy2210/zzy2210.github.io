---
title: "数据结构-二叉树与其他树"
date: 2022-11-14T20:46:54+08:00
lastmod: 2022-11-14T20:46:54+08:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: ""

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