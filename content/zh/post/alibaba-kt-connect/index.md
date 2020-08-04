---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kt Connect：研发侧利器，本地连通 Kubernetes 集群内网"
subtitle: ""
summary: "研发侧利器，云原生 VPN：Kt Connect，可在本地调用 Kubernetes 集群服务，或将 Kubernetes 集群流量转发到本地。"
authors: ["guoxudong"]
tags: ["Kubernetes"]
categories: ["Kubernetes"]
date: 2020-03-24T09:14:06+08:00
lastmod: 2020-03-24T09:14:06+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3i33qlz7j21qm15o4f6.jpg"
  caption: ""
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

随着 Kubernetes 的普及，越来越多的应用被容器化，并部署到 Kubernetes 上。随之而来的问题是当容器中发生错误时，对错误的定位和调试也变得很复杂。当一个工具给你带来便利时，它也可能给你带来另一些麻烦。

那么有没有工具可以在本地联通 Kubernetes 集群并进行调试呢？当然是有的，这里就介绍一款研发侧利器：`Kt Connect`

## Kt Connect

`Kt Connect` 是阿里巴巴开源的一款云原生协同开发测试解决方案，目前的功能包括：

- 直接访问 Kubernetes 集群
- 转发集群流量到本地
- Service Mesh 支持
- 基于 SSH 的轻量级 VPN 网络
- 作为 kubectl 插件，集成到 Kubectl

