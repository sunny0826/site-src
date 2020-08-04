---
title: "阿里云容器服务新建集群优化方案(更新版)"
date: 2019-04-25T22:26:06+08:00
draft: false
type: blog
banner: "https://wx4.sinaimg.cn/large/ad5fbf65gy1g1ltc0evxsj20rs0ijq4s.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "这里记录了在工作中遇到阿里云容器服务的调优优化方案，帮助您绕过阿里云容器服务中的一些坑，来使用更好更优质的阿里云容器服务。"
tags: ["阿里云","kubernetes","容器"]
categories: ["kubernetes"]
keywords: ["阿里云","kubernetes","容器"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3i87o5kgj20rs0ijq4s.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

选择阿里云的```容器服务```，主要原因是公司主要业务基本都在运行在阿里云上。相较自建 kubernetes 集群，容器服务的优势在于部署相对简单，与阿里云 VPC 完美兼容，网络的配置相对简单，而如果使用 ```kubeadm``` 安装部署 kubernetes 集群，除了众所周知的科学上网的问题，还有一系列的问题，包括 ```etcd``` 、 ```Scheduler``` 和 ```Controller-Manager``` 的高可用问题等。并且如果使用托管版的阿里云 kubernetes 容器服务，还会省掉3台 master 节点的钱，并且可能将 master 节点的运维问题丢给阿里云解决，并且其提供的 master 节点性能肯定会比自购的配置好，这点是阿里云容器服务的研发小哥在来我司交流时专门强调的。

## 问题

前面吹了阿里云容器服务的优势，那这里就说说在实践中遇到的容器服务的问题：

- 在新建集群的时候需要选择相应的 VPC 并选择 ```Pod``` 和 ```Service``` 所在的网段，这两个网段不能和 Node 节点存在于同一网段，但是如果您在阿里云中存在不止一个 VPC （VPC的网段可以是 10.0.0.0/8，172.16-31.0.0/12-16，192.168.0.0/16 ），如果网段设置不对的话，就可能会使原本存在该网段的 ECS 失联，需要删除集群重新创建。如果删除失败的话，还需要手动删除路由表中的记录（**别问我是怎么知道的**）。

- 在使用容器服务创建集群后，会创建2个 SLB （之前是3个），一个是 SLB 是在 VPC 上并且绑定一个弹性IP（需要在创建的时候手动勾选创建弹性IP）用于 API Server，一个是经典网络的 SLB 使用提供给 Ingress 使用。但是这两个外网IP创建后的规格都是默认最大带宽、按流量收费，这个并不符合我们的要求，需要手动修改，~~然而这个修改都会在第二天才能生效~~。

- 容器服务创建集群后，Node 节点的名称会使```{region-id}.{ECS-id}```的形式，这个命名方式在集群监控，使用 ```kubectl``` 操作集群方面就显得比较反人类了，每次都要去查 ```ECS id``` 才能确定是哪个节点，而这个 Node 节点名称是不能修改的！

## 网段问题解决

这个比较好解决，甚至可以说不用解决，只要把网段规划好，不要出现网段冲突就好

## Node 节点名称无法修改问题解决

这个功能之前已有人在阿里聆听平台提出这个问题了，咨询了容器服务的研发小哥，得到的反馈是该功能已经在灰度测试了，相信很快就可以上线了。

## 创建 SLB 规格问题解决

相较之前自动创建3个 SLB 的方式，目前的版本只会自动创建2个并且有一个是 VPC 内网+弹性IP的方式，已经进行了优化，但是 ingress 绑定的 SLB 还是经典网络类型，无法接入云防火墙并且规格也是不合适的。这里给出解决方案：

### 方法一：使用 ```kubectl``` 配置 

#### 1. 创建新的 SLB 

- 这里需要创建一个新的 SLB 用来代替自动创建的不符合要求的 SLB。这里可以先私网 SLB 先不绑定弹性IP。***这里要注意的事，新建的 SLB 需要与 k8s集群处于同一 VPC 内，否则在后续会绑定失败***。
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1ma5lxgvdj21ws0s6qa5.jpg)
- 查看新购买 SLB 的 ID
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1ma8zuq1gj20sa0hoq4b.jpg)

