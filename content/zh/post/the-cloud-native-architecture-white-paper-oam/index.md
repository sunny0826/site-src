---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "读完《云原生架构白皮书》，我们来谈谈开放应用模型（OAM）"
subtitle: ""
summary: "读完《云原生架构白皮书》，我们来谈谈开放应用模型（OAM）"
authors: ["guoxudong"]
tags: ["OAM"]
categories: ["OAM"]
date: 2020-07-24T10:53:04+08:00
lastmod: 2020-07-24T10:53:04+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65gy1gh4iqcxmu7j20om0gomzp.jpg"
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
## 前言

7月21日阿里云发布了《云原生架构白皮书》，该书由阿里云众多技术专家共同编撰而成，从云原生定义、技术、架构、产品、实践和发展趋势几个方面详细介绍了云原生这一近些年来大火的技术概念。受阿里云邀请，我有幸在该书发布前试读了该书，但是由于最近比较忙，现在才有空和大家分享我的试读感受。

熟悉我的朋友肯定知道，去年开放应用模型（OAM）概念一经提出，我就十分关注这一技术模型，最近更是参与到了该模型的实现项目 [Crossplane](https://github.com/crossplane/oam-kubernetes-runtime) 中，同社区中的同学共同实现云原生技术“以应用为中心”这一终极愿景。但是苦于社区中的资料都是英文，同时自己的理解又比较片面，在向身边同事和其他不了解该项技术的同学科普 OAM 时，往往很难准确表达我的观点。

OAM 是什么？OAM 能做什么？我们为什么需要 OAM？每每被同事进行灵魂拷问时，总是不能拿出完整、条理、有说服力的东西，只能根据自己的理解以及一些零零散散的技术文章来说明我的观点，很是不爽。但是当我读到《云原生架构白皮书》第三章中的开放应用模型（OAM）章节时，我知道我的问题解决了。该章系统的介绍了 OAM 这项技术的背景、定义、概念、实现和未来，读者只要对云原生稍有理解，就能轻松从这章中找到前面那些问题的答案。

## 那么 OAM 到底是什么？

从《云原生架构白皮书》的内容出发，结合我的理解，大致将 OAM 的特点分为以下三点：

### 以应用为中心

今年是 Kubernetes 项目诞生的第六年，在这六年中，以 Kubernetes 为首的云原生技术快速的改变着我们的技术架构，一个又一个的应用被拆分成微服务，打包成容器，运行在 Kubernetes 上。然而随着微服务越拆越多，管理微服务的难度也呈指数型增长，Kubernetes 中并没有”应用“这一概念，提供给我们的只有 deployment、StatefulSet 这样工作负载粒度的资源，而一个应用，可能由多个 Deployment、Service、以及各种相关配套资源组成（如：HPA 用于弹性伸缩、Ingress 用于外部访问等）。Kubernetes 并没有提供给我们一个统一的资源或者说是方法来管理这些相关资源，各个公司只能开发自己的 PASS 平台或设立规范约束自己的应用。

OAM 的出现补充了“应用”这一概念，建立对应用和它所需的运维能力定义与描述的标准规范。换言之，OAM 既是标准“应用定义”同时也是帮助封装、组织和管理 Kubernetes 中各种“运维能力”的工具。通过 OAM 中应用的可交付对象 - Application Configuration，我们可以轻松的掌握我们的应用到底有那些 Kubernetes 工作负载组成，这些工作负载都使用了哪些运维特性，这些内容都会以 Kubernetes API 对象的形式展示，查看起来和查看 Deployment 与 Service 资源一样方便。

![Application Configuration](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gh3bn5n23zj20t80f0myj.jpg)

### 关注点分离

在实践中，如果基础架构和应用是由不同团队维护的，由于各个团队的关注点不同、对 Kubernetes 了解的程度不同、使用习惯不同，很容易产生混乱。实际上，对于业务研发人员和运维人员而言，他们并不想配置这些如此底层的资源信息，而希望有更高维度的抽象。这就要求一个真正面向最终用户侧的应用定义，一个能够为业务研发和应用运维人员提供各自所需的应用定义原语。

![](https://tva2.sinaimg.cn/large/ad5fbf65gy1gh3cl2hzsaj20w80gmgnl.jpg)

通过组件（Component）和运维特征（Trait）将业务研发人员与运维人员关注的不同特征进行分离，再将不同的运维特征（Trait）与业务组件（Component）进行绑定，最终再由OAM 可交付物 – Application Configuration 将所有组件组装为一个统一的应用。研发与运维对资源的控制进行细粒度的划分，可以有效的解决实际情况中存在的类似”我比你更懂 Kubernetes，要听我的“的现象，避免了研发与运维之间的甩锅与扯皮的情况。

### 面向最终用户的应用管理平台

这部分白皮书中并未详细提及，但这也是我们现阶段的主要工作和努力方向，经过不到一年的时间，OAM 的概念、思想已经基本成熟，而基于 OAM 的实现也已经出现 -  Crossplane 项目，该项目目前为 CNCF 的 [Sandbox](https://www.cncf.io/sandbox-projects/) 项目。

Crossplane 的出现解决了平台维护者，也就是负责维护 Kubernetes 的基础设施工程师的难题。但是对于应用研发和运维人员，也就是 OAM 的最终用户，操作起来并不是十分的友好。基础设施工程师为他们提供了一堆 CRD，他们必须逐个去挑选、测试和甄别，尤其是一些运维特征（Trait）可能存在功能冲突，不能同时与一个业务组件（Component）绑定，这都都要应用研发和运维人员自己去学习和测试，虽然可以通过文档来规范，但显然这样做并不优雅，这时 OAM App Engine（暂定名 RdurX）就出现了。

![OAM App Engine 所在位置](https://tva3.sinaimg.cn/large/ad5fbf65gy1gh3da963ouj20sj0at0ux.jpg)

OAM App Engine 的目标用户群体是应用开发者，是希望终端开发者用户可以感受到 OAM 提倡的各类应用管理理念带来的价值。相比于其他基于 K8s 的应用管理平台（如 [rio](https://github.com/rancher/rio) ），OAM App Engine 将至少具备如下三大核心价值。

1. 插件系统：App Engine 可以通过 OAM 具备快速纳管 operator 的能力，轻松扩展各种能力。
2. 用户体验：贴近开发者，一切设计以最终开发者使用体验至上，复杂的概念做抽象，用户熟悉的概念不隐藏。
3. 最佳实践：App Engine 将成为 OAM 实现的最佳实践。

![OAM 架构](https://tva1.sinaimg.cn/large/ad5fbf65gy1gh3cutnty0j227415w1kx.jpg)

OAM App Engine 由 CLI 命令行工具、 Dashboard UI 管理页面和一系列编排文件/DSL 组成，目前还处于功能设计与开发当中，预计在8月底会和用户见面。OAM App Engine 的开发者均来自 OAM 中国社区，来自不同的公司和组织，是真正的从社区中来，服务社区用户。

欢迎对 OAM 有兴趣的朋友加入，社区每双周都会进行视频例会，欢迎大家发表自己的见解或提出相关疑问。

![OAM 中国社区](https://tva1.sinaimg.cn/wap360/ad5fbf65gy1gh3cx41p0gj20nc0uqtfu.jpg)

## 结语

《云原生架构白皮书》的编写集合了阿里云众多技术专家，基于这些年阿里云海量的技术实践，对云原生这一当下十分火爆概念进行了十分深入的剖析，在分享知识和实践经验的同时还对云原生相关技术、架构设计和发展趋势等内容进行分析和描述，为那些对于云原生这一概念还十分陌生和迷茫的开发者/管理者提供了一份干货满满的参考资料。这里借用白皮书序言里的一句话：

>云计算的下一站，就是云原生；
>
>IT 架构的下一站，就是云原生架构 ;
>
> 希望所有的开发者、架构师和技术决策者们，共同定义、共同迎接云原生时代。

《云原生架构白皮书》下载链接 ： https://developer.aliyun.com/topic/download?id=721

## 参考

- [《云原生架构白皮书》](https://developer.aliyun.com/topic/download?id=721)
- 基 Kubernetes 与 OAM 构建统一、标准化的应用管理平台
- OAM App Engine CLI 设计文档【实现基准】
