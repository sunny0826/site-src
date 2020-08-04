---
title: "如何构建混合Kubernetes平台"
date: 2019-08-06T14:01:30+08:00
draft: false
type: blog
banner: "https://wx2.sinaimg.cn/large/ad5fbf65gy1g5q0b3o8laj20jg06sq36.jpg"
author: "David Donchez"
authorlink: "https://medium.com/@david.donchez"
translator: "郭旭东"
translatorlink: "https://github.com/sunny0826"
originallink: "https://medium.com/dailymotion/how-we-built-our-hybrid-kubernetes-platform-d121ea9cb0bc"
summary: "随着3年前重构 Dailymotion 核心API的决定，我们希望提供一种更有效的方式来托管应用程序，促进我们的开发和生产工作流程。 最终决定使用容器编排平台来实现这一目标，那么自然就选择了 Kubernetes。"
tags: ["翻译","kubernetes"]
categories: ["翻译"]
keywords: ["翻译","kubernetes"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3ijjy5q6j20jg06sq36.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

>随着3年前重构 [Dailymotion](https://www.dailymotion.com/) 核心API的决定，我们希望提供一种更有效的方式来托管应用程序，[促进我们的开发和生产工作流程](https://medium.com/dailymotion/deploying-apps-on-multiple-kubernetes-clusters-with-helm-19ee2b06179e)。 最终决定使用容器编排平台来实现这一目标，那么自然就选择了 Kubernetes。

那么为什么要建立自己的Kubernetes平台？

## 借由 Google Cloud 快速推动的 API 投入生产

> 2016年夏

三年前，在 [Vivendi](https://www.vivendi.com/) 收购 Dailymotion 之后，所有开发团队都专注于一个目标：提供全新的 Dailymotion 产品。

根据对容器、编排解决方案和以前的经验的分析，使我们确信 Kubernetes 是正确的选择。许多开发人员已经掌握了这一概念并知道如何使用 Kubernetes ，这对我们的基础设施转型来说是一个巨大的优势。在基础架构方面，我们需要一个强大而灵活的平台来托管这些新型的云原生应用程序。而公有云为我们提供了极大的便利，于是我们决定在 Google Kubernetes Engine 上部署我们的应用程序，即使之后我们也会在自己的数据中心中进行混合部署。

### 为何选择 GKE ？

我们做出这个选择主要是出于技术原因，但也因为我们需要快速提供基础设施来满足 Dailymotion 的业务需求。并且对托管的应用程序（如地理分布，可伸缩性和弹性）有一些要求。

![](https://ws2.sinaimg.cn/large/ad5fbf65gy1g5py1vm2k2j20hd0bbjtq.jpg)
<center>Dailymotion 的 GKE 集群</center>

Dailymotion 作为一个全球性的视频平台，需要通过减少延迟来改善用户体验。之前我们仅在巴黎提供 [API](https://developer.dailymotion.com/) ，但这样并非最佳，我们希望能够在欧洲、亚洲以及美国托管我们的应用程序。

这种延迟限制意味着我们在平台的网络设计方面面临着巨大的挑战。大多数云供应商要求我们在每个地区创建一个网络，并将所有这些网络通过 VPN 与托管服务互连，但 Google Cloud 允许我们在所有 Google 地区创建一个完全路由的单一网络，该网络在运营方面提供了便利并提高了效率。

此外，Google Cloud 的网络和负载均衡服务非常棒。它可以将我们的用户路由到最近的集群，并且在发生故障的情况下，流量会自动路由到另一个区域而无需任何人为干预。

![](https://wx4.sinaimg.cn/large/ad5fbf65gy1g5pytelbwnj20jg0avq4x.jpg)
<center>Google 负载均衡监控</center>

我们的平台同样需要使用 GPU，而 Google Cloud 允许我们以非常有效的方式直接在我们的 Kubernetes 集群中使用它们。

所有这一切使我们在启动后6个月开始接入 Google Cloud 基础架构上的生产流量。

但是，尽管具有整体优势，但使用共有云服务还是要花费不少成本。这就是为什么我们要评估采取的每项托管服务，以便将来将其内部化。事实上，我们在2016年底开始构建我们的本地集群，并启动了我们的混合策略。

## 在 Dailymotion 的内部构建容器编排平台

> 2016年秋

看到整个技术栈已经准备好在生产环境中应用，但[API仍在开发中](https://tartiflette.io/)，这使得我们有时间专注搭建我们的本地集群。

Dailymotion 多年来在全球拥有自己的内容分发网络，每月有超过30亿的视频播放量。显然，我们希望利用现有的优势并在我们现有的数据中心部署自己的 Kubernetes 集群。

我的目前拥有6个数据中心的2500多台服务器。所有这些都使用 Saltstack 进行配置，我们开始准备所有需要的公式来创建主节点、工作节点以及 Etcd 集群。

![](https://wx2.sinaimg.cn/large/ad5fbf65gy1g5pzm4m985j20jg06tgm7.jpg)

### 网络部分

我们的网络是一个完全路由的网络。每个服务器使用Exabgp通过网络广播自己的IP。我们比较了几个网络插件， [Calico](https://www.projectcalico.org/) 使用的是三层网络，因此这是唯一满足我们需求的网络插件。

由于我们想要重用基础架构中的所有现有工具，首先要解决的问题是插入一个自制网络工具（我们所有服务器都使用它），通过我们的 Kubernetes 节点通过网络广播 IP 范围。我们让 Calico 为 pod 分配 IP，但不使用它与我们的网络设备进行BGP会话。路由实际上是由Exabgp处理的，它宣布了Calico使用的子网。这使我们可以从内部网络访问任何pod，尤其是来自我们的负载均衡器。

### 我们如何管理入口流量

为了将传入的请求路由到正确的服务，我们希望使用 Ingress Controllers 与 Kubernetes 的入口资源集成。

3年前，nginx-ingress-controller 是最成熟的控制器 ，并且 Nginx 已经使用多年，并以其稳定性和性能而闻名。

在我们的设计中，我们决定在专用的 10Gbps 刀片服务器上托管我们的控制器。每个控制器都插入其所属集群的 kube-apiserver 端点。在这些服务器上，我们还使用Exabgp来广播公共或私有IP。我们的网络拓扑允许我们使用来自这些控制器的BGP将所有流量直接路由到我们的pod，而无需使用NodePort服务类型。这样可以避免节点之间的水平流量，从而提高效率。

![](https://ws3.sinaimg.cn/large/ad5fbf65gy1g5q05ex27bj20in0fbt9q.jpg)
<center>从 Internet 到 pods 的流量</center>

现在我们已经看到了我们如何构建混合平台，我们可以深入了解流量迁移本身。

## 将流量从 Google Cloud 迁移到 Dailymotions 基础架构

> 2018年秋

经过近2年的构建、测试和微调，我们发现自己拥有完整的 Kubernetes 技术栈，可以接收部分流量。

![](https://wx2.sinaimg.cn/large/ad5fbf65gy1g5q0b3o8laj20jg06sq36.jpg)

目前，我们的路由策略非常简单，但足以解决我们的问题。除了我们的公共IP（Google Cloud和Dailymotion）之外，我们还使用AWS Route 53 来定义策略并将终端用户流量引入我们选择的集群。

![](https://ws2.sinaimg.cn/large/ad5fbf65gy1g5q0ds3spjj20jg07a0tk.jpg)
<center>使用Route 53的路由策略示例</center>

在 Google Cloud 上很简单，因为我们为所有群集使用唯一的IP，并且用户被路由到他最近的 GKE 群集。对于我们来说，我们不使用相同的技术，因此我们每个群集都有不同的IP。

在此次迁移过程中，我们将目标国家逐步纳入我们的集群并分析其收益。

由于我们的GKE集群配置了自动调节自定义指标，因此它们会根据传入流量进行扩展/缩小。

在正常模式下，区域的所有流量都路由到我们的内部部署集群，而GKE集群则使用Route 53提供的运行状况检查作为故障转移。

## 结语

我们接下来的步骤是完全自动化我们的路由策略，以实现自动混合策略，不断增强我们的用户体验。在效益方面，我们大大降低了云的成本，甚至改善了API响应时间。我们相信我们的云平台足以在需要时处理更多流量。
