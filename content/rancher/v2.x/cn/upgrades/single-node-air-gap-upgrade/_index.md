---
title: 2 - 单节点离线升级
weight: 2
---

要离线升级Rancher Server，需要先同步最新的Rancher镜像到私有镜像仓库中，然后再进行下面的升级操作。

## 一、先决条件

从v2.0.7开始，Rancher引入了`system`项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到`v2.0.7+`前，请检查环境中有没有创建`system`项目，如果有则删除。`并检查确认所有系统命名空间未分配到任何项目下，如果有则移到出去，以防止集群网络问题。`

## 二、离线升级Rancher Server

1. （可选）如果你无法找到之前运行rancher server的`docker run`命令，可通过以下命令找会执行命令。

    ```bash
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
         registry.cn-shanghai.aliyuncs.com/rancher/run-config <rancher_container_name>
    ```

1. 按照离线安装方法[准备离线镜像]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/prepare-private-reg/)。

1. 按照[单节点升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/single-node-upgrade/)的方法，进行Rancher Server升级。

    >**注意:** 在执行单节点升级时，`docker run`参数中的镜像名，需要添加私有仓库地址。
    >
    > 例如: `<registry.yourdomain.com:port>/rancher/rancher:stable (或者rancher/rancher:latest)`

>**升级后出现网络问题？**
>
> 请查阅[Restoring Cluster Networking]({{< baseurl >}}/rancher/v2.x/en/upgrades/upgrades/namespace-migration/#restoring-cluster-networking)。
