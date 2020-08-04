---
title: "8分钟入门 K8S"
date: 2019-04-30T13:38:12+08:00
draft: false
type: blog
banner: "https://ws3.sinaimg.cn/large/ad5fbf65gy1g2ji8i9dx3j20xc0m87ci.jpg"
author: "Omer Hamerman"
authorlink: "https://medium.com/@omerxx"
translator: "郭旭东"
translatorlink: "https://github.com/sunny0826"
originallink: "https://medium.com/prodopsio/an-8-minute-introduction-to-k8s-94fda1fa5184"
summary: "8分钟快速了解 Kubernetes 基本概念，快速入门 K8S。"
tags: ["翻译", "kubernetes"]
categories: ["翻译"]
keywords: ["翻译", "kubernetes"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3ic1v5x3j20xc0m87ci.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
> 读完 [Kubernetes: Up and Running](https://www.amazon.com/Kubernetes-Running-Dive-Future-Infrastructure/dp/1491935677) 后，我写下了这篇文章。旨在为那些认为文章 [TL;DR](https://blog.maoxianplay.com/posts/cant/) 的人进行一些总结，这同时也是一种强迫自己检查所阅读内容的好方法。

基于 Google [Borg](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes/) 的开源系统 K8S( Kubernetes ) 是一个非常强大的容器编排调度系统。 其整个生态系统，包括：工具，模块，附加组件等，都是使用 Golang 语言编写的，这使得 K8S 及其周边生态系统基本上是面向 API 对象、运行速度非常快的二进制文件的集合，并且这些二进制文件都有很好的文档记录，易于编写和构建应用程序。

在深入了解之前，我想先介绍一下 K8S 的竞争对手：ECS 、 Nomad 和 Mesos 。ECS 是 AWS 自己的业务编排解决方案，而最近 AWS 上也引入了一个托管的 K8S 系统 -- EKS 。两者都提供 [FARGATE](https://aws.amazon.com/fargate/) ，允许用户运行其应用并忽略其运行物理资源。

K8S 作为一个开源系统，在采用量上毫无疑问是最大赢家，同时它也可以以托管形式在三个主要云提供商上提供服务。然而，它比其他系统更加复杂。K8S 可以处理几乎任何类型的容器化工作负载，但这并不意味着每个人都需要它。用户也可以选择其他解决方案，例如，单独部署在 AWS 上的互联网产品可以在生产环境很好的使用 ECS 而非 K8S。

话虽如此，k8s也有其神奇之处 -- 它可以在任何地方部署，同时拥有一个活跃的社区和数百个核心开发人员，以及其广泛生态系统中的数千个其他开源贡献者。它快速、新颖、模块化和面向 API ，使其成为对于构建插件和服务非常友好的系统。

## 话不多说，这里把 K8S 分为的11个部分介绍


### 1. Pods

**Pods** 是 K8S 中创建或部署的最小基本单位。一个 pod 可以由多个容器组成，这些容器将形成一个部署在单个节点上的单元。一个 pod 包含一个容器之间共享的 IP。在微服务中， pod 将是执行某些后端工作或提供传入请求的微服务的单个实例。

### 2. Nodes

**Node** 就是服务器。它们是 K8S 部署其 pod 的“裸机”（也可以是虚拟机）。Nodes 为 K8S 提供可用的群集资源，以保持数据，运行作业，维护工作负载和创建网络路由。

### 3. Labels & Annotations

**Labels** 是 K8S 及其终端用户过滤和筛选系统中类似资源的方式，也是一个资源需要“访问”或与另一个资源关联的粘合剂。例如：一个 Service 想要开放 Deployment 的端口。无论是监控，记录，日志，测试，任何k8s资源都应添加 Labels 以供进一步检查。例如： `app=worker` ，一个给系统中所有工作 pod 的标签，稍后可以使用 `kubectl` 工具或 k8s api 使用 `--selector` 字段进行选择。

**Annotations** 与 Labels 非常类似，但是它通常以字符串的形式用于存储元数据，但他不能用于标识和选择对象，通常也不会被 Kubernetes 直接使用，其主要目的是方便工具或用户的阅读和查找等。

### 4. 服务发现

作为编排调度器，控制不同工作负载的资源，K8S 管理 pods 、jobs 和其他任何需要网络通信的物理资源。 K8S 使用 [etcd](https://kubernetes.io/docs/concepts/overview/components/#etcd) 管理这些。 etcd 是 K8S 的“内部”数据库， Master 节点使用它来知道一切资源都在哪里。K8S 还为您的服务提供了实时的 “服务发现” - 所有 Pod 都使用的自定义 DNS 服务（CoreDNS），您可以通过解析其他服务的名称来获取其 IP 地址和端口。它不需要任何设置，在 K8S 集群中“开箱即用”。

### 5. ReplicaSets

虽然 pod 运行任务，但通常单个实例是不够的。出于对冗余和负载处理的考虑，需要进行复制容器，即“弹性缩放”。K8S 使用 **ReplicaSet** 来实现伸缩扩展。根据副本的数量来表示系统的期望状态，并且在任何给定时刻保持系统的当前状态。

这也是配置自动扩展的地方，在系统负载高时创建新的副本，以及在不再需要这些资源来支持运行工作负载时减少扩展。简单的讲就是：少则增加，多增删除。

### 6. DaemonSets

有时，某些应用程序在每个节点上只需要一个实例。最好的例子就是像 [FileBeat](https://www.elastic.co/products/beats/filebeat) 这样的日志采集组件。为了让 agent 从节点上收集日志，它需要运行在所有节点上，但只需要一个实例即可。为了创建满足上面需求的的工作负载，K8S 使用 **DaemonSets** 来完成这个工作。

### 7. StatefulSets

虽然大多数微服务都是无状态的应用程序，但是还是有一部分并不是。有状态的工作负载需要由某种可靠的磁盘卷来支持。虽然应用程序容器本身可以是不变的，并且可以用更新的版本或更健康的实例来替换它们，但是即使使用其他副本也是需要持久化的数据。为此，**StatefulSets** 允许部署整个生命周期内需要运行在同一节点的应用程序。它还保留了它的 “名称” ; 容器内的 `hostname` 和整个集群中服务发现的名称。一个包含3个 ZooKeeper 的 StatefulSet 可以命名为 `zk-1` ，`zk-2` 和 `zk-3` 还可以扩展为包含其他成员，如 `zk-4` ， `zk-5` 等... StatefulSets 还需要管理 PVC 。

### 8. Jobs

K8S 核心团队考察了绝大多数需要使用编排系统的应用程序。虽然大多数应用程序需要持续的正常运行时间来处理服务请求，例如 Web 服务，但有时也需要运行批量任务并在任务完成后进行清理。如果您愿意，可以使用小型无服务器环境。而在 K8S 中实现这一功能，可以使用 **Job** 资源。Jobs 正是听起来的那样，一个工作负载容器来完成特定的工作，并在成功后被销毁。一个很好的例子是设置一组 worker ，从要处理和存储的队列中读取任务。一旦队列为空，直到下一批准备好进行处理，都不再需要启动 worker。

### 9. ConfigMaps & Secrets

如果您还不熟悉 [Twelve-Factor App manifest](https://12factor.net/) 《[十二要素应用](../12-factor)》 ，可以点击链接了解一下。现代应用程序的一个关键概念是无环境，可通过注入的环境变量进行配置。应用程序应完全与其所在位置无关。**ConfigMaps** 在 K8S 中实现这一重要概念。其本质上是环境变量的 key-value 列表，这些变量被传递给正在运行的工作负载以确定不同的 runtime 行为。

**Secrets** 与 **ConfigMaps** 类似，通过加密的方式防止密钥、密码、证书等敏感信息泄漏。

> 我个人认为在任何系统上使用密码的最佳选择是 Hashicorp 的 Vault 。请务必阅读我去年写的关于它的[文章](https://medium.com/prodopsio/security-for-dummies-protecting-application-secrets-made-easy-5ef3f8b748f7)，关于 Vault 可以为你的产品提供的功能，以及我的一位同事写的另一篇更具技术性的[文章](https://medium.com/prodopsio/taking-your-hashicorp-vault-to-the-next-level-8549e7988b24）。

### 10. Deployments

为了使新版本快速替换原有的应用程序，我们希望将构建、测试和发布在一块来实现 [short feedback loops](https://www.ibm.com/developerworks/community/blogs/beingagile/entry/short_feedback_loops_everywhere?lang=en) 。K8S 使用 Deployments 来不断部署新软件，Deployments 是一组用来描述特定运行工作负载的元数据。例如：发布新版本，bug 修复，甚至是回滚（这是k8s的另一个内部选项）。

在 K8S 中部署软件有两个主要__策略__：

1. **Replacement**：将使用新副本替换您的整个工作负载，整个过程需要强制停机。

2. **RollingUpdate**：k8s通过两种特定配置来实现使用新的 Pods 实例滚动更新：

    1. `MaxAvailable` ： 该设置表示在部署新版本时可用的工作负载的百分比（或确切数量），100％表示“我有2个容器，保持2个存活并在整个部署期间正常提供服务”。
    2. `MaxSurge` ： 该设置表示升级期间总 Pod 数最多可以超出期望的百分比（或数量），100％表示“我有 X 个容器，再部署 X 个容器，然后开始推出旧容器”。

### 11. Storage

K8S 在存储上添加了一层抽象，工作负载可以为不同的任务请求特定的存储，甚至可以管理持续的时间可以超过某个pod的生命周期。为了简短起见，我想向您介绍我最近发布的关于k8s存储的[文章](https://medium.com/prodopsio/k8s-will-not-solve-your-storage-problems-5bda2e6180b5)，特别是为什么它不能完全解决数据库部署等数据持久性要求。

## 概念性理解

K8S 是根据一些指导方向设计和开发的，考虑到社区的性质，每个特征、概念和想法都被内置于系统中。此外，终端用户会以某种方式使用该系统，作为一个开源和免费的系统，不属于任何人，你可以用它做任何你想要做的事。

面向 API ：系统中的每个部分都以一种可通过记录良好且可操作的 API 进行交互的方式进行构建。核心开发人员确保作为终端用户的您可以进行更改，查询和更新，用来提供更好的用户体验。

工具友好 ： 作为上面一点的衍生，K8S 是热衷于在其 API 周围创建工具的。它将自身做为一个原始平台，以可定制的方式构建，以供其他人使用，并进一步开发用于不同的工具。有些已经变得非常有名并被广泛使用，如 Spinnaker ，Istio 和许多其他功工具。

声明性状态 ： 鼓励用户使用具有声明性描述的系统而不是命令式描述。这意味着系统的状态和组件最好被描述为在某种版本控制（如 git ）中管理的代码，而不会因为某一处手动更改也对整体有影响。这样，k8s更容易[灾难恢复](https://en.wikipedia.org/wiki/Disaster_recovery) ，易于在团队之间分享和传递。

## 最后

本文试图将重点放在 K8S 的介绍和主要概念上，当然，K8S 还有其他非常重要的领域，比如物理系统构建模块，如 `kubelet`， `kube-proxy` ， `api-server` 和终端操作工具：`kubectl`。我将在下一篇文章中讨论以及介绍这些很酷的功能。

原文地址： https://medium.com/prodopsio/an-8-minute-introduction-to-k8s-94fda1fa5184