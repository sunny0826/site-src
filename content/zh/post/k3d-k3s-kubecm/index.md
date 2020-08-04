---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "K3d+Kubecm 本地开发运维两不误"
subtitle: "k3d+k3s+kubecm 快速搭建集群并管理"
summary: "使用 k3d 在本地快速搭建轻量级 k8s 集群 - k3s，并使用 kubecm 管理所有集群。"
authors: ["guoxudong"]
tags: ["k3s","k3d","kubecm","Kubernetes"]
categories: ["边缘计算"]
date: 2020-02-17T11:51:39+08:00
lastmod: 2020-02-17T11:51:39+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3ioptt0jj21qi15owjf.jpg"
  caption: ""
  focal_point: "Center"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## 前言

k3s 是由 Rancher Labs 于2019年年初推出的一款轻量级 Kubernetes 发行版，满足在边缘计算环境中运行在 x86、ARM64 和 ARMv7 处理器上的小型、易于管理的 Kubernetes 集群日益增长的需求。

k3s 除了在边缘计算领域的应用外，在研发侧的表现也十分出色。我们可以快速在本地拉起一个轻量级的 k8s 集群，而 k3d 则是 k3s 社区创建的一个小工具，可以在一个 docker 进程中运行整个 k3s 集群，相比直接使用 k3s 运行在本地，更好管理和部署。

在日常工作中，时长要在本地集群和多个远程集群之间切换来完成运维工作，这时使用 `kubecm` 快速将 k3s 集群的 kubeconfig 与现有集群的 kubeconfig 合并，并可快速切换集群，开发运维两不误。


## 安装 k3d

k3d 提供了多种安装方式，十分方便。

### 使用脚本安装

直接使用 `wget` 和 `curl` 安装

```bash
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
# 或
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
```

安装指定版本

```bash
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | TAG=v1.3.4 bash
# 或
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | TAG=v1.3.4 bash
```

### 使用 Homebrew 安装

MacOS 或安装了 Homebrew 的 Linux 可以使用 brew 安装：

```bash
brew install k3d
```

### 其他

还可以直接前往 [release 页面](https://github.com/rancher/k3d/releases) 下载二进制可执行文件，或者直接使用 `go install github.com/rancher/k3d` 安装。

## 创建 k3s 集群

创建 k3s 集群也十分简单，一行命令就可拉起，速度非常快。

```go
$ k3d create -n k3s-local
INFO[0000] Created cluster network with ID facae4a046b169721805f93ec21ba1acb65b9efb8cf35866529178cb0fba75a9
INFO[0000] Created docker volume  k3d-k3s-local-images
INFO[0000] Creating cluster [k3s-local]
INFO[0000] Creating server using docker.io/rancher/k3s:v1.0.1...
INFO[0000] SUCCESS: created cluster [k3s-local]
INFO[0000] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-local')"
kubectl cluster-info
```

但是一般情况下，如果没有梯子的话，k3s 集群虽然拉起来很快，但因为拉不到镜像，集群组件都无法正常拉起。

```go
$ export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-local')"
$ kubectl get pod -n kube-system
NAME                                      READY   STATUS              RESTARTS   AGE
helm-install-traefik-8wxmr                0/1     ContainerCreating   0          3m30s
metrics-server-6d684c7b5-j4sc7            0/1     ContainerCreating   0          3m30s
coredns-d798c9dd-j6lpw                    0/1     ContainerCreating   0          3m30s
local-path-provisioner-58fb86bdfd-wv7sw   0/1     ContainerCreating   0          3m30s
$ kubectl describe pod coredns-d798c9dd-j6lpw -n kube-system
...
Events:
  Type     Reason                  Age                 From                           Message
  ----     ------                  ----                ----                           -------
  Normal   Scheduled               <unknown>           default-scheduler              Successfully assigned kube-system/coredns-d798c9dd-j6lpw to k3d-k3s-local-server
  Warning  FailedCreatePodSandBox  7s (x7 over 4m30s)  kubelet, k3d-k3s-local-server  Failed create pod sandbox: rpc error: code = Unknown desc = failed to get sandbox image "k8s.gcr.io/pause:3.1": failed to pull image "k8s.gcr.io/pause:3.1": failed to pull and unpack image "k8s.gcr.io/pause:3.1": failed to resolve reference "k8s.gcr.io/pause:3.1": failed to do request: Head https://k8s.gcr.io/v2/pause/manifests/3.1: dial tcp 64.233.189.82:443: i/o timeout
```

### 离线安装

如果没有梯子的话，就只能选择使用离线安装。

#### 下载离线镜像

前往 [release 页面](https://github.com/rancher/k3s/releases) 下载指定版本的镜像，这里我们下载最新的 [v1.17.2+k3s1](https://github.com/rancher/k3s/releases/tag/v1.17.2%2Bk3s1) 镜像。

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gbzdedmqpdj20sh0k776o.jpg)

下载到 `~/airgap` 目录中，并进行解压，将解压后的目录重命名为 `1.17.2`。

### 运行离线镜像

这里再次运行 k3d，部署 k3s 集群。这里要注意的是，挂载离线镜像的话，必须使用 `-i` flag 来指定镜像版本，这里我们使用的是 [v1.17.2+k3s1](https://github.com/rancher/k3s/releases/tag/v1.17.2%2Bk3s1) 版本，而镜像的 tag 则是 `v1.17.2-k3s1`，如果不确定 tag，可以去 [DockerHub](https://hub.docker.com/r/rancher/k3s/tags) 上查看。

```go
$ k3d create -n k3s-local -i rancher/k3s:v1.17.2-k3s1  -v $(pwd)/airgap/v1.17.2/:/var/lib/rancher/k3s/agent/images/
INFO[0000] Created cluster network with ID 10b3fca995fcb491ae1fe1c901672bf6f0a0fd6f51785ba8403947d2773ebd43
INFO[0000] Created docker volume  k3d-k3s-local-images
INFO[0000] Creating cluster [k3s-local]
INFO[0000] Creating server using docker.io/rancher/k3s:v1.17.2-k3s1...
INFO[0000] SUCCESS: created cluster [k3s-local]
INFO[0000] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-local')"
kubectl cluster-info
```

查看 k3s 集群组件启动状态：

```go
$ export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-local')"
$ kubectl get pod -A -w
NAMESPACE     NAME                                      READY   STATUS              RESTARTS   AGE
kube-system   local-path-provisioner-58fb86bdfd-7jzbw   0/1     ContainerCreating   0          6m35s
kube-system   coredns-d798c9dd-jhmds                    1/1     Running             0          6m35s
kube-system   metrics-server-6d684c7b5-4x2cd            1/1     Running             0          6m35s
kube-system   traefik-6787cddb4b-9v7r4                  0/1     ContainerCreating   0          16s
kube-system   svclb-traefik-fzrqj                       0/2     ContainerCreating   0          15s
kube-system   helm-install-traefik-h8k2j                0/1     Completed           0          6m35s
kube-system   svclb-traefik-fzrqj                       2/2     Running             0          21s
```

## 使用 kubecm

在 k3s 集群启动成功后，使用 [`kubecm`](https://github.com/sunny0826/kubecm)，将 k3s 的 kubeconfig 与现有 kubeconfig 合并。

```bash
kubecm add -f $(k3d get-kubeconfig --name='k3s-local') -n k3s -c
```

切换集群，选择 k3s。

```go
$ kubecm s
Use the arrow keys to navigate: ↓ ↑ → ←  and / toggles search
Select Kube Context
  😼 k3s(*)
    prod-tg
    test
↓   banma

--------- Info ----------
Name:           k3s
Cluster:        cluster-485d6mhcfm
User:           user-485d6mhcfm
```

现在就可以在本地使用 k3s 集群进行开发工作，而有运维工作的时候，使用 `kubecm switch` 快速切换集群。

## 结语

![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1gbzegsyex5j20x90n70vv.jpg)

k3s 同时支持 **x86_64**、**ARM64** 和 **ARMv7** 架构，它可以十分灵活地跨任何边缘基础架构工作。不提 k3s 在边缘计算领域的应用，与之前使用的 [minikube](https://github.com/kubernetes/minikube) 相比，k3s 裁剪掉了许多用不到的功能，并且安装更简单，启动更快，空间占用也更小。相信 k3s 在开发侧的作用也会越来越大，使云原生应用的开发更加的便利。
