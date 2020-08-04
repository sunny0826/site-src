---
title: "阿里云环境Istio初探"
date: 2019-03-13T15:45:43+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g117v91a61j21qi15otcd.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "极简的istio样例部署，可以帮助新手快速入门，相较官方提供的Bookinfo应用更容易上手。"
tags: ["istio","service mesh","阿里云"]
categories: ["istio"]
keywords: ["istio","service mesh","阿里云"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3inbf8ahj21qi15otcd.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# istio应用部署样例

该实例为一套istio服务上线流程：```注入```->```部署```->```创建目标规则```->```创建默认路由```。就大多数istio服务网格应用均可基于这一流程上线。

### 部署istio
istio有多种部署方式，阿里云、华为云等云服务商均提供一键安装，同时也可以通过GitHub下载release包，使用```install/kubernetes/istio-demo.yaml```部署，或者使用helm部署。**这里采用阿里云容器服务一键部署istio**。

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g117xxixlvj20a00ajdgb.jpg)

### 部署两个版本的服务
这里选择一个简单的Python项目作为服务端，这里使用[崔秀龙](https://github.com/fleeto)老哥的[flaskapp](https://github.com/fleeto/flaskapp/blob/master/app/main.py)服务，该服务的作用就是提供2个url路径：
    
- 一个是/env，用户获取容器中的环境变量，例如 http://flaskapp/env/version
- 另一个是/fetch ，用于获取在参数url中指定的网址的内容，例如 http://flaskapp/fetch?url=http://weibo.com

创建2个Deployment，分别命名为 flaskapp-v1 和 flaskapp-v2 ，同时创建一个 Service ,将其命名为flaskapp。代码文件为 ```flaskapp.istio.yaml```。

```yaml
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: v1
kind: Service
metadata:
name: flaskapp
labels:
    app: flaskapp
spec:
selector:
    app: flaskapp
ports:
- name: http
    port: 80
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
name: flaskapp-v1
spec:
replicas: 1
template:
    metadata:
    labels:
        app: flaskapp
        version: v1
    spec:
    containers:
    - name: flaskapp
        image: dustise/flaskapp
        imagePullPolicy: IfNotPresent
        env:
        - name: version
        value: v1
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
name: flaskapp-v2
spec:
replicas: 1
template:
    metadata:
    labels:
        app: flaskapp
        version: v2
    spec:
    containers:
    - name: flaskapp
        image: dustise/flaskapp
        imagePullPolicy: IfNotPresent
        env:
        - name: version
        value: v2
```

**注意**

- 两个版本Deployment的镜像一致，但是使用了不同的version标签区分，分别为 v1 和 v2 。实际环境中的镜像是不同的
- 在两个Deployment中都有一个名为version的环境变量，分别为 v1 和 v2 。这里设置是为了方便后续区分服务。
- 两个Deployment中都使用了 app 和 version 标签，在 istio 网格应用中通常会使用这两个标签作为应用和版本的标识。
- Service 中的 Selector 仅使用了一个 app 标签，这意味着该 Service 对两个 Deployment 都是有效的。
- 将在 Service 中定义的端口根据 **istio 规范**命名为http。

**istio注入并部署服务端**

```bash
$ istioctl kube-inject -f flask.istio.yaml | kubectl apply -f -
service/flaskapp created
deployment.extensions/flaskapp-v1 created
deployment.extensions/flaskapp-v2 created
```

在rancher查看注入情况

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g1045ku3dcj20cj05kglp.jpg)

这里也可以使用```kubectl describe po flaskapp-v1-7d4f9b8459-2ncnf```命令查看Pod容器，这里可以看到Pod中多了一个容器，名为```istio-proxy```，这就表示注入成功了。而前面```istio-init```的初始化容器，这个容器是用于初始化劫持的。

### 部署客户端

这里的客户端是一个安装了测试工具的镜像，测试的内容可以在容器内通过shell完成。代码文件为 ```sleep.istio.yaml```。

```yaml
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: v1
kind: Service
metadata:
name: sleep
labels:
    app: sleep
    version: v1
spec:
selector:
    app: sleep
    version: v1
ports:
- name: ssh
    port: 80
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
name: sleep
spec:
replicas: 1
template:
    metadata:
    labels:
        app: sleep
        version: v1
    spec:
    containers:
    - name: sleep
        image: dustise/sleep
        imagePullPolicy: IfNotPresent
```

**istio注入并部署客户端**

```bash
$ istioctl kube-inject -f sleep.istio.yaml | kubectl apply -f -
service/sleep created
deployment.extensions/sleep created
```

**```sleep```应用的Pod进入Running状态就可以进行验证了**

### 验证服务

直接在sleep容器中执行命令行

```bash
$ for i in `seq 10`;do http --body http://flaskapp/env/version;done
v1
v2
...
v1
```

该命令使用一个for循环，重复访问 http://flaskapp/env/version ，查看内容，结果为 v1 和 v2 随机出现，各占一半。出现 v1 和 v2 版本轮流调用的效果，达到了基本的负载均衡的功能。

### 创建目标规则

目标规则代码 ```flaskapp-destinationrule.yaml```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
name: flaskapp
spec:
host: flaskapp
subsets:
- name: v1
    labels:
    version: v1
- name: v2
    labels:
    version: v2
```

**部署目标规则（这里使用kubectl和istioctl均可）**

```bash
$ kubectl apply -f flaskapp-destinationrule.yaml
Created config destination-rule/default/flaskapp at revision 59183403
```

### 创建默认路由

默认路由代码 ```flaskapp-default-vs-v2.yaml```

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
name: flaskapp-default-v2
spec:
hosts: 
- flaskapp
http:
- route:
    - destination:
    host: flaskapp
    subset: v2
```

**部署默认路由**

```bash
$ kubectl apply -f flaskapp-default-vs-v2.yaml
Created config virtual-service/default/flaskapp-default-v2 at revision 59185583
```

### 验证路由规则是否生效

再次在sleep容器中执行命令，查看新定义的流量管理规则是否生效

```bash
$ for i in `seq 10`;do http --body http://flaskapp/env/version;done
v2
v2
v2
v2
v2
v2
v2
v2
v2
v2
```

这里就可以看到，设置的默认路由已经生效了，多次重复访问，返回的内容都是来自环境变量 version 设置为 v2 的版本，也就是v2版本。

#### kiali查看调用情况

![image](http://wx4.sinaimg.cn/large/ad5fbf65ly1g104tydblxj21az0li40i.jpg)

可以看到流量都进入了v2版本中

### 小结

这里实现了一个极简的istio应用，可以帮助新手快速入门，官网提供的Bookinfo应用较为复杂。这里提供的小例子更为简洁易懂，非常利于入门。

### 参考
- [《深入浅出Istio》](https://github.com/fleeto/istio-for-beginner)    ---   崔秀龙
