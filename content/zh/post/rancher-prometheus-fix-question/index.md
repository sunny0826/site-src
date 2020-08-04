---
title: "Rancher 2.2.1 解决工作负载监控为空问题"
date: 2019-04-18T17:46:08+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65gy1g26yrcsxvhj21qx15owo8.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "记一次在 Rancher 官方小哥帮助下解决 Rancher 问题的过程。"
tags: ["rancher","kubernetes","prometheus"]
categories: ["问题解决"]
keywords: ["rancher","kubernetes","prometheus"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3jaudev1j21qx15owo8.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

Rancher 2.2.X 版本于3月底正式GA，新版本处理其他部分的优化以外，最大亮点莫过于本身集成了 Prometheus ，可以通过 Rancher 自带 UI 或者 Grafana 查看集群的实时监控，对所有监控进行了一次聚合，不用再和之前一样，每个集群都要安装一个 Prometheus 用于监控，而告警部分也可使用 Rancher 自带的通知组件进行告警。通知方式目前支持 Slack 、 邮件、 PagerDuty 、 Webhook 、 企业微信，由于我司办公使用钉钉，所以我们使用了 Webhook 的方式，告警触发后通知我们的消息服务，然后消息服务将其发送到钉钉进行告警。

![image](http://wx2.sinaimg.cn/large/ad5fbf65gy1g26xsh6omvj20rk0ilta6.jpg)

## 问题

Rancher 集成 Prometheus 后，监控方面变的十分强大，不用再徘徊于多个集群的 Grafana ，直接在 Rancher 上即可查看，非常方便

![image](http://wx2.sinaimg.cn/large/ad5fbf65gy1g26xuv2frnj212b0onn1h.jpg)

但是在使用的时候，我发现了一个问题：就是在查看 工作负载和 Pod 的时候会显示 ***没有足够的数据绘制图表***

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g26xzvi2cpj20po057q31.jpg)

进入 Grafana 查看会发现，其实监控参数是存在的，但是没有采集到值，所以并没有展示出来。

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g26y4j4s3yj21f50m9wqj.jpg)

## 解决

在检查了配置后并没有找到原因，只好去 GitHub 上提一个 issue 来询问一下开发者或者其他用户有无遇到这个问题。

Rancher 官方的开发者还是十分负责的， GitHub 上用户名为 [Logan](https://github.com/loganhz) 的官方小哥来我指导解决这个问题。

小哥发现我是导入的集群，要我进入 Prometheus 查看，发现 ```cattle-prometheus/exporter-kube-state-cluster-monitoring``` 果然没有起来

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g26yb1p4eoj21db0am782.jpg)

解决这个问题，需要在集群监控配置中添加一个高级选项，插入值为：`exporter-kubelets.https=false`

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g26ycq6amfj221q0uggp8.jpg)

点击保存，问题就解决了！

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g26yheqwp7j213e0g3di5.jpg)

## 后记

使用 Rancher 有半年，从2.0版本一直用到2.2版本，而18年分别在云栖大会和 KubeCon 上听了 Rancher 创始人梁胜博士的演讲。而从这一个小问题上就可以看到 Rancher 官方对每一个用户都是十分重视的。
