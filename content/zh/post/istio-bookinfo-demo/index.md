---
title: "Istio初探之Bookinfo样例部署"
date: 2019-03-21T09:42:18+08:00
draft: flase
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g1agvmgy2aj21qi15odmu.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "正如Linux 的创始人 Linus Torvalds 的那句话：Talk is cheap. Show me the code. 这里我们部署一个demo，由四个单独的微服务构成（注意这里的四个微服务是由不同的语言编写的），用来演示多种 Istio 特性。"
tags: ["istio","service mesh","阿里云","华为云"]
categories: ["istio"]
keywords: ["istio","service mesh","阿里云","华为云"]
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3imuejxuj21qi15odmu.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言
之前介绍了 Istio 和 Service Mesh 能给我们带来什么，我们为什么要用 Istio ，但大家对 Istio 的认识可能还没有那么深刻。正如Linux 的创始人 [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) 的那句话：**Talk is cheap. Show me the code.** 这里我们部署一个demo，由四个单独的微服务构成**（注意这里的四个微服务是由不同的语言编写的）**，用来演示多种 Istio 特性。这个应用模仿在线书店的一个分类，显示一本书的信息。页面上会显示一本书的描述，书籍的细节（ISBN、页数等），以及关于这本书的一些评论。

## Bookinfo 应用

Bookinfo 应用分为四个单独的微服务：

- ```productpage``` ：```productpage``` 微服务会调用 ```details``` 和 ```reviews``` 两个微服务，用来生成页面。
- ```details``` ：这个微服务包含了书籍的信息。
- ```reviews``` ：这个微服务包含了书籍相关的评论。它还会调用 ratings 微服务。
- ```ratings``` ：ratings 微服务中包含了由书籍评价组成的评级信息。

这里主要使用```reviews```来演示 Istio 特性，```reviews``` 微服务有 3 个版本：

- v1 版本不会调用 ```ratings``` 服务。
- v2 版本会调用 ```ratings``` 服务，并使用 1 到 5 个黑色星形图标来显示评分信息。
- v3 版本会调用 ```ratings``` 服务，并使用 1 到 5 个红色星形图标来显示评分信息。

