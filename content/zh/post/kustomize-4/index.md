---
title: "使用 Kustomize 帮你管理 kubernetes 应用（四）：简述核心配置 kustomization.yaml"
date: 2019-05-23T12:50:12+08:00
draft: false
type: blog
banner: "https://ws4.sinaimg.cn/large/ad5fbf65gy1g3bbl0silkj21qi15o7bb.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本篇为系列文章第四篇，将简述 kustomize 的核心配置文件 kustomization.yaml"
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3j5pw2egj21qi15o7bb.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

在前面的文章中已经介绍了 kustomize 是什么，以及如何开始使用和如何简单的在 CI/CD 中使用，本篇文章将会介绍 kustomize 的核心文件 [kustomization.yaml](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/zh/kustomization.yaml)。

另外，博主已经向 kustomize 贡献了中文文档，已被官方采纳，现在在 kustomize 中的 [`docs/zh`](https://github.com/kubernetes-sigs/kustomize/tree/master/docs/zh) 目录中就可看到，翻译的不好的地方欢迎指正。同时也在 GitHub 上新建了一个 名为 [kustomize-lab](https://github.com/sunny0826/kustomize-lab) 的 repo 用于演示 kustomize 的各种用法及技巧，本文中介绍的内容也会同步更新到该 repo 中，欢迎 fork、star、PR。

## `kustomization.yaml` 的作用

> Kustomize 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件。

有前面的文章[《使用 Kustomize 帮你管理 kubernetes 应用（二）： Kustomize 的使用方法》](../kustomize-2)中已经介绍了，每个 `base` 或 `overlays` 中都必须要有一个 `kustomization.yaml`，这里我们看一下官方示例 `helloWorld` 中的 `kustomization.yaml`：

```YAML
commonLabels:
  app: hello

resources:
- deployment.yaml
- service.yaml
- configMap.yaml
```
可以看到该项目中包含3个 resources ， `deployment.yaml`、`service.yaml` 、 `configMap.yaml`。

```bash
.
└── helloWorld
    ├── configMap.yaml
    ├── deployment.yaml
    ├── kustomization.yaml
    └── service.yaml
```

直接执行命令：
```bash
kustomize build helloWorld
```

就可以看到结果了：
```YAML
apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  labels:
    app: hello
  name: the-map
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello
  name: the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello
    deployment: hello
  type: LoadBalancer
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello
  name: the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        deployment: hello
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080
```
从上面的结果可以看大 kustomize 通过 `kustomization.yaml` 将3个 resources 进行了处理，给三个 resources 添加了共同的 labels `app: hello` 。这个示例展示了 `kustomization.yaml` 的作用：**将不同的 resources 进行整合，同时为他们加上相同的配置**。

## 进阶使用

上面只不过是一个简单的示例，下面将结合实际情况分享一些比较实用的用法

### 根据环境生成不同配置

在实际的使用中，使用最多的就是为不同的环境配置不同的 `deploy.yaml`，而使用 kustomize 可以把配置拆分为多个小的 patch ，然后通过 kustomize 来进行组合。而根据环境的不同，每个 patch 都可能不同，包括分配的资源、访问的方式、部署的节点都可以自由的定制。

```bash
.
├── flask-env
│   ├── README.md
│   ├── base
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   └── service.yaml
│   └── overlays
│       ├── dev
│       │   ├── healthcheck_patch.yaml
│       │   ├── kustomization.yaml
│       │   └── memorylimit_patch.yaml
│       └── prod
│           ├── healthcheck_patch.yaml
│           ├── kustomization.yaml
│           └── memorylimit_patch.yaml
```
这里可以看到配置分为了 `base` 和 `overlays`， `overlays` 则是继承了 `base` 的配置，同时添加了诸如 healthcheck 和 memorylimit 等不同的配置，那么我们分别看一下 `base` 和 `overlays` 中 `kustomization.yaml` 的内容

- base

```YAML
commonLabels:
  app: test-cicd

resources:
- service.yaml
- deployment.yaml
```
`base` 中的 `kustomization.yaml` 中定义了一些基础配置

- overlays

```YAML
bases:
- ../../base
patchesStrategicMerge:
- healthcheck_patch.yaml
- memorylimit_patch.yaml
namespace: devops-dev
```
`overlays` 中的 `kustomization.yaml` 则是基于 `base` 新增了一些个性化的配置，来达到生成不同环境的目的。

执行命令
```bash
kustomize build flask-env/overlays/dev
```

结果
```YAML
apiVersion: v1
kind: Service
metadata:
  labels:
    app: test-cicd
  name: test-cicd
  namespace: devops-dev
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: test-cicd
  type: ClusterIP
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test-cicd
  name: test-cicd
  namespace: devops-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-cicd
  template:
    metadata:
      labels:
        app: test-cicd
        version: 0.0.3
    spec:
      containers:
      - env:
        - name: ENV
          value: dev
        image: guoxudongdocker/flask-python:latest
        imagePullPolicy: Always
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 20
        name: test-cicd
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 20
        resources:
          limits:
            cpu: 300m
            memory: 500Mi
          requests:
            cpu: 300m
            memory: 500Mi
        volumeMounts:
        - mountPath: /etc/localtime
          name: host-time
      imagePullSecrets:
      - name: registry-pull-secret
      volumes:
      - hostPath:
          path: /etc/localtime
        name: host-time
```
可以看到包括 `replicas`、`limits`、`requests`、`env` 等 dev 中个性的配置都已经出现在了生成的 yaml 中。由于篇幅有限，这里没有把所有的配置有罗列出来，需要的可以去 [GitHub](https://github.com/sunny0826/kustomize-lab) 上自取。


## 结语

上面所有的 `kustomize build dir/` 都可以使用 `kubectl apply -k dir/` 实现，但是需要 `v14.0` 版以上的 `kubectl`，也就是说，其实我们在集成到 CI/CD 中的时候，甚至都不需要用来 `kustomize` 命令集，有 `kubectl` 就够了。

由于篇幅有限，这里没法吧所有 `kustomization.yaml` 的用途都罗列出来，不过可以在官方文档中找到我提交的中文翻译版 [`kustomization.yaml`](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/zh/kustomization.yaml)，可以直接去官方 GitHub 查看。同时 [kustomize-lab](https://github.com/sunny0826/kustomize-lab) 会持续更行，敬请关注。