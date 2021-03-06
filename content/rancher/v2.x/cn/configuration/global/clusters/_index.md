---
title: 1 - 集群
weight: 1
---

全局中的集群，它是Rancher中的概念。这里归纳了所有创建的K8S集群，您可以删除、新建、搜索集群。Rancher支持通过UI创建Kubernetes集群，而不是使用配置文件，从而简化了Kubernetes集群的创建。

### 1、节点角色

Kubernetes集群包含3种角色类型的节点：etcd节点，control plane节点和worker节点。创建Kubernetes集群，每种角色至少需要一个，`多个角色可以运行在一台节点上`。

- etcd Nodes

etcd节点用于运行etcd数据库。 etcd是一个键值存储，用作Kubernetes对所有集群数据的后备存储。即使您可以在单个节点上运行etcd实例，也需要3个、5个或7个节点来实现冗余,具体参考[ETCD集群容错表]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/basic-environment-configuration/#7、ETCD集群容错表)。

- Control Plane Nodes

control plane节点用于运行Kubernetes API服务、scheduler和controller manager。control plane节点是无状态的，因为所有集群数据都存储在etcd节点上。您可以在1个节点上运行control plane，但需要2个或更多节点实现冗余。您还可以在etcd节点上运行控制平面。

- Worker Nodes

Worker节点用于运行kubelet和工作负载。它还在需要时运行存储和网络驱动程序和ingress controllers。您可以根据工作负载需要创建尽可能多的Worker节点。

## 2、Kubernetes集群创建方式

Rancher包含多种选项:

- 可以通过`自定义`方式，使用独立的Linux节点来创建和运行Kubernetes集群；
- 可以通过对接Kubernetes托管服务商来创建Kubernetes集群；
- 可以通过`host driver`对接云服务商创建主机，然后自动创建Kubernetes集群；
- 可以导入纳管现有的或其他工具创建的Kubernetes集群；