---
title: '最小化安装'
weight: 1
description: '在单节点的 Linux 服务器上安装 Kubernetes 和 Rainbond。'
---

最小化安装会把 Kubernetes 和 Rainbond 全安装在一个 Linux 节点上，可以帮助你节省资源。但是对生产环境确实不友好的，如果你需要部署一个生产集群，请产考[在 Linux 上安装高可用的 Rainbond](/docs/install/install-from-linux/high-availability/)。

## 前提条件

- 如果开启了防火墙，确保其满足[端口要求](/docs/install/requirements/#port-requirements)。
- 硬件：2 核 CPU，8G 内存，50G 磁盘。
- 操作系统：`64 bit CentOS 7`, `64 bit Ubuntu 1604/1804`, `64 bit Debian 9/10`

详情请参考[安装要求](/docs/install/requirements/)。

## 搭建 Kubernetes

在安装 Rainbond 之前，需要一个 `1.13` 及以上版本的 Kubernetes。请参考 [Kubernetes 的 all-in-one安装方式](/docs/user-operations/install/kubernetes-install/#kubernetes的all-in-one安装方式)。

## 使用 Helm 3 安装 Rainbond Operator

推荐使用 [Helm](https://helm.sh/) 来安装 Rainbond Opeartor。

### 安装 Helm 3

使用以下命令镜像安装：

```bash
wget https://goodrain-pkg.oss-cn-shanghai.aliyuncs.com/pkg/helm && chmod +x helm && mv helm /usr/local/bin/
```

helm 的安装详情，请查阅 [Installing Helm](https://helm.sh/docs/intro/install/)。

### 安装 Rainbond Operator

1. 创建 namespace, 推荐使用 `rbd-system`：

    ```bash
    kubectl create ns rbd-system
    ```

1. 下载 Rainbond Operator 的 chart 包：

    ```bash
    wget https://rainbond-pkg.oss-cn-shanghai.aliyuncs.com/offline/5.2/rainbond-operator-chart-v5.2.0-release.tgz && tar xvf rainbond-operator-chart-v5.2.0-release.tgz
    ```

1. 安装 Rainbond Operator

    ```bash
    helm install rainbond-operator ./chart --namespace=rbd-system
    ```

    如果想了解 Rainbond Operator 的参数，请查阅[这里](http://localhost:1313/docs/user-operations/rainbond-operator/configuration/)。

1. 确认 Rainbond Operator 状态

    ```bash
    $ kubectl get pod -n rbd-system
    NAME                  READY   STATUS    RESTARTS   AGE
    rainbond-operator-0   2/2     Running   0          110s
    ```

    稍微等待一会（根据具体的网络环境而定），直到 rainbond-operator-0 的状态（STATUS）变为 `Running`。

## 安装 Rainbond

访问 Rainbond Operator，开始安装 Rainbond。

1. 打开浏览器，输入主机 IP 地址：`http://<SERVER_IP>:30008`. 可以通过以下命令获取 `SERVER_IP`：

    ```bash
    echo $(kubectl get po rainbond-operator-0 -n rbd-system -o jsonpath="{..hostIP}")
    ```

    > 注意，获取到的 `SERVER_IP` 是内网地址，请根据实际情况直接使用或替换为外网地址。

1. 配置**网关安装节点**

    Rainbond Operator 默认会选择 Kubernetes 集群中符合条件的 master 节点去安装**网关**。
    如果你的集群中没有 master 节点，那么你可以`搜索选择`一个 `80`，`443` 等端口没有被占用的 node 节点，作为网关节点。

    > 提示：如果你无法搜索并选择一个网关 IP，请参考[无法选择网关节点](/docs/user-operations/install/troubleshooting/#无法选择网关节点)。

1. 配置**构建服务运行节点**

    Rainbond Operator 默认会选择 Kubernetes 集群中的 master 节点去安装**构建服务**。
    如果你的集群中没有 master 节点，那么你可以`搜索选择`一个 node 节点, 作为**构建服务运行节点**。

1. 可选项：**网关外网 IP**

    Rainbond Operator 默认会选择第一个**网关节点** 的 IP 地址作为 **网关外网 IP**。你也填写合适其他的 **网关节点 IP** 或 **公网 IP**.

1. 其他配置

    跳过其他的配置项，它们现在不重要。

1. 完成了上述配置后，单击 **配置就绪，开始安装**。

> 如果安装受阻，可以参考[故障排查](/docs/user-operations/install/troubleshooting/)，或联系相应管理人员。

## 验证安装

当安装的进度全部走完，会跳转到以下页面：

![image-20200204141936123](https://grstatic.oss-cn-shanghai.aliyuncs.com/images/5.2/rainbond-install-4.jpg)

说明已经安装完成。点击 **访问地址**，注册并开始使用 Rainbond。

## 安装命令行工具

为了方便运维管理集群请参照[文档](/docs/user-operations/tools/grctl/)安装 `grctl` 命令行工具。

## 卸载

卸载程序将删除 RBAC 权限，rbd-system 命名空间和所有相关资源。

### 卸载 Rainbond

访问 Rainbond Operator 的 UI 界面，单击 **卸载** 即可。

### 卸载 Rainbond Operator

```bash
helm delete rainbond-operator -n rbd-system
```
