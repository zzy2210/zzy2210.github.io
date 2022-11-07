---
title: "kubernetes-环境搭建"
date: 2022-11-07T17:45:08+08:00
lastmod: 2022-11-07T17:45:08+08:00 
draft: true
keywords: ["kubernetes"，"云原生"]
description: "完成k8s的集群环境搭建"
tags: ["kubernetes"]
keywords: ["kubernetes"，"云原生"]
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

# kubernetes 环境搭建

## 前言

尝试开学学习云原生与k8s，一开始尝试先看看官方文档中对各个名次的解析，但是感觉个人对架构等并不是很清晰，于是干脆直接上手搭建一个集群测试环境，边搭建边学习

## docker 环境按照

参考官方文档，使用minikube搭建k8s环境，而minikube的搭建环境有以下几点要求：

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

对于最后的容器方面，我们使用docker 