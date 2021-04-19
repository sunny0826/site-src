---
title: "自动合并Kubeconfig，实现多k8s集群切换"
date: 2019-03-17T10:45:02+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65gy1g15zla19o4j21y013ete4.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "随着kubernetes集群的增加，集群管理的问题就凸显出来，不同的环境存在不同的集群，不同的业务线不同的集群，甚至有些开发人员都有自己的集群。这里介绍一款工具来自动合并Kubeconfig，实现多k8s集群切换。"
tags: ["kubernetes","工具"]
categories: ["kubernetes"]
keywords: ["kubernetes","工具"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3j8edws4j21y013ete4.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言
>随着微服务和容器化的深入人心，以及kubernetes已经成为容器编排领域的事实标准，越来越多的公司将自己的服务迁移到kubernetes集群中。而随着kubernetes集群的增加，集群管理的问题就凸显出来，不同的环境存在不同的集群，不同的业务线不同的集群，甚至有些开发人员都有自己的集群。诚然，如果集群是使用公有云如阿里云或华为云的容器服务，可以登录其控制台进行集群管理；或者使用rancher这用的多集群管理工具进行统一的管理。但是在想操作```istio```特有的容器资源，或者想使用```istioctl```的时候，或者像我一样就是想使用```kubectl```命令的同学，这个时候多集群的切换就显的十分重要了。

## 简介
```kubectl```命令行工具通过```kubeconfig```文件的配置来选择集群以及集群的API Server通信的所有信息。```kubeconfig```用来保存关于集群，用户，名称空间和身份验证机制的信息。默认情况下```kubectl```使用的配置文件名称是在```$HOME/.kube```目录下的```config```文件，可以通过设置环境变量KUBECONFIG或者--kubeconfig指定其他的配置文件。详情可看官方文档https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

## 原理
使用```kubeconfig```文件，您可以组织您的群集，用户和名称空间。 还可以定义上下文以快速轻松地在群集和名称空间之间切换。

### 上下文(Context) 
```kubeconfig```文件中的上下文元素用于以方便的名称对访问参数进行分组。 每个上下文有三个参数：集群，命名空间和用户。 默认情况下，kubectl命令行工具使用当前上下文中的参数与集群进行通信。可以使用下面的命令设置上下文：

    kubectl config use-context

### 配置内容

    kubectl config view

1. 如果设置了```--kubeconfig```标志，则只使用指定的文件。该标志只允许有一个实例。 
2. 如果环境变量```KUBECONFIG```存在，那么就使用该环境变量```KUBECONFIG```里面的值，如果不存在该环境变量```KUBECONFIG```，那么默认就是使用```$HOME/.kube/config```文件。

### ```kubeconfig```内容
从下面kubeconfig文件的配置来看集群、用户、上下文、当前上下文的关系就比较明显了。

    apiVersion: v1
    kind: Config
    preferences: {}
    
    clusters:
    - cluster:
    name: {cluster-name}
    
    users:
    - name: {user-name}
    
    contexts:
    - context:
        cluster: {cluster-name}
        user: {user-name}
    name: {context-name}

    current-context: {context-name}

## 为何要自动合并
在日常的工作中，如果我们需要操作多个集群，会得到多个kubeconfig配置文件。一般的kubeconfig文件都是yaml格式的，但是也有少部分的集群kubeconfig时已json文件的形式给出的（比如华为云的=。=），比如我们公司再阿里云、华为云和自建环境上均存在kubernetes集群，平时操作要在多集群之间切换，这也就催生了我写这个工具（其实就是一个脚本）的动机。

## 自动合并生成kubeconfig
众所周知，yaml是一种直观的能够被电脑识别的数据序列化格式，是一个可读性高并且容易被人类阅读的语言和json相比（没有格式化之前）可读性更强。而我这个工具并不是很关心kubeconfig的格式，只要将想要合并的kubeconfig放入指定文件即可。

GitHub：https://github.com/sunny0826/mergeKubeConfig

### 适用环境

* 需要在终端使用命令行管理多集群
* kubernetes集群中安装了istio，需要使用```istioctl```命令，但是集群节点并没有安装```istioctl```，需要在本地终端操作
* 不愿频繁编辑.kube目录中的config文件的同学

### 准备工作

* Python环境：2.7或者3均可
* 需要依赖包：```PyYAML```

### 开始使用

* 安装依赖：

        pip install PyYAML
        
* 运行脚本

    * 默认运行方式，kubeconfig文件放入```configfile```文件,注意删掉作为示例的两个文件
    
            python merge.py
            
    * 自定义kubeconfig文件目录
    
            python merge.py -d {custom-dir}
            
### 运行后操作

* 将生成的config文件放入.kube目录中

        cp config ~/.kube

* 查看所有的可使用的kubernetes集群角色

        kubectl config get-contexts

* 更多关于kubernetes配置文件操作

        kubectl config --help

* 切换kubernetes配置

        kubectl config use-context {your-contexts}

## 结语
在使用kubernetes初期，在多集群之间我一直是频繁的切换```.kube/config```文件来达到切换操作集群的目的。这也导致了我的```.kube```目录中存在这多个类似于```al_test_config.bak```、```al_prod_config.bak```、```hw_test_config.bak```的文件，本地环境已经自建环境，在集群切换的时候十分头疼。而后来使用```--kubeconfig```来进行切换集群，虽然比之前的方法要方便很多，但是并不十分优雅。这个简单的小工具一举解决了我的文件，对于我这个```kubectl```重度依赖者来说十分重要。
