---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "去指挥你的舰队吧！体验使用 Fleet 批量管理 K8S 集群"
subtitle: "体验使用 Fleet 批量管理 K8S 集群"
summary: "体验 Fleet 是怎么管理海量 Kubernetes 集群的。"
authors: ["guoxudong"]
tags: ["rancher","kubernetes"]
categories: ["kubernetes"]
date: 2020-04-23T14:03:53+08:00
lastmod: 2020-04-23T14:03:53+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3o9u1zfgj20e8086t99.jpg"
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

{{% alert note %}}
2020年4月3日，Rancher Labs 宣布推出全新开源项目 Fleet，致力于为用户提供海量 Kubernetes 集群的集中管理体验。
{{% /alert %}}


最早听说到这个消息时，我还是很疑惑的，Fleet 不是 CoreOS 早已经停止维护的一个项目吗？怎么又和 Rancher Labs 扯上了关系？

**“为用户提供海量 Kubernetes 集群的集中管理体验”**这句话是否言过其实：

- “海量”这个量到底有多大？
- 又有多少公司或团队有管理海量的 Kubernetes 集群的需求？
- 又是怎么一个**集中管理**法？

带着这些疑问，我仔细了解了一下 Fleet 这个开源项目。

## Fleet

首先，这里的 Fleet 是一个新项目，起这个名字应该算是一种致敬，经过了解后我个人觉得这个名字起的还是挺贴切的，比一大波 KubeXXX 有创意多了。

>“我一直是它的忠实粉丝，将这一项目命名为 Fleet 也包含了我的私心。”Darren Shepherd 解释道：“所以我希望重新使用 Fleet 这一名字，这是对这个非常出色的容器领域早期项目的致敬。同时，对于推动 Kubernetes 集群管理的演进，我们感到十分兴奋及万分期待。”

> --- 摘自 RancherLabs 官方微信公众号《Rancher开源Fleet：业界首个海量K8S集群管理项目》

