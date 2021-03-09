---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubectl Plugin 推荐（二）| 简化操作篇"
subtitle: ""
summary: "推荐一些 Kubectl Plugin，本篇主要是简化操作的插件。"
authors: ["guoxudong"]
tags: ["Kubernetes","Kubernetes Plugin","Krew"]
categories: ["Kubernetes Plugin"]
date: 2021-03-09T11:18:48+08:00
lastmod: 2021-03-09T11:18:48+08:00
draft: false
type: blog
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65gy1godp4trcrnj20p00an3zg.jpg"
---
## 补充

开始介绍简化操作的插件之前，先补充一个增强可观测性的插件。

### pod-lens

该插件使用树状图和表格展示 pod 相关资源，在排查问题可以非常方便的查看 Pod 相关资源信息和状态。

项目地址：https://github.com/sunny0826/kubectl-pod-lens

#### 安装

```bash
$ kubectl krew install pod-lens
$ kubectl pod-lens --help
```

#### 示例

![pod-lens](https://tvax4.sinaimg.cn/large/ad5fbf65gy1godp0s6wj6j219014ugx8.jpg)

## 前言

[上一篇文章](../kubectl-plugin-recommended)中我们介绍一些提升观测性的 Kubectl Plugin，本篇笔者将继续推荐一些能够简化操作，提升工作效率的 Kubectl Plugin。

## 插件推荐

### iexec

工作中，我们经常会使用 `kubectl exec` 命令进入容器中进行问题排查 和 debug。而在实际操作中，除了需要加 `-it` 等参数外，还需要选择 Pod name 和 Container name，比较费事且经常操作失误。而 `kubectl-iexec` 这款插件很好的简化了这一系列操作。

项目地址：https://github.com/gabeduke/kubectl-iexec

#### 安装

```bash
$ kubectl krew install iexec
$ kubectl iexec --help
```

#### 用法

该插件极大的简化了 `kubectl exec` 操作。其可以模糊匹配 pod name，如果只有一个 pod 匹配输入的名称，将会直接进入该 Pod。

![单个匹配](https://tva1.sinaimg.cn/large/ad5fbf65gy1godnu79ttxj20rs0fkq5e.jpg)

如果匹配到多个 Pod，则会出现下拉菜单来选择要进入的 Pod 和 Container。

![多个匹配](https://tva4.sinaimg.cn/large/ad5fbf65gy1godnw22airj20pw0gajts.jpg)


### open-svc

日常工作中，常常需要使用 `kubectl port-forward` 命令来在本地访问部署在 k8s 中的服务，`open-svc` 则简化了这一步骤，输入命令，相应服务会直接在浏览器中弹出。

项目地址：https://github.com/superbrothers/kubectl-open-svc-plugin

#### 安装

```bash
$ kubectl krew install open-svc
$ kubectl open-svc --help
```

#### 示例

![open-svc](https://tva4.sinaimg.cn/large/ad5fbf65gy1godnz8c836g20ok0aymzp.gif)


### view-secret

当需要查看 Secret 中的信息时，往往需要执行以下步骤：

1. `kubectl get secret <secret> -o yaml`
2. 复制 secret 中的数据
3. `echo "<secret-data>" | base64 -d`

而 `view-secret` 这个插件就简化了这一步骤，直接在终端打印解码后的内容。

项目地址：https://github.com/elsesiy/kubectl-view-secret

#### 安装

```bash
$ kubectl krew install view-secret
$ kubectl view-secret --help
```

#### 示例

![view-secret](https://tvax3.sinaimg.cn/large/ad5fbf65gy1godoks7qwbj20oa11oq9n.jpg)

### ksniff

当需要在 k8s 中进行抓包时，`ksniff` 是个不错的选择，该插件使用 tcpdump 和 Wireshark 在 k8s 中进行抓包。

项目地址：https://github.com/eldadru/ksniff

#### 安装

```bash
$ kubectl krew install sniff
$ kubectl sniff --help
```

#### 示例

![ksniff](https://tva4.sinaimg.cn/large/ad5fbf65gy1godop5l1hqg21bp0oval7.gif)

## 总结

在日常工作中使用好这些插件，可以极大的提高工作效率，将运维人员从繁琐的工作中释放出来。**把有意义的事情做的有意思**，这也是工作的乐趣所在吧。