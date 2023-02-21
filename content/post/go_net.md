---
title: "go CIDR的各种转换"
date: 2023-02-17T23:31:04+08:00
lastmod: 2023-02-17T23:31:04+08:00
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

## 前言

最近项目开发和网卡等内容做了一些交道。遇到了一些常见的操作。比如 IP/CIDR 与 ip和netmask的相互转换，IP/CIDR 转换为IP段形式以及转换回来，判断某IP是否在一个IP/CIDR内

干脆就统一写一下，也做记录，若是日后需要要好回忆

##  基础思路

CIDR，即无类域间路由（Classless Inter-Domain Routing）可以将路由集中起来，在路由表中更灵活地定义地址，CIDR 标记使用一个斜线分隔符，后面跟一个十进制数值表示地址中网络部分所占的位数。

即 如 10.10.1.1/16 这类形式的地址。16是子网掩码

如果要将/16 转换为netmask ip形式

则说明二进制形式下的ip，前16个值为1，后面为0，也就是

`1111 1111,1111 1111,0000 0000,0000 0000`

也就是 

`255，255，0，0`

这样，它对应的IP段就是

10.10.0.0 - 10.10.255.255

根据这种方式，判断一个ip是否在一个IP/CIDR 内，就可以根据子网掩码来计算

如 10.10.7.6 与 10.10.1.1/16

我们只需判断 两者的前16位是否相同，若相同，则在其内
 
`10.10.1.1 = 0000 1010.0000 1010.0000 0001.0000 0001`

`10.10.7.6 = 0000 1010.0000 1010.0000 0101.0000 0100`

前16位都是 0000 1010.0000 1010 故可以判断 10.10.7.6 在 10.10.1.1/16 内

## 代码实现

基础思路有了，接下来就可以写代码实现了。有的方式Go的net包内是提供的 // 写完校对的时候改一下 因为我现在还不确定我要写啥

###  IP/CIDR 与 IP netmask 的转换


``` GO
type IpNetmask struct {
	IP      string
	Netmask string
}

 strings.Join(s, ".")func CIDRToIPNetmase(cidr string) (*IpNetmask, error) {
	_, ipNet, err := net.ParseCIDR(cidr)
	if err != nil {
		return nil, fmt.Errorf("parse CIDR failed: err= %v", err)
	}
	// 直接使用 ipNet.Mask.String() 会输出一个十六进制

	ones, _ := ipNet.Mask.Size()
	mask := net.CIDRMask(ones,32)
	return &IpNetmask{
		IP:      ipNet.IP.String(),
		Netmask: net.IP(mask).String,
	}, nil
}
```

```go
netmask, bits := net.IPv4Mask(255, 255, 255, 248).Size()
```

### CIDR 转换为 地址段

```go
 ip,ipnet,_ := net.ParseCIDR(address)
var ips []string
for ip:= ip.Mask(ipnet.Mask);ipnet.Contains(ip);inc(ip){
ips = append(ips,ip.string)
}
return fmt.Sprintf("%s-%s",ips[0],ips[len(ips)-1])


func inc(ip net.IP){
	for j:= len(ip) -1;j>0;j-- {
		ip[i]++
		if ip[j]>0 {
			break
		}
	}
}
```

### 判断某IP是否在CIDR内

```go
ipnet.Contains()
```