（以上内容来自[官方文档](https://alibaba.github.io/kt-connect/#/zh-cn/)）

目前使用下来最实用的功能就是**直接连接 Kubernetes 网络**实现在本地使用 k8s 内网域名调用 Kubernetes 集群内的服务以及**将 Kubernetes 集群中的流量转发到本地**，作用类似于一个 VPN，将本地网络与 Kubernetes 集群网络连接。
![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gd4wu5p3rmj20pb0dl75m.jpg)

## 安装

`Kt Connect` 使用 Go 开发，支持 Mac、Linux 和 Windows，安装方式也很简单

前往[Github Releases](https://github.com/alibaba/kt-connect/releases) 下载可执行文件

### Mac

安装sshuttle

```bash
brew install sshuttle
```

下载并安装KT

```bash
$ curl -OL https://rdc-incubators.oss-cn-beijing.aliyuncs.com/stable/ktctl_darwin_amd64.tar.gz
$ tar -xzvf ktctl_darwin_amd64.tar.gz
$ mv ktctl_darwin_amd64 /usr/local/bin/ktctl
$ ktctl -h
```

### Linux

安装sshuttle

```bash
pip install sshuttle
```

下载并安装KT

```bash
$ curl -OL https://rdc-incubators.oss-cn-beijing.aliyuncs.com/stable/ktctl_linux_amd64.tar.gz
$ tar -xzvf ktctl_linux_amd64.tar.gz
$ mv ktctl_linux_amd64 /usr/local/bin/ktctl
$ ktctl -h
```

### Windows

下载并解压可执行文件，并确保ktctl在PATH路径下

## 本地连接集群

{{% alert note %}}
以MacOS为例
{{% /alert %}}

使用 `ktctl connect` 命令，启动的时候需要 admin 权限，需要输入密码

```bash
$ ktctl --namespace=default connect

1:51PM INF Connect Start At 69444
1:51PM INF Client address 192.168.7.121
1:51PM INF deploy shadow deployment kt-connect-daemon-rcacy in namespace default

1:51PM INF pod label: kt=kt-connect-daemon-rcacy
1:51PM INF pod: kt-connect-daemon-rcacy-fd4c587f-zmn4z is running,but not ready
1:51PM INF pod: kt-connect-daemon-rcacy-fd4c587f-zmn4z is running,but not ready
1:51PM INF Shadow pod: kt-connect-daemon-rcacy-fd4c587f-zmn4z is ready.
Forwarding from 127.0.0.1:2222 -> 22
Forwarding from [::1]:2222 -> 22
1:51PM INF port-forward start at pid: 69445
[local sudo] Password: 1:51PM INF vpn(sshuttle) start at pid: 69449
1:51PM INF KT proxy start successful
# 这里需要输入密码
Handling connection for 2222
Warning: Permanently added '[127.0.0.1]:2222' (ECDSA) to the list of known hosts.
bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
client: Connected.
```

这里可以看到在 `namespace:default` 中部署了一个 `kt-connect-daemon-*` 的 `Deployment`，如果这个 `Deployment` 启动正常，就可以直接在本地访问 Kubernetes 集群内的服务了。
```bash
$ kubectl get deploy | grep kt

kt-connect-daemon-rcacy   1/1     1            1           5m35s
```

访问集群服务，可以使用 `curl` 或者直接在浏览器访问。（这里使用之前文章[《使用 Grafana 展示肺炎疫情动态》](../feiyan-grafana)中部署的服务）

### 使用 `curl`
```bash
$ curl kk-feiyan
UP
```
### 直接使用浏览器

![image](https://tvax3.sinaimg.cn/large/ad5fbf65ly1gd4zc1ddfij20fq03zglp.jpg)

## 转发集群流量到本地

使用 `ktctl exchange` 命令，这个命令的前提条件是 Kubernetes 集群中必须有已经已经存在的 `Deployment`，在运行该命令时，将会起一个 shadow 容器，来代替已存在的 Deployment，调用该容器的流量，都会被转发到本地的指定端口。

{{% alert note %}}
要注意的是：该命令会将其代替的 Deployment 的 replicas 设置为0，可能会导致业务的暂停，请勿在生产环境中使用！
{{% /alert %}}

本地启动一个服务

![image](https://tvax4.sinaimg.cn/large/ad5fbf65ly1gd4zsl7r14j20eq03r76o.jpg)

运行命令

```bash
$ ktctl exchange kk-feiyan --expose 8088
2:13PM INF 'KT Connect' is runing, you can access local app from cluster and localhost
2:13PM INF Client address 192.168.7.121
2:13PM INF deploy shadow deployment kk-feiyan-kt-yssnq in namespace default

2:13PM INF pod label: kt=kk-feiyan-kt-yssnq
2:13PM INF pod: kk-feiyan-kt-yssnq-6464bbf74d-smvhc is running,but not ready
2:13PM INF pod: kk-feiyan-kt-yssnq-6464bbf74d-smvhc is running,but not ready
2:13PM INF Shadow pod: kk-feiyan-kt-yssnq-6464bbf74d-smvhc is ready.
2:13PM INF create exchange shadow kk-feiyan-kt-yssnq in namespace default
2:13PM INF scale deployment kk-feiyan to 0

2:13PM INF  * kk-feiyan (0 replicas) success
2:13PM INF remote 172.22.1.166 forward to local 8088
Forwarding from 127.0.0.1:2266 -> 22
Forwarding from [::1]:2266 -> 22
2:13PM INF exchange port forward to local start at pid: 70269
2:13PM INF redirect request from pod 172.22.1.166 22 to 127.0.0.1:2266 starting

Handling connection for 2266
Warning: Permanently added '[127.0.0.1]:2266' (ECDSA) to the list of known hosts.
bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
2:13PM INF ssh remote port-forward start at pid: 70270
```

查看 Deployment

```bash
$ kubectl get deploy | grep kk-feiyan
kk-feiyan                 0/0     0            0           39d    # 原服务
kk-feiyan-kt-eclcc        1/1     1            1           89s    # 转发流量服务
```

这样的话，集群内调用 `kk-feiyan` 这个服务的流量都会被转发到本地

**集群内调用：**
```bash
$ curl kk-feiyan
UP
```

**可以看到流量被抓发到了本地**
![image](https://tvax3.sinaimg.cn/large/ad5fbf65ly1gd4zuym6ofj20eq052n0d.jpg)

## 将本地服务暴露到 Kubernetes 集群

有些时候，我们并不想使用 `exchange` 来代替已经存在的 Deployment，只想在集群内新建一个服务来将流量转发到本，以完成调试。

这个时候使用 `ktctl run`，就可以满足需求，该命令会在 Kubernetes 集群中新建一个服务，并将访问该服务的流量被转发到本地的指定端口。

```bash
$ ktctl run localservice --port 8088 --expose
2:33PM INF Client address 192.168.7.121
2:33PM INF deploy shadow deployment localservice in namespace default

2:33PM INF pod label: kt=localservice
2:33PM INF pod: localservice-77d565c488-64hpp is running,but not ready
2:33PM INF pod: localservice-77d565c488-64hpp is running,but not ready
2:33PM INF Shadow pod: localservice-77d565c488-64hpp is ready.
2:33PM INF create shadow pod localservice-77d565c488-64hpp ip 172.22.1.74
2:33PM INF expose deployment localservice to localservice:8088
2:33PM INF remote 172.22.1.74 forward to local 8088
Forwarding from 127.0.0.1:2274 -> 22
Forwarding from [::1]:2274 -> 22
2:33PM INF exchange port forward to local start at pid: 70899
2:33PM INF redirect request from pod 172.22.1.74 22 to 127.0.0.1:2274 starting

Handling connection for 2274
Warning: Permanently added '[127.0.0.1]:2274' (ECDSA) to the list of known hosts.
bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
2:33PM INF ssh remote port-forward start at pid: 70903
2:33PM INF forward remote 172.22.1.74:8088 -> 127.0.0.1:8088
```

可以看到该服务已经被拉起了
```bash
$ kubectl get deploy localservice
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
localservice   1/1     1            1           86s
```

访问该服务
```bash
$ curl localservice:8088
UP
```

可以看到流量被转发到了本地
![image](https://tva3.sinaimg.cn/large/ad5fbf65ly1gd50e6lkquj20ff05z782.jpg)

## 总结

本地访问 k8s 内网，将 k8s 流量转发到本地，靠着这两大功能 `Kt Connect` 可以称之为研发侧的利器，我们可以轻松的在本地调用集群服务，或者让集群调用本地的服务，这就让开发/测试 k8s 集群中发起调用的服务，在本地断点 debug 成为了现实，非常好用。同时还有其他一些没有介绍的功能，比如：

- Service Mesh 支持，可以支持用户可以基于Service Mesh的能力做更多自定义的流量规则定义
- Dashboard 功能，管理所以使用 kt 连入集群的用户等等

值得一提的是，`ktctl run` 功能是我提出该场景并希望能实现，该 [issue](https://github.com/alibaba/kt-connect/issues/89) 提出仅一天就通过并完成了开发。给高效的开发人员点赞。
