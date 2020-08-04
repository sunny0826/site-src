---
title: "Pod质量服务类别(QoS)"
date: 2019-03-04T19:18:13+08:00
draft: false
type: blog
tags: ["kubernetes"]
categories: ["kubernetes"]
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0t1d4atzlj21qf15oq8r.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "根据Pod对象的requests和limits属性，kubernetes将Pod对象归类到BestEffort、Burstable和Guaranteed三个服务质量（Quality of Service，QoS）类别。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3iqlrxapj21qf15oq8r.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

> 根据Pod对象的requests和limits属性，kubernetes将Pod对象归类到BestEffort、Burstable和Guaranteed三个服务质量（Quality of Service，QoS）类别。

- Guaranteed
    - cpu:requests=limits
    - memory:requests=limits
    - 这类Pod具有最高优先级
- Burstable
    - 至少一个容器设置了cpu或内存资源的requests
    - 这类Pod具有中等优先级
- BestEffort
    - 未有任何一个容器设置requests或limits属性
    - 这类Pod具有最低优先级

![](http://ww1.sinaimg.cn/large/ad5fbf65ly1g0rv2ipzqkj20hx0edmx8.jpg)

同级别优先级的Pod资源在OOM时，与自身的requests属性相比，其内存占用比例最大的Pod对象将被首先杀死。如上图同属Burstable类别的Pod A将先于Pod B被杀死，虽然其内存用量小，但与自身的requests值相比，它的占用比例95%要大于Pod B的80%。
