---
title: "Kustomize: 无需模板定制你的 kubernetes 配置"
date: 2019-04-15T17:23:21+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65gy1g24hv1hekoj21y012cqc6.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "本文介绍了 Kubernetes 原生的应用管理工具 Kustomize。"
tags: ["翻译", "kustomize", "kubernetes", "工具"]
categories: ["翻译"]
keywords: ["翻译", "kustomize", "kubernetes", "工具"]
image:
  curl: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3imde420j21y012cqc6.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
> 作者：Jeff Regan (Google), Phil Wittrock (Google) 2018-05-29

如果你在运行 kubernetes 集群，你可能会拷贝一些包含 kubernetes API 对象的 YAML 文件，并且根据你的需求来修改这些文件，通过这些 YAML 文件来定义你的 kubernetes 配置。

但是这种方法存在很难找到配置的源头并对其进行改进。今天 Google 宣布推出 **Kustomize** ，一个作为 [SIG-CLI](https://github.com/kubernetes/community/tree/master/sig-cli) 子项目的命令行工具。这个工具提供了一个全新的、纯粹的声明式的方法来定制 kubernetes 配置，遵循并利用我们熟悉且精心设计的 Kubernetes API。

有这样一个常见的场景，在互联网上可以看到别人的 CMS（content management system，内容管理系统）的 kubernetes 配置，这个配置是一组包括 Kubernetes API 对象的 YAML 描述文件。然后，在您自己公司的某个角落，您找到一个你非常了解的数据库，希望用它来该 CMS 的数据。

你希望同时使用它们，此外，你希望自定义配置文件以便你的资源实例在集群中显示，并通过添加一个标签来区分在同一集群中做同样事情的其他资源。同时也希望为其配置适当的 CPU 、内存和副本数。

此外，你还想要配置整个配置的多种变化：一个专门用于测试和实验的小服务实例（就计算资源而言），或更大的用于对外提供服务的生产级别的服务实例。同时，其他的团队也希望拥有他们自己的服务实例。

## 定制就是复用
kubernetes 的配置并不是代码（是使用 YAML 描述的 API 对象，严格来说应该是数据），但是配置的生命周期与代码的生命周期有许多相似之处。

你需要在版本控制中保留配置。所有者的配置不必与使用者的配置相同。配置可以作为整体的一部分。而用户希望为在不同的情况下复用这些配置。

与代码复用相同，一种复用配置的方法是简单的全部拷贝并进行自定义。像代码一样，切断与源代码的联系使得从改进变的十分困难。许多团队和环境都使用这种方法，每个团队和环境都拥有自己的配置，这使得简单的升级变得十分棘手。

另一种复用方法是将源代码抽象为参数化模板。使用一个通过执行脚本来替换所需参数的模板处理工具生成配置，通过为同一模板设置不同的值来达到复用的目的。而这种方式面临的问题是模板和参数文件并不在 kubernetes API 资源的规范中，这种方式必定是一种包装了 kubernetes API 的新东西、新语言。虽然这种方式很强大，但是也带来了学习成本和安装工具的成本。不同的团队需要不同的更改，因此几乎所有可以包含在 YAML 文件中的规范都会需要抽象成参数。

## 自定义配置的新选择
**kustomize** 中工具的声明与规范是由名为 ```kustomization.yaml``` 的文件定义。

**kustomize** 将会读取声明文件和 Kubernetes API 资源文件，将其组合然后将完整的资源进行标准化的输出。输出的文本可以被其他工具进一步处理，或者直接通过 **kubectl** 应用于集群。

例如，如果 ```kustomization.yaml``` 文件包括：

```yaml
commonLabels:
  app: hello
resources:
- deployment.yaml
- configMap.yaml
- service.yaml
```

确保这三个文件与 ```kustomization.yaml``` 位于同一目录下，然后运行：

```bash
kustomize build
```

将创建包含三个资源的 YAML 流，其中 ```app: hello``` 为每个资源共同的标签。

同样的，你可以使用 ***commonAnnotations*** 字段给所有资源添加注释， ***namePrefix*** 字段为所有的资源添加共同的前缀名。这些琐碎而有常见的定制只是一个开始。

一个更常见的例子是，你需要为一组相同资源设置不同的参数。例如：开发、演示和生产的参数。

为此，**Kustomize** 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件。两者都是由 kustomization 文件表示。基础（Base）声明了共享的内容（资源和常见的资源配置），Overlay 则声明了差异。

这里是一个目录树，用于管理集群应用程序的 ***演示*** 和 ***生产*** 配置参数：

```bash
someapp/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── configMap.yaml
│   └── service.yaml
└── overlays/
    ├── production/
    │   └── kustomization.yaml
    │   ├── replica_count.yaml
    └── staging/
        ├── kustomization.yaml
        └── cpu_count.yaml
```

**```someapp/base/kustomization.yaml``` 文件指定了公共资源和常见自定义配置（例如，它们一些相同的标签，名称前缀和注释）。**

**```someapp/overlays/production/kustomization.yaml``` 文件的内容可能是：**

```yaml
commonLabels:
  env: production
bases:
- ../../base
patches:
- replica_count.yaml
```

这个 kustomization 指定了一个 ***patch*** 文件 ```replica_count.yaml``` ，其内容可能是：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 100
```

***patch*** 是部分的资源声明，在这个例子中是 Deployment 的补丁 ```someapp/base/deployment.yaml``` ，仅修改了副本数用以处理生产流量。

该补丁不仅仅是一个无上下文 {parameter name，value} 元组。其作为部分 deployment spec，可以通过验证，即使与其余配置隔离读取，也具有明确的上下文和用途。

要为生产环境创建资源，请运行：

```bash
kustomize build someapp/overlays/production
```

运行结果将作为一组完整资源打印到标准输出，并准备应用于集群。可以用类似的命令定义演示环境的配置。

## 综上所述
使用 **kustomize** ，您可以仅使用 Kubernetes API 资源文件就可以管理任意数量的 Kubernetes 定制配置。kustomize 的每个产物都是纯 YAML 的，每个都可以进行验证和运行的。**kustomize** 鼓励通过 fork/modify/rebase 这样的[工作流](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/workflows.md)来管理海量的应用描述文件。

尝试[hello world](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/helloWorld)示例，开始使用 **kustomize** 吧！有关的反馈与讨论，可以通过加入[邮件列表](https://groups.google.com/forum/#!forum/kustomize)或提 [issue](https://github.com/kubernetes-sigs/kustomize/issues/new)，欢迎提交PR。

## 译者按
随着 kubernetes 1.14 的发布，kustomize 被集成到 ```kubectl``` 中，用户可以利用 ```kubectl apply -k dir/``` 将指定目录的 ```kustomization.yaml``` 提交到集群中。

**原文链接** https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/