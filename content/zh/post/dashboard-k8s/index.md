---
title: "阿里云日志服务采集k8s日志并实现livetail功能"
date: 2019-02-14T14:07:06+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0t0tydo76j21qi15oaqs.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "阿里云日志服务采集k8s日志和livetail功能调研"
tags: ["阿里云", "日志", "kubernetes"]
categories: ["部署安装"]
keywords: ["阿里云","日志","kubernetes"]
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3idgsfk2j21qi15oaqs.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# 前言
>目前的项目日志都是通过Logtail直接采集，投递到OSS持久化，同时可以通过阿里云日志服务、devops自建平台进行查看（虽然大部分人是直接登录ECS查看=。=），
在开始进行容器化之后，同样遇到日志的问题，目前的解决方案是阿里云日志服务持久化和展现格式化后的日志、使用rancher查看实时日志，
但是之前由于rancher平台出现一些问题，导致不能及时查看日志的情况，在这个背景下对阿里云日志服务采集k8s日志和livetail进行搭建并调研此方案是否可行。

# 简介（转自阿里云官方文档）
日志服务（Log Service，简称 LOG）是针对日志类数据的一站式服务，在阿里巴巴集团经历大量大数据场景锤炼而成。您无需开发就能快捷完成日志数据采集、消费、投递以及查询分析等功能，提升运维、运营效率，建立 DT 时代海量日志处理能力。

# kubernetes日志采集组件安装

## 安装Logtail

* 进入阿里云容器服务找到集群id
![image](/images/source/log_ser.png)

* 通过ssh登录master节点，或者任意安装了kubectl并配置了该集群kubeconfig的服务器

* 执行命令，将${your_k8s_cluster_id}替换为集群id

    ```bash
    wget http://logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com/kubernetes/alicloud-log-k8s-install.sh -O alicloud-log-k8s-install.sh; chmod 744 ./alicloud-log-k8s-install.sh; sh ./alicloud-log-k8s-install.sh ${your_k8s_cluster_id}
    ```

    * Project k8s-log-${your_k8s_cluster_id}下会自动创建名为config-operation-log的Logstore，用于存储alibaba-log-controller的运行日志。请勿删除此Logstore，否则无法为alibaba-log-controller排查问题。
    * 若您需要将日志采集到已有的Project，请执行安装命令sh ./alicloud-log-k8s-install.sh${your_k8s_cluster_id} ${your_project_name} ，并确保日志服务Project和您的Kubernetes集群在同一地域。

