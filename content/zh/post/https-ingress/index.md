---
title: "为ingress配置SSL证书，实现HTTPS访问"
date: 2018-12-29T21:28:13+08:00
draft: false
type: blog
tags: ["kubernetes","阿里云","rancher"]
categories: ["kubernetes"]
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t15tmebvj21qi15oah8.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "devops平台率先在公司内使用kubernetes集群提供后端服务，但是由于之前一直处于探索阶段，所以使用的事http的方式提供后端服务，但是在开发统一入口后，出现了访问HTTPS页面的跨域问题，由此引出了后端服务配置SSL证书的问题。"
keywords: ["kubernetes","阿里云","rancher"]
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3ik1hhcbj21qi15oah8.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>devops平台率先在公司内使用kubernetes集群提供后端服务，但是由于之前一直处于探索阶段，所以使用的事http的方式提供后端服务，但是在开发统一入口后，出现了访问HTTPS页面的跨域问题，由此引出了后端服务配置SSL证书的问题

# 使用rancher配置SSL证书

## 下载SSL证书文件
首先需要获得SSL证书文件，可以直接在阿里云SSL证书管理控制台下载

选中需要下载证书，选择下载nginx证书
![image](/images/source/zhengshu.png)

## 将证书上传项目
打开rancher，选择要使用证书的项目，点击资源中的证书
## 将证书上传项目
打开rancher，选择要使用证书的项目，点击资源中的证书
![image](/images/source/https-1.png)
添加证书，点击从文件上传
![image](/images/source/https-2.png)
上传证书文件中的秘钥和证书，点击保存即可

# 使用yaml上传证书
这个证书的原理其实是在相应的命名空间创建了一个包含证书信息的secrets

```yaml
apiVersion: v1
data:
    tls.crt: {私钥}
    tls.key: {证书}
kind: Secret
metadata:
    name: keking-cn
    namespace: devops-plat
type: kubernetes.io/tls
```

在kubernetes上运行该yaml即可

# rancher中证书绑定
选中需要绑定证书的ingress，点击编辑，选中证书，保存即可（由于ingress-controller中没有绑定默认证书，所以这里不能选中默认）
![image](/images/source/https-3.png)
保存完毕，证书即可生效