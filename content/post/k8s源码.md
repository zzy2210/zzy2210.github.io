# k8s 源码

## k8s 组件

### kube apiserver

apiserver 负责向外提供资源信息与操控的能力，并且是唯一与etcd集群交互的组件

### kube controller manager

管理器，负责node pod service endpoint等资源的管理。

比如node挂了，将上面的pod 迁移到其他node 就是由它负责

他提供 statefulset控制器 namespace控制器等，控制器通过与apiserver 的交互，实时监测当前集群的状况，并且根据情况来做行动

controller manager 是分布式高可用的，它是多实例运行的，基于etcd提供的分布式锁功能实现领导者选举

抢占到锁的实例变成领导节点，运行组建的主逻辑 ，其他节点变成 candidate节点(候选节点) 被阻塞到 leader 节点推出，重新开始选举

### kube scheduler

调度器 负责 将 pod 调度到 合适的 node 上去，每次只调度一个 pod。

### kubelet

运行在节点上，用于 接收 运行 上报apiserver 下发的任务相关信息，负责 该节点上 pod 资源对象的管理

它提供了三种接口
- CRI
- CNI
- CSI