#### 2. 在创建集群后重新绑定 ```ingress-controller``` 的 ```Service```

首先需要使用 ```kubectl``` 或者直接在阿里云控制台操作，创建新的 ```nginx-ingress-svc```

```yaml
# nginx ingress service
apiVersion: v1
kind: Service
metadata:
name: nginx-ingress-lb-{new-name}
namespace: kube-system
labels:
    app: nginx-ingress-lb-{new-name}
annotations:
    # set loadbalancer to the specified slb id
    service.beta.kubernetes.io/alicloud-loadbalancer-id: {SLB-ID}
    # set loadbalancer address type to intranet if using private slb instance
    #service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet
    service.beta.kubernetes.io/alicloud-loadbalancer-force-override-listeners: 'true'
    #service.beta.kubernetes.io/alicloud-loadbalancer-backend-label: node-role.kubernetes.io/ingress=true
spec:
type: LoadBalancer
# do not route traffic to other nodes
# and reserve client ip for upstream
externalTrafficPolicy: "Local"
ports:
- port: 80
    name: http
    targetPort: 80
- port: 443
    name: https
    targetPort: 443
selector:
    # select app=ingress-nginx pods
    app: ingress-nginx
```

创建成功后，可以进到 SLB 页面查看，可以看到 ```80``` 和 ```443``` 端口的监听已经被添加了
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1maej57c1j21ru0rwq8b.jpg) 

#### 3. 绑定符合要求的弹性IP

确定 SLB 创建成功并且已经成功监听后，这里就可以为 SLB 绑定符合您需求的弹性IP了，这里我们绑定一个按宽带计费2M的弹性IP

![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1mak2r0p3j207k07mq33.jpg)

#### 4. 验证连通性

到上面这步，我们的 ingress 入口 SLB 已经创建完成，这里我们验证一下是否联通。

- 在k8s集群中部署一个 ```nginx``` ，直接在阿里云容器服务控制台操作即可
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1mant7ec6j21s40qegpr.jpg)
    这里创建 ingress 路由，**注意：这里的域名需要解析到刚才创建的 SLB 绑定的弹性IP**
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1maqf7gdjj21ns0kymz8.jpg)

- 访问该域名，显示 ```nginx``` 欢迎页，则证明修改成功
    ![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g1mat8srhnj21ak0hmact.jpg)

### 方法二： 使用阿里云容器服务控制台配置

#### 1. 阿里云容器控制台创建新 ```service```

- 在阿里云容器服务控制台：```路由与负载均衡``` --> ```服务``` 点击```创建```
- 选择 ```kube-system``` 命名空间
- 类型选中```负载均衡``` - ```内网访问```
- 关联 ```nginx-ingress-controller```
- 并添加端口映射
- 点击创建

![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g2g4fwfgevj20i50hsgmp.jpg)

#### 2. 进入负载均衡查看 SLB 是否创建

可见 SLB 已经成功创建

![image](https://wx4.sinaimg.cn/large/ad5fbf65gy1g2g4pb1d45j215303c74r.jpg)

#### 3. 绑定符合要求的弹性IP
同方法一

#### 4.验证连通性
同方法一

### 后续操作

- 在确定新的 SLB 创建成功后，就可以将容器服务自动创建的 SLB 释放了
- 删除 ```kube-system``` 中原本绑定的 ```Service``` **（目前版本已经可以关联删除绑定的 SLB 了，不用分开操作）**
- **这里别忘了，自动创建给API Server 的SLB还是按流量付费的，记得降配**

## 后记

上面的这些问题和解决方案都属于临时方案，已在阿里的聆听平台提出了上面的问题，相信很快就会有所改进。总的来说，阿里云容器服务在提供优质的 kubernetes 功能，并且只收 ECS 的钱，对于想学习 kubernetes 又没有太多资金的同学也比较友好，直接买按量付费实例，测试完释放即可，不用购买 master 节点，十分良心！