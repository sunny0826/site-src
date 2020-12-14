---
title: "轻量快速的 CI 工具 Drone"
date: 2019-05-21T08:59:00+08:00
draft: false
type: blog
banner: "https://ws2.sinaimg.cn/large/ad5fbf65gy1g38swbabmaj21jk15ognu.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本文介绍一款轻量级的 CI 工具 Drone ，同时也介绍在实践中遇到的一些坑，帮助你快速搭建持续集成流水线。"
tags: ["devops","drone","工具"]
categories: ["devops"]
keywords: ["devops","drone","工具"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3ifzmi4yj21jk15ognu.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

公司之前一直在使用 Jenkins 作为 CI/CD 工具， Jenkins 非常强大，它完成了几乎所有 CI/CD 的工作，并且应用于整个团队有好长一段时间了。但是随着公司推荐数字化、智慧化，以及服务容器化的推进， Jenkins 的一些弊端也凸显了出来：

- **重量级：** Jenkins 功能十分齐全，几乎可以做所有的事情。但是这也是他的一个弊端，过于重量级，有时候往往一个小的修改需要改动许多地方，升级\下载插件后需要进行重启等。
- **升级不易：** 在一些安全 Jenkins 相关的安全漏洞被公开后，我们会对 Jenkins 进行升级，但这也不是一件容易的事。之前就出现过升级\重启后，所有 job 丢失，虽然我们所有项目配置都是以 Jenkinsfile 的形式统一存储，但是每个 job 都需要重新重新创建，包括每个 job 的权限.....\_(´ཀ`」 ∠)_
- **权限控制复杂：** 这其实也是 Jenkins 的一大优势，可以精确控制每个用户的权限，但是需要花费更多时间去配置，时间长了也会出现权限混乱的问题。
- **UI 界面：** 这个其实是吐槽最多的部分，虽然有诸如：Blue Ocean 这样的插件来展示 pipeline ，但是还是没有从根本改变它简陋的 UI 界面。

那么为什么选择使用 Drone 呢？

其实在 GitHub 上提交 PR 后，大部分开源项目都会使用 [travis-ci](http://travis-ci.org/) 对提交的代码进行 CI 及检查，而如果是 Kubernetes 相关的项目，则会使用 [prow](https://github.com/k8s-ci-robot) 进行 CI。但是 [travis-ci](http://travis-ci.org/) 只能用于 GitHub ，在寻找类似项目的时候， Drone 进入了我的视野。

大道至简。和 Jenkins 相比， Drone 就轻量的多了，从应用本身的安装部署到流水线的构建都简洁的多。由于是和源码管理系统相集成，所以 Drone 天生就省去了各种账户\权限的配置，直接与 gitlab 、 github 、 Bitbucket 这样的源码管理系统操作源代码的权限一致。正如它官网上写的那样：

> **Any Source Code Manager**

> Drone integrates seamlessly with multiple source code management systems, including GitHub, GitHubEnterprise, Bitbucket, and GitLab.

> **Any Platform**

> Drone natively supports multiple operating systems and architectures, including Linux x64, ARM, ARM64 and Windows x64.

> **Any Language**

> Drone works with any language, database or service that runs inside a Docker container. Choose from thousands of public Docker images or provide your own.

Drone 天生支持任何源码管理工具、任何平台和任何语言。

**而写这篇文章的目的，并不是要吹捧这个工具有多么的好用，而是要总结在搭建 drone 和使用时候需要的各种坑，帮助读者绕过这些坑。**

## 声明

鉴于在使用 Drone CI 中，遇到的各种坑都和 Drone 的版本有关，这里首先声明我使用的 Drone 版本为`1.1`，使用`0.8`版本的同学请绕道。

## 搭建 Drone

这里要说的就是在使用 drone 中遇到的第一个坑，在最初正准备搭建 drone 的时候 Google 了很多相关的 blog ，大部分 blog （包括某些 [medium.com](https://medium.com/) 上面近期的英文 blog） 推荐的安装方式都是使用 `docker-compose`，而无一例外的都失败了...走投无路之下，我回到了[官网的文档](https://docs.drone.io/installation/)，发现`1.0`之后许多参数都发生了变化，并且官方推荐使用 docker 的方式运行 Drone。

**所以在使用任何开源软件之前都要去阅读它的文档，不要跟着一篇 blog 就开始了（包括我的），这样会少踩很多坑！！！**

这里以 gitlab 为例，展示网上版本启动参数和实际参数的不同：

| 作用 | 各种blog | 官网文档 |
| --- | --- | --- |
| 设置 Drone 的管理员 | DRONE_ADMIN=admin | DRONE_USER_CREATE=username:admin,admin:true |
| 设置GitLab的域名 | DRONE_GITLAB_URL | DRONE_SERVER_HOST |
| GitLab的Application中的key | DRONE_GITLAB_CLIENT | DRONE_GITLAB_CLIENT_ID |
| GitLab的Application中的secret | DRONE_GITLAB_SECRET | DRONE_GITLAB_CLIENT_SECRET |
| Drone 域名 | DRONE_HOST | DRONE_GITLAB_SERVER |

上面只是列举了部分官方文档和网上流产版本的不同，所以在使用之前一定要仔细阅读官方文档。下附运行 drone 的命令：

```bash
docker run \
  --volume=/var/run/docker.sock:/var/run/docker.sock \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GIT_ALWAYS_AUTH=false \
  --env=DRONE_GITLAB_SERVER={your-gitlab-url} \  # gitlab 的 URL
  --env=DRONE_GITLAB_CLIENT_ID={your-gitlab-applications-id} \  #GitLab的Application中的id
  --env=DRONE_GITLAB_CLIENT_SECRET={your-gitlab-applicati-secret} \ #GitLab的Application中的secret
  --env=DRONE_SERVER_HOST={your-drone-url} \    # drone 的URl
  --env=DRONE_SERVER_PROTO=http \
  --env=DRONE_TLS_AUTOCERT=false \
  --env=DRONE_USER_CREATE=username:{your-admin-username},admin:true \   # Drone的管理员
  --publish=8000:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1.1
```

关于 `gitlab Application` 的配置和 Drone 其他参数含义请参考[官方文档](https://docs.drone.io/installation/gitlab/single-machine/)，这里只展示单节点办的运行方式。

## 核心文件 `.drone.yml`

要使用 Drone 只需在项目根创建一个 `.drone.yml` 文件即可，这个是 Drone 构建脚本的配置文件，它随项目一块进行版本管理，开发者不需要额外再去维护一个配置脚本。其实现代 CI 程序都是这么做了，这个主要是相对于 Jekins 来说的。虽然 Jekins 也有插件支持，但毕竟还是需要配置。

> 值得注意的事这个文件时 `.drone.yml`，由于 Kubernetes 使用的多了，第一次创建了一个 `.drone.yaml` 文件，导致怎么都获取不到配置...\_(´ཀ`」 ∠)_... YAML 工程师石锤了...

这里放一个 Java 的 .drone.yml ，这个项目是 fork 别人的项目用作演示，记得要修改 `deployment.yaml` 中的镜像仓库地址修改为自己的私有仓库。

示例项目源码：https://github.com/sunny0826/pipeline-example-maven

```YAML
kind: pipeline
name: pipeline-example-maven

steps:
- name: Maven编译
  image: maven:3-jdk-7
  volumes:
  - name: cache
    path: /root/.m2
  commands:
    - mvn clean install

- name: 构建镜像  
  image: plugins/docker
  volumes:
  - name: docker
    path: /var/run/docker.sock
  settings:
    username: 
      from_secret: docker_user
    password: 
      from_secret: docker_pass
    repo: {your-repo}
    registry: {your-registry}
    tags: ${DRONE_BUILD_NUMBER}

- name: Kubernetes 部署
  image: guoxudongdocker/kubectl:v1.14.1 
  volumes:
  - name: kube
    path: /root/.kube
  commands:
    - sed -i "s/#Tag/${DRONE_BUILD_NUMBER}/g" deployment.yaml
    - kubectl apply -f deployment.yaml

- name: 钉钉通知
  image: guoxudongdocker/drone-dingtalk 
  settings:
    token: 
      from_secret: dingding
    type: markdown
    message_color: true
    message_pic: true
    sha_link: true
  when:
    status: [failure, success]

volumes:
- name: cache
  host:
    path: /tmp/cache/.m2
- name: kube
  host:
    path: /tmp/cache/.kube/.test_kube
- name: docker
  host:
    path: /var/run/docker.sock

trigger:
  branch:
  - master
```

值得注意的事：上面的这个 `.drone.yml` 文件将本地的`.m2`文件、kubeconfig文件、`docker.sock` 文件挂载到 pipeline 中以实现 maven 打包缓存，k8s 部署、docker 缓存的作用，以提高 CI 速度。而是用挂载需要管理员在项目 settings 中勾选 `Trusted` ，这个操作只能管理员进行，普通用户是看不到这个选项的。而管理员就是在docker运行时候 `--env=DRONE_USER_CREATE=username:{your-admin-username},admin:true ` 设置的。

![WX20190521-104717@2x](https://ws3.sinaimg.cn/large/ad5fbf65gy1g38qvifxwij21d40tk76s.jpg)

而上传镜像和钉钉同时需要在 settings 设置中添加 secret

- docker_user：docker 仓库用户名
- docker_pass：docker 仓库密码
- dingding： 钉钉机器人 token

> 注意这里的钉钉 token 是 webhook 中 `https://oapi.dingtalk.com/robot/send?access_token=` 后这部分
![WX20190521-105337](https://ws2.sinaimg.cn/large/ad5fbf65gy1g38r1mkoztj20iy0ezgmg.jpg)

![WX20190521-104942@2x](https://tva2.sinaimg.cn/large/ad5fbf65gy1g38qxizsg1j21ia0tujtb.jpg)

## 构建结果

添加 `.drone.yml` 文件后，向 master 分支提交代码即可出发 CI 构建

![WX20190521-105809@2x](https://wx3.sinaimg.cn/large/ad5fbf65gy1g38r68yb8pj21l40sawit.jpg)

CI 结束后，会在钉钉机器人所在群收到通知

![WX20190521-110009](https://ws4.sinaimg.cn/large/ad5fbf65gy1g38r8cttcrj20e90bzacr.jpg)


## 插件支持

可以看到，每一步的镜像都是一个镜像，上面 pipeline 中的 Kubernetes 及钉钉通知插件就是我开发的，具体开发方法可以参考[官方文档](https://docs.drone.io/)，而官方也提供了许多[官方插件](http://plugins.drone.io/)。

- 构建后部署：[Kubernetes](http://plugins.drone.io/mactynow/drone-kubernetes/)、[helm](http://plugins.drone.io/ipedrazas/drone-helm/)、[scp](http://plugins.drone.io/appleboy/drone-scp/)
- 构建后通知：[钉钉](http://plugins.drone.io/lddsb/drone-dingtalk-message/) 、[Email](http://plugins.drone.io/drillster/drone-email/)、[Slack](http://plugins.drone.io/drone-plugins/drone-slack/)、[微信](http://plugins.drone.io/lizheming/drone-wechat/)


## 后记

Drone 整体用起来还是很方便的，搭建、上手速度都很快，但是官方文档给的不够详实，而网上充斥着各种各样0.8版本的的实例，但是其实官网早就发布了1.0版本，而官方并没有 `example` 这样的示例项目，这样就又把本来降下来的学习曲线拉高了。许多坑都需要自己去趟，我在测试 drone 的时候，就构构建了上百次，不停的修改 `.drone.yml` ， commit 信息看起来是很恐怖的。后续抽空会向官方贡献 `example` 这样的 PR。
