---
title: 1 - 单节点升级
weight: 1
---

## 一、先决条件

从v2.0.7开始，Rancher引入了`system`项目，该项目是自动创建的，用于存储Kubernetes需要运行的重要命名空间。在升级到`v2.0.7+`前，请检查环境中有没有创建`system`项目，如果有则删除。`并检查确认所有系统命名空间未分配到任何项目下，如果有则移到出去，以防止集群网络问题。`

## 二、升级步骤

> **注意:** 打开Rancher web并记下浏览器左下方显示的版本号(例如:`v2.0.0`) ，在升级过程中您需要此版本号

1. （可选）如果你无法找到之前运行rancher server的`docker run`命令，可通过以下命令找会执行命令。

    ```bash
    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
         registry.cn-shanghai.aliyuncs.com/rancher/run-config <rancher_container_name>
    ```

1. 运行以下命令，停止当前运行Rancher Server的容器

      ```bash
      docker stop <RANCHER_CONTAINER_ID>
      ```

      >**提示:** 您可以输入`docker ps`命令获取Rancher容器的ID

1. 创建当前Rancher Server容器的数据卷容器，以便在升级Rancher Server中使用，命名为rancher-data容器。

    - 替换`<RANCHER_CONTAINER_ID>`为上一步中的容器ID。
    - 替换`<RANCHER_CONTAINER_TAG>`为您当前正在运行的Rancher版本，如上面的先决条件中所述。

    ```bash
    docker create --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-data rancher/rancher:<RANCHER_CONTAINER_TAG>
    ```

1. 创建`rancher-data`数据卷容器的备份容器

    如果升级失败，可以通过此备份还原Rancher Server，容器命名:`rancher-data-snapshot-<CURRENT_VERSION>`.

    - 替换`<RANCHER_CONTAINER_ID>`为上一步中的容器ID。
    - 替换`<CURRENT_VERSION>`为当前安装的Rancher版本的标记。
    - 替换`<RANCHER_CONTAINER_TAG>`为当前正在运行的Rancher版本，如先决条件中所述 。

    ```bash
    docker create --volumes-from <RANCHER_CONTAINER_ID> \
    --name rancher-data-snapshot-<CURRENT_VERSION> rancher/rancher:<RANCHER_CONTAINER_TAG>
    ```

1. 拉取Rancher的最新镜像。

      ```bash
      docker pull rancher/rancher:stable (或者rancher/rancher:latest)
      ```

    >**注意** 如果您正在进行[离线升级]({{< baseurl >}}/rancher/v2.x/cn/upgrades/air-gap-upgrade/)，请在运行docker run命令时将您的私有镜像仓库URL添加到镜像名中
    >
    >例如: `<registry.yourdomain.com:port>/rancher/rancher:stable (或者rancher/rancher:latest)`

1. 通过`rancher-data`数据卷容器启动新的`Rancher Server`容器。

    ```bash
    docker run -d --volumes-from rancher-data --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:stable (或者rancher/rancher:latest)
    ```

    >**注意:** 升级过程会需要一定时间，不要在升级过程中终止升级，强制终止可能会导致数据库迁移错误。
    >
    >升级Rancher Server后，server容器中的数据会保存到`rancher-data`容器中，以便将来升级。

1. 删除旧版本`Rancher Server`容器

    如果您只是停止以前的Rancher Server容器(并且不删除它),则旧版本容器可能随着主机重启后自动运行，导致容器端口冲突。

1. 登录rancher，通过检查浏览器左下角显示的版本，确认是否升级成功。

    >**注意:** 如果升级未成功完成，则可以将Rancher Server及其数据恢复到上一个健康状态。有关更多信息，请查阅[单节点恢复]({{< baseurl >}}/rancher/v2.x/cn/backups-and-restoration/restorations/single-node/)。