* 该条命令其实就是执行了一个shell脚本，使用helm安装了采集kubernetes集群日志的组件

    ```vim
    #!/bin/bash

    if [ $# -eq 0 ] ; then
        echo "[Invalid Param], use sudo ./install-k8s-log.sh {your-k8s-cluster-id}"
        exit 1
    fi
    
    clusterName=$(echo $1 | tr '[A-Z]' '[a-z]')
    curl --connect-timeout 5  http://100.100.100.200/latest/meta-data/region-id
    
    if [ $? != 0 ]; then
        echo "[FAIL] ECS meta server connect fail, only support alibaba cloud k8s service"
        exit 1
    fi
    
    regionId=`curl http://100.100.100.200/latest/meta-data/region-id`
    aliuid=`curl http://100.100.100.200/latest/meta-data/owner-account-id`
    
    helmPackageUrl="http://logtail-release-$regionId.oss-$regionId.aliyuncs.com/kubernetes/alibaba-cloud-log.tgz"
    wget $helmPackageUrl -O alibaba-cloud-log.tgz
    if [ $? != 0 ]; then
        echo "[FAIL] download alibaba-cloud-log.tgz from $helmPackageUrl failed"
        exit 1
    fi
    
    project="k8s-log-"$clusterName
    if [ $# -ge 2 ]; then
        project=$2
    fi
    
    echo [INFO] your k8s is using project : $project
    
    helm install alibaba-cloud-log.tgz --name alibaba-log-controller \
        --set ProjectName=$project \
        --set RegionId=$regionId \
        --set InstallParam=$regionId \
        --set MachineGroupId="k8s-group-"$clusterName \
        --set Endpoint=$regionId"-intranet.log.aliyuncs.com" \
        --set AlibabaCloudUserId=":"$aliuid \
        --set LogtailImage.Repository="registry.$regionId.aliyuncs.com/log-service/logtail" \
        --set ControllerImage.Repository="registry.$regionId.aliyuncs.com/log-service/alibabacloud-log-controller"
    
    installRst=$?
    
    if [ $installRst -eq 0 ]; then
        echo "[SUCCESS] install helm package : alibaba-log-controller success."
        exit 0
    else
        echo "[FAIL] install helm package failed, errno " $installRst
        exit 0
    fi
    ```
* 命令执行后，会在kubernetes集群中的每个节点运行一个日志采集的pod：logatail-ds，该pod位于kube-system

    ![image](/images/source/log_detail.png)

* 安装完成后，可使用以下命令来查看pod状态，若状态全部成功后，则表示安装完成

    ```bash
    $ helm status alibaba-log-controller
    LAST DEPLOYED: Thu Nov 22 15:09:35 2018
    NAMESPACE: default
    STATUS: DEPLOYED
    
    RESOURCES:
    ==> v1/ServiceAccount
    NAME                    SECRETS  AGE
    alibaba-log-controller  1        6d
    
    ==> v1beta1/CustomResourceDefinition
    NAME                                   AGE
    aliyunlogconfigs.log.alibabacloud.com  6d
    
    ==> v1beta1/ClusterRole
    alibaba-log-controller  6d
    
    ==> v1beta1/ClusterRoleBinding
    NAME                    AGE
    alibaba-log-controller  6d
    
    ==> v1beta1/DaemonSet
    NAME        DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
    logtail-ds  16       16       16     16          16         <none>         6d
    
    ==> v1beta1/Deployment
    NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
    alibaba-log-controller  1        1        1           1          6d
    
    ==> v1/Pod(related)
    NAME                                     READY  STATUS   RESTARTS  AGE
    logtail-ds-2fqs4                         1/1    Running  0         6d
    logtail-ds-4bz7w                         1/1    Running  1         6d
    logtail-ds-6vg88                         1/1    Running  0         6d
    logtail-ds-7tp6v                         1/1    Running  0         6d
    logtail-ds-9575c                         1/1    Running  0         6d
    logtail-ds-bgq84                         1/1    Running  0         6d
    logtail-ds-kdlhr                         1/1    Running  0         6d
    logtail-ds-lknxw                         1/1    Running  0         6d
    logtail-ds-pdxfk                         1/1    Running  0         6d
    logtail-ds-pf4dz                         1/1    Running  0         6d
    logtail-ds-rzsnw                         1/1    Running  0         6d
    logtail-ds-sqhbv                         1/1    Running  0         6d
    logtail-ds-vvtwn                         1/1    Running  0         6d
    logtail-ds-wwmhg                         1/1    Running  0         6d
    logtail-ds-xbp4j                         1/1    Running  0         6d
    logtail-ds-zpld9                         1/1    Running  0         6d
    alibaba-log-controller-85f8fbb498-nzhc8  1/1    Running  0         6d
    ```
    
# 配置日志组件展示

* 在集群内安装好日志组件后，登录阿里云日志服务控制台，就会发现有一个新的project，名称为k8s-log-{集群id}
![image](/images/source/log_src_de.png)

* 创建Logstore
![image](/images/source/log-1.png)

* 数据导入
![image](/images/source/log-2.png)

* 选择数据类型中选择docker标准输出
![image](/images/source/log-3.png)

* 数据源配置，这里可以使用默认的
![image](/images/source/log-4.png)

* 选择数据源
![image](/images/source/log-5.png)

* 配置好之后等待1-2分钟，日志就会进来了
![image](/images/source/log-6.png)

* 为了快速查询和过滤，需要配置索引
![image](/images/source/log-7.png)

* 添加容器名称、命名空间、pod名称作为索引（后续使用livetail需要）
![image](/images/source/log-8.png)

* 这样就完成了一个k8s集群日志采集和展示的基本流程了

# livetail功能使用

## 背景介绍
在线上运维的场景中，往往需要对日志队列中进入的数据进行实时监控，从最新的日志数据中提取出关键的信息进而快速地分析出异常原因。在传统的运维方式中，如果需要对日志文件进行实时监控，需要到服务器上对日志文件执行命令tail -f，如果实时监控的日志信息不够直观，可以加上grep或者grep -v进行关键词过滤。日志服务在控制台提供了日志数据实时监控的交互功能LiveTail，针对线上日志进行实时监控分析，减轻运维压力。

## 使用方法

* 这里选择来源类型为kubernetes，命名空间、pod名称、容器名称为上一步新建的3个索引的内容，过滤关键字的功劳与tail命令后加的grep命令是一样的，用于关键词过滤
![image](/images/source/log-9.png)

* 点击开启livetail，这时就有实时日志展示出来了
![image](/images/source/log-10.png)

**以上就是阿里云livetail日志服务功能**