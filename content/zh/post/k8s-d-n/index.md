---
title: "Kubernetes删除一直处于Terminating状态的namespace"
date: 2018-11-16T18:18:13+08:00
draft: false
type: blog
tags: ["kubernetes"]
categories: ["问题解决"]
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t1c036hkj21qi15odl1.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "kubernetes解决删除的namespace一直处于Terminating状态的情况。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3ip90wwoj21qi15odl1.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>近期由于公司需要将部署在ucloud上的rancher迁移到阿里云上，所以需要将原有Rancher依赖的namespace（cattle-system）删除，但在删除中出现了删除的namespace一直处于Terminating状态的情况

![imgage](/images/source/d-n-1.png)

# 解决方案

运行命令：

```bash
kubectl edit namespaces cattle-system
```

可以看到namespaces的yaml配置：

![imgage](/images/source/d-n-2.png)

将finalizer的value删除，这里将其设置为[]

保存即可看到该namespace已被删除

![imgage](/images/source/d-n-3.png)
