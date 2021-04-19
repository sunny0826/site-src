---
title: "解决kubernetes中ingress-nginx配置问题"
date: 2019-03-06T14:42:05+08:00
draft: false
type: blog
banner: "https://tva2.sinaimg.cn/large/ad5fbf65gy1g61hrqctjnj20dw099aa9.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "随着公司容器化的深入，越来越多的服务陆续迁移到kubernetes集群中，有些问题在测试环境并未凸显，但是在生产环境中这些问题就显得格外的扎眼。这里就对实践中kubernetes集群中的7层负载均衡器ingress遇到的问题进行总结。"
tags: ["kubernetes","容器"]
categories: ["kubernetes"]
keywords: ["kubernetes","容器"]
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3ipmye8jj20dw099aa9.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

随着公司容器化的深入，越来越多的服务陆续迁移到kubernetes集群中，有些问题在测试环境并未凸显，但是在生产环境中这些问题就显得格外的扎眼。这里就对实践中kubernetes集群中的7层负载均衡器ingress遇到的问题进行总结。

## HTTP(S)负载均衡器-ingress

Ingress是kubernetes API的标准资源类型之一，其本质就是一组基于DNS名称(host)或URL路径把请求转发至指定的Service资源的规则，**用于将集群外的请求流量转发至集群内部完成服务发布**。

Ingress控制器(Ingress Controller)可以由任何具有反向代理(HTTP/HTTPS)功能的服务程序实现，如Nginx、Envoy、HAProxy、Vulcand和Traefik等。Ingress控制器本身也作为Pod对象与被代理的运行为Pod资源的应用运行于同一网络中。我们在这里选择了NGINX Ingress Controller，由于对NGINX的配置较为熟悉，同时我们使用的kubernetes是阿里云的容器服务，构建集群的时候，容器服务会自带NGINX Ingress Controller。

![image](http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t3yj7wecj20w50doab9.jpg)

## 根据实际情况Ingress调优

### 1. 解决400 Request Header Or Cookie Too Large问题
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
    
#### 现象

微信小程序需要调用后端接口，需要在header中传一段很长的token参数，直接使用浏览器访问该端口可以访问通，但是在加上token访问之后，会报“400 Request Header Or Cookie Too Large”

```html
<html>
    <head>
        <title>400 Request Header Or Cookie Too Large</title>
    </head>
    <body>
        <center>
            <h1>400 Bad Request</h1>
        </center>
        <center>Request Header Or Cookie Too Large</center>
        <hr>
        <center>nginx/1.15.6</center>
    </body>
</html>
```

#### 问题定位

直接修改Service使用nodeport的形式访问，则没有报错，初步定位需要在ingress中nginx配置客户端的请求头，进入Ingress Controller的Pod查询配置，果然是请求头空间不足。

```bash
$ cat nginx.conf | grep client_header_buffer_size
    client_header_buffer_size       1k;
$ cat nginx.conf | grep large_client_header_buffers
    large_client_header_buffers     4 8k;
```

#### 解决方法

在ingress中添加注释

```nginx
nginx.ingress.kubernetes.io/server-snippet: client_header_buffer_size 2046k;
```
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
**Server snippet**<br>Using the annotation ```nginx.ingress.kubernetes.io/server-snippet``` it is possible to add custom configuration in the server configuration block.
<br>该注释是将自定义配置加入nginx的server配置中

image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

### 2. 解决请求超时问题

#### 现象

有一个数据导出功能，需要将大量数据进行处理，然后以Excel格式返回，在导出一个大约3W条数据的时候，出现访问超时情况。

![image](https://tva2.sinaimg.cn/mw690/ad5fbf65ly1g0ubdwwzo5j21b30bjaat.jpg)

#### 解决方法

调整proxy_read_timeout，连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理
在ingress中添加注释 

```nginx
nginx.ingress.kubernetes.io/proxy-read-timeout: 600
```

>这里需要注意的事该注释的value需要时number类型，不能加s，否则将不生效

### 3. 增加白名单

#### 现象

在实际的使用中，会有一部分应用需要设置只可以在办公场地的网络使用，之前使用阿里云 SLB 的时候可以针对端口进行访问控制，但是现在走 ingress ，都是从80 or 443端口进，所以需要在 ingress 设置

#### 解决方法

> **Whitelist source range**

>You can specify allowed client IP source ranges through the nginx.ingress.kubernetes.io/whitelist-source-range annotation. The value is a comma separated list of CIDRs, e.g. 10.0.0.0/24,172.10.0.1.

在 ingress 里配置 ```nginx.ingress.kubernetes.io/whitelist-source-range``` ，如有多个ip段，用逗号分隔即可

```nginx
nginx.ingress.kubernetes.io/whitelist-source-range: 10.0.0.0/24
```
如果想全局适用，可以在阿里云 SLB 里操作，也可以将该配置加入到 ```NGINX ConfigMap``` 中。

image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
根据工作中遇到的实际问题，持续更新中...

## 总结
使用NGINX ingress controller的好处就是对于nginx配置相对比较熟悉，性能也不差。相关nginx配置的对应的ingress可以在 https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/ 上查到。