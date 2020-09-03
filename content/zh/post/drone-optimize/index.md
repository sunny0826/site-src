---
title: "玩转 Drone CI"
date: 2019-09-11T13:53:09+08:00
draft: false
type: blog
banner: "https://wx1.sinaimg.cn/large/ad5fbf65gy1g6vnfwh6a2j21hc120dpl.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "通过这篇文章总结一下目前我们对 drone 进行了一些定制化开发以及使用技巧，由于 drone 官方的文档不是很详细，所以也希望通过这种方法来和其他使用 drone 的用户分享和交流使用经验。"
tags: ["devops","drone","工具"]
categories: ["devops"]
keywords: ["devops","drone","工具"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3ighqw3uj21hc120dpl.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## 前言

使用 drone CI 已有小半年，在将原有项目的 CI 系统从 jenkins 向 drone 迁移的时候，也陆陆续续遇到了一些问题。在这段时间，也完成了使用官方插件到插件定制的转变，使得 drone CI 流程更贴合我们 devops 开发流程。通过这篇文章总结一下目前我们对 drone 进行的一些定制化开发以及使用技巧，由于 drone 官方的文档不是很详细，所以也希望通过这种方法来和其他使用 drone 的用户分享和交流使用经验。

## 并行构建

在默认情况下，drone 会按照步骤执行，但是有时会遇到前后端在同一个 repo 的情况，这时使用并行构建就可以省去很多的构建时间。

### 构建流程

在下面的示例里会展示一个如下流程：repo 中包含一个由 Java 写的服务以及一个 vue 前端项目，maven 构建和 npm 构建同时进行，maven 构建成功后会镜像 docker 镜像构建并上传镜像仓库，docker 构建成功后会镜像 k8s 部署，部署成功后会进行 vue 项目前端发布，在 k8s 部署成功并且前端发布成功后，进行钉钉构建成功同时，否则进行钉钉构建失败通知。

```text
前端构建 ————————————          前端发布
                    \      /        \
                     \    /       钉钉通知
                      \  /          /
后端构建 —— 镜像构建 —— k8s部署 ——————

```
### .drone.yml 配置

```yaml
kind: "pipeline"
name: "default"
steps:
  - name: "Maven编译"
    image: "guoxudongdocker/drone-maven"
    commands:
      - "mvn clean install"
    depends_on: [ "clone" ]
  - name: "构建镜像"
    image: "guoxudongdocker/drone-docker"
    settings:
      username:
        from_secret: "docker_user"
      password:
        from_secret: "docker_pass"
      dockerfile: "Dockerfile"
      repo: "registry-vpc.cn-shanghai.aliyuncs.com/guoxudong/test"
      registry: "registry-vpc.cn-shanghai.aliyuncs.com"
      tags: "${DRONE_BUILD_NUMBER}"
    depends_on: [ "Maven编译" ]
  - name: "Kubernetes 部署"
    image: "guoxudongdocker/kubectl"
    settings:
      config: "deploy/overlays/uat"
      timeout: 300
      check: false
    depends_on: [ "构建镜像" ]
  - name: "前端构建"
    image: "guoxudongdocker/node-drone"
    commands:
      - "npm install"
      - "npm run build"
    depends_on: [ "clone" ]
  - name: "前端上传"
    image: "guoxudongdocker/node-drone"
    commands:
      - "do something"
    depends_on: [ "前端构建","Kubernetes 部署" ]
  - name: "钉钉通知"
    image: "guoxudongdocker/drone-dingtalk"
    settings:
      token:
        from_secret: "dingding"
      type: "markdown"
      message_color: true
      message_pic: true
      sha_link: true
    depends_on: [ "前端上传","Kubernetes 部署" ]
    when:
      status:
        - "failure"
        - "success"
```

## 多子项目构建

在使用 drone 中遇到的最大问题就是，我们有很多项目都是在一个 repo 中有很多子项目，而每个子项目都是 k8s 中的一个服务，这时一个 `.drone.yml` 文件很难把所有的服务都囊括。而又不想每个子项目拉一个分支管理，当前的模式就很不合适。

### 插件开发

针对这个问题，我们对 drone 进行了定制化开发，会在每次提交代码后，对新提交的代码和老代码进行比较，筛选出做了修改的子项目，然后对有修改的子项目尽心 CI ，其余的子项目则不进行发布。

而以上的方式仅适用于测试环境的快速迭代，生产环境则采用 tag 的模式，针对不同的子项目，打不同前缀的 tag ，比如子项目为 test1 ，则打 `test1-v0.0.1` 的 tag，就会对该子项目进行生产发布。

### 构建效果

- 有修改的子项目

    ![image](https://ws1.sinaimg.cn/large/ad5fbf65gy1g6vm2ul2zfj21ky148jx0.jpg)

- 无修改的子项目

    ![image](https://wx1.sinaimg.cn/large/ad5fbf65gy1g6vm49on4kj21jk11iaf7.jpg)


## Kubernetes 发布状态检查

之前的 Kubernetes 发布只是将服务发布到 Kubernetes 集群，并不管服务是否正常启动。针对这个问题以及我们的 Kubernetes 应用管理模式，我们开发了 drone 的 Kubernetes 发布插件，该插件包括 `kubectl` 、`kustomize`、`kubedog` ，来完善我们的 Kubernetes 发布 step 。

> .drone.yml

```yaml
steps:
- name: Kubernetes 部署
  image: guoxudongdocker/kubectl
  volumes:
  - name: kube
    path: /root/.kube
  settings:
    check: false                 # 该参数为是否开启子模块检查
    config: deploy/overlays/uat  # 这里使用 kustomize ,详细使用方法请见 https://github.com/kubernetes-sigs/kustomize
    timeout: 300                 # kubedog 的检测超时
    name: {your-deployment-name} # 如果开启子模块检查则需要填入子模块名称

...

volumes:
- name: kube
  host:
    path: /tmp/cache/.kube  # kubeconfig 挂载位置

trigger:
  branch:
  - master  # 触发 CI 的分支
```

使用该插件会如果为测试构建，则会自动设置 docker 镜像 tag 为 `DRONE_BUILD_NUMBER` ；如果为生产构建（git tag），则叫自动设置 docker 镜像 tag 为 `DRONE_TAG` ，然后通过 `kubectl apply -k .` 进行部署，同时使用 `kubedog` 进行部署状态检查，如果服务正常启动则该 step 通过，如果超时或者部署报错则该 step 失败。

## 结语

根据我们目前的开发模式，对 drone 插件进行了全方位的开发。由于 dockerhub 的镜像拉取经常超时，则将镜像推送到了我们自己的镜像仓库；对钉钉通知也进行了优化；同时也根据我们目前的开发语言进行了插件的开发，提供了基于 Java 、Python 以及 Node.js 的 drone 插件，基本可以满足我们现在的 CI 需求，但随着 drone 的深入使用，越来越多的问题将会暴露出来。后续将会不断解决遇到的问题，持续优化。
