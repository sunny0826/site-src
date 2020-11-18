---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "K3d vs Kind 谁更适合本地研发"
subtitle: ""
summary: "对比 K3d 和 Kind 在本地开发侧的能力。"
authors: ["guoxudong"]
tags: ["k3s","k3d","kind"]
categories: ["kubernetes"]
date: 2020-11-18T09:44:37+08:00
lastmod: 2020-11-18T09:44:37+08:00
draft: false
type: blog
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65gy1gktbsfimhuj21qi15o78d.jpg"
---
## 前言

随着 Kubernetes 及其周边生态的发展壮大，新项目层出不穷，越来越多的开发者加入到了开发云原生应用的行列。随之而来的，就是 Kubernetes 极高的复杂度和研发人员想要快速开发云原生应用之间的矛盾。为了解决这个矛盾，同时也为了降低学习 Kubernetes 的门槛，社区出现了各种各样的解决方案。如 minikube 用于生成一个单节点的 k8s VM，而 katacoda 则是在 web 端提供交互式的 k8s 操作教程。

在这些方案中，最有意思的一类方案是使用 docker 运行整个 k8s 集群，以极高的启动速度、极低的使用成本以及简单的操作深受广大开发者欢迎，并广泛应用于各种云原生应用开发和 e2e 测试中。其中最具代表性的就是 [Kubernetes SIGs 项目 Kind](https://kind.sigs.k8s.io/) 和 [Rancher Lab 开源的 k3d](https://k3d.io/)。在这篇文章中，我们就来探究一下这两个项目在本地开发侧的优缺点，站在一个开发者的的角度看看使用哪个项目更能提升我们的效率。

## Kind

![](https://tvax3.sinaimg.cn/wap360/ad5fbf65gy1gkt56jeqr7j20rd0gimz7.jpg)

Kind 顾名思义 Kubernetes in docker，是一个使用 docker 容器在本地运行 Kubernetes 集群的工具。其本身就是为了测试 Kubernetes 而设计，所以天生就和 CI 紧密关联，广泛应用于各种云原生项目的 CI 中，同时因为其可以快速拉起集群和操作简单，深受开发者喜爱，可谓是“有 Kind 不思 Minikube”。

Kind 使用 kubeadm 进行集群的创建，内部使用 containerd 运行组件容器，可以通过指定配置文件 `config.yaml` 来拉起相应配置的集群，支持多节点集群，同时也可以把本地的镜像加载到集群中，实现测试镜像无需上传镜像仓库的功能。并且之前国内拉取不到镜像的问题已经解决，直接在 dockerhub 拉取 `kindest/node` 镜像，镜像中均已包含创建 Kubernetes 集群所需的全部资源，无需再额外下载。

## K3d

![](https://tvax2.sinaimg.cn/wap360/ad5fbf65gy1gkt5pu3ifhj21s00ocaef.jpg)

与 Kind 类似，K3d 是使用 docker 容器在本地运行 k3s 集群，k3s 是由 Rancher Lab 开源的轻量级 Kubernetes。k3d 完美继承了 k3s 的简单、快速和占用资源少的优势，镜像大小只有 100 多 M，启动速度快，支持多节点集群。虽然 k3s 对 Kubernetes 进行了轻量化的裁剪，但是提供了完整了功能，像 Istio 这样复杂的云原生应用都可以在 k3s 上顺利运行。

K3d 同时也有着自己的优势，除了启动速度快和占用资源少以外，在边缘计算和嵌入式领域也有着不俗的表现。因为 k3s 本身应用场景主要在边缘侧，所以支持的设备和架构很多，如：ARM64 和 ARMv7 处理器。很多老旧 PC 和树莓派这样的设备都可以拿来做成 k3s 集群，为本地研发测试燃尽最后的生命。

## Kind vs K3d

下面就对 Kind 和 K3s 的进行一些简单的对比，对比数据均来自同一台 macbook pro，使用相同的资源进行。采用目前最新版本，版本如下：

- k3d v3.2.1
- kind v0.7.0

### 工具安装

K3d 和 Kind 均支持使用 `brew` 安装，且均在安装时自动注入命令补全 `completion` 脚本，安装好之后即可实现按 `<TAB>` 自动补全命令，无需手动操作，体验极佳。

### 本地镜像注入

`k3d image import` 和 `kind load` 命令均可将本地镜像注入集群且均支持注入 docker 镜像或镜像文件。

### base 镜像大小

本项 k3s 完胜，由于 k3s 本身就是轻量级的 Kubernetes，所以镜像极小，大小还没有 Kind base 镜像 `kindest/node` 的零头大。

![镜像大小比较](https://tva2.sinaimg.cn/large/ad5fbf65gy1gkta0t5u1mj20sk023q5p.jpg)

### 启动速度

这里的启动速度排除了镜像拉取的时间，两者的镜像均已拉取到本地，且启动的均为默认配置的集群。

k3d 启动时间：

![k3d 启动时间](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gkta5bjvuhj20gp03radf.jpg)

kind 启动时间：

![kind 启动时间](https://tva1.sinaimg.cn/large/ad5fbf65gy1gkta5meujxj20hz06a79q.jpg)

可以看到还是 k3s 的启动速度要快于 Kind，因为 k3s 本身就是主打轻量级和快速启动，但 kind 的启动速度也很快，耗时均在本地用户的可接受范围内。

### 资源占用

这里对比的两个集群均是默认配置，无运行任何其他组件和服务。同样 k3d 占用的资源更少，但总体都没有超出不可承受的范围。

![资源占用](https://tva4.sinaimg.cn/large/ad5fbf65gy1gktaadtyvqj20ul01rdik.jpg)

### ingress

这项更偏向使用者的体验，k3s 自带了 traefik 作为 Ingress Controller 开箱即用，用户如果不想关心使用什么 ingress 可以无需关心，直接上手使用。而 Kind 则没有预装 Ingress Controller，如果需要使用则需要手动部署。

### 架构支持

Kind 目前只支持 x86 的 CPU 架构，而 k3d 则支持  x86、ARM64 和 ARMv7，如果想在使用 M1 芯片的新 MacBook 或在有 ARM 架构 CPU 的 PC 上使用，暂时只能使用 k3d。

## 总结

通过对比可以看出 k3d 和 kind 的相似点很多，但两方的使用场景还是略有差别。Kind 更贴近原生 Kubernetes，适合用于开发测试 Kubernetes 原生组件、资源比较充沛的开发者；而 K3s 则更适合边缘计算场景应用开发、资源紧张、使用非 x86 CPU 架构设备的开发者。如果你只是想学习 Kubernetes 集群的操作、各种资源的使用、Kubernetes 相关项目的尝鲜，则这两个工具都是不错的选择。

{{% alert title="注意" color="warning" %}}
两者都是为了方便测试 kubernetes 集群,切不可用于生产环境。
{{% /alert %}}
