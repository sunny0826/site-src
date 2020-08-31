---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "OpenKruise：Kubernetes 核心控制器 Plus"
subtitle: ""
summary: "Kruise 是 OpenKruise 中的核心项目之一，Kruise 是 cruise的谐音，字面意义巡航，豪华游艇（'K' for Kubernetes）。寓意 Kubernetes 上应用的自动巡航，如果把原生 Kubernetes 资源 Deployment 或 StatefulSet 比作小船，那 Kruise 确实就是豪华游艇了。"
authors: ["guoxudong"]
tags: ["Openkruise"]
categories: ["Openkruise"]
date: 2020-08-27T11:51:25+08:00
lastmod: 2020-08-27T11:51:25+08:00
draft: false
type: blog
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65gy1gi5c4x0ekpj20j608qtam.jpg"
---
## 前言

在去年的 KubeCon 上海 2019，我有幸在现场见证了 [OpenKruise](https://github.com/openkruise/kruise) 项目的开源，当时在台下的我非常兴奋，因为找到了一套让我的 Kubernetes 集群的核心资源 Pod 升级和发布更自动更简单的方案。如今一年多过去了，前不久 Openkruise 刚发布了最新的 `v0.6.0` 版本，目前已经有很多企业在生产环境应用了 OpenKruise，借助 OpenKruise 提供的自动化能力，大大提升了部署升级效率与质量。

## Kruise

Kruise 是 OpenKruise 中的核心项目之一，Kruise 是 cruise的谐音，字面意义巡航，豪华游艇（'K' for Kubernetes）。寓意 Kubernetes 上应用的自动巡航，如果把原生 Kubernetes 资源 Deployment 或 StatefulSet 比作小船，那 Kruise 确实就是豪华游艇了。

<center>
  <img src="https://tvax4.sinaimg.cn/large/ad5fbf65gy1gi6knwioawj20ge0dp0ty.jpg">
</center>
<br>

Kruise 提供一套在 Kubernetes 核心控制器之外的扩展 workload，但我更愿意称之为核心控制器 plus。因为目前 Kruise 提供的一系列 workload，更像是核心控制器资源（Deployment、 StatefulSet、Job 和 DaemonSet）的增强版。比如：Advanced StatefulSet 的介绍里就写着是 StatefulSet 的增强版本，在原生 StatefulSet 的基础上增加了诸多功能。下面笔者就来简单介绍一下 Kruise 目前提供的所有 workload 控制器，由于篇幅限制每个 workload 的详细介绍及使用示例将在后续文章中做单独介绍，本篇只是简单介绍各个 workload 可能的使用场景及用途。

### CloneSet

记得 Kruise 最早还没有 CloneSet 这个 Workload，所以我之前还是将其归类为有状态应用的控制器增强（最早放出来的是 Advanced StatefulSet），但是自 `v0.4.0` 版本推出之后，CloneSet 一跃成为了最受欢迎，使用率最高的 Kruise 控制器之一，同时也补齐了 Kruise 没有无状态应用控制器这个短板。

<center>
  <img src="https://tvax4.sinaimg.cn/large/ad5fbf65gy1gi6knhrsvlj20d00ae0sy.jpg">
</center>


CloneSet 其实就是 Deployment plus，其提供更加高效、稳定可控的无状态应用管理和部署能力，支持优雅原地升级、指定删除、发布顺序、并行/灰度发布等丰富的策略，可以满足更多样化的应用场景。CloneSet 也是目前使用最广的一类 Kruise 控制器，是 OAM 官方 Kubernetes 套件 [crossplane/oam-kubernetes-runtime](https://github.com/crossplane/oam-kubernetes-runtime) 支持的 Workload 之一。

CloneSet 的增强分为两类：**扩缩容功能增强**和**升级功能增强**。扩缩容功能增强包括：支持 PVC 模板、指定 Pod 缩容等；升级功能增强包括：原地升级、分批灰度、控制最大不可用数量、控制最大弹性数量、按照不同测控顺序升级、发布暂停等功能。

**使用场景**：代替原生的 Deployment，将升级过程控制的更加精细、自动和优雅，利用原地升级可以大大降低由于原来重建升级导致的网络、存储等方面的损耗，同时还能加快升级速度。

### Advanced StatefulSet

Advanced StatefulSet 是 Kruise 最早发布的控制器之一，是原生 StatefulSet 的增强版本，默认行为与原生完全一致，在此之外提供了原地升级、并行发布（最大不可用）、发布暂停等功能。因为是在原生基础上进行开发的，所以只需将原生 
StatefulSet 的 `apiVersion` 由 `apps/v1` 改为 `apps.kruise.io/v1alpha1` 即可完成迁移，非常直接。

Advanced StatefulSet 为 StatefulSet 提供了和 Deployment 一样的 `MaxUnavailable` 策略，可以并行发布 Pod，而不再像原生 StatefulSet 一样 one by one 的串行发布；支持原地升级策略，无需重建 Pod，即可原地升级镜像，同时也提供了优雅原地升级的策略，控制器在升级前将 Pod status 改为 not-ready，等待指定时间再升级镜像，这就为将 Pod 从 endpoints 端点列表中去除留出了充足的时间（CloneSet 也支持该策略）；还支持指定升级顺序、发布暂停等策略。

**使用场景**：代替原生 StatefulSet，有效利用原地升级、并行发布等功能，提升有状态应用的发布速度，为其配置合适的升级策略，提升发布速度。

### SidecarSet

<center>
  <img src="https://tva1.sinaimg.cn/large/ad5fbf65gy1gi6kn5e9t8j20cu0b5gm2.jpg">
</center>

SidecarSet 的作用就是对 Sidecar 容器做统一管理，支持在一个单独的 CR 中定义 Sidecar 容器，向将满足条件的 Pod 中注入指定的 Sidecar 容器，同时 SidecarSet 也支持 Sidecar 容器原地升级。这样就可以将业务容器和 Sidecar 容器的管理分离，更有利于分工合作，不同的团队只需关心和自己业务有关的容器，免去了大量的沟通成本。

**使用场景**：将所有 Sidecar 进行统一管理，一个 CR 管理一类 Sidecar，真正做到业务容器和 Sidecar 容器管理分离，权责清晰。

### UnitedDeployment

<center>
  <img src="https://tva3.sinaimg.cn/large/ad5fbf65gy1gi6kmw2uy3j20b707m74o.jpg">
</center>
<br>

UnitedDeployment 为由多个区域组成的集群中实现高可用部署提供了一种新模式，在一个 Kubernetes 集群中可能存在不同的 node 类型，比如多个可用区、或不同的节点技术（比如 Virtual kueblet）等，这些不同类型的 node 上都有 label/taint 标识。UnitedDeployment 控制器可以提供一个模板来定义应用，每个 UnitedDeployment 下每个区域的 workload 被称为 `subset`，有一个期望的 `replicas` Pod 数量。目前 subset 支持使用 `StatefulSet` 和 `Advanced StatefulSet`。通过 UnitedDeployment 可以同时管理位于多个可用区的同一应用。

**使用场景**：用于管理跨可用区的有状态应用，做到管理更精细，过程更自动。

### BroadcastJob

BroadcastJob 更像 Job 和 DaemonSet 的组合，BroadcastJob 控制器将 Pod 分发到集群中每个 node 上，类似于 DaemonSet， 但是 BroadcastJob 管理的 Pod 并不是长期运行的 daemon 服务，而是类似于 Job 的任务类型 Pod。最终在每个 node 上的 Pod 都执行完成退出后，BroadcastJob 和这些 Pod 并不会占用集群资源。 这个控制器非常有利于做升级基础软件、巡检等过一段时间需要在整个集群中跑一次的工作。此外，BroadcastJob 还可以维持每个 node 跑成功一个 Pod 任务。如果采取这种模式，当后续集群中新增 node 时 BroadcastJob 也会分发 Pod 任务上去执行。

**使用场景**：用于管理升级基础软件、巡检等需要在集群中所有节点或指定类型节点执行的单次任务。

### Advanced DaemonSet

Advanced DaemonSet 是 `v0.6.0` 新增的控制器，是原生 DaemonSet 的增强版本，默认行为与原生一致，在此之外提供了灰度分批、按 Node label 选择、暂停、热升级等发布策略。迁移方式同 StatefulSet 类似，将 DaemonSet 的 `apiVersion` 由 `apps/v1` 改为 `apps.kruise.io/v1alpha1` 即可完成迁移。

Advanced DaemonSet 也有 `RollingUpdateDaemonSet` 的增强策略，同时也提供了多种升级方式如：按照 Selector 标签选择升级、分批灰度升级、热升级和暂停升级。

**使用场景**：代替原生 DaemonSet，利用原地升级、灰度升级、选择性升级以及热升级等增强特性，更好的维护和管理 DaemonSet 资源。

## 结语

Kruise 项目源自于阿里巴巴多年的大规模应用部署、发布与管理的最佳实践，以 **automate everything on Kubernetes** 为目标。经过了一年多的发展，OpenKruise 的大部分功能都经过了各种生产环境的洗礼，应用在越来越多的 Kubernetes 系统中，代替原生核心控制器，使应用的升级和管理更加的方便高效。

## 参考

[OpenKruise - openkruise.io](https://openkruise.io)