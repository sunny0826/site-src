---
title: "记一次使用 Kustomize 时遇到的愚蠢问题"
date: 2019-07-03T13:44:50+08:00
draft: false
type: blog
banner: "https://tva2.sinaimg.cn/large/ad5fbf65gy1g4mmuqm6n2j2098048a9z.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "解决使用 Kustomize 时遇到的报错： error: failed to find an object with apps_v1_Deployment|myapp to apply the patch "
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3j7v8sguj2098048a9z.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## 现象

在日常 CI/CD 流程中，已经将 Kustomize 集成到 pipeline 中使用，但是在对一个项目进行 Kustomize 改造时，将单个 `deploy.yaml` 拆分为了若干个 patch 以达到灵活 Kubernetes 部署的目的。但是在使用 `kubectl apply -k .` 命令进行部署的时候遇到了 `error: failed to find an object with apps_v1_Deployment|myapp to apply the patch` 的报错。

![image](http://tva2.sinaimg.cn/large/ad5fbf65gy1g4mm1m3vx9j21oe10y102.jpg)

## 解决之路

由于之前的使用中没有遇到此类报错，看报错信息像是 `apiVersion` 的问题，所以先检查了所有 patch 的 `apiVersion` ，但是并没有找到有什么问题。

### Google 搜索

对该报错进行了搜索，搜索到如下结果：

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4mmee8ctxj21900ns44c.jpg)
![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4mmgrdz0fj21ou1b6wro.jpg)

？？？ 为何这个 issue 没有解决就被提出者关闭了？

### 问题解决

在 Google 了一圈之后还是没有找到什么有营养的回答，问题又回到了原点...只能对所有的 patch 的每个字符和每个配置逐一进行了检查。结果发现是 `name` 的内容 base 与 overlays 不同... base 中是 `name:myapp` ，而 overlays 中是 `name:my-app` ...

好吧，issue 关的是有道理的...

![](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4mmuqm6n2j2098048a9z.jpg)