顾名思义 Fleet 是“舰队”的意思，而 Kubernetes 在希腊语意为 “舵手”。从名称上看，Fleet 的目标就是管理或是指挥众多 Kubernetes 集群。而在了解这个项目时，我发现了这个项目和 Rancher Labs 另一个受欢迎项目 [k3s](https://k3s.io/) 有个千丝万缕的联系，甚至在我看来 Fleet 可能就是就是为了管理众多 k3s 集群而生的，是 Rancher Labs 布局边缘计算和 IoT 领域的重要组成部分。

k3s 是一款轻量级的 Kubernetes 集群，主要面向边缘计算和 IOT 领域，相比原生 Kubernetes，k3s 体量更轻、部署简单且快速，同时还具有完整的 Kubernetes 体验。可以说只要是 Linux 系统（配合周边工具甚至可以运行在 Mac 和 Windows 系统），无论是树莓派、各种开发板还是 PC 机，都可以独立运行起 k3s，**这也为运行海量 Kubernetes 集群**提供了可能。以汽车为例，我们可以为每一辆汽车都部署一个 k3s 集群，所有汽车相关的软件（导航、广播甚至是无人驾驶程序）都部署在 k3s 集群中，每次这些软件发布新版本，只需使用 Fleet 进行批量操作该种车型的所有 k3s 集群即可，无需将车开回 4S 店进行手动更新。

{{% alert note %}}
联系美国空军是 Kubernetes 与 Istio 项目的重要用户，这种实践可能早就开始了。
{{% /alert %}}

解释了海量 Kubernetes 集群的疑问，下面就从 Fleet 的架构入手，讲讲如何**集中管理**。

![](https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3o40xe41j20qx0ljdm7.jpg)

Fleet 包含`Manager`和`agent`，`Manager`所在集群作为控制平面管理所有`agent`集群，同时 Fleet 根据 Kubernetes 部署 Pod 的模型，定义了一个 Bundles 对象，并且提供了一种内置机制，可以使用诸如`Helm`和`Kustomize`等行业标准工具为每个目标集群定制 Bundles，在我看来这种模式以及`bundle.yaml`的写法都和`Kustomize`很像(套娃行为？)...一旦用户在集群之间部署了 Bundles，Fleet 就会主动监视资源是否已就绪，以及是否被更改过。总的来说就是通过部署 Bundles，就可以将部署内容批量分发到所有目标集群，从而达到**集中管理**的目的。

## 尝鲜体验

说那么多其实意义不大，好不好用，只有试过才知道。这里使用的 Fleet 版本为`v0.2.0`，是目前的最新版本。

**下载 CLI 工具**

首先需要下载`fleet`的 CLI 工具，这里的体验和 k3s 类似，都是直接`curl` GitHub 上的安装脚本并执行：

```bash
$ curl -sfL https://raw.githubusercontent.com/rancher/fleet/master/install.sh | sh -
```

**部署控制平面**

使用 CLI 工具将`Fleet Manager`部署到 Kubernetes 集群上：

```bash
# Kubeconfig should point to Manager cluster
$ fleet install manager | kubectl apply -f -
```

**生成 Cluster group token**

到这控制平面就部署好了，接下来部署`agent`目标集群。这里生成的其实是一个 yaml 文件，内容包含 fleet 需要的 RBAC 权限和 fleet-agent 的 Deployment：

```bash
# Kubeconfig should point to Manager cluster
$ fleet install agent-token > token
```

**目标集群注册**

将需要纳管的目标集群加入到 fleet 中，**注意**：这里需要将 kubeconfig 切换到目标集群，也就是需要部署`agent`的集，每个需要注册的集群都要部署`agent`：

```bash
# Kubeconfig should point to AGENT cluster
$ kubectl apply -f token
```

**部署 bundles**

这里就是向多个集群同时部署 bundles，使用方法也和`Kustomize`类似（`example` 目录是 fleet 官方仓库中的示例目录）：

```bash
# Kubeconfig should point to Manager cluster
$ fleet apply ./examples/helm-kustomize
```

**查看状态**

现在就可以查看所有集群 bundles 的状态了，这里可以看到 bundles 在多个集群都部署成功了（这里是我起的两个 k3s 集群做的测试）：

```bash
$ kubectl get fleet
NAME                                   CLUSTER-COUNT   BUNDLES-READY   BUNDLES-DESIRED   STATUS
clustergroup.fleet.cattle.io/default   2               3               4                 Modified: 1 (helm-kustomize )

NAME                                    CLUSTERS-READY   CLUSTERS-DESIRED   STATUS
bundle.fleet.cattle.io/fleet-agent      2                2
bundle.fleet.cattle.io/helm-kustomize   1                2                  Modified: 1 (default-default-group/cluster-5a186072-acbd-4f54-8f22-fb1651ce902f )
```

## 总结

总的来说，Fleet 的架构简洁且十分轻量，部署方式简单，使用`YAML`、`Helm`、`Kustomez`都可以进行资源的描述和配置，甚至可以使用`Helm`+`Kustomeze`的模式，部署体验不错。

但遗憾的是，目前 Fleet 还处于项目早期，实践也仅限于尝鲜体验，并不能用于生产环境，项目 README 中还专门提到了**目前 Fleet 仅适用于 10 个集群以下的小规模部署**。目前文档不足且项目维护人员并不积极，文档勘误的 [RP](https://github.com/rancher/fleet/pull/32) 和相关 ISSUE 也没有得到相关的反馈。项目是做到了业界首个，但是要真正生产可用甚至做到业界第一还有很长的一段路要走。

## 参考

* [Fleet - Github](https://github.com/rancher/fleet)
* [Rancher开源Fleet：业界首个海量K8S集群管理项目 - RancherLabs](https://mp.weixin.qq.com/s/byErGqVBtm4kdv58OZFt_w)