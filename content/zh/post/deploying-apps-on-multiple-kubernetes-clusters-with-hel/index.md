---
title: "使用 Helm 在多集群部署应用"
date: 2019-07-14T14:16:56+08:00
draft: false
type: blog
banner: "https://wx2.sinaimg.cn/large/ad5fbf65gy1g51gkx94erj21qy15owly.jpg"
author: "Smaine Kahlouch"
authorlink: "https://medium.com/@smainklh"
translator: "郭旭东"
translatorlink: "https://github.com/sunny0826"
originallink: "https://medium.com/dailymotion/deploying-apps-on-multiple-kubernetes-clusters-with-helm-19ee2b06179e"
summary: "本文将重点介绍我们如何在全球多个 Kubernetes 集群上部署我们的应用程序。为了将应用一次部署到多个 Kubernetes 集群，我们使用了 Helm ，并将所有 chart 存储在一个 git 仓库中。"
tags: ["翻译","kubernetes"]
categories: ["翻译"]
keywords: ["翻译","kubernetes"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3idyfogjj21qy15owly.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
> [Dailymotion](https://www.dailymotion.com/) 在生产环境使用 Kubernetes 已经3年了，但是也面临着多集群部署应用的挑战，这也是在过去的几年中我一直努力优化工具和改进工作流的原因。

## 前言

本文将重点介绍我们如何在全球多个 Kubernetes 集群上部署我们的应用程序。

为了将应用一次部署到多个 Kubernetes 集群，我们使用了 [Helm](https://helm.sh/)，并将所有 chart 存储在一个 git 仓库中。我们使用 __umbrella__ 来部署由多个服务组成的完整应用程序，这基本上是一个声明依赖关系的 chart ，其允许我们在单个命令行中引导我们的 API 及其服务。

此外，我们在使用 Helm 之前会运行一个 python 脚本，用来进行检查，构建 chart ，添加 secrets 并部署我们的应用程序。所有这些任务都是使用 docker 镜像在 CI 平台上完成的。

下面就进行详细介绍

__注意！：__ 当你阅读这篇博文的时候，Helm 3 的第一个 [release](https://github.com/helm/helm/releases/tag/v3.0.0-alpha.1) 已经发布。这个版本带来了一系列增强功能，肯定会解决我们过去遇到的一些问题。

image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## Charts 开发流程

在开发应用程序时，我们使用[分支工作流](https://git-scm.com/book/en/v2/Git-Branching-Branching-Workflows)，开发 chart 时也使用相同流程。

- 首先，__dev__ 分支用于构建要在开发集群上进行测试的 chart 。
- 然后，当发起 PR 请求到 __master__ 分支时，将发布到演示环境中进行验证。
- 最终，我们将 PR 请求提交的修改合并到 __prod__ 分支，将这个修改应用于生产环境。

我们使用 [Chartmuseum](https://chartmuseum.com/) 作为私有仓库来存储 chart ，每个环境都有一个 。这样我们就可以在__环境之间实现明确的隔离__，并且确保该 chart 在生产环境中使用之前已经过测试。

![](https://wx2.sinaimg.cn/large/ad5fbf65gy1g50h10d4xbj20ys0ee75e.jpg)
<center>每个环境的 Chart 仓库</center>

值得注意的是，当开发人员 push 代码到他们的 dev 分支时，他们的 chart 版本也会自动 push 到 dev 环境的 Chartmuseum 。因此，所有开发人员都使用相同的 dev 存储库，他们必须小心的指定自己的 chart 版本，以避免使用其他人的对 chart 的更改。

此外，我们的 python 脚本通过使用 [Kubeval](https://kubeval.instrumenta.dev/) 在它们推送到 Chartmusem 之前验证 Kubernetes 对象与 Kubernetes OpenAPI 规范。

> chart 开发工作流

![](https://ws3.sinaimg.cn/large/ad5fbf65gy1g50hg9gmh2j20gr047t8o.jpg)

1. 根据 [gazr.io](https://gazr.io/) 规范设置我们的 pipeline 任务（lint，unit-test）。
2. push docker 镜像，该镜像包含部署应用程序的 Python 工具。
3. 根据分支名称设置相应环境。
4. 使用 Kubeval 检查 Kubernetes yamls 。
5. 自动增加 chart 版本及其父项（取决于更改的 chart ）。
6. 将 chart push 到与其环境对应的 Chartmuseum 。

## 管理集群差异

> Cluster federation

我们使用 [Kubernetes cluster federation](https://kubernetes.io/docs/concepts/cluster-administration/federation/)，它允许我们从单个 API 端声明 Kubernetes 对象。但是我们遇到的问题是，无法在 federation 端中创建某些 Kubernetes 对象，因此很难维护 federation 对象和其他的群集对象。

为了解决这个问题，我们决定独立管理我们的集群，反而使这个过程变得更加容易（我们使用的是 federation v1，v2 可能有所改善）。

> 平台地理分布

目前，我们的平台分布在6个地区，3个在自己的数据中心，3个在公有云。

![](https://ws1.sinaimg.cn/large/ad5fbf65gy1g50klup6yaj212w0ftq4h.jpg)
<center>分布式部署</center>

> Helm global values

4个全局的 Helm value 定义集群间的差异。这些是我们所有 chart 的最小默认值。

```yaml
    global:
        cloud: True
        env: staging
        region: us-central1
        clusterName: staging-us-central1
```
<center>global values</center>

这些信息有助于我们为应用程序定义上下文，它们可用于监控，跟踪，记录，进行外部调用，扩展等许多内容......

- __cloud__：我们有一个混合 Kubernetes 集群。例如，我们的 API 部署在 GCP 和我们自己的数据中心。
- __env__：对于非生产环境，某些值可能会发生变化。本质上是资源定义和自动扩展配置。
- __region__：此信息用于标识群集的位置，并可用于定义外部服务的最近端点。
- __clusterName__：如果我们想要为每个群集定义一个值。

下面是一个具体的示例：
```yaml
{{/* Returns Horizontal Pod Autoscaler replicas for GraphQL*/}}
{{- define "graphql.hpaReplicas" -}}
{{- if eq .Values.global.env "prod" }}
{{- if eq .Values.global.region "europe-west1" }}
minReplicas: 40
{{- else }}
minReplicas: 150
{{- end }}
maxReplicas: 1400
{{- else }}
minReplicas: 4
maxReplicas: 20
{{- end }}
{{- end -}}
```
<center>helm template example</center>

请注意，此逻辑在帮助模板中定义，以保持 Kubernetes YAML 的可读性。

> 声明应用

我们的部署工具基于几个 YAML 文件，下面是我们声明服务及其每个集群的扩展拓扑（副本数量）的示例。

```yaml
releases:
  - foo.world
 
foo.world:                # Release name
  services:               # List of dailymotion's apps/projects
    foobar:
      chart_name: foo-foobar
      repo: git@github.com:dailymotion/foobar
      contexts:
        prod-europe-west1:
          deployments:
            - name: foo-bar-baz
              replicas: 18
            - name: another-deployment
              replicas: 3
```
<center>service definition</center>

这是部署工作流的所有步骤，最后一步将在多个生产集群上同时部署应用程序。

![](https://ws1.sinaimg.cn/large/ad5fbf65gy1g50ldllp33j20mw0bxglz.jpg)
<center>Jenkins deployment steps</center>

## Secrets 怎么办

在安全领域，我们专注于跟踪可能在不同位置传播的所有的 Secrets ，并将其存储在巴黎的 [Vault](https://www.vaultproject.io/) 。

我们的部署工具负责从 Vault 检索加密的值，并在部署时将其注入 Helm 。

为此，我们定义了存储在 Vault 中的 Secrets 与我们的应用程序所需的 Secrets 之间的映射，如下所示：

```yaml
secrets: 
     - secret_id: "stack1-app1-password"
       contexts:
         - name: "default"
           vaultPath: "/kv/dev/stack1/app1/test"
           vaultKey: "password"
         - name: "cluster1"
           vaultPath: "/kv/dev/stack1/app1/test"
           vaultKey: "password"
```

- 定义将 Secrets 写入 Vault 时要遵循的通用规则。
- 如果 Secrets 有特定的上下文/集群，则必须添加特定条目。
- 否则，将使用默认值。
- 对于此列表中的每个项目，将在 Kubernetes Secrets 中插入一个 key/value 。这样我们 chart 中的 Secrets 模板非常简单。

```yaml
apiVersion: v1
data:
{{- range $key,$value := .Values.secrets }}
  {{ $key }}: {{ $value | b64enc | quote }}
{{ end }}
kind: Secret
metadata:
  name: "{{ .Chart.Name }}"
  labels:
    chartVersion: "{{ .Chart.Version }}"
    tillerVersion: "{{ .Capabilities.TillerVersion.SemVer }}"
type: Opaque
```

## 警告与限制

image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

### 操作多个存储库

目前，chart 和应用程序开发是分离的。这意味着开发人员必须处理两个 git 存储库，一个用于应用程序，另一个用于定义如何在 Kubernetes 上部署。而2个 git 存储库意味着两个工作流程，这对于新手来说可能相当复杂。

### 管理 umbrella charts 可能很棘手

如前所述，umbrella charts 非常适合定义依赖关系并快速部署多个应用程序。同时我们使用 `--reuse-values` 选项，以避免每次部署作为 umbrella charts 一部分的应用程序时都要传递所有值。

在我们的 CD 工作流中，只有2个值会定期更改：副本数量和镜像标签（版本）。对于其他更稳定的值，需要手动更新，而且这些值并不是很容易弄清楚。此外，我们曾遇到过部署 umbrella charts 的一个错误导致严重的中断。

### 更新多个配置文件

添加新应用程序时，开发人员必须更改多个文件：应用程序声明， Secrets 列表，如果应用程序是 umbrella charts 的一部分，则将其添加到依赖。

### 在 Vault 上， Jenkins 权限过大

目前，我们有一个 [AppRole](https://www.vaultproject.io/docs/auth/approle.html) 可以读取 Vault 的所有 Secrets 。

### 回滚过程不是自动化的

回滚需要在多个集群上运行该命令，这可能容易出错。我们制作本操作手册是因为我们要确保应用正确的版本 ID 。

## GitOps 实践

> 目标

我们的想法是将 chart 放回到它部署的应用程序的存储库下。工作流程与应用同时开发，例如，无论何时在主服务器上合并分支，都会自动触发部署。这种方法与当前工作流程的主要区别在于，所有内容都将通过 git 进行管理（应用程序本身以及我们在 Kubernetes 中部署它的方式）。

这样做优点：

- 从开发人员的角度来看，更容易理解。学习如何在本地 chart 中应用更改将更容易。
- 将服务 deployment 定义在与此服务的代码相同的位置。
- 移除 umbrella charts 管理。服务将拥有自己的 Helm 版本。这使得应用程序生命周期管理（回滚，升级）形成闭环，不会影响其他服务。
- git 功能对 chart 管理的好处：回滚，审计日志......如果要还原 chart 更改，可以使用 git 进行更改。同时部署将自动触发。
- 我们考虑使用 Skaffold 等工具改进开发工作流程，这些工具允许开发人员在类似于生产的环境中测试他们的更改。

> 2步迁移

我们的开发人员已使用上述工作流程2年，因此我们需要尽可能顺利地进行迁移。这就是为什么我们决定在达到目标之前添加一个中间步骤。

第一步很简单：

- 我们将保留一个类似的结构来配置我们的应用程序部署，但是在名为 “DailymotionRelease” 的单个对象中

```yaml
apiVersion: "v1"
kind: "DailymotionRelease"
metadata:
  name: "app1.ns1"
  environment: "dev"
  branch: "mybranch"
spec:
  slack_channel: "#admin"
  chart_name: "app1"
  scaling:
    - context: "dev-us-central1-0"
      replicas:
        - name: "hermes"
          count: 2
    - context: "dev-europe-west1-0"
      replicas:
        - name: "app1-deploy"
          count: 2
  secrets:
    - secret_id: "app1"
      contexts:
        - name: "default"
          vaultPath: "/kv/dev/ns1/app1/test"
          vaultKey: "password"
        - name: "dev-europe-west1-0"
          vaultPath: "/kv/dev/ns1/app1/test"
          vaultKey: "password"
```

- 每个应用程序一个版本（不再使用 umbrella charts ）
- 将 chart 加入应用程序 git 存储库中

我们已经开始向所有开发人员科普这个词，并且迁移过程已经开始。第一步仍然使用 CI 平台进行控制。我将在短期内撰写另一篇博文，介绍第二步：我们如何通过 [Flux](https://github.com/weaveworks/flux) 实现向 GitOps 工作流程的迁移。将描述我们的设置和面临的挑战（多个存储库，Secrets 等）。 敬请期待！
