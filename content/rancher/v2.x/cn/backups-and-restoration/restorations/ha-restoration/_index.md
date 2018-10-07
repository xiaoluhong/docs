---
title: 2 - HA集群恢复
weight: 2
---

{{% accordion id="1" label="一、 恢复准备" %}}

1、需要在进行操作的主机上提前[安装RKE]({{< baseurl >}}/rke/v0.1.x/en/installation/)([RKE下载]({{< baseurl >}}/rancher/v2.x/cn/installation/download/#rancher-rke))和[kubectl]({{< baseurl >}}/rancher/v2.x/cn/installation/kubectl/)。

2、在开始还原之前，请确保已停止旧群集节点上的所有kubernetes服务。

{{% /accordion %}}
{{% accordion id="2" label="二、添加新ETCD节点并复制最新快照" %}}

假设集群中一个或者多个etcd节点发生故障，或者整个集群数据丢失，则需要进行etcd集群恢复。

**添加新ETCD节点并复制最新快照:**

1、添加你选择的`新ETCD节点`，可以是物理主机、本地虚拟机、云主机等；

2、通过远程终端登录新主机；

3、创建快照目录:

```bash
mkdir -p /opt/rke/etcd-snapshots/
```

4、复制备份的最新快照到`新etcd节点`的`/opt/rke/etcd-snapshots/`目录下:

```bash
cp $PWD/<SNAPSHOT.db> /opt/rke/etcd-snapshots/<SNAPSHOT.db>
```

{{% /accordion %}}
{{% accordion id="3" label="三、配置RKE配置文件" %}}

制作原始`rancher-cluster.yml`文件的副本

`cp rancher-cluster.yml rancher-cluster-restore.yml`

对副本配置文件进行以下修改:

- 删除或注释掉整个`addons:`部分。
- 将`nodes:`部分更改为`新etcd节点`,注释掉其他节点。

`例: rancher-cluster-restore.yml`

```bash
nodes:
- address: 52.15.238.179     # 新etcd节点
  user: ubuntu
  role: [ etcd, controlplane, worker ]
# - address: 52.15.23.24
#   user: ubuntu
#   role: [ etcd, controlplane, worker ]
# - address: 52.15.238.133
#   user: ubuntu
#   role: [ etcd, controlplane, worker ]

# addons: |-
#   ---
#   kind: Namespace
#   apiVersion: v1
#   metadata:
#     name: cattle-system
#   ---
...
```

{{% /accordion %}}
{{% accordion id="4" label="四、恢复ETCD数据" %}}

1、打开`shell终端`，切换到RKE二进制文件所在的目录，并且上一步修改的`rancher-cluster-restore.yml`文件也需要放在同一路径下。

2、根据系统类型，选择运行以下命令还原`etcd`数据：

```bash
# MacOS
./rke_darwin-amd64 etcd snapshot-restore --name <snapshot>.db --config ./rancher-cluster-restore.yml

# Linux
./rke_linux-amd64 etcd snapshot-restore --name <snapshot>.db --config ./rancher-cluster-restore.yml
```

>RKE将在`新ETCD节点`上创建包含已还原数据的`ETCD`容器。此容器将保持运行状态，但无法完成etcd初始化并。

{{% /accordion %}}
{{% accordion id="5" label="五、恢复集群" %}}

使用RKE并在`新ETCD节点`单节点上启动集群。根据系统类型，选择运行以下命令更新集群：

```bash
# MacOS
./rke_darwin-amd64 up --config ./rancher-cluster-restore.yml
# Linux
./rke_linux-amd64 up --config ./rancher-cluster-restore.yml
```

1、测试群集

RKE运行完成后会创建`kubectl`的配置文件`kube_config_rancher-cluster-restore.yml`，可通过这个配置文件查询K8S集群节点状态：

```bash
kubectl  --kubeconfig=kube_config_rancher-cluster-restore.yml  get nodes

NAME            STATUS    ROLES                      AGE       VERSION
52.15.238.179   Ready     controlplane,etcd,worker    1m       v1.10.5
18.217.82.189   NotReady  controlplane,etcd,worker   16d       v1.10.5
18.222.22.56    NotReady  controlplane,etcd,worker   16d       v1.10.5
18.191.222.99   NotReady  controlplane,etcd,worker   16d       v1.10.5
```

2、清理旧节点

通过kubectl从群集中删除旧节点

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml  delete node 18.217.82.189 18.222.22.56 18.191.222.99
```

3、重新启动`新ETCD节点`

4、`新ETCD节点`运行起来后，检查`Kubernetes Pods`的状态

```bash
kubectl --kubeconfig=kube_config_rancher-cluster-restore.yml  get pods --all-namespaces

NAMESPACE       NAME                                    READY     STATUS    RESTARTS   AGE
cattle-system   cattle-cluster-agent-766585f6b-kj88m    0/1       Error     6          4m
cattle-system   cattle-node-agent-wvhqm                 0/1       Error     8          8m
cattle-system   rancher-78947c8548-jzlsr                0/1       Running   1          4m
ingress-nginx   default-http-backend-797c5bc547-f5ztd   1/1       Running   1          4m
ingress-nginx   nginx-ingress-controller-ljvkf          1/1       Running   1          8m
kube-system     canal-4pf9v                             3/3       Running   3          8m
kube-system     cert-manager-6b47fc5fc-jnrl5            1/1       Running   1          4m
kube-system     kube-dns-7588d5b5f5-kgskt               3/3       Running   3          4m
kube-system     kube-dns-autoscaler-5db9bbb766-s698d    1/1       Running   1          4m
kube-system     metrics-server-97bc649d5-6w7zc          1/1       Running   1          4m
kube-system     tiller-deploy-56c4cf647b-j4whh          1/1       Running   1          4m
```

>直到Rancher服务器启动并且DNS/负载均衡器指向新群集，`cattle-cluster-agent和cattle-node-agent`pods将处于`Error或者CrashLoopBackOff`状态。

5、添加其他节点

编辑RKE配置文件`rancher-cluster-restore.yml`,添加或者取消其他节点的注释。

`例：rancher-cluster-restore.yml`

```bash
nodes:
- address: 52.15.238.179     # 新ETCD节点
  user: ubuntu
  role: [ etcd, controlplane, worker ]
- address: 52.15.23.24
  user: ubuntu
  role: [ etcd, controlplane, worker ]
- address: 52.15.238.133
  user: ubuntu
  role: [ etcd, controlplane, worker ]

# addons: |-
#   ---
#   kind: Namespace
...
```

6、更新集群

根据系统类型，选择运行以下命令更新集群：

```bash
# MacOS
./rke_darwin-amd64 up --config ./rancher-cluster-restore.yml
# Linux
./rke_linux-amd64 up --config ./rancher-cluster-restore.yml
```

{{% /accordion %}}