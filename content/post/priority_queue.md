---
title: "数据结构-优先级队列"
date: 2022-11-30T22:55:11+08:00
lastmod: 2022-11-30T22:55:11+08:00
draft: true
keywords: ["go","data structure"]
description: "完成对优先级队列的定义，实现与应用"
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

# 数据结构-优先级队列

## 前言

本文主要介绍了优先级队列的线性表、堆与左高树的实现方式

当然，也会讲解堆、左高树的相关概念等

## 定义

零或多个元素的集合，每个元素都有一个优先权/值。

### 最小优先级队列

查找、删除都是对优先级最小的元素进行操作

### 最大优先级队列

查找、删除都是对优先级最大的元素进行操作
