---
title: "由一封邮件看 Mailing List 在开源项目中的重要性"
date: 2019-07-04T09:16:41+08:00
draft: false
type: blog
banner: "https://tva2.sinaimg.cn/large/ad5fbf65gy1g4nkephdw7j21qf15odp5.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "从发现、使用 Kubernetes Client/Python 这个项目的过程，谈谈 mailing list 在开源项目中的重要性。"
tags: ["kubernetes", "python", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "python", "工具"]
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3iyhw047j21qf15odp5.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

> 只要仔细找，想要的轮子总会有的。
> --- 某不知名 DevOps 工程师

感谢 `kubernetes-dev` 的 Mailing List ！早上在浏览邮件时发现了下面这封有趣的邮件：

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4nkmrb8scj21780q0afv.jpg)

接触 Kubernetes 也有不短的时间了，也见证了 Kubernetes 干掉 Swarm 和 Mesos 成为容器编排领域的事实标准的过程。在享受 Kubernetes 及其生态圈带来的便利的同时也在为 Kubernetes 及 CNCF 项目进行贡献。而使用 [`kubectl`](https://github.com/kubernetes/kubectl)、[`rancher`](https://github.com/rancher/rancher) 甚至是 [`kui`](https://github.com/IBM/kui) 这些 CLI 和 UI 工具对 Kubernetes 集群进行操作和观察。

虽然上面这些工具为操作 Kubernetes 集群带来了极大的便利，但是归根到底还是一些开源项目，并不能满足我们的全部需求。所以我们只能根据我们自己的需求和 Kubernetes 的 api-server 进行定制，但是由于 Kubernetes 的 api-server 比较复杂，短时间内并不是那么好梳理的。

## kubernetes-client/python

由于我们自研的 DevOps 平台是使用 python 开发的，所以我也基于 python 语言开发了一套 Kubernetes Client ，但总的来说由于 Kubernetes 的功能实在太多，而我的开发实践并不是很多，开发出来的功能只是差强人意。

而 [`kubernetes-client/python`](https://github.com/kubernetes-client/python) 这个官方给出的轮子是真的香！

### 安装方便

这个安装方式简单的令人发指，支持的 python 版本为 `2.7 | 3.4 | 3.5 | 3.6 | 3.7` 并且和所有 python 依赖包一样，只需要使用 `pip` 安装即可：

```bash
pip install kubernetes
```

### 简单示例

查看所有的 pod ：

```python
#!/usr/bin/env python
#encoding: utf-8
#Author: guoxudong
from kubernetes import client, config

# Configs can be set in Configuration class directly or using helper utility
config.load_kube_config()

v1 = client.CoreV1Api()
print("Listing pods with their IPs:")
ret = v1.list_pod_for_all_namespaces(watch=False)
for i in ret.items:
    print("%s\t%s\t%s" % (i.status.pod_ip, i.metadata.namespace, i.metadata.name))
```

运行查看结果：

```python
Listing pods with their IPs:
172.22.1.126	kube-system	coredns-5975fdf55b-bqgkx
172.22.0.2	kube-system	coredns-5975fdf55b-vxbb4
10.16.16.13	kube-system	flexvolume-9ccf7
10.16.16.15	kube-system	flexvolume-h5xn2
10.16.16.14	kube-system	flexvolume-kvn5x
10.16.16.17	kube-system	flexvolume-mf4zv
10.16.16.14	kube-system	kube-proxy-worker-7lpfz
10.16.16.15	kube-system	kube-proxy-worker-9wd9s
10.16.16.17	kube-system	kube-proxy-worker-phbbj
10.16.16.13	kube-system	kube-proxy-worker-pst5d
172.22.1.9	kube-system	metrics-server-78b597d5bf-wdvqh
172.22.1.12	kube-system	nginx-ingress-controller-796ccc5d76-9jh5s
172.22.1.125	kube-system	nginx-ingress-controller-796ccc5d76-jwwwz
10.16.16.17	kube-system	terway-6mfs8
10.16.16.14	kube-system	terway-fz9ck
10.16.16.13	kube-system	terway-t9777
10.16.16.15	kube-system	terway-xbxlp
172.22.1.8	kube-system	tiller-deploy-5b5d8dd754-wpcrc
...
```

果然是一个好轮子，引入 kubeconfig 的方式及展示所有 namespace 的 pod 的方法封装的也十分简洁，是个非常漂亮的范例。建议可以看一下[源码](https://github.com/kubernetes-client/python)，肯定会有收获的！

### 支持版本

`client-python` 遵循 [semver](https://semver.org/lang/zh-CN/) 规范，所以在 `client-python` 的主要版本增加之前，代码将继续使用明确支持的 Kubernetes 集群版本。

|                    | Kubernetes 1.5 | Kubernetes 1.6 | Kubernetes 1.7 | Kubernetes 1.8 | Kubernetes 1.9 | Kubernetes 1.10 | Kubernetes 1.11 | Kubernetes 1.12 | Kubernetes 1.13 | Kubernetes 1.14 |
|--------------------|----------------|----------------|----------------|----------------|----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| client-python 1.0  | ✓              | -              | -              |-               |-               |-                |-                |-                |-                |-                |
| client-python 2.0  | +              | ✓              | -              |-               |-               |-                |-                |-                |-                |-                |
| client-python 3.0  | +              | +              | ✓              |-               |-               |-                |-                |-                |-                |-                |
| client-python 4.0  | +              | +              | +              |✓               |-               |-                |-                |-                |-                |-                |
| client-python 5.0  | +              | +              | +              |+               |✓               |-                |-                |-                |-                |-                |
| client-python 6.0  | +              | +              | +              |+               |+               |✓                |-                |-                |-                |-                |
| client-python 7.0  | +              | +              | +              |+               |+               |+                |✓                |-                |-                |-                |
| client-python 8.0  | +              | +              | +              |+               |+               |+                |+                |✓                |-                |-                |
| client-python 9.0  | +              | +              | +              |+               |+               |+                |+                |+                |✓                |-                |
| client-python 10.0 | +              | +              | +              |+               |+               |+                |+                |+                |+                |✓                |
| client-python HEAD | +              | +              | +              |+               |+               |+                |+                |+                |+                |✓                |

## Mailing List 的重要性

这次的收获很大程度得益于 `kubernetes-dev` 的 Mailing List 也就是邮件列表。这种沟通方式在国内不是很流行，大家更喜欢使用 QQ 和微信这样的即时通讯软件进行交流，但是大多数著名开源项目都是主要使用 __Mailing List__ 进行交流，交流的数量甚至比在 GitHub issue 中还多，在与 Apache 、 CNCF 项目开源的贡献者和维护者交流中得知了使用 __Mailing List__ 主要考虑是一下几点：

- 这种异步的交流方式可以让更多关心该话题的开发人员一起加入到讨论中。
- mailing list 是永久保留的，如果你对某个话题感兴趣，可以随时回复邮件，关注这个话题的开发者都会收到邮件，无论这个话题是昨天提出的，还是去年提出的，有助于解决一些陈年老 BUG （俗称技术债）。
- 即时通讯软件虽然很便利，但是问题很快会被评论顶掉，虽然诸如 slack 这样的工具解决了部分这方面的问题，但是还是不如 mailing list 好用。
- 并不是所有地区的开发者都有高速的宽带，性能优秀的PC，在地球上很多地区还是只能使用拨号上网，网速只有几kb/s，他们甚至 GitHub issue 都无法使用。但是你不能剥夺他们参与开源项目的权利，而 mailing list 是一种很好的交流方式。
- 通过 mailing list 可以很好掌握社区动态，效果明显好于 GitHub watch ，因为并不是项目的所有 commit 都是你关心的。

## 结语

如果你有志于参与到开源运动，在享受开源软件带来便利的同事，还想为开源软件做出自己的贡献，那么 mailing list 是你进入社区最好的选择。在 mailing list 中和来自世界各地志同道合的开发者交流中提升自己的能力，创造更大的价值，迈出你参与开源运动的第一步。
