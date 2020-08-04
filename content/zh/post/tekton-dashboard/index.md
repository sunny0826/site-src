---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "可视化 Tekton 组件 Tekton Dashboard"
subtitle: "Tekton Dashboard"
summary: "Tekton Dashboard 使用指南。"
authors: ["guoxudong"]
tags: ["CI/CD","Kubernetes","devops"]
categories: ["devops"]
date: 2020-05-13T09:55:51+08:00
lastmod: 2020-05-13T09:55:51+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65gy1geblt77f4bj22bd0qqq91.jpg"
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

Tekton 作为一款开源的云原生 CI/CD 框架，前身是 Knative 的 build-pipeline 项目。作为 CI/CD 框架，其本身并不是一个 CI/CD 产品，所以不应拿 Tekton 与 Jenkins 或者 Drone 这样的 CI/CD 产品进行比较，Tekton 本质是一个强大而灵活的 CI/CD 框架，开发者可以基于它开发自己的 CI/CD 工具或产品，一些有能力的团队可以使用 Tekton 做为底座开发出更适合自己团队使用的 CI/CD 工具。

而 Tekton 的可视化组件 Tekton Dashboard 则为用户提供了可视化界面，使 Tekton 的体验更接近与 Jenkins 这样的 CI/CD 产品，同时开发者可以在使用 Tekton Dashboard 时也会对 Tekton 的一些概念进行更深入的了解。

本文将会使用 Tekton Dashboard，通过 UI 界面在 K8S 集群中部署一个 Java 项目：[pipeline-example-maven](https://github.com/sunny0826/pipeline-example-maven)

## 交互式学习

[katacoda]: https://katacoda.com

本文还提供 [katacoda] 交互式学习版本，用户可以直接访问 katacoda 页面：https://katacoda.com/guoxudong/scenarios/tekton-dashboard ，使用 [katacoda] 在浏览器端学习使用 Tekton Dashboard。

该教程属于官方教程的汉化版，并得到了[许可](https://github.com/ncskier/katacoda/issues/2)。

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1geqt0wmbtvj21hb0q779v.jpg)

## Tekton Dashboard

### 安装

这是所有步骤中最麻烦的一步，由于官方提供的 Tekton 镜像都在 `gcr.io` 上，在国内并不能直接拉取，所以在测试的时候着实花费了不少时间。

我特意将这些镜像转储到 dockerhub 上，如果官方版无法使用，可以使用克隆版：

**安装 [Tekton Pipelines](https://github.com/tektoncd/pipeline/blob/master/docs/install.md)**

```shell
# 官方
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.10.1/release.yaml
# 克隆版
$ kubectl apply -f https://raw.githubusercontent.com/sunny0826/tekton-local/v0.10.1/tekton-pipeline.yaml
```

**安装 [Tekton Dashboard](https://github.com/tektoncd/dashboard#install-dashboard)**

```shell
# 官方
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/previous/v0.5.3/tekton-dashboard-release.yaml
# 克隆版
$ kubectl apply -f https://raw.githubusercontent.com/sunny0826/tekton-local/v0.10.1/tekton-dashboard.yaml
```

安装成功之后需要配置 Tekton Dashboard 的访问地址，可以使用 ingress 或 Nodeport 暴露端口，这里采用 `port-forward` 的形式将端口映射到本地：

```shell
$ kubectl port-forward svc/tekton-dashboard 8097:9097 -n tekton-pipelines
Forwarding from 127.0.0.1:8097 -> 9097
Forwarding from [::1]:8097 -> 9097
...
```

**访问 Tekton Dashboard**

打开浏览器访问访问 http://localhost:8097

![](https://tvax3.sinaimg.cn/large/ad5fbf65gy1geqnhem9i9j21mk0tu425.jpg)

### 导入资源

点击 `Import Tekton resources` 进入资源导入页面，导入资源：

- Repository URL: `https://github.com/sunny0826/pipeline-example-maven`
- Namespace: `default`
- Repository directory: `tekton/`
- Service Account `tekton-dashboard`

输入内容如下：

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1geqnp36mk0j20yu0memze.jpg)

点击 `Import and Apply` 按钮，之后 Dashboard 会创建一个 PipelineRun 来导入指定的 Tekton 资源。

点击页面底部的 `View status of this run` 链接，查看 MyApp 导入 Tekton 资源的状态。

![](https://tvax4.sinaimg.cn/large/ad5fbf65gy1geqnqyx5g2j20a403et8q.jpg)

PipelineRun 完成后，Tekton 资源已导入成功。

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1geqns0gqi8j21go0ozwhl.jpg)

### 创建 PipelineResource

选择 `default` 命名空间，并点击 `PipelineResource` 按钮。

![](https://tvax3.sinaimg.cn/large/ad5fbf65gy1geqnukrb3aj20yb0enmyg.jpg)

点击页面右上方的 `Create +` 按钮，将弹出一个创建 PipelineResource 的表单。

我们要在 `default` 命名空间中为 pipeline-example-maven 的 `master` 分支创建一个 git PipelineResource，故在弹出的表单中填写以下信息：

- Name: `pipeline-example-maven`
- Namespace: `default`
- Type: `Git`
- URL: `https://github.com/sunny0826/pipeline-example-maven`
- Revision: `master`

该表单内容应如下：

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1geqnxalh2pj20pl0dbq3h.jpg)

点击 `Create` 按钮，创建 PipelineResource。

### 创建 PipelineRun

选择 `default` 命名空间，并点击 `PipelineRuns` 按钮。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1geqo2iatnhj20yb0ewjso.jpg)

点击页面右上方的 `Create +` 按钮，将弹出一个创建 PipelineRun 的表单。该表单是动态的，会根据所选的 Pipeline 提供 PipelineResource 和 Param 字段。

我们需要 `default` 命名空间中使用 `pipeline-example-maven` 的 Pipeline 和 PipelineResource，创建一个 PipelineRun，故在弹出的表单中填写以下信息：

- Namespace: `default`
- Pipeline: `pipeline-example-maven`
- PipelineResources source: `pipeline-example-maven`
- 其余字段保留默认值。

该表单内容应如下：

![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1geqrdvoaquj20pi0lzt9o.jpg)

点击 `Create` 按钮，创建 PipelineRun。

### 查看 PipelineRun 日志

点击页面顶部创建通知中的链接或在 PipelineRun 列表中对应的 PipelineRun，查看 pipeline-example-maven PipelineRun 的日志。

![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1geqrhrwspcj217x0i7425.jpg)

> deploy 步骤中，有时会出现权限错误，需要给 default:default 绑定上 admin 的 clusterrole 权限：

```shell
$ kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=default:default
```

确认 `build` 和 `deploy` 任务均已成功。

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1geqrmy2mc9j218w0jo0uj.jpg)

{{% alert note %}}
**注意**：这里为了方便，使用的是单节点的 Kubernetes，构建完并没有推送到镜像仓库，镜像拉取策略为 `imagePullPolicy: Never` ，所以启动时候也没有从远程仓库拉取镜像，而是启动的本地镜像。
{{% /alert %}}

### 查看构建结果

```shell
$ kubectl get deploy
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
example-greenhouse   1/1     1            1           5h2m
```

## 总结

Tekton Dashboard 将 Tekton 的资源进行了可视化展示，指导用户快速理解 Tekton pipeline 流程以及配置方式，快速上手 Tekton。但是由于镜像的原因，导致新手体验不佳，所幸官方还提供了 [katacoda] 交互式教程，该教程我已汉化完成并获得了官方的许可，可以在浏览器端快速体验从安装 Tekton 到部署应用的整个过程。