下图展示了这个应用的端到端架构。
![Istio 注入之前的 Bookinfo 应用](https://istio.io/docs/examples/bookinfo/noistio.svg)
<center>Istio 注入之前的 Bookinfo 应用</center>

Bookinfo 是一个异构应用，几个微服务是由不同的语言编写的。这些服务对 Istio **并无依赖**，但是构成了一个有代表性的服务网格的例子：它由多个服务、多个语言构成，并且 reviews 服务具有多个版本。

## 部署应用
这里 Istio 的安装部署就不在赘述了。

值得注意的是：如果使用的是**阿里云**容器服务安装的 Istio ，需要在 ```容器服务```-```市场```-```应用目录``` 中选择 ```gateway``` 进行安装，这里提供了多种 ```gateway``` ，我们选择 ```istio-ingressgateway```，选择直接安装的话会默认创建 ```LoadBalancer``` 类型的Service，会自动创建一个经典网络SLB，这里是可以调整的，会在后续的文章中进行详细讲解，这里不做赘述。

在 Istio 中运行这一应用，无需对应用自身做出任何改变。我们只要简单的在 Istio 环境中对服务进行配置和运行，具体一点说就是把 Envoy sidecar 注入到每个服务之中。这个过程所需的具体命令和配置方法由运行时环境决定，而部署结果较为一致，如下图所示：

![Bookinfo 应用](https://istio.io/docs/examples/bookinfo/withistio.svg)
<center>Bookinfo 应用</center>

所有的微服务都和 Envoy sidecar 集成在一起，被集成服务所有的出入流量都被 sidecar 所劫持，这样就为外部控制准备了所需的 Hook，然后就可以利用 Istio 控制平面为应用提供服务路由、遥测数据收集以及策略实施等功能。

### 下载安装
到 GitHub 中 istio 的 [release](https://github.com/istio/istio/releases) 中下载相应版本的 istio 包，下载后将 ```bin``` 目录配置到环境变量 ```PATH``` 中 ```export PATH="/istio/bin:$PATH"``` ，这里我们使用的是 ```istio 1.0.5``` 版本

Bookinfo 这个应用就在 ```samples/```目录下

## 在 阿里云容器服务（kubernetes） 中运行

启动应用容器，这里提供两种注入方法：**手工注入**和**自动注入**

- 自动注入

    需要修改 namespace ，为其添加 label 标签，这样所以在这个 namespace 中创建的应用都会被自动注入 sidecar 

    ```bash
    $ kubectl label namespace {inject-namespace} istio-injection=enabled
    $ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
    ```

- 手工注入

    需要使用 istioctl 命令生成注入后应用的配置，然后在部署应用

    ```bash
    $ istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml | kubectl apply -f -
    ```

由于是测试，这里我们使用手工注入的方法。
上面的命令会启动全部的四个服务，其中也包括了 ```reviews``` 服务的三个版本（```v1```、```v2``` 以及 ```v3```）

```bash
$ istioctl kube-inject -f bookinfo.yaml | kubectl apply -f -
service/details created
deployment.extensions/details-v1 configured
service/ratings created
deployment.extensions/ratings-v1 created
service/reviews created
deployment.extensions/reviews-v1 created
deployment.extensions/reviews-v2 created
deployment.extensions/reviews-v3 created
service/productpage created
deployment.extensions/productpage-v1 created
$ kubectl get po
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-8685d68cf9-8fwdb       2/2     Running   0          1h
productpage-v1-5fd9fddc97-tx88z   2/2     Running   0          1h
ratings-v1-7c4d756c55-cn76d       2/2     Running   0          1h
reviews-v1-5d868db586-w28q5       2/2     Running   0          1h
reviews-v2-787647c7d9-7sc52       2/2     Running   0          1h
reviews-v3-6964c86584-8728m       2/2     Running   0          1h
$ kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)            AGE
details              ClusterIP   10.11.224.17    <none>        9080/TCP           1h
productpage          ClusterIP   10.11.16.86     <none>        9080/TCP           1h
ratings              ClusterIP   10.11.244.59    <none>        9080/TCP           1h
reviews              ClusterIP   10.11.162.37    <none>        9080/TCP           1h
```

可以看到 Bookinfo 应用已经正常运行

### 指定 ingress 和 IP 的端口

1. 为为应用程序定义入口网关：

    ```bash
    $ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
    ```

2. 确认网关创建完成

    ```bash
    $ kubectl get gateway
    NAME               AGE
    bookinfo-gateway   1h
    ```

3. 快速查询访问地址，这里的是之前在阿里云上创建的 ```LoadBalancer``` 类型的 Service

    ```bash
    $ kubectl get svc istio-ingressgateway -n istio-system
    NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP       PORT(S)                  AGE
    istio-ingressgateway   LoadBalancer   10.11.18.83   xxx.xxx.xxx.xxx   80:xxx/TCP,443:xxx/TCP   2h
    ```

### 查看效果
访问 http://{EXTERNAL-IP}/productpage 注意：这里最后不能有/，否则将找不到页面
![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1ad2jg6p3j21g90mxgo7.jpg)
多次刷新浏览器，将在 ```productpage``` 中看到评论的不同的版本，它们会按照 round robin（红星、黑星、没有星星）的方式展现，这三个展示分来来自```v1```、```v2```和```v3```版本，因为还没有使用 Istio 来控制版本的路由，所以这里显示的是以轮询的负载均衡算法进行展示。

### 请求路由
BookInfo示例部署了三个版本的reviews服务，因此需要设置一个缺省路由。否则当多次访问该应用程序时，会发现有时输出会包含带星级的评价内容，有时又没有。出现该现象的原因是当没有为应用显式指定缺省路由时，Istio会将请求随机路由到该服务的所有可用版本上。

在使用 Istio 控制 Bookinfo 版本路由之前，你需要在目标规则中定义好可用的版本 。

运行以下命令为 Bookinfo 服务创建的默认的目标规则：

- 如果不需要启用双向TLS，请执行以下命令：

    ```bash
    $ kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
    ```

- 如果需要启用双向 TLS，请执行以下命令：

    ```bash
    $ kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
    ```

    等待几秒钟，等待目标规则生效。你可以使用以下命令查看目标规则：

    ```bash
    kubectl get destinationrules
    NAME          AGE
    details       28s
    productpage   28s
    ratings       28s
    reviews       28s
    ```

### 将所有微服务的缺省版本设置为v1
通过运行如下命令，将所有微服务的缺省版本设置为v1：

```bash
$ kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

可以通过下面的命令来显示所有已创建的路由规则：

```bash
$ kubectl get virtualservices
NAME       AGE
bookinfo      33m
details       8s
productpage   8s
ratings       8s
reviews       8s
```

显示已创建的详细路由规划：

```bash
$ kubectl get virtualservices -o yaml
```

由于路由规则是通过异步方式分发到代理的，过一段时间后规则才会同步到所有pod上。因此需要等几秒钟后再尝试访问应用。

在浏览器中打开 Bookinfo 应用程序的URL: http://{EXTERNAL-IP}/productpage。

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1adqyf9dej21g70oitbd.jpg)

可以看到 Bookinfo 应用程序的 ```productpage``` 页面，显示的内容中不包含带星的评价信息，这是因为 ```reviews:v1``` 服务不会访问ratings服务。

### 将来自特定用户的请求路由到reviews:v2
本例中，首先使用 Istio 将100%的请求流量都路由到了 Bookinfo 服务的```v1```版本；然后再设置了一条路由规则，路由规则基于请求的 header（例如一个用户cookie）选择性地将特定的流量路由到了 ```reviews``` 服务的```v2```版本。

通过运行如下命令，把来自测试用户"jason"的请求路由到 ```reviews:v2 ```，以启用ratings服务。

```bash
$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

通过如下命令确认规则是否创建：

```bash
$ kubectl get virtualservice reviews -o yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
    {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"match":[{"headers":{"end-user":{"exact":"jason"}}}],"route":[{"destination":{"host":"reviews","subset":"v2"}}]},{"route":[{"destination":{"host":"reviews","subset":"v1"}}]}]}}
creationTimestamp: "2019-03-21T06:01:10Z"
generation: 1
name: reviews
namespace: default
resourceVersion: "62486214"
selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/reviews
uid: b9e41681-4b9e-11e9-a679-00163e045478
spec:
hosts:
- reviews
http:
- match:
    - headers:
        end-user:
        exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
- route:
    - destination:
        host: reviews
        subset: v1
```

确认规则已创建之后，在浏览器中打开BookInfo应用程序的URL: http://{EXTERNAL-IP}/productpage。

以"jason"用户登录 ```productpage``` 页面，您可以在每条评价后面看到星级信息。

这里登录用户名为 ```jason``` ，密码随便输入即可

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1adtjugp3j21gb0iygoa.jpg)

### 流量转移
除了基于内容的路由，Istio还支持基于权重的路由规则。

首先，将所有微服务的缺省版本设置为v1：

```bash
$ kubectl replace -f samples/bookinfo/networking/virtual-service-all-v1.yaml
```

其次，使用下面的命令把50%的流量从reviews:v1转移到reviews:v3:

```bash
$ kubectl replace -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

在浏览器中多次刷新productpage页面，大约有50%的几率会看到页面中出现带红星的评价内容。

说明： 注意该方式和使用容器编排平台的部署特性来进行版本迁移是完全不同的。容器编排平台使用了实例scaling来对流量进行管理。而通过Istio，两个版本的reviews服务可以独立地进行扩容和缩容，并不会影响这两个版本服务之间的流量分发。

如果觉得 ```reviews：v3``` 微服务已经稳定，你可以通过以下命令， 将 ```virtual service``` 100％的流量路由到 ```reviews：v3```，从而实现一个灰度发布的功能。

```bash
$ kubectl replace -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
```

## 在华为云（CCE）上运行
华为云率先将 Istio 作为产品投入到公有云中进行商业应用，开通方式十分简单，只要在华为云CCE上创建集群，然后申请 Istio 公测即可。

为了方便测试 Bookinfo 应用在华为云上提供了一键体验应用，点击即可省去刚刚那一系列的 ```kubectl``` 操作

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afbs7oq4j21g90id0vv.jpg)
<center>一键创建体验应用</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afgth1cgj219b0a7tb1.jpg)
<center>点击灰度发布即可</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afjc5hvgj21fv0o1q6q.jpg)
<center>创建金丝雀发布</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afnqyqlhj20ze0o00vl.jpg)
<center>选择灰度发布的组件</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afp1c5ltj20zk0le765.jpg)
<center>填写版本号</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afq846bjj20z80nowgl.jpg)
<center>选择镜像版本</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afra8rmhj21050mfgpb.jpg)
<center>版本创建完成后配置灰度策略</center>

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1afwpan6qj21090mste1.jpg)
<center>选择相应策略，策略下发即可</center>

总的来说，华为云的 Istio 确实已经是商业化应用，这里只是展示了部分灰度发布的功能。其他比如流量治理，流量监控等功能还没展示，这些功能做的十分细致，值得尝试。
## 参考
[在Kubernetes上基于Istio实现Service Mesh智能路由](https://help.aliyun.com/document_detail/90563.html?spm=a2c4g.11186623.6.759.5dbd1f5fSB2m9T)

[基于ISTIO服务网格的灰度发布](https://support.huaweicloud.com/bestpractice-cce/cce_bestpractice_0012.html)