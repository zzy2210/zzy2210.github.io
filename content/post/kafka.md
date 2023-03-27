---
title: "kafak 集群搭建"
date: 2023-01-11T10:03:31+08:00
draft: false
keywords: ["kafka"]
description: "完成基于内置zookeeper的kafka集群构建"
tags: ["kafka"]
categories: ["kafka"]
author: "y1nhui"
---

## 背景

基于 kafka_2.13-3.4.0 版本 编写

## 环境搭建

下载 kafka 文件

`wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz`

安装java

本人环境为ubuntu：
`apt update`

`apt install default-jdk`

### kafka 组件介绍

刚解压完文件，看到其内的数据，新手应该是很迷茫的。
比如bin目录下有

![ls bin](/static/kafka/kafka-1.png)

我初看的时候是很茫然的。为什么那么多二进制？没有类似 nginx那样的，sbin下面直接 nginx 启动的吗？

介绍这里介绍一下kafka的情况，就可以理解了

kafka 是 天然支持集群的。也可以说哪怕是个单机部署的kafka也其实是集群模式下的单机部署。

而kafka的集群是依赖于 zookeeper 做协调服务。 一个 zookeeper 管理一个kafka集群。

如果查阅资料的话会发现很多国内构建高可用的kafka集群的文章都是额外下载一个zookeeper，而kafka的官方快速开始则是启用自己的bin下的zookeeper

是的，为了方便用户使用，kafka在自己的包中内置了一个zookeeper。如果只是单纯的学习测试等是可以直接应用该zookeeper的，而若是为了项目使用，则一般建议去zookeeper的官网下载稳定版本。

我们这里使用kafka内置的zookeeper即可

ps: kafka-2.8.0 之后移除了对 zookeeper 的完全依赖，也可以通过自带的 KRaft 来做集群管理

## zookeeper 配置与启动

修改 config下的 `zookeeper.properties` 文件

``` properties
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
# maxClientCnxns=0
# Disable the adminserver by default to avoid port conflicts.
# Set the port to something non-conflicting if choosing to enable this
admin.enableServer=false
# admin.serverPort=8080

#为zk的基本时间单元，毫秒
tickTime=2000
#Leader-Follower初始通信时限 tickTime*10
initLimit=10
#Leader-Follower同步通信时限 tickTime*5
syncLimit=5

server.1=10.0.1.102:2888:3888
server.2=10.0.1.178:2888:3888
server.3=10.0.10.137:2888:3888
```

之后需要在对应的 server 下 写 myid文件，内容为编号。

如 在10.0.1.102机器：`echo 1 > /data/zookeeper/myid`

之后将文件夹直接通过`scp -r`的方式发送到其他两台机器上去

## kafka 配置

修改 `server.properties`文件

需要改的几个基础点方便为:

```properties
broker.id=0
listeners=PLAINTEXT://10.0.1.102:9092
advertised.listeners=PLAINTEXT://10.0.1.102:9092
# A comma separated list of directories under which to store log files
log.dirs=/datakafka
zookeeper.connect=10.0.1.102:2181,10.0.1.178:2181,10.0.10.137:2181
```

他们分别是:

- brokerID
- kafka监听地址 (机器自己ip)
- kafka数据存储目录
- zookeeper集群连接地址

## 启动

首先启动zookeeper。

先启动 主节点，即 myid=1 的机器，之后启动其他两个

`nohup bin/zookeeper-server-start.sh config/zookeeper.properties > /data/logs/zookeeper/zookeeper.log 2>1 &`

主节点防火墙记得放行`3888`端口

`ufw allow 3888/tcp`

kafka启动：

类似的，直接运行 `bin/kafka-server-start.sh -daemon config/server.properties`

接着我们创界一个测试用的topic

 `./bin/kafka-topics.sh --bootstrap-server 10.0.1.102:9092 --create --partitions 3 --replication-factor 3 --topic study`

老版本的创建命令是 --zookeeper ip:2181 而不是bootstrap-server 不过笔者使用的这个版本，已经没有这个命令了

## 生产与消费

 接下来对集群做生产与消费来测试。

 因为 使用 zookeeper 维护集群信息。所以对任意的 broker 做读写操作都是不受影响的。

Kafka 的写入操作会自动将消息的副本分布到集群中的其他 broker 节点上，以提高数据的可靠性和冗余性。因此，无论你向哪个 broker 写入数据，这些数据都会被复制到其他 broker 节点上。
同时读取操作，如果调用的broker并非对应分区备份的leader节点，则会通过zookeeper进行转发。

这样很明显会有网络延迟与性能损耗，因而实际生产环境，为了最大程度地提高读取效率和性能，建议使用专业的 Kafka 客户端库来编写自己的消费者程序，并根据实际情况合理选择所需的 broker 和分区来消费数据。

不过我们这里做测试，这是验证这种读写能力。对于分区算法等，后续会再开一篇文章

在 broker 0 上 启用写

`bin/kafka-console-producer.sh --topic study --bootstrap-server 10.0.1.102:9092`

在 1，2 上启用读

`bin/kafka-console-consumer.sh --topic study --from-beginning --bootstrap-server 10.0.1.178:9092`

我们 与 broker 0 的写入，便可以在 1，2 上看到。
