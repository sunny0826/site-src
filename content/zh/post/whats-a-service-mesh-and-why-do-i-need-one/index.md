---
title: "Service Mesh是什么，我们又为什么需要它"
date: 2019-03-25T18:17:20+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g1ji5k4b8dj21qg15ojvt.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "Service Mesh 是一个专门用于使服务与服务之间的通信变得安全、快速和可靠的的基础设施。如果你正在在构建一个云原生（ Cloud Native ）应用，那么 Service Mesh 是你需要的。"
tags: ["翻译","service mesh"]
categories: ["翻译"]
keywords: ["翻译","service mesh"]
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3je975r1j21qg15ojvt.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>作者：William Morgan 发表于2017年4月25日，2018年11月26日有所修改。

Service Mesh 是一个专门使服务与服务之间的通信变得安全、快速和可靠的的基础设施。如果你正在在构建一个云原生（ Cloud Native ）应用，那么你一定需要 Service Mesh 。

在过去的一年中， Service Mesh 成为了云原生技术栈的关键组件。像 Paypal ,  Ticketmaster 和 Credit Karma 这样的大厂，已经将 Service Mesh 加入到他们的全部应用中。并且在2017年1月，开源的 Service Mesh 软件 Linkerd 加入云原生基金会（ CNCF ），成为云原生基金会（ CNCF ）的官方项目。但是什么是真正的 Service Mesh ？它又为何突然变的如此重要？

在这篇文章，我会讲解 Service Mesh 的定义，并通过应用服务架构过去十年的发展追溯其起源。并将 Service Mesh 与其他相似的概念（包括 API 网关，边缘代理以及 ESB （enterprise service bus））进行区分。最终，将会描述 Service Mesh 的发展方向，以及随着云原生概念的普及，Service Mesh 发生的变化。

## 什么是 Service Mesh
Service Mesh 这个服务网络专注于处理服务和服务间的通讯。其主要负责构造一个稳定可靠的服务通讯的基础设施，并让整个架构更为的先进和 Cloud Native。在工程中，Service Mesh 基本来说是一组轻量级的服务代理和应用逻辑的服务在一起，并且对于应用服务是透明的。

Service Mesh 作为独立层的概念与云原生应用的兴起有关。在云原生模式，单个应用可能有数百个服务组成，每个服务又可能有上千个实例，而每个实例都有可能被像 kubernetes 这样的服务调度器不断调度从而不断变化状态。而这些复杂的通信又普遍是服务运行时行为的一部分，这时确保端到端的通信的性能和可靠性就变的至关重要。

## Service Mesh 就是一个网络模型吗？
Service Mesh 是一个位于 TCP/IP 上的抽象层的网络模型。它假定底层 L3/L4 网络存在并且能够从一点向另一点传输数据。（它还假定这个网络和环境的其他方面一样不可靠，所以 Service Mesh 也必须能够处理网络故障。）

在某些方面，Service Mesh 就像是网络七层模型中的第四层 TCP 协议。其把底层的那些非常难控制的网络通讯方面的控制面的东西都管了（比如：丢包重传、拥塞控制、流量控制），而更为上面的应用层的协议，只需要关心自己业务应用层上的事了。

与 TCP 不同的是， Service Mesh 想要达成的目的不仅仅是正常的网络通讯。它为应用提供了统一的，可视化的以及可控制的控制平面。Service Mesh 是要将服务间的通信从无法发现和控制的基础设施中分离出来，并对其进行监控、管理和控制。

