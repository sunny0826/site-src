---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Istio 升级新方式：金丝雀升级"
subtitle: ""
summary: "Istio 1.6 推出了渐进式的升级方式：金丝雀升级，为相对头疼的 Istio 升级问题提供了一种解决方案。"
authors: ["guoxudong"]
tags: ["istio","service mesh"]
categories: ["istio"]
date: 2020-07-08T15:08:09+08:00
lastmod: 2020-07-08T15:08:09+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ggjn49yrfgj228011bjuj.jpg"
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

自 1.5 版本开始，Istio 就弃用了之前使用 `Helm` 的安装方式。而 1.6 发布也有一段时间了，目前都已经到了 `1.6.4` 版本，就升级部分，Istio 1.6 推出了渐进式的升级方式：金丝雀升级，为相对头疼的 Istio 升级问题提供了一种解决方案。

>Istio 不支持跨版本升级。本文仅讲解从 1.5 版本升级到 1.6 版本。如果您使用的是旧版本，请先升级到 1.5 版本。

## 金丝雀升级

顾名思义，金丝雀升级可以让新老版本的 `istiod` 同时存在，并允许将所有流量在由新版控制平面 `istiod-canary` 控制之前，先将一小部分工作负载交由新版本 `istiod-canary` 控制，并进行监控，渐进式的完成升级。该方式比原地升级安全的多，也是现在推荐的升级方式。

### 控制平面升级

首先需要[下载新版本 Istio](https://github.com/istio/istio/releases) 并切换目录为新版本目录。

安装`canary`版本，将`revision`字段设置为`canary`：

```bash
$ istioctl install --set revision=canary
```

这里会部署新的 `istiod-canary`，但并不会对原有的控制平面造成影响，部署成功后会看到两个并行的 `istiod`：

```bash
$ kubectl get pods -n istio-system
NAME                                        READY   STATUS    RESTARTS   AGE
pod/istiod-85745c747b-knlwb                 1/1     Running   0          33m
pod/istiod-canary-865f754fdd-gx7dh          1/1     Running   0          3m25s
```

这里还可以看到新版的 `sidecar injector` 配置：

```bash
$ kubectl get mutatingwebhookconfigurations
NAME                            CREATED AT
istio-sidecar-injector          2020-07-07T08:39:37Z
istio-sidecar-injector-canary   2020-07-07T09:06:24Z
```

### 数据平面升级

只安装 `canary` 版本的控制平面并不会对现有的代理造成影响，要升级数据平面，并将他们指向新的控制平面，就需要在 namespace 中插入 `istio.io/rev` 标签。

例如，想要要升级 `default` namespace 的数据平面，需要添加 `istio.io/rev` 标签并删除`istio-injection`标签，以指向 `canary` 版本的控制平面：

```bash
$ kubectl label namespace default istio-injection- istio.io/rev=canary
```

注意：`istio-injection` 标签必须删除，因为该标签的优先级高于 `istio.io/rev` 标签，保留该标签将导致无法升级数据平面。

在 namespace 更新成功后，需要重启 Pod 来重新注入 Sidecar：

```bash
$ kubectl rollout restart deployment -n default
```

当重启成功后，该 namespace 的 pod 将被配置指向新的`istiod-canary`控制平面，使用如下命令查看启用新代理的 Pod：

```bash
$ kubectl get pods -n default -l istio.io/rev=canary
```

同时可以使用如下命令验证新 Pod 的控制平面为`istiod-canary`：

```bash
$ istioctl proxy-config endpoints ${pod_name}.default --cluster xds-grpc -ojson | grep hostname
    "hostname": "istiod-canary.istio-system.svc"
```

这时 `default` namespace 内的 Pod 就完成了金丝雀升级，接下来就可以进行验证，确定这些 Pod 有没有因为 Istio 升级而导致功能异常。

## 原地升级 & 降级

目前原地升级有很大的概率通不过升级检测，导致无法升级，不推荐这种升级方式，这里就不做介绍了，详情见[官方文档](https://istio.io/latest/docs/setup/upgrade/)。

并且虽然 Istio 提供了降级方式，但是经过测试降级的体验并不好，并且出现了由于不支持的 CRD `apiVersion` 导致无法降级的情况，所以请谨慎升级。

## 总结

总体来说，金丝雀升级的出现很好的解决了控制平面渐进式升级的需求，但是由于`istioctl upgrade`命令支持的场景和版本太少以及 Istio 整体架构的更改，目前的原地升级体验很差。最近还爆出了谷歌将把 Istio 捐给一个新成立基金会的消息，看来 Istio 要走的路还很长。

## 参考

- [Upgrade Istio - istio.io](https://istio.io/latest/docs/setup/upgrade/)
