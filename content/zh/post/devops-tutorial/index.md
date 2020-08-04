---
title: "Devops入门手册"
date: 2019-04-09T13:21:56+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wiaimt98j21qi15owk9.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "“DevOps”这个词是 “development” 和 “operations”这两个词的组合。它是一种促进开发和运维团队之间的协作，以自动化和可重复的方式更快地将代码部署到生产中的文化。"
tags: ["devops","翻译"]
categories: ["翻译"]
keywords: ["devops","翻译"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3ielgql8j21qi15owk9.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# DevOps 是什么？
“DevOps” 这个词是 ```development``` 和 ```operations``` 这两个词的组合。它是一种促进开发和运维团队之间的协作，以自动化和可重复的方式更快地将代码部署到生产中的**文化**。

DevOps 帮助团体提高软件和服务的交付速度。它使团队能够更好地为客户服务，并提高在市场中的竞争力。

简而言之， DevOps 可以定义为通过更好的沟通和协作，使开发和运维保持一致。

![what is devops](http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wbobryucj20db07dq2w.jpg)

本手册中，您将学到：

- [DevOps 是什么？](#devops-是什么)
- [为什么需要 DevOps ？](#为什么需要-devops)
- [DevOps 与传统运维有什么不同？](#devops-与传统运维有什么不同)
- [为什么使用 DevOps ？](#为什么使用-devops)
- [DevOps 的生命周期](#devops-的生命周期)
- [DevOps 的工作流](#devops-的工作流)
- [DevOps 与敏捷有什么不同？ DevOps VS Agile](devops-与敏捷有什么不同-devops-vs-agile)
- [DevOps 原则](#devops-原则)
- [谁可以做 DevOps 工程师？](#谁可以做-devops-工程师)
- [DevOps 工程师的角色、职责和技能](#devops-工程师的角色-职责和技能)
- [DevOps 工程师可以挣多少钱？](#devops-工程师可以挣多少钱)
- [DevOps 培训认证](#devops-培训认证)
- [DevOps 自动化工具](#devops-自动化工具)
- [DevOps 的未来是怎样的？](#devops-的未来是怎样的)
- [总结](#总结)

# 为什么需要 DevOps ？
- 在实行 DevOps 之前，开发和运维团队是完全孤立的。
- 测试和部署是设计在构建之后完成的独立活动。因此，他们比实际构建周期消耗更多时间。
- 在不使用 DevOps 的情况下，团队成员将大量时间花在测试，部署和设计上，而不是构建项目。
- 手动部署代码会导致生产中出现人为错误。
- 开发和运维团队都有各自的时间表，时间的不同步导致生产交付进一步延误。

提高软件交付率是业务方最迫切的需求。根据 Forrester Consulting Study 统计，只有17％的团队可以足够快地交付软件。更是证明了这一痛点。

# DevOps 与传统运维有什么不同？
让我们将传统软件瀑布开发模型与 DevOps 进行比较，以了解 DevOps 带来的变化。

我们假设有一个应用程序计划在2周内上线，代码完成80％。该应用程序是一个新的发布，从购买服务器开始

|  瀑布式开发 | DevOps |
| --- | --- |
| 订购新服务器后，开发团队需要进行测试。运维团队根据需求文档开始部署基础设施。 | 订购新服务器后，开发和运维团队根据需求文档共同调试部署新服务器。这样开发人员可以更好地了解服务器的基础架构。|
| 关于故障转移，冗余策略，数据中心位置和存储要求的规划存在偏差，因为开发人员对应用程序有深入了解，但他们无法提供任何协助。 |  由于开发人员的加入，有关故障转移，冗余策略，灾难恢复，数据中心位置和存储要求的规划非常准确。 |
| 运维团队对开发团队的进展一无所知。只能根据运维团队理解制定监控计划。 | 在 DevOps 中，运维团队完全了解开发人员的进展。通过互动，共同制定满足运维和业务需求的监控计划。他们还使用应用程序性能监视（APM）工具以优化应用。 |
| 在上线之前，压力测试使应用程序崩溃。发布延迟了。 | 在上线之前，压力测试使应用程序有点慢。开发团队迅速解决了瓶颈问题。该应用程序按时发布。 |

# 为什么使用 DevOps ？
DevOps 允许敏捷开发团队实施持续集成和持续交付。这有助于他们更快地将产品推向市场。

其他重要原因是：

1. **可预测性：** DevOps 可以显着降低新版本的故障率。
2. **自愈性：** 可以随时将应用回滚到较早的版本。
3. **可维护性：** 在新版本崩溃或当前系统不可用的情况下，可以毫不费力地进行恢复。
4. **上线时间：** DevOps 通过简化软件交付流程将上线时间缩短至50％。对于互联网和移动应用时间更短。
5. **更高的质量：** DevOps 帮助团队提高应用程序开发的质量。
6. **降低风险：** DevOps 在软件交付的生命周期中包含安全检查。它有助于减少整个软件生命周期中的安全风险。
7. **弹性：** 软件系统的运行状态更稳定，更安全，更改是可审计的。
8. **成本效益：** DevOps 在软件开发过程中提供了成本效益，这始终是互联网公司管理层所期望的。
9. **将大的代码库分成小块：** DevOps 是基于敏捷编程方法的。因此，它允许将大的代码库分解为更小且易于管理的块。

### 什么时候使用 DevOps ？
DevOps 应该用于大型分布式应用程序，例如电子商务站点或托管在云平台上的应用程序。
### 什么时候不使用 DevOps？
它不应该用于关键任务应用程序，如银行，电力设施和其他敏感数据站点。此类应用程序需要对生产环境进行严格的访问控制，详细的变更管理策略，完善的数据中心访问控制策略。

# DevOps 的生命周期
![devops Lifecycle](http://wx3.sinaimg.cn/large/ad5fbf65gy1g1wekkedpcj20k509mjrp.jpg)

DevOps 是开发和运维之间的深度集成。在不了解 DevOps 生命周期的情况下，是无法真正理解 DevOps 的。

以下是有关 DevOps生命周期的简要信息：

1. 开发

    在此阶段，整个开发过程分为小的开发周期。这有利于 DevOps 团队加快软件开发和交付过程。

2. 测试

    QA 团队使用 ```Selenium``` 等自动化测试工具来识别和修复新代码中的错误。

3. 集成

    在此阶段，新功能与主分支代码集成，并进行测试。只有持续集成和测试才能实现持续交付。

4. 部署

    在此阶段，部署过程持续进行。它的执行方式是任何时候在代码中进行的任何更改都不应影响高流量网站的运行。

5. 监测

    在此阶段，运维团队将负责处理不合适的系统行为或生产中发现的错误。

# DevOps 的工作流

![ DevOps Work Flow ](http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wewdq1elj20g009fa9y.jpg)

工作流允许排列和分离用户最需要的任务。它还能够在配置任务时反应其最理想过程。

# DevOps 与敏捷有什么不同？ DevOps VS Agile
这是一个典型的IT流程

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wfmrcbafj20nq05wdg0.jpg)

敏捷解决了客户和开发人员沟通中的问题

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wfn81bchj20no05q3ys.jpg)

DevOps 解决了开发人员运维人员沟通中的问题

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g1wfnk7fi3j20nt05vt90.jpg)

| 敏捷 | DevOps |
| --- | --- |
| 强调打破开发人员和管理层之间的障碍。 | DevOps 是关于软件开发和运维团队的。 |
| 解决客户需求与开发团队之间的距离。 | 解决开发和运维团队之间的距离。 |
| 重点关注功能和非功能准备。 | 它侧重于运维和业务准备。 |
| 敏捷开发主要涉及公司对开发方式的思考。 | DevOps 强调以最可靠和最安全的方式部署软件，而这些方式并不总是最快的。 |
| 敏捷开发非常注重培训所有团队成员，使他们拥有各种相同的技能。因此，当出现问题时，任何团队成员都可以在没有团队领导的情况下从别的成员那里获得帮助。 | DevOps 在开发和运维团队之间传播技能，并保持一致的沟通。|
| 敏捷开发管理 “sprint” ，意味着时间更短（不到一个月），并且在此期间将产生和发布多个功能。 | DevOps 努力争取主要版本的稳定可靠，而不是更小和更频繁的发布版本。 |

# DevOps 原则
这里有六个在采用 DevOps 时必不可少的原则：

1. **以客户为中心：** DevOps 团队必须以客户为中心，因为是他们不断向我的产品和服务投资。
2. **端到端的责任：** DevOps 团队需要在产品的整个生命周期提供性能支持。这提高了产品的水平和质量。
3. **持续改进：** DevOps 文化专注于持续改进，以尽量减少浪费。它不断加快产品或服务改进的速度。
4. **自动化一切：** 自动化是 DevOps 流程的重要原则。这不仅适用于软件开发，同时也适用于整个基础架构环境。
5. **作为一个团队工作：** 在 DevOps 文化角色中，设计人员，开发人员和测试人员已经定义。他们所需要做的就是作为一个团队完成合作。
6. **监控和测试所有内容：** DevOps 团队拥有强大的监控和测试程序是非常重要的。

# 谁可以做 DevOps 工程师？
DevOps 工程师是一名IT专业人员，他与软件开发人员，系统运维人员和其他IT人员一起管理代码发布。DevOps 应具备与开发，测试和运维团队进行沟通和协作的硬技能和软技能。

DevOps 方法需要对代码版本进行频繁的增量更改，这意味着频繁的部署和测试方案。尽管 DevOps 工程师需要偶尔从头开始编码，但重要的是他们应该具备软件开发语言的基础知识。

DevOps 工程师将与开发团队的工作人员一起解决连接代码的元素（如库或软件开发工具包）所需的编码和脚本。

# DevOps 工程师的角色、职责和技能

DevOps 工程师负责软件应用程序平台的生产和持续维护。

以下是 DevOps 工程师的一些角色，职责和技能：

- 能够跨平台和应用程序域执行系统故障排除和问题解决。
- 通过开放的，标准的平台有效管理项目。
- 提高项目可见性和可追溯性。
- 通过协作提高开发质量并降低开发成本。
- 分析、设计和评估自动化脚本和系统。
- 通过使用最佳的云安全解决方案确保系统的安全。
- DevOps 工程师应该具备问题解决者和快速学习者的软技能。

# DevOps 工程师可以挣多少钱？

DevOps 是最热门的IT专业之一。这就是为什么那里都有很多机会的原因。因此，即使是初级DevOps工程师的薪酬水平也相当高。在美国，初级DevOps工程师的平均年薪为78,696美元。

# DevOps 培训认证

DevOps 培训认证可以帮助任何渴望成为 DevOps 工程师职业的人。认证可从 Amazon web services 、 Red Hat 、 Microsoft Academy 、 DevOps Institute 获得。

[AWS Certified DevOps Engineer](https://aws.amazon.com/cn/certification/certified-devops-engineer-professional/)

此 DevOps 工程师证书将测试您如何使用最常见的 DevOps 模式在 AWS 上开发，部署和维护应用程序。它还会评估 DevOps 方法的核心原则。

该认证有两个必要条件：认证费用为300美元，持续时间为170分钟。

[Red Hat Certification](https://www.redhat.com/en/services/training-and-certification)

红帽为 DevOps 专业人士提供不同级别的认证，如下所示:

- Red Hat Certificate of Expertise in Platform-as-a-Service
- Red Hat Certificate of Expertise in Containerized Application Development
- Red Hat Certificate of Expertise in Ansible Automation
- Red Hat Certificate of Expertise in Configuration Management
- Red Hat Certificate of Expertise in Container Administration

[Devops Institute](http://devopsinstitute.com/)

Devops Institute是围绕新兴 DevOps 实践的全球学习社区。该组织正在为 DevOps 能力资格设置质量标准。Devops Institute目前提供三个课程和认证。

公司提供的认证课程有：

- DevOps Foundation
- DevOps Foundation Certified
- Certified Agile Service Manager
- Certified Agile Process Owner
- DevOps Test Engineering
- Continuous Delivery Architecture
- DevOps Leader
- DevSecOps Engineering

# DevOps 自动化工具

所有测试流程自动化并对其进行配置以实现至关重要的速度和灵活性。此过程称为 DevOps 自动化。

维护庞大的IT基础架构的大型 DevOps 团队面临的困难可以简要分为六个不同的类别。

1. 基础设施自动化
2. 配置管理
3. 部署自动化
4. 性能管理
5. 日志管理
6. 监测

让我们看看每个类别中的工具以及它们如何解决痛点：

### 基础设施自动化

亚马逊网络服务（AWS）：作为云服务，您无需建立实际的数据中心。此外，它们易于按需扩展。没有前期硬件成本。它可以配置为自动根据流量配置更多服务器。

### 配置管理

Chef：它是一个有用的 DevOps 工具，用于提升速度，规模和一致性。它可用于简化复杂任务并执行配置管理。使用此工具，DevOps 团队可以避免在一万台服务器上进行更改。相反，只需要在一个地方进行更改，这些更改会自动反映在其他服务器中。

### 部署自动化

Jenkins：该工具有助于持续集成和测试。通过在部署构建后快速查找问题，更​​轻松地集成项目更改。

### 日志管理

Splunk：可以解决在一个地方聚合，存储和分析所有日志的问题的工具。

### 性能管理

App Dynamic：它是一个 DevOps 工具，提供实时性能监控。此工具收集的数据可帮助开发人员在发生问题时进行调试。

### 监控

Nagios：在基础架构和相关服务出现故障时通知相关人员也很重要。Nagios 就是这样一种工具，它可以帮助 DevOps 团队找到并纠正问题。

# DevOps 的未来是怎样的？

- 团队将代码部署周期转换为数周和数月，而不是数年。
- 很快就会看到，DevOps 工程师可以比企业中的任何其他人更多地接近和管理终端用户。
- DevOps 正在成为IT人员的重要技能。例如，Linux 招聘进行的一项调查发现，25％的受访者的求职者寻求 DevOps 工作。
- DevOps 和持续交付将继续存在。因为公司需要发展，他们别无选择，只能改变。然而，DevOps 概念的主流化则需要5到10年。

# 总结

- DevOps 是一种促进开发和运维团队之间的协作，以自动化和可重复的方式更快地将代码部署到生产中的**文化**。
- 在 DevOps 出现之前运维和开发团队完全独立。
- 手动部署代码会导致生产中出现人为错误。
- 在旧的软件开发流程中，运维团队不了解开发团队的进度。因此，运维团队只能根据他们自己的理解制定了基础设施的购买和监控计划。
- 在 DevOps 流程中，运维团队充分了解开发人员的进度。采购和监控计划准确无误。
- DevOps 提供可维护性，可预测性，更高质量的代码和更准确的上线时间。
- 敏捷流程侧重于功能和非功能准备，而 DevOps 则侧重于IT基础架构方面。
- DevOps 生命周期包括开发，测试，集成，部署和监控。
- DevOps 工程师将与开发团队工作人员合作，以解决编码和脚本编写需求。
- DevOps 工程师应该具备问题解决者的软技能，并且是一个快速学习者。
- DevOps 认证可从 Amazon web services，Red Hat，Microsoft Academy，DevOps Institute 获得
- DevOps 可帮助团队将代码部署周期转换为数周和数月，而不是数年。


**原文链接** https://www.guru99.com/devops-tutorial.html