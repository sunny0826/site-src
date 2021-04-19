---
title: "kubernetes集群概述"
date: 2018-10-03T12:18:13+08:00
draft: false
type: blog
tags: ["kubernetes"]
categories: ["kubernetes"]
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t1e34opvj21qi15owjf.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "随着2017年AWS，Azure和阿里云相继在其原有容器服务上新增了对kubernetes的支持，而Docker官网也在同年10月宣布同时支持Swarm好kubernetes容器编排系统。kubernetes俨然已成为容器编排领域事实上的标准，而2018年更是各大公司相继将服务迁移到kubernetes上，而kubernetes则以惊人更新速度，保持着每个季度发布一个大版本的速度高速发展着。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3ivbmskjj21qi15owjf.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>随着2017年AWS，Azure和阿里云相继在其原有容器服务上新增了对kubernetes的支持，而Docker官网也在同年10月宣布同时支持Swarm好kubernetes容器编排系统。kubernetes俨然已成为容器编排领域事实上的标准，而2018年更是各大公司相继将服务迁移到kubernetes上，而kubernetes则以惊人更新速度，保持着每个季度发布一个大版本的速度高速发展着。

# kubernetes特征

kubernetes是一种在一组主机上运行和协同容器化应用程序的系统，旨在提供可预测性、可拓展性与高可用性的方法来完全管理容器化应用和服务的生命周期平台。用户可以定义应用程序的运行方式，以及与其他应用程序或外部世界交互的途径，并能实现服务的扩容和缩容，执行平滑滚动更新，以及在不同版本的应用程序之间调度流量以测试功能或回滚有问题的部署。kubernetes提供了接口和可组合帆软平台原语，使得用户能够以高度的灵活性和可靠性定义及管理应用程序。

# kubernetes组件及网络通信

kubernetes集群的客户端可以分为两类：API Server客户端和应用程序（运行为Pod中的容器）客户端。
![image](/images/source/kubernetes-topo.png)

* 第一类客户端通常包含用户和Pod对象两种，它们通过API Server访问kubernetes集群完成管理任务，例如，管理集群上的各种资源对象。
* 第二类客户端一般也包含人类用户和Pod对象两种，它们的访问目标是Pod上运行于容器中的应用程序提供的各种具体的服务，如redis或nginx等，不过，这些访问请求通常要经由Service或Ingress资源对象进行。另外，第二类客户端的访问目标对象的操作要经由第一类客户端创建和配置完成后才进行。

    访问API Server时，人类用户一般借助于命令行工具kubectl或图形UI（例如kubernetes dashboard）进行，也通过编程接口进行访问，包括REST API。访问Pod中的应用时，其访问方式要取决于Pod中的应用程序，例如，对于运行Nginx容器的Pod来说，其最常用工具就是浏览器。

    管理员（开发人员或运维人员）使用kubernetes集群的常见操作包括通过控制器创建Pod，在Pod的基础上创建Service供第二类客户端访问，更新Pod中的应用版本（更新和回滚）以及对应用规模进行扩容或缩容等，另外还有集群附件管理、存储卷管理、网络及网络策略管理、资源管理和安全管理等。