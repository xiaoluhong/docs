---
title: 8 - 升级
weight: 8
---

## 1. rancher2.1.x升级到2.2.x，无法`查看/编辑yaml`文件

_此问题将在 v2.2.4 修复_

![image-20190430094043207](assets/image-20190430094043207.png)

### 解决方法：

- 架构为使用外部七层负载均衡器代理rancher ha

   检查外部负载均衡器的配置，如果有`http2`的参数，把http2取消。比如nginx的配置：![image-20190430094333165](assets/image-20190430094333165.png)

- 架构为使用内部ingress 七层代理Rancher HA

  1. 进入`local`集群，切换到`系统项目`\\`资源`\\`配置映射`页面，找到并编辑`nginx-configuration`![image-20190430094846660](assets/image-20190430094846660.png)
  2. 添加`use-http2=false`禁止`http2`![image-20190430094945817](assets/image-20190430094945817.png)

## 2. 升级rancher后出现`unable to handle the request`

   ![image-20190529230049960](assets/image-20190529230049960.png)

rancher v2.0.x 、v2.1.x与v2.2.x之间证书存储方式有变化，出现以上问题因为存储方式发生变化后rancher无法连接到业务集群。