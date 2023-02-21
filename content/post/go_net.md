---
title: "go CIDR的各种转换"
date: 2023-02-17T23:31:04+08:00
lastmod: 2023-02-17T23:31:04+08:00
draft: false
keywords: ["go"]
description: ""
tags: ["go"]
categories: ["go"]
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

IP/CIDR 转换为 ip与netmask

``` GO
type IpNetmask struct {
	IP      string
	Netmask string
}

func CIDRToIPNetmase(cidr string) (*IpNetmask, error) {
	_, ipNet, err := net.ParseCIDR(cidr)
	if err != nil {
		return nil, fmt.Errorf("parse CIDR failed: err= %v", err)
	}
	ones, _ := ipNet.Mask.Size()
	mask := net.CIDRMask(ones, 32)
	return &IpNetmask{
		IP:      ipNet.IP.String(),
		Netmask: net.IP(mask).String(),
	}, nil
}
```
上文中为了得到perfix 或者说 子网掩码我们做了多次操作，但是在 go 1.18之后，net/netip包提出了一个新的方法可以快速得到

```go 
	netip.ParsePrefix(t)
```
该方法会在返回一个address与int类型的perfix	

netmask 反向转换

```go
func netmaskToSub(mask string) (int, error) {
	masks := strings.Split(mask, ".")
	if len(masks) != 4 {
		return 0, fmt.Errorf("invalid mask, mask: %s", mask)
	}
	var subs []int
	for _, i := range masks {
		one, err := strconv.Atoi(i)
		if err != nil {
			return 0, fmt.Errorf("invalid mask, mask: %s", mask)
		}
		subs = append(subs, one)
	}
	ones, _ := net.IPv4Mask(byte(subs[0]), byte(subs[1]), byte(subs[2]), byte(subs[3])).Size()
	return ones, nil
}netmask, bits := net.IPv4Mask(255, 255, 255, 248).Size()
```

### CIDR 转换为 地址段

```go
func CIDRtoIPRange(address string) (string, error) {
	ip, ipnet, err := net.ParseCIDR(address)
	if err != nil {
		return "", fmt.Errorf("parse cidr failedL err= %v", err)
	}
	var ips []string
	for ip := ip.Mask(ipnet.Mask); ipnet.Contains(ip); inc(ip) {
		ips = append(ips, ip.String())
	}
	return fmt.Sprintf("%s-%s", ips[0], ips[len(ips)-1]), nil
}

func inc(ip net.IP) {
	for j := len(ip) - 1; j > 0; j-- {
		ip[j]++
		if ip[j] > 0 {
			break
		}
	}
}
```
### 判断某IP是否在CIDR内
这个没什么做的，前文通过`net.ParseCIDR`方法得出的 ipNet 有一个方法就是 contains  判断输入ip是否在其内，在的话返回true
```go
ipnet.Contains()
```

## 总结

本文介绍了 ipCIDR 形式转换为 ip ，网关，ip段以及判断一个ip是否在一个ipcidr内的方法。

其实最开始是打算从0开始实现的，结果在写的过程中发现官方库提供了很多方法，就直接使用了