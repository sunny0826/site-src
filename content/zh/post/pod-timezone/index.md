---
title: "kubernetes中pod同步时区问题"
date: 2019-01-30T20:18:13+08:00
draft: false
type: blog
tags: ["kubernetes"]
categories: ["问题解决"]
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t1f80hkgj21qo15oglu.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "解决容器中时区问题。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3jag24tsj21qo15oglu.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>新版监控大屏于18年最后一天正式上线，之后陆续进行了几次优化和修改，最近发现一个比较大的bug，就是监控显示的时间轴不对，显示的就是和目前的时间相差8小时，这就引出了docker中的时区问题

# 问题的原因
默认的情况，在K8S里启动一个容器，该容器的设置的时区是UTC0，但是对用户而言，主机环境并不在UTC0。我们在UTC8。如果不把容器的时区和主机主机设置为一致，则在查找日志等时候将非常不方便，也容易造成误解。但是K8S以及Docker容器没有一个简便的设置/开关在系统层面做配置。都需要我们从单个容器入手做设置，具体有两个方法：

* 直接修改镜像的时间设置，好处是应用部署时无需做特殊设置，但是需要手动构建Docker镜像。
* 部署应用时，单独读取主机的“/etc/localtime”文件，即创建pod时同步时区，无需修改镜像，但是每个应用都要单独设置。

# 问题的解决
这里我们选择第二种方法，即修改部署应用的yaml文件，创建pod时同步时区

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: myweb
spec:
    replicas: 2
    template:
        metadata:
        labels:
            app: myweb
        spec:
        containers:
        - name: myweb
            image: nginx:apline
            ports:
            - containerPort: 80
        #挂载到pod中
            volumeMounts:
            - name: host-time
            mountPath: /etc/localtime    
        #需要被挂载的宿主机的时区文件
        volumes:
        - name: host-time
            hostPath:
            path: /etc/localtime
```

# 效果对比
## 修改时区前
![image](/images/source/time-1.png)
## 修改时区后
![image](/images/source/time-2.png)