---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "小工具介绍：KubeWatch"
subtitle: "Kubernetes 资源观测利器"
summary: "用于观测 Kubernetes 资源情况，并实时通知到各种协作软件/聊天软件"
authors: ["guoxudong"] 
tags: ["kubernetes","容器"]
categories: ["kubernetes"]
date: 2019-12-04T17:09:51+08:00
lastmod: 2019-12-04T17:09:51+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3j3k1yrrj21qx15on2p.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

这次要介绍一个 Kubernetes 资源观测工具，实时监控 Kubernetes 集群中各种资源的新建、更新和删除，并实时通知到各种协作软件/聊天软件，目前支持的通知渠道有：

- `slack`
- `hipchat`
- `mattermost`
- `flock`
- `webhook`

我这边开发了钉钉的通知渠道，但是在上游 [ISSUE#198](https://github.com/bitnami-labs/kubewatch/issues/198) 中提出的贡献请求并没有得到回应，所以这边只能 fork 了代码，然后自己进行了开发，以支持钉钉通知。

## 安装

这里推荐使用 helm 进行安装，快速部署

```bash
helm install kubewatch stable/kubewatch \
--set rbac.create=true \
--set slack.channel='#YOUR_CHANNEL' \
--set slack.token='xoxb-YOUR_TOKEN' \
--set resourcesToWatch.pod=true \
--set resourcesToWatch.daemonset=true
```

如果想使用钉钉通知，则可以在 [GitHub](https://github.com/sunny0826/kubewatch-chat) 上拉取我的代码，代码中包含 helm chart 包，可直接进行安装

```bash
git clone https://github.com/sunny0826/kubewatch-chat.git
cd kubewatch-chat
helm install kubewatch kubewatch \
--set dingtalk.sign="XXX" \
--set dingtalk.token="XXXX-XXXX-XXXX"
```

## 钉钉配置

在钉钉中创建 `智能群助手` ，之后

### 获取 token

复制的 webhook 中 `https://oapi.dingtalk.com/robot/send?access_token={YOUR_TOKEN}`, `{YOUR_TOKEN}` 就是要填入的 token。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1g9ku2hvs16j20ep05smxk.jpg)

## 安全设置

钉钉智能群助手在更新后新增了安全设置，提供三种验证方式 `自定义关键词` `加签` `IP地址（段）`，这里推荐使用 `IP地址（段）的方式`，直接将 Kubernetes 集群的出口 IP 填入设置即可。同时也提供了 `加签` 的方式，拷贝秘钥，将其填入 `dingtalk.sign` 中。

![](https://tva1.sinaimg.cn/large/ad5fbf65gy1g9ku6qjwy2j20fo077glw.jpg)


## 项目配置

编辑 `kubewatch/value.yaml` ，修改配置

```yaml
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName

slack:
  enabled: false
  channel: ""
  token: "xoxb"

hipchat:
  enabled: false
  # room: ""
  # token: ""
  # url: ""
mattermost:
  enabled: false
  # channel: ""
  # url: ""
  # username: ""
flock:
  enabled: false
  # url: ""
webhook:
  enabled: false
  # url: ""
dingtalk:
  enabled: true
  token: ""
  sign: ""

# namespace to watch, leave it empty for watching all.
namespaceToWatch: ""

# Resources to watch
resourcesToWatch:
  deployment: true
  replicationcontroller: false
  replicaset: false
  daemonset: false
  services: false
  pod: true
  job: false
  persistentvolume: false

image:
  registry: docker.io
#  repository: bitnami/kubewatch
  repository: guoxudongdocker/kubewatch-chart
#  tag: 0.0.4-debian-9-r405
  tag: latest
  pullPolicy: Always
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

## String to partially override kubewatch.fullname template (will maintain the release name)
##
# nameOverride:

## String to fully override kubewatch.fullname template
##
# fullnameOverride:

rbac:
  # If true, create & use RBAC resources
  #
  create: true

serviceAccount:
  # Specifies whether a ServiceAccount should be created
  create: true
  # The name of the ServiceAccount to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 300Mi
  # requests:
  #   cpu: 100m
  #   memory: 300Mi

# Affinity for pod assignment
# Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
# affinity: {}

# Tolerations for pod assignment
# Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
tolerations: []

# Node labels for pod assignment
# Ref: https://kubernetes.io/docs/user-guide/node-selection/
nodeSelector: {}

podAnnotations: {}
podLabels: {}
replicaCount: 1

```

使用 `value.yaml` 安装

```bash
git clone https://github.com/sunny0826/kubewatch-chat.git
cd kubewatch-chat
helm install my-release -f kubewatch/values.yaml
```

## Slack 配置

Slack 为 kubewatch 默认的通知软件，这里就不简介 Slack 的安装和注册，直接从创建 APP 开始

### 创建一个 APP

进去创建 [APP 页面](https://api.slack.com/apps)

![image](https://tva1.sinaimg.cn/large/ad5fbf65gy1g9kum3x5npj21h40p6tdx.jpg)

选择 `App Name` 和 `Development Slack Workspace`

![](https://tva1.sinaimg.cn/large/ad5fbf65gy1g9kupp0av1j210c0uejvj.jpg)

### 添加 Bot 用户

![image](https://tvax3.sinaimg.cn/large/ad5fbf65gy1g9kuszmgggj21n4156gu2.jpg)

### 添加 App 到 Workspace

![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1g9kuyzwzetj21qu0wmq9n.jpg)

### 获取 Bot-token

![image](https://tvax3.sinaimg.cn/large/ad5fbf65gy1g9kv06dva8j21s60uajxf.jpg)


## 通知效果

在 Slack 中，`创建` `更新` `删除` 分别以绿、黄和红色代表

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1g9kv23nvmoj213c0mewj4.jpg)

在钉钉中，我进行了汉化

![image](https://tvax3.sinaimg.cn/large/ad5fbf65gy1g9kv5fppglj20dd08zdgs.jpg)

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1g9kv5uuxn4j20ea08fgmk.jpg)


## 结语

对于 kubewatch 我们这里主要用作监控各种 CronJob 的定时触发状态，已经 ConfigMap 和 Secrets 的状态变化，同时也观察 HPA 触发的弹性伸缩的状态，可以实时观测到业务高峰的到来，是一个不错的小工具。