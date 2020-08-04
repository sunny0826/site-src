---
title: "使用 Kustomize 帮你管理 kubernetes 应用（一）：什么是 Kustomize ？"
date: 2019-04-15T13:32:59+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65gy1g23bdrc5tuj20x80wudlh.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "本篇为系列文章第一篇，介绍我对 Kustomize 的了解过程以及 Kustomize 是什么，为什么它能解决我的燃眉之急。"
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3j40z93gj20x80wu7fr.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 初识 Kustomize
第一次听说 Kustomize 其实是在 kubernetes 1.14 发布时候，它被集成到 ```kubectl``` 中，成为了一个子命令，但也只是扫了一眼，并没有深究。真正让我注意到它，并主动开始了解其功能和使用方法的，是张磊大神在云栖社区发表的一篇文章[《从Kubernetes 1.14 发布，看技术社区演进方向》](https://yq.aliyun.com/articles/697883)，他在文中是这么说的：

> Kustomize 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件，而不是像 Helm 那样只提供应用描述文件模板，然后通过字符替换（Templating）的方式来进行定制化。

这不正我在苦苦寻找的东西嘛！自从公司确定了应用容器化的方案，至今已有半年多了，这期间我们的服务一个接一个的实现了容器化，部署到了 kubernetes 集群中。kubernetes 集群也有原先了1个测试集群，几个节点，发展到了如今的多个集群，几十个节点。而在推进容器化的过程中，每个服务都对对应多个应用描述文件（ YAML 文件），而根据环境的不同，又配置了多套的应用描述文件。随着服务越部越多，应用描述文件更是呈爆炸式的增长。

感谢 devops 文化，它是我不需要为每个应用去写 YAML 文件，各个应用的开发组承担了这一工作，我只需要为他们提供基础模板即可。但应用上线后出现的 OOM 、服务无法拉起等 YAML 文件配置有误导致的问题接踵而至，使得我必须要深入各个服务，为他们配置符合他们配置。虽然也使用了 ```helm``` ，但是其只提供应用描述文件模板，在不同环境拉起一整套服务会节省很多时间，而像我们这种在指定环境快速迭代的服务，并不会减少很多时间。针对这种情况，我已经计划要自己开发一套更符合我们工作这种场景的应用管理服务，集成在我们自己的 devops 平台中。

这时 Kustomize 出现了，我明锐的感觉到 Kustomize 可能就是解决我现阶段问题的一剂良药。

## 什么是 Kustomize ？
> #### Kubernetes native configuration management

> Kustomize introduces a template-free way to customize application configuration that simplifies the use of off-the-shelf applications. Now, built into ```kubectl``` as ```apply -k```.

```kustomize```  允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件。而其他用户可以完全不受影响的使用任何一个 Base YAML 或者任何一层生成出来的 YAML 。这使得每一个用户都可以通过类似fork/modify/rebase 这样 Git 风格的流程来管理海量的应用描述文件。这种 PATCH 的思想跟 Docker 镜像是非常相似的，它可以规避“字符替换”对应用描述文件的入侵，也不需要用户学习额外的 DSL 语法（比如 Lua）。

而其成为 ```kubectl``` 子命令则代表这 ```kubectl``` 本身的插件机制的成熟，未来可能有更多的工具命令集成到 ```kubectl``` 中。拿张磊大神的这张图不难看出，在 kubernetes 原生应用管理系统中，应用描述文件在整个应用管理体系中占据核心位置，通过应用描述文件可以组合和编排多种 kubernetes API 资源，kubernetes 通过控制器来保证集群中的资源与应用状态与描述文件完全一致。

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g23cqlrodkj21bq0r8znk.jpg)

Kustomize 不像 Helm 那样需要一整套独立的体系来完成管理应用，而是完全采用 kubernetes 的设计理念来完成管理应用的目的。同时使用起来也更加的得心应手。

## 参考
- [Kustomize - kustomize.io](https://kustomize.io/)

- [从Kubernetes 1.14 发布，看技术社区演进方向 - yq.aliyun.com](https://yq.aliyun.com/articles/697883)