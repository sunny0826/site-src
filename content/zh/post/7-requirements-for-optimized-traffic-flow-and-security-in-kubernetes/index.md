---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubernetes 中优化流量和安全性需要注意的7点要求"
subtitle: ""
summary: "翻译【7 Requirements for Optimized Traffic Flow and Security in Kubernetes】"
authors: ["Almas Raza","John Allen"]
tags: ["翻译","Kubernetes","安全","流量"]
categories: ["翻译"]
date: 2020-02-18T14:15:42+08:00
lastmod: 2020-02-18T14:15:42+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3i18tkb7j20sg0iyjui.jpg"
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

根据 [Portworx 在2018年进行的一项调查](https://portworx.com/wp-content/uploads/2018/12/Portworx-Container-Adoption-Survey-Report-2018.pdf)，五分之四的企业现在正在使用容器，其中83％的企业正在生产环境中使用。而这个数字在2017年只有67％，很明显，容器不仅仅是一种时尚。

但是，随着容器的流行，一些公司开始在 Kubernetes 内建立有效的流量控制和安全策略。

作为容器调度和集群管理平台，Kubernetes 致力于提供出色的基础架构，因此被无数公司采用。它刚刚开源五周年，最近在福布斯发表的一篇名为[《Kubernetes “the most popular open source project of our times”》](https://www.forbes.com/sites/janakirammsv/2019/05/25/5-exciting-facts-about-kubernetes-on-the-eve-of-its-5th-anniversary/#87a930c3e736)的文章表示，Kubernetes 已被 Capital One，ING Group，Philips，VMware 和 Huawei 等公司使用。

对于使用微服务架构（MSA）开发来应用程序的公司来说，Kubernetes 具有许多优势，特别是在应用程序部署方面。

出于上面这些原因，研发团队有必要了解 Kubernetes 独有的流量和安全情况。在本文中，我们将介绍：

- Kubernetes 是什么。
- Kubernetes 面临的挑战。
- Kubernetes 中的七个最重要的流量和安全要求。
- 关于开发和操作简便性的注意事项。

让我们开始吧。

## Kubernetes 是什么

Kubernetes 是一个开源的容器编排系统。根据 [Kubernetes’ own definition](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)，它是一个可移植且可扩展的程序，用于管理容器化的工作负载和服务，并提供以容器为中心的管理环境。

下图描述了 Kubernetes 的基本工作方式。图中可以看到一个主节点和两个工作节点。主节点用来告诉工作程序节点需要做什么工作，而工作程序节点则执行主节点提供给它们的指令。同时可以添加其他 Kubernetes 工作节点以扩展基础架构。

![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gc0k2knw9zj20r30czq6p.jpg)

如果仔细观察，您会发现在每个部分中都出现了 “Docker” 一词。Docker 是一个容器平台，非常适合在单个物理机或虚拟机（VM）上运行容器。

但是，如果您要在多个不同的应用程序中使用数百个容器，且您不希望将它们全部放在一台计算机上。这是催生 Kubernetes 的挑战之一。

使用 overlay 网络（如上图中的红色条所示），主节点中的容器不必知道它需要与之通信的容器位于哪个节点，就可以直接与之通信。

Kubernetes 的另一个主要功能是将信息打包到 “pod” 中，如果应用程序由多个容器组成，则可以将这些容器组成一个 pod ，并共享整个生命周期。

## Kubernetes 面临的挑战

像所有其他容器编排系统一样，Kubernetes 也面临的诸多挑战，其中包括：

- 内部和外部网络是隔离的。
- 容器和容器的 IP 地址会发生变化。
- 微服务之间没有访问控制。
- 没有应用程序层的可见性。

让我们更深入地探讨这些挑战。Kubernetes 的网络不是常规的网络，因为尽管使用了 overlay 网络，但内部和外部网络却是彼此不通的。

另外，Kubernetes 会隔离发生故障的节点或 Pod，以防止它们关闭整个应用程序。这可能导致节点之间的IP地址频繁更改。想要发现容器或容器的IP地址的服务就必须弄清楚新的IP地址是什么。

当涉及微服务之间的访问控制时，对于企业而言，重要的是要认识到 Kubernetes 节点之间的流量也能够流入外部物理设备或 VM。这可能会消耗资源并削弱安全性。

最后，无法在应用程序层检查信息是一个大问题。没有这种可见性，企业可能会错过收集详细分析信息的关键机会。

## Kubernetes 和云安全要求

到目前为止，我们已经讨论了 Kubernetes 的基本功能以及它所带来的挑战。现在，基于 [A10 Networks](https://www.a10networks.com/) 15年的经验，我们将继续讨论 Kubernetes 和云安全性的要求。

我们将讨论如下七点要求：

1. 高级应用程序交付控制器（ADC）
2. 使负载均衡器（LB）配置与基础架构保持同步
3. 南北向流量的安全
4. 为大规模部署准备的中央控制器
5. 微服务之间的访问控制
6. 东西向流量加密
7. 应用流量分析


### 1. 高级应用程序交付控制器（ADC）

![](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gc0ldvmd2ij20r30bjad0.jpg)

虽然企业可能已经在其基础架构的其他区域使用了高级应用程序交付控制器，但也有必要为 Kubernetes 部署一个。默认情况下，这将允许管理员操作在 Kubernetes 前的高级负载均衡器。

Kubernetes 已经配备了名为 *kube-proxy* 的网络代理。它提供了简单的用法：通过在三层中调整 iptables 规则来工作。但这是非常基本的，并与大多数企业操作习惯的有所不同。

许多人会将 ADC 或负载均衡器放在他们的 Kubernetes 前。这样就可以创建一个静态的虚拟 IP，所有人都可以使用它，并动态配置所有内容。

随着 Pod 和容器的启动，可以动态配置 ADC，以提供对新应用程序的访问，同时实现网络安全策略，并在某些情况下实施业务数据规则。通常，这是通过使用 “Ingress controller” 来实现的，其可以监控到新的容器和容器的启动，并且可以配置 ADC 以提供对新应用程序的访问权限，或者将更改通知给另一个 “Kubernetes controller” 节点。

### 2. 使负载均衡器（LB）配置与基础架构保持同步

![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gc0ll8lr83j20r30aytbc.jpg)

由于在 Kubernetes 中一切都是可以不断变化的，因此位于集群前的负载均衡器是无法追踪所有事情的。除非您有类似上图紫色框所示的东西。

该紫色框为 Ingress Controller，当容器启动或停止时，会在 Kubernetes 中创建一个事件。然后，Ingress Controller 会识别该事件并做出相应的响应。

如上图所示，Ingress Controlle 识别到容器已启动，并将其放入负载均衡池。这样，应用程序控制器（无论是在云之上还是内部）都可以保持最新状态。

这减轻了管理员的负担，并且比手动管理效率更高。

### 3. 南北向流量的安全

![](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gc1hnwxcqlj20r30bpgon.jpg)

南北和东西方都是用来描述流量流向的通用术语。南北流量是指流量流入和流出 Kubernetes。

如前所述，企业需要在 Kubernetes 前放置一些设备来监视流量。例如，防火墙，DDoS 防护或任何其他可捕获恶意流量的设备。

这些设备在流量管理方面也很有用。因此，如果流量需要流向特定的区域，这是理想的选择。Ingress Controller 在这方面也可以提供很多帮助。

如果企业可以通过统一的解决方案使这种功能自动化，那么他们可以得到：

- 更简化操作
- 更好的应用程序性能
- 可在不中断前端的情况下进行后端更改
- 自动化的安全策略


### 4. 为大规模部署准备的中央控制器

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1gc1i8wydpyj20r30bamzf.jpg)

企业还需要考虑到横向扩展，特别是在安全性方面。

如上图所示，Ingress Controller（由紫色框表示）仍然存在，但是这次它正在处理来自多个 Kubernetes 节点的请求，并且正在观测整个 Kubernetes 集群。

Ingress Controller 前方的蓝色圆圈是 [A10 Networks Harmony Controller](https://www.a10networks.com/products/harmony-controller/)。这种控制器可以实现高效的负载分配，并且可以将信息快速发送到适当的位置。

使用这样的中央控制器，必须选择一种在现有解决方案上进行少量额外配置，就可进行扩容和缩容的解决方案。

### 5. 微服务之间的访问控制

![](https://tva3.sinaimg.cn/large/ad5fbf65gy1gc1ikekni3j20r30ckjuz.jpg)

与流入和流出 Kubernetes 的南北流量相反，东西向流量在 Kubernetes 节点之间流动。在上图中，您可以看到东西向流量是如何运作的。

当流量在 Kubernetes 节点之间流动时，可以通过物理网络，虚拟网络或 overlay 网络来发送该流量。如果不通过某种方式来监控那些东西向的流量，那么对流量如何从一个 pod 或容器流向另一个 pod 或容器的了解就变得非常困难。

另外，它还可能带来严重的安全风险：**获得对一个容器的访问权限的攻击者可以访问整个内部网络**。

幸运的是，企业可以通过“服务网格”（例如 A10 Secure Service Mesh）来解决这个问题。通过充当容器之间的代理以实现安全规则，这可以确保东西向的流量安全，并且还可以帮助扩展，负载均衡，服务监视等。

此外，服务网格可以在 Kubernetes 内部运行，而无需将流量发送到物理设备或 VM。使用服务网格，东西向的流量状况如下所示：

![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gc1ikyysvtj20r30bcn0n.jpg)

通过这种解决方案，像金融机构这样的企业可以轻松地将信息保留在应有的位置，而不用担心影响安全性。

### 6. 东西向流量加密

![](https://tva1.sinaimg.cn/large/ad5fbf65gy1gc1ivrlln4j20r309ojtt.jpg)

如果没有适当的加密，未加密的信息可能会从一个物理 Kubernetes 节点流到另一个。这是一个严重的问题，特别是对于需要处理特别敏感信息的金融机构和其他企业。

这就是为什么对于企业而言，在评估云安全产品时，重要的是选择一种可以在离开节点时对流量进行加密，并在进入节点时对其进行解密的方法。

供应商可以通过两种方式提供这种类型的保护：

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1gc1ixe7n4xj20r30b0aci.jpg)

第一个选择是 Sidecar 代理部署，这种方法也是最受欢迎的。

通过这样的部署，管理员可以告诉 Kubernetes，每当启动特定 pod 时，应在该 pod 中启动一个或多个其他容器。

通常，其他容器是某种类型的代理，可以管理从 Pod 流入和流出的流量。

从上图可以看出，Sidecar 代理部署的不利之处在于，每个 pod 都需要启动一个 Sidecar，因此将占用一定数量的资源。

另一方面，企业也可以选择中心辐射代理部署。在这种类型的部署中，一个代理会处理从每个 Kubernetes 节点流出的流量。这样只需要较少的资源。

### 7. 应用流量分析

![](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gc1j83rredj20r30dfn2i.jpg)

最后一点是，企业了解应用程序层流量的详细信息至关重要。

有了可同时监控南北和东西向流量的控制器，就已经有了两个理想的点来收集流量信息。

这样做既可以帮助优化应用程序，又可以提高安全性，还可以拓展多种不同的功能。从最简单到最高级的顺序排列，这些功能可以实现：

- 通过描述性分析进行**性能监控**。大多数供应商都提供此功能。
- 通过诊断分析**更快地进行故障排除**。少数供应商提供此功能。
- 通过机器学习系统生成的预测分析获得**建议**。更少的供应商提供此功能。
- 通过真实直观的AI生成的规范分析进行**自适应控制**。只有最好，最先进的供应商才能提供此功能。

因此，当企业与供应商交流时，至关重要的是确定他们的产品可以提供哪些功能。

使用 A10 Networks 的类似产品，可以查看大图分析以及相关的单个数据包，日志条目或问题。具有这种粒度的产品是企业应寻求的产品。

## 关于开发和操作简便性的注意事项

最后，让我们看一下企业在 Kubernetes 中的流量和安全性方面应该追寻的东西。考虑这些因素还可以为开发和运维团队大大简化工作：

- 具有统一解决方案的简单体系结构。
- 集中管理和控制，便于进行分析和故障排除。
- 使用常见的配置格式，例如 YAML 和 JSON。
- 无需更改应用程序代码或配置即可实现安全性和收集分析信息。
- 自动化应用安全策略。

如果公司优先考虑以上这些，则企业可以在使用 Kubernetes 时享受简单、自动化和安全的流量。您的基础设施、架构和运维团队都会对此感到满意。

{{% alert note %}}
**原文信息** 

作者：Almas Raza、John Allen

发表时间：31 Jul 2019 9:51am

地址：https://thenewstack.io/7-requirements-for-optimized-traffic-flow-and-security-in-kubernetes/
{{% /alert %}}