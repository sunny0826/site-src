---
title: "困难的 Kubernetes"
date: 2019-04-24T10:18:46+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65gy1g2dicvhncxj20xc0hi7wh.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "虽然 Kubernetes 赢得了容器战争，但是其仍然很难使用并且引起很多事故。"
tags: ["翻译","kubernetes"]
categories: ["翻译"]
keywords: ["翻译","kubernetes"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3izhwj5kj20xc0hi1kx.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
> 虽然 Kubernetes 赢得了容器之站，但是其仍然很难使用并且时长引起事故。

我想我应该给这篇文章做一点序言。 [kubernetes](https://kubernetes.io/) 为许多应用程序提供新的 runtime ，如果使用得当，它可以成为一个强大的工具，并且可以将您冲复杂的开发生命周期中解放出来。然而在过去的几年里，我看到很多人和公司都会搭建他们的 Kubernetes ，但常常只是处于测试阶段，从未进入到生产。

## Kubernetes 是如何运作的？

粗略的讲， Kubernetes 或者 K8S 看起来十分简单。您运行的 Kubernetes 节点至少被分为两类：Master 和 Workers。Master 节点通常不运行任何真实的工作负载，那是 Workers 节点的工作。

Kubernetes 的 Master 节点包含一个名叫 API server 的组件，其提供的 API 可以通过 ```kubectl``` 调用。此外还包括一个 scheduler ，负责调度容器，决定容器运行在哪个节点。最后一个组件是 controller-manager ，它实际上是一组多个控制器，负责处理节点中断、复制、加入 services 和 pods ，并且处理授权相关内容。所有的数据都存储在 etcd 中，这是一个可信赖的分布式键值存储服务（包含一些非常酷的功能）。总而言之，Master 节点负责管理集群，这里没什么特别大的惊喜。

另一方面， 真实的工作负载运行在 Worker 节点上。为此，它还包括许多组件。首先，Worker 节点上会运行 ***kubelet*** ，它是与该节点上的容器一起运行的 API ，负责与管控组件沟通，并按照管控组件指示管理 Worker 节点。另一个组件就是 ***kube-proxy*** ，其负责转发网络连接，根据您的配置运行容器。可能还有其他东西，如 ***kube-dns*** 或 ***gVisor***。您还需要集成某种 ***overlay network*** 或底层网络设置，以便 Kubernetes 可以管理您的 pod 之间的网络。

如果您想要一个更完整的概述，建议去看 Kelsey Hightowers 的 [Kubernetes  -  The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)。

## 生产就绪的 Kubernetes 

到目前为止，这听起来并不太糟糕。只是安装几个程序、配置、证书等。不要误会我的意思，这仍然是一个学习曲线，但这也不是系统管理员不能处理的问题。

然而，简单地手动安装 Kubernetes 并不代表其已经完全准备就绪，所以让我们谈谈让这个东西运行起来所需的步骤。

首先，**安装**。如果您想要某种自动安装，无论是使用 Ansible ， Terraform 还是其他工具。[kops](https://github.com/kubernetes/kops) 可以帮助您解决这个问题，但是使用 kops 意味着您将不知道它是如何设置的，并且当您以后想要调试某些东西时可能会引起一些其他问题。应对此自动化进行测试，并定期进行检查。

其次，您需要**监控**您的 Kubernetes 安装。所以您需要 [Prometheus](https://prometheus.io/) 、 [Grafana](https://grafana.com/) 等工具。您是在 Kubernetes 里面运行它吗？ 如果您的 Kubernetes 有问题，那么您的监控是否会也会挂掉？ 或者您单独运行它？ 如果是，那么您在哪里运行它？

另外值得注意的是**备份**。如果您的 Master 崩溃，数据无法恢复并且您需要重新配置系统上的所有 pod ，您会怎么做？您是否测试了再次运行 CI 系统中所有作业所需的时间？您有灾难恢复计划吗？

现在，既然我们在谈论 CI 系统，那么您需要为您的镜像运行 Docker 镜像仓库。当然，您可以再次在 Kubernetes 上做，但如果 Kubernetes 崩溃......您知道这个后果。当然，CI 系统与运行版本控制系统都有这个问题。理想情况下，这些系统是与生产环境隔离的，以便在系统出现问题时，至少可以访问 git ，来进行重新部署等操作。

## 数据存储

最后，我们来谈谈最重要的部分：存储。Kubernetes 本身并不提供存储解决方案。当然，您可以将存储挂载到主机安装目录，但这既不推荐也不简单。

基本上需要在 Kubernetes 下使用某种存储。例如，[rook](https://rook.io/) 使得运行 [Ceph](https://ceph.com/) 作为底层块存储需求的变得相对简单，但我对 Ceph 的体验是它还有有很多地方需要调整，所以您绝不是只需点击下一步就能走出困境。

## 调试

在与开发人员谈论 Kubernetes 时，一种常见的回答经常出现：在使用 Kubernetes 时，人们常常在调试应用程序时遇到问题。即使是一个例如容器未能启动的简单问题，也会引起混乱。

当然，这是一个教育问题。在过去的几十年中，开发人员已经学会了调试的“经典”步骤：在 ```/vat/log/``` 中查看日志等。但是对于容器，我们甚至不知道容器运行在哪个服务器上，因此它呈现出了一种范式转换。

## 问题：复杂

您可能已经注意到我正在跳过共有云提供商给您的东西，即使它不是一个完整的托管 Kubernetes。当然，如果您使用托管的 Kubernetes 解决方案，这很好，除了调试之外，您不需要处理上面这些问题。

Kubernetes 拥有许多可以移动组件，但 Kubernetes 本身也并不能提供完整的解决方案。例如，[RedHat OpenShift](https://www.openshift.com/) 可以，但它需要花钱，并且仍然需要添加自己的东西。

现在Kubernetes正处于 [Gartner hype cycle](https://www.gartner.com/en/research/methodologies/gartner-hype-cycle) 的顶峰，每个人都想要它，但很少有人真正理解它。在接下来的几年里，不少公司必须意识到 Kubernetes 并不是银弹，而如何正确有效地使用它才是关键。

我认为，如果您有能力将 Ops 团队专门用于为开发人员来维护底层平台，那么运行自己的 Kubernetes 是值得的。

> 本文作者：[Janos Pasztor](https://pasztor.at/)  2018-12-04

原文地址：https://pasztor.at/blog/kubernetes-is-hard