## Service Mesh 实际上做了什么？
在云原生应用中传递可靠的请求是十分复杂的。而 [Linkerd](https://linkerd.io/#_ga=2.114183109.310878331.1553762133-1927878916.1553476024) 提供了服务熔断、重试、负载均衡、熔断降级等功能，通过其强大的功能来管理那些必须运行在复杂环境中的服务。

这里列举一个通过 Linkerd 向服务发出请求的简单流程：

1. 通过 Linkerd 的动态路由规则来确定打算连接哪个服务。这个请求是要路由到生产环境还是演示环境？是请求本地数据中心的服务还是云上的服务？是请求正在测试的最新版的服务还是已经在生产中经过验证的老版本？所有的这些路由规则都是动态配置的，可以全局应用也可以部分应用。
2. 找到正确的目的服务后， Linkerd 从一个或几个相关的服务发现端点检索实例池。如果这些信息与 Linkerd 的服务发现信息不同， Linkerd 会决定信任哪些信息来源。
3. Linkerd 会根据观察到的最近的响应延迟来选择速度最快的实例。
4. Linkerd 发送请求给这个实例，记录延迟和响应类型。
5. 如果这个实例挂了、无响应或者无法处理请求， Linkerd 会再另一个实例上重试这个请求。（但只有在请求是幂等的时候）
6. 如果一个实例一直请求失败， Linkerd 会将其移出定时重试的负载均衡池。
7. 如果请求超时， Linkerd 会主动将请求失效，而不是进一步重试从而增加负载。
8. Linkerd 会记录指标和分布式的追踪上述行为的各个方面，将他们保存在集中的指标系统中。

以上只是简化版的介绍， Linkerd 还可以启动和重试 TLS ，执行协议升级，动态切换流量，甚至在故障之后数据中心的切换。
![image](http://tva2.sinaimg.cn/large/ad5fbf65ly1g1in1q1jnuj20sg0gbt99.jpg)

值得注意的是，这些功能旨在为每个实例和应用程序提供弹性伸缩。而大规模的分布式系统（无论是如何构建的）都有一个共同特点：都会因为许多小的故障，而升级为全系统灾难性的故障。Service Mesh 则被设计为通过快速的失效和减少负载来保护整个系统免受这样灾难性的故障。

## 为什么 Service Mesh 是必要的？
Service Mesh 本质上并不是什么新技术，而是功能所在位置的转变。Web 应用需要管理复杂的服务通信，Service Mesh 模式的起源和演变过程可以追溯到15年前。

参考2000年左右中型 Web 应用的典型三层架构，在这个架构中，应用被分为三层：应用逻辑、web 服务逻辑、存储逻辑。层之间的通信虽然复杂，但是毕竟范围有限，最多只有2跳。这里并不是 “Mesh” 的，但在每层中处理跳转的代码是存在通信逻辑的。

当这种架构向更大规模发展的时候，这种通信方式就无以为继了。像 Google , Netflix , 和 Twitte ，在面临巨大的请求流量的时候，他们的实现了云原生应用的前身：应用被分割成了许多服务（现在称作“微服务”），这些服务组成了一种网格结构。在这些系统中，通用通信层突然兴起，表现为“胖客户端”的形式 - Twitter 的 Finagle, Netflix 的 Hystrix 和 Google 的 Stubby 都是很典型的例子。

现在看来，像 Finagle 、Stubby 和 Hystrix 这样的库就是最早的 Service Mesh。虽然它们是为特定环境、语言和框架定制了，但都是作为基础设施专门用于管理服务间的通信，并（在 Finagle 和 Hystrix 开源的情况下）在其他公司的应用中被使用。

这三个组件都有应用自适应机制，以便在负载中进行拓展，并处理在云环境中的部分故障。但是对于数百个服务或数千个实例，以及不时需要重新调度的业务层实例，单个请求通过的调用链可能变的非常复杂，而且服务可能由不同的语言编写，这时基于库的解决方案可能就不再适用了。

服务通信的复杂性和重要性导致我们急需一个专门的基础设施层来处理服务间的通信，该层需要与业务代码解耦，并且具有捕获底层环境的动态机制。这就是 Service Mesh 。


## Service Mesh 的未来
Service Mesh 在云生态下迅速的成长，并且有着令人激动的未来等待探索。对无服务器计算（Serverless， 例如 Amazon 的 [Lambda](https://aws.amazon.com/lambda/)）适用的 Service Mesh 网络模型，在云生态系统中角色的自然拓展。Service Mesh 可能成为服务身份和访问策略这些在云原生领域还是比较新的技术的基础。最后，Service Mesh ，如之前的TCP / IP，将推进加入到底层的基础架构中。就像 Linkerd 是由像 Finagle 这样的系统发展而来，Service Mesh 将作为单独的用户空间代理添加到云原生技术栈中继续发展。

## 结语
Service Mesh 是云原生技术栈的关键技术。Linkerd 成立仅1年就成为了云原生基金会（CNCF）的一部分，拥有蓬勃发展的社区和贡献者。使用者从像 Monzo 这样颠覆英国银行业的创业公司，到像 Paypal、 Ticketmaster 和 Credit Karma 这样的互联网大厂，再到像 Houghton Mifflin Harcourt 这样经营了数百年的公司。

使用者和贡献者每天都在 Linkerd 社区展示 Service Mesh 创造的价值。我们将致力于打造这一令人惊叹的产品，并继续发展壮大我们的社区，[加入我们吧](https://linkerd.io/#_ga=2.40265824.310878331.1553762133-1927878916.1553476024)！

**原文链接** https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/