---
title: "kubernetes-环境搭建"
date: 2023-06-14T17:35:08+08:00
lastmod: 2023-06-14T17:35:08+08:00 
draft: false
keywords: ["kubernetes","云原生"]
description: "完成k8s的单节点环境搭建"
tags: ["kubernetes"]
keywords: ["kubernetes","云原生"]
author: "y1nhui"

# Uncomment to pin article to front page
# weight: 1
# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
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

组件： kubeadm+docker
操作系统： ubuntu

## 禁用交换分区

为了保证 kubelet 的正常工作，需要禁用交换分区

`vi /etc/fstab  # 将swap一行注释 `

## docker 环境安装

[官方ubuntu安装教程](https://docs.docker.com/engine/install/ubuntu/)

直接对着官方教程安装即可。

首先载入证书等

``` shell
 $ sudo apt-get update
 $ sudo apt-get install ca-certificates curl gnupg
 $ sudo install -m 0755 -d /etc/apt/keyrings
 $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 $ sudo chmod a+r /etc/apt/keyrings/docker.gpg
 $ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

之后安装docker， 笔者这里采用了特定版本的docker

``` shell
$ VERSION_STRING=5:19.03.15~3-0~ubuntu-focal
$ apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

## kubeadm 等工具安装

安装 1.18.17 版本的 kubeadm kubectl kubelet

[官方安装教程](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

这里同样安装官方文档来

``` shell
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
$curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
$echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
```

之后安装特定版本

``` shell
$ apt install -y kubelet=1.18.17-00 kubeadm=1.18.17-00 kubectl=1.18.17-00
```

之后标记工具，防止更新

## 创建集群

使用 yaml 文件

``` yaml

apiVersion: kubeadm.k8s.io/v1beta1
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.5.9
---
apiVersion: kubeadm.k8s.io/v1beta1
apiServer:
  timeoutForControlPlane: 4m0s
  certSANs:
    - 192.168.5.9
  extraVolumes:
    - name: localtime
      hostPath: /usr/share/zoneinfo/Asia/Shanghai
      mountPath: /etc/localtime
      pathType: File
scheduler:
  extraVolumes:
    - name: localtime
      hostPath: /usr/share/zoneinfo/Asia/Shanghai
      mountPath: /etc/localtime
      pathType: File
controllerManager:
  extraVolumes:
    - name: localtime
      hostPath: /usr/share/zoneinfo/Asia/Shanghai
      mountPath: /etc/localtime
      pathType: File
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: ""
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
    ServerCertSANs:
      - 192.168.5.9
    peerCertSANs:
      - 192.168.5.9
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.18.17
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
   
```
`kubeadm config images pull --config=kubeadm.yaml`

`kubeadm init --config=kubeadm.yaml`

安装成功后，会有语句提醒执行一下命令
``` shell 
  $ mkdir -p $HOME/.kube
  $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
之后执行 ` kubectl get pod -n kube-system ` 就可以看到有数据

但是会有两个 `coredns` 处于pending 状态，这是因为还需要网络组建支持

## 安装 calico

`wget  https://docs.projectcalico.org/v3.8/manifests/calico.yaml`

接着修改 yaml 文件， 在 -name: IP 数据下增加新的内容：
``` yaml
           # Auto-detect the BGP IP address.
            - name: IP
              value: "autodetect"
            # 增加内容，ip为master 地址
            - name: IP_AUTODETECTION_METHOD
              value: "can-reach=192.168.5.9"
```

`kubectl apply -f calico.yaml`

至此 安装完成

##  总结

本文简单叙述了通过 kubeadm 安装单节点 k8s 的方式。
单节点场景过于简单，后续有多余机器后，笔者将会输出一篇三节点k8s集群安装方法