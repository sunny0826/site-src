---
title: "使用 Kustomize 帮你管理 kubernetes 应用（五）：配合 kubedog 完善 CI/CD 的最后一步"
date: 2019-07-03T15:20:31+08:00
draft: false
type: blog
banner: "http://wx2.sinaimg.cn/large/ad5fbf65gy1g4mm41w485j21qc15oq6d.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "在以往的 pipeline 中，使用 kubectl 进行部署 Deployment 后无法检查 Deployment 是否部署成功，只能通过使用命令/脚本来手动检查 Deployment 状态，而 kubedog 这个小工具完美解决了这个问题，完善了 CI/CD 流水线的最后一步。"
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3j6dyag5j21qc15oq6d.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>在以往的 pipeline 中，使用 kubectl 进行部署 Deployment 后无法检查 Deployment 是否部署成功，只能通过使用命令/脚本来手动检查 Deployment 状态，而 kubedog 这个小工具完美解决了这个问题，完善了 CI/CD 流水线的最后一步。

## KubeDog

kubedog 是一个 lib 库和 CLI 小工具，允许在 CI/CD 部署 pipeline 中观察和跟踪 Kubernetes 资源。与 kustomize 配合，集成到 pipeline 之后，完美的解决了 CI/CD 的最后一步，完美的替代了之前不够灵活的脚本（好吧，其实我也开发了类似的小工具，但是有这么好用的轮子，拿来直接用何乐而不为呢？）。

kubedog 提供了 lib 库和 CLI 小工具，这里由于是介绍 CI/CD 中的实践，所以只介绍其中的 `rollout track` 功能。 lib 库的使用和 CLI 的 `follow` 功能这里就不做介绍了，有兴趣的同学可以去 [GitHub](https://github.com/flant/kubedog) 上查看该项目的各种使用方式。

### 集成 KubeDog

由于我司目前使用的是 [drone](https://drone.io/) 进行 CI ，每个 step 都是由一个 docker 制作的插件组成。我制作了一个包含 `kubectl` 、 `kustomize` 和 `kubedog` 的镜像。该镜像已上传 dockerhub ，需要的可以自行拉取使用 `guoxudongdocker/kubectl` ,而该插件的使用也在 [GitHub](https://github.com/sunny0826/kubectl-kustomize) 和 [DockerHub](https://cloud.docker.com/u/guoxudongdocker/repository/docker/guoxudongdocker/kubectl) 上查看。

而集成方式也比较简单，直接将 `kubectl` 、 `kustomize` 和 `kubedog` 的可执行包下载到 `/usr/local/bin` 并赋予执行权限即可，下面就是 `Dockerfile` 文件：

```dockerfile
FROM alpine

LABEL maintainer="sunnydog0826@gmail.com"

ENV KUBE_LATEST_VERSION="v1.14.1"

RUN apk add --update ca-certificates \
 && apk add --update -t deps curl \
 && curl -L https://storage.googleapis.com/kubernetes-release/release/${KUBE_LATEST_VERSION}/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl \
 && chmod +x /usr/local/bin/kubectl \
 && curl -L https://github.com/kubernetes-sigs/kustomize/releases/download/v2.0.3/kustomize_2.0.3_linux_amd64 -o /usr/local/bin/kustomize \
 && chmod +x /usr/local/bin/kustomize \
 && curl -L https://dl.bintray.com/flant/kubedog/v0.2.0/kubedog-linux-amd64-v0.2.0 -o /usr/local/bin/kubedog \
 && chmod +x /usr/local/bin/kubedog \
 && apk del --purge deps \
 && rm /var/cache/apk/*


WORKDIR /root
ENTRYPOINT ["kubectl"]
CMD ["help"]
```

## Kustomize 配合 KubeDog 使用

在镜像构建好之后就可以直接使用了，这里使用的是 DockerHub 的镜像仓库，这里建议将镜像同步到私有仓库，比如阿里云的容器镜像服务或者 Habor ，因为国内拉取 DockerHub 的镜像不太稳定，经常会拉取镜像失败或者访问超时，在 CI/CD 流水线中推荐使用更稳定镜像。

以下是 `.drone.yml` 示例：

```yaml
kind: pipeline
name: {your-pipeline-name}

steps:
- name: Kubernetes 部署
  image: guoxudongdocker/kubectl
  volumes:
  - name: kube
    path: /root/.kube
  commands:
    - cd deploy/overlays/dev    # 这里使用 kustomize ,详细使用方法请见 https://github.com/kubernetes-sigs/kustomize
    - kustomize edit set image {your-docker-registry}:${DRONE_BUILD_NUMBER}
    - kubectl apply -k . && kubedog rollout track deployment {your-deployment-name} -n {your-namespace} -t {your-tomeout}

...

volumes:
- name: kube
  host:
    path: /tmp/cache/.kube  # kubeconfig 挂载位置

trigger:
  branch:
  - master  # 触发 CI 的分支
```

从上面的配置可见，在该 step 中执行了如下几步：

1. 进入 patch 所在路径
2. 使用了 Kustomize 命令 `kustomize edit set image {your-docker-registry}:${DRONE_BUILD_NUMBER}` 方式将前面 step 中构建好的镜像的 tag 插入到 patch 中
3. 使用 `kubectl apply -k .` 进行 k8s 部署，要注意最后的那个 `.`
4. 使用 kubedog 跟踪 Deployment 部署状态

> 命令解析：`kubedog rollout track deployment {your-deployment-name} -n {your-namespace} -t {your-tomeout}`
>
- deployment {your-deployment-name} : Deployment 的名称
- -n {your-namespace} : Deployment 所在的 namespace
- -t {your-tomeout} : 超时时间，单位为秒，超时后会报错，这里请根据实际部署情况进行设置

## 结语

从 Kubernetes release v1.14 版本开始，`kustomize` 集成到 `kubectl` 中，越来越多 k8S 周边的小工具出现。这些小工具的出现帮助了 Kubernetes 的使用者来拉平 Kubernetes 的使用曲线，同时也标志着 K8S 的成熟，越来越多的开发人员基于使用 K8S 的痛点开发相关工具。套用一句今年 KubeCon 的 Keynote 演讲上，阿里云智能容器平台负责人丁宇的话： __Kubernetes 正当时，云原生未来可期__ 。
