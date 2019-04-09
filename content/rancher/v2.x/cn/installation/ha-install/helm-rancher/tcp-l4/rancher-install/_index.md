---
title: 4 - Helm安装Rancher
weight: 4
---

>**注意：** 对于离线环境，请参考[离线安装Rancher]({{< baseurl >}}/rancher/v2.x/cn/installation/air-gap-installation/)。

## 一、添加Chart仓库地址

使用`helm repo add`命令添加Rancher chart仓库地址,访问[Rancher tag和Chart版本]({{< baseurl >}}/rancher/v2.x/cn/installation/server-tags/)

替换`<CHART_REPO>`为您要使用的Helm仓库分支(即latest或stable）。

```bash
helm repo add rancher-stable \
https://releases.rancher.com/server-charts/stable
```

## 二、使用自签名或者权威SSL证书安装Rancher server

Rancher server设计默认需要开启SSL/TLS配置来保证安全，将ssl证书以`Kubernetes Secret`卷的形式传递给`rancher server或Ingress Controller`。首先创建证书密文，以便`Rancher和Ingress Controller`可以使用。

- 使用权威CA机构颁发的证书

1. 将`服务证书`和`CA中间证书链`合并到`tls.crt`,将`私钥`复制到或者重命名为`tls.key`；

1. 使用`kubectl`创建`tls`类型的`secrets`；

    >**注意:** `证书、私钥`名称必须是`tls.crt、tls.key`。

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml

    kubectl --kubeconfig=$KUBECONFIG create namespace cattle-system
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system \
    create secret tls tls-rancher-ingress \
    --cert=./tls.crt \
    --key=./tls.key
    ```

1. 安装rancher server

    >修改`hostname`

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml
    helm --kubeconfig=$KUBECONFIG install rancher-stable/rancher \
      --name rancher \
      --namespace cattle-system \
      --set hostname=<你自己的域名> \
      --set ingress.tls.source=secret
    ```

    >**注意:** 1.创建证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

- 使用自签名ssl证书

1. 如果没有自签名ssl证书，可以参考[自签名ssl证书]({{< baseurl >}}/rancher/v2.x/cn/install-prepare/self-signed-ssl/#四-生成自签名证书)，一键生成ssl证书；

1. 一键生成ssl自签名证书脚本将自动生成`tls.crt、tls.key、cacerts.pem`三个文件。如果使用你自己的自签名ssl证书，则需要将`服务证书`和`CA中间证书链`合并到`tls.crt`文件,将`私钥`复制到或者重命名为`tls.key`文件，将`CA证书`复制到或者重命名为`cacerts.pem`文件;

1. 使用`kubectl`在命名空间`cattle-system`中创建`tls-ca`和`tls-rancher-ingress`两个`secret`;

    >**注意:** `证书、私钥、ca`名称必须是`tls.crt、tls.key、cacerts.pem`。

    ```bash
    # 指定配置文件
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml
    # 创建命名空间
    kubectl --kubeconfig=$KUBECONFIG create namespace cattle-system
    # 服务证书和私钥密文
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system create secret \
    tls tls-rancher-ingress \
    --cert=./tls.crt \
    --key=./tls.key
    # ca证书密文
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system create secret \
    generic tls-ca \
    --from-file=cacerts.pem
    ```

1. 安装rancher server

    >修改`hostname`

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml
    helm --kubeconfig=$KUBECONFIG install rancher-stable/rancher \
      --name rancher \
      --namespace cattle-system \
      --set hostname=<你自己的域名> \
      --set ingress.tls.source=secret \
      --set privateCA=true
    ```

    >**注意:** 1.证书对应的`域名`需要与`hostname`选项匹配，否则`ingress`将无法代理访问Rancher。

### 4、高级配置

Rancher chart有许多配置选项,可用于自定义安装以适合你的特定环境，点击查看[Rancher高级设置](../advanced-settings)

### 5、保存配置参数

确保保存了所有的配置参数，Rancher下一次升级时，helm需要使用相同的配置参数来运行新版本Rancher。

### 6、(可选)为Agent Pod添加主机别名(/etc/hosts)

如果你没有内部DNS服务器而是通过添加`/etc/hosts`主机别名的方式指定的Rancher server域名，那么不管通过哪种方式(自定义、导入、Host驱动等)创建K8S集群，K8S集群运行起来之后，因为`cattle-cluster-agent Pod`和`cattle-node-agent`无法通过DNS记录找到`Rancher server`,最终导致无法通信。

- 解决方法

可以通过给`cattle-cluster-agent Pod`和`cattle-node-agent`添加主机别名(/etc/hosts)，让其可以正常通信`(前提是IP地址可以互通)`。

1. cattle-cluster-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system \
    patch deployments cattle-cluster-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "demo.cnrancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

2. cattle-node-agent pod

    ```bash
    export KUBECONFIG=xxx/xxx/xx.kubeconfig.yaml #指定kubectl配置文件
    kubectl --kubeconfig=$KUBECONFIG -n cattle-system \
    patch  daemonsets cattle-node-agent --patch '{
        "spec": {
            "template": {
                "spec": {
                    "hostAliases": [
                        {
                            "hostnames":
                            [
                                "xxx.rancher.com"
                            ],
                                "ip": "192.168.1.100"
                        }
                    ]
                }
            }
        }
    }'
    ```

    > **注意**
    >1、替换其中的域名和IP \
    >2、别忘记json中的引号。

## 三、故障排除

[故障排除]({{< baseurl >}}/rancher/v2.x/cn/faq/troubleshooting-helm/)