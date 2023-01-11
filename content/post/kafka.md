---
title: "Kafka"
date: 2023-01-11T10:03:31+08:00
draft: true
---

kafka启动：
1. 启动zooker: `zookeeper-server-start.sh config/zookeeper.properties`
2. 启动kafka : `kafka-server-start.sh config/server.properties`
3. 创建topis `kafka-topic.sh --zookeeper 127.0.0.1:2181 --create cloudbg --partition 1 --replication-factor 1`



2. 查看 kafka topis : `kafka-consoloe-consumer.sh -- bootstrap-server 127.0.0.1:9092 --topic cloudbg --from-beginning`