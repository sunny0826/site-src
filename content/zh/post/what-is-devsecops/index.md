---
title: "什么是 DevSecOps ?"
date: 2019-05-03T14:37:14+08:00
draft: false
type: blog
banner: "https://ws2.sinaimg.cn/large/ad5fbf65gy1g2o4u644j7j21qi15owio.jpg"
author: "Red Hat"
authorlink: "https://www.redhat.com"
translator: "郭旭东"
translatorlink: "https://github.com/sunny0826"
originallink: "https://www.redhat.com/en/topics/devops/what-is-devsecops"
summary: "DevOps 不仅仅是开发和运营团队。如果您想要充分发挥出 DevOps 方法的敏捷性和响应力，则必须在应用的整个生命周期内同时兼顾 IT 安全性。"
tags: ["devops"]
categories: ["devops"]
keywords: ["devops"]
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3jds2cxfj21qi15owio.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
DevOps 不仅仅是开发和运营团队。如果您想要充分发挥出 DevOps 方法的敏捷性和响应力，则必须在应用的整个生命周期内同时兼顾 [IT 安全性](https://www.redhat.com/zh/topics/security)。

为什么？以往，安全性会在开发的最后阶段由特定的团队来负责实现。当开发周期长达数月、甚至数年时，上述做法不存在任何问题；但是，这种做法现在已经行不通了。有效的 DevOps 可顺利推进快速频繁的开发周期（有时全程只有数周或数天），但是过时的安全措施会对此造成负面影响，即使对于最高效的 DevOps 计划也是如此。

![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g2o4h6asbfj20b9077q3d.jpg)

现在，安全防护在 DevOps 协作框架中属于共同责任，而且需要在整个周期中[整合](https://www.redhat.com/zh/challenges/integration)相应的安全功能。这是一个非常重要的理念。它还使得“DevSecOps”一词应运而生，以用于强调必须为 DevOps 计划打下扎实的安全基础。

![image](https://ws4.sinaimg.cn/large/ad5fbf65gy1g2o4i7spd6j20b908k0t7.jpg)

DevSecOps 意味着，从一开始就要考虑应用和基础架构的安全性；同时还要让某些安全网关实现[自动化](https://www.redhat.com/zh/topics/automation)，以防止 DevOps 工作流程变慢。选择正确的工具来持续确保安全性有助于实现安全目标。但是，有效的 DevOps 安全防护需要的不仅是新工具。它建立在 DevOps 文化变革的基础上，以便尽早集成安全团队的工作。

## DevOps 安全性为内置特性

无论您将其称为“DevOps”还是“DevSecOps”，最好始终能在应用的整个生命周期内确保安全性。DevSecOps 关乎内置安全性，而不是应用和数据层面的安全性。如果将安全性问题留到开发流程的最后环节再加以考虑，那么采用 DevOps 方案的组织会发现自己的开发周期又变长了，而这是他们从一开始就想要避免的情况。

在某种程度上，DevSecOps 强调，在 DevOps 计划刚启动时就要邀请安全团队来确保信息的安全性，并制定自动安全防护计划。它还强调，要帮助开发人员从代码层面确保安全性；在这个过程中，安全团队需要针对已知的威胁分享可见性信息、提供反馈并进行智能分析。这可能还包括为开发人员提供新的安全培训，因为 DevSecOps 并非始终着眼于较为传统的应用开发模式。

那么，怎样才算是真正地实现了内置安全性？对于新手而言，优质的 DevSecOps 策略应能确定风险承受能力并进行风险/收益分析。在一个给定的应用中，需要配备多少个安全控制功能？对于不同的应用，上市速度又有多重要？自动执行重复任务是 DevSecOps 的关键所在，因为在管道中运行手动安全检查可能会非常耗时。

## DevOps 安全性可自动实现

企业应该：确保采用时间短、频率高的开发周期；采取安全措施，以最大限度地缩短运营中断时间；采用创新技术，如[容器](https://www.redhat.com/zh/topics/containers)和[微服务](https://www.redhat.com/zh/topics/microservices)；同时，还要促使常见的孤立团队加强合作 — 这对所有企业来说都是一项艰巨的任务。上述所有举措都与人有关，而且企业内部需要协同合作；但是，[自动化](https://www.redhat.com/zh/topics/automation/whats-it-automation)才是有助于在 DevSecOps 框架中实现这些人员变化的关键所在。

![image](https://ws2.sinaimg.cn/large/ad5fbf65gy1g2o4kwhrauj20f9065aao.jpg)

那么，企业应该在哪些方面实现自动化？具体又该怎么做呢？红帽提供了相应的[书面指南](https://itrevolution.com/book/devops-and-audit/)来帮助解答上述问题。企业应该退后一步，并着眼于整个开发和运营环境。其中涉及：源控制存储库；容器注册表；持续集成和持续部署 (CI/CD) 管道；应用编程接口 (API) 的管理、编排和发布自动化；以及运营管理和监控。

全新的自动化技术已帮助企业提高了开发实践的敏捷性，还在推动采用新的安全措施方面起到了重要作用。但是，自动化并不是近年来 IT 领域发生的唯一变化。现在，对于大多数 DevOps 计划而言，容器和微服务等[云原生技术](https://www.redhat.com/zh/challenges/cloud-infrastructure)也是一个非常重要的组成部分。所以，企业必须调整 DevOps 安全措施，以适应这些技术。

## DevOps 安全性适用于容器和微服务

可通过容器实现的规模扩展和基础架构动态性提升改变了许多组织开展业务的方式。因此，DevOps 安全性实践必须适应新环境并遵循[特定于容器的安全准则](https://csrc.nist.gov/publications/detail/nistir/8176/final)。云原生技术不适合用来落实静态安全策略和检查清单。相反，组织必须在应用和基础架构生命周期的每个阶段确保持续安全并整合相应的安全功能。

DevSecOps 意味着，要在应用开发的整个过程中确保安全性。要实现与管道的这种集成需要秉持一种全新的思维方式，就像使用新工具一样。考虑到这一点，DevOps 团队应该实现安全防护自动化，以保护整体环境和数据；同时实现持续集成/持续交付流程——可能还要确保容器中的微服务的安全性。

| 环境和数据安全性： | CI/CD 流程安全性： |
| --- | --- |
| 实现环境的标准化和自动化。<br>每项服务都应具有最小的权限，以最大限度地减少未经授权的连接和访问。 | 集成适用于容器的安全性扫描程序。<br>应在向注册表添加容器的过程中实现这一点。 |
| 实现用户身份和访问控制功能的集中化。<br>由于要在多个点发起身份验证，因此严格的访问控制和集中式身份验证机制对于确保微服务安全性而言至关重要。 | 自动在 CI 过程中完成安全性测试。<br>其中包括在构建过程中运行安全性静态分析工具；而且在构建管道中提取任何预构建容器映像时，都要进行扫描，以检查是否存在已知的安全漏洞。 |
| 使运行微服务的容器相互隔离并与网络隔离。<br>这包括传输中和静止的数据，因为获取这两类数据是攻击者的高价值目标。 | 在验收测试流程中加入针对安全性功能的自动化测试。<br>自动执行输入验证测试，并针对验证操作实现身份验证和授权功能的自动化。 |
| 加密应用与服务间的数据。<br>具有集成式安全功能的容器编排平台有助于最大限度地降低发生未经授权访问的可能性。 | 自动执行安全性更新，<br>例如针对已知漏洞打修补。通过 DevOps 实现这一点。这样，在创建记录在案的可跟踪更改日志时，管理员便无需登录生产系统。 |
| 引入安全的 API 网关。<br>安全的 API 可提高授权和路由的可见性。通过减少公开的 API，组织可以减小攻击面。 | 实现系统和服务配置管理功能的自动化。<br>这样可以确保遵守安全策略，避免出现人为错误。审核和补救操作也应实现自动化。 |