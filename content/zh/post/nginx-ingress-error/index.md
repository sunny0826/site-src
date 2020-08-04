---
title: "解决 Nginx-Ingress 重定向失败问题"
date: 2019-08-16T11:15:37+08:00
draft: false
type: blog
banner: "https://wx2.sinaimg.cn/large/ad5fbf65gy1g61duu66q4j20dw08ft8w.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "记录 Nginx-Ingress 重定向失败问题的解决过程。"
tags: ["kubernetes","容器"]
categories: ["kubernetes"]
keywords: ["kubernetes","容器","ingress","nginx-ingress-controller"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3j9d3bf0j20dw08ft8w.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## 前言

最近对公司 Kubernetes 集群的 `nginx-ingress-controller` 进行了升级，但是升级后却出现了大问题，之前所有采用 `nginx.ingress.kubernetes.io/rewrite-target: /` 注释进行重定向的 Ingress 路由全部失效了，但是那些直接解析了域名，没有进行重定向的却没有发生这个问题。

## 问题分析

1. 首先检查对应服务健康状态，发现所有出问题的服务的状态均正常，同时受影响的之后 http 调用，而 RPC 调用却不受影响，这时问题就定位到了 ingress。
2. 然后检查 nginx-ingress-controller ，发现 nginx-ingress-controller 的状态也是正常的，路由也是正常的。
3. 最后发现受影响的只有添加了重定向策略的 ingress 。

## 问题解决

问题已经定位，接下来就是着手解决问题，这时候值得注意的就是之前进行了什么变更：升级了 nginx-ingress-controller 版本！看来问题就出现在新版本上，那么就打开官方文档：https://kubernetes.github.io/ingress-nginx/examples/rewrite/ 看一下吧。

### Attention

>Starting in Version 0.22.0, ingress definitions using the annotation `nginx.ingress.kubernetes.io/rewrite-target` are not backwards compatible with previous versions. In Version 0.22.0 and beyond, any substrings within the request URI that need to be passed to the rewritten path must explicitly be defined in a [capture group](https://www.regular-expressions.info/refcapture.html).

文档上给出了非常明显的警告⚠️：从 V0.22.0 版本开始将不再兼容之前的入口定义，再查看一下我的 nginx-ingress-controller 版本，果然问题出现来这里。


### Note

>[Captured groups](https://www.regular-expressions.info/refcapture.html) are saved in numbered placeholders, chronologically, in the form `$1`, `$2` ... `$n`. These placeholders can be used as parameters in the `rewrite-target` annotation.

### 示例

到这里问题已经解决了，在更新了 ingress 的配置之后，之前所有无法重定向的服务现在都已经可以正常访问了。修改见如下示例：

```bash
$ echo '
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
' | kubectl create -f -
```

## 结语

解决这个问题的实际时间虽然不长，但是着实让人出了一身冷汗，同时也给了我警示：变更有风险，升级需谨慎。在升级之前需要先浏览新版本的升级信息，同时需要制定完善的回滚策略，确保万无一失。
