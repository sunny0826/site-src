---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubectl Plugin 推荐（一）| 可观测性篇"
subtitle: ""
summary: "推荐一些 Kubectl Plugin，本篇主要是提升可观测性篇的插件。"
authors: ["guoxudong"]
tags: ["Kubernetes","Kubernetes Plugin","Krew"]
categories: ["Kubernetes Plugin"]
date: 2021-03-04T17:36:15+08:00
lastmod: 2021-03-04T17:36:15+08:00
draft: false
type: blog
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65gy1go82bo07nyj20p00angmk.jpg"
---
## 前言

`kubectl` 作为最重要的 Kubernetes 客户端工具一直以来都被广泛的应用与各种场景，其对于 YAML 工程师的作用就像战士手中的枪，用的好不好完全可以影响到 YAML 工程师的整体工作效率。虽然 `kubectl` 本身迭代的速度非常快，但是也很难满足所有人的全部需求，这时 kubectl 的插件机制就可以很好的弥补这个问题。

## Krew

Krew 是 kubernetes SIG 项目，是 `kubectl` 的插件管理器，其提供了类似 brew 的包管理功能，用户可以方便的使用 krew 安装和使用 kubectl 插件，极大的方便了 kubectl 插件的开发和管理。

### 安装

Krew 虽然也可以使用 `brew` 进行安装，但是官方貌似并不积极支持该方式。这里使用如下命令脚本来完成安装：

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)
```

脚本运行成功后，将 `$HOME/.krew/bin` 添加到 `PATH` 中，在 `.bashrc` 或 `.zshrc` 文件中添加以下内容：

```bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

添加完成后请重启终端，使用 `kubectl krew` 检查是否安装成功。

## 插件推荐

接下来推荐一些增强可观测性的 kubectl plugin。

### tree

该插件是由 Google 大佬开发，通过 `ownerReferences` 来发现 kubernetes 对象之间的相互关联，并通过树状图来展示，对资源的关系一目了然。

项目地址：https://github.com/ahmetb/kubectl-tree

#### 安装

```bash
$ kubectl krew install tree
$ kubectl tree --help
```
#### 示例

![kubectl tree](https://tvax2.sinaimg.cn/large/ad5fbf65gy1go810e9dtxj21ps0rwtyh.jpg)

### status

kubectl-status 这个插件简化了 `get` 和 `describe` 操作，采用不同颜色和箭头等元素来展示 kubernetes 资源的生命周期和状态信息，可以查看单个资源，也可以查看该 namespace 下的所有该资源的状态，极大的缩短了问题排查的时间，减少了操作步骤。

项目地址：https://github.com/bergerx/kubectl-status

#### 安装

```bash
$ kubectl krew install status
$ kubectl status --help
```

#### 示例

![Pod](https://tvax2.sinaimg.cn/large/ad5fbf65gy1go81g4jkwxj21da0yg0vn.jpg)

![StatefulSet](https://tvax1.sinaimg.cn/large/ad5fbf65gy1go81gfyc56j21da0s8abh.jpg)

### view-allocations

kubectl-view-allocations 可以非常方便的展示 CPU、内存、GPU 等资源的分布情况，并可以对 namespace、node、pod 等维度进行展示。

项目地址：https://github.com/davidB/kubectl-view-allocations

#### 安装

**脚本安装**

```bash
$ curl https://raw.githubusercontent.com/davidB/kubectl-view-allocations/master/scripts/getLatest.sh | bash
```

**krew**

```bash
$ kubectl krew install view-allocations
```

**cargo**

```bash
$ cargo install kubectl-view-allocations
```

#### 示例

**展示 GPU 的分配情况**

```bash
$ kubectl-view-allocations -r gpu

 Resource                   Requested       Limit  Allocatable  Free 
  nvidia.com/gpu           (71%) 10.0  (71%) 10.0         14.0   4.0 
  ├─ node-gpu1               (0%) 0.0    (0%) 0.0          2.0   2.0 
  ├─ node-gpu2               (0%) 0.0    (0%) 0.0          2.0   2.0 
  ├─ node-gpu3             (100%) 2.0  (100%) 2.0          2.0   0.0 
  │  └─ fah-gpu-cpu-d29sc         2.0         2.0                    
  ├─ node-gpu4             (100%) 2.0  (100%) 2.0          2.0   0.0 
  │  └─ fah-gpu-cpu-hkg59         2.0         2.0                    
  ├─ node-gpu5             (100%) 2.0  (100%) 2.0          2.0   0.0 
  │  └─ fah-gpu-cpu-nw9fc         2.0         2.0                    
  ├─ node-gpu6             (100%) 2.0  (100%) 2.0          2.0   0.0 
  │  └─ fah-gpu-cpu-gtwsf         2.0         2.0                    
  └─ node-gpu7             (100%) 2.0  (100%) 2.0          2.0   0.0 
     └─ fah-gpu-cpu-x7zfb         2.0         2.0    
```

**展示 namespace 维度资源的分配情况**

```bash
$ kubectl-view-allocations -g namespace

 Resource              Requested          Limit  Allocatable     Free 
  cpu                 (21%) 56.7    (65%) 176.1        272.0     95.9 
  ├─ default                42.1           57.4                       
  ├─ dev                     5.3          102.1                       
  ├─ dns-external         200.0m            0.0                       
  ├─ docs                 150.0m         600.0m                       
  ├─ ingress-nginx        200.0m            1.0                       
  ├─ kube-system             2.1            1.4                       
  ├─ loki                    1.2            2.4                       
  ├─ monitoring              3.5            7.0                       
  ├─ sharelatex           700.0m            2.4                       
  └─ weave                   1.3            1.8                       
  ephemeral-storage     (0%) 0.0       (0%) 0.0        38.4T    38.4T 
  memory             (8%) 52.7Gi  (15%) 101.3Gi      675.6Gi  574.3Gi 
  ├─ default              34.6Gi         60.0Gi                       
  ├─ dev                   5.3Gi         22.1Gi                       
  ├─ dns-external        140.0Mi        340.0Mi                       
  ├─ docs                448.0Mi        768.0Mi                       
  ├─ ingress-nginx       256.0Mi          1.0Gi                       
  ├─ kube-system         840.0Mi          1.0Gi                       
  ├─ loki                  1.5Gi          1.6Gi                       
  ├─ monitoring            5.9Gi          5.7Gi                       
  ├─ sharelatex            2.5Gi          7.0Gi                       
  └─ weave                 1.3Gi          1.8Gi                       
  nvidia.com/gpu      (71%) 10.0     (71%) 10.0         14.0      4.0 
  └─ dev                    10.0           10.0                       
  pods                (9%) 147.0     (9%) 147.0         1.6k     1.5k 
  ├─ cert-manager            3.0            3.0                       
  ├─ default                13.0           13.0                       
  ├─ dev                     9.0            9.0                       
  ├─ dns-external            2.0            2.0                       
  ├─ docs                    8.0            8.0                       
  ├─ ingress-nginx           2.0            2.0                       
  ├─ kube-system            43.0           43.0                       
  ├─ loki                   12.0           12.0                       
  ├─ monitoring             38.0           38.0                       
  ├─ sharelatex              3.0            3.0                       
  └─ weave                  14.0           14.0   
```

### images

kubectl-images 可以展示集群中正在使用的镜像，并对 namespace 进行一个简单的统计。使用这个插件可以非常方面的查看 namespace 中使用了哪些镜像，尤其在排查问题需要查看镜像版本时非常有用。

项目地址：https://github.com/chenjiandongx/kubectl-images

#### 安装

```bash
$ kubectl krew install images
$ kubectl images --help
```

#### 示例

![kubectl-images](https://tva4.sinaimg.cn/large/ad5fbf65gy1go81ui9oidj21sk17qx3l.jpg)

## 结语

有了 plugin 的加持，可以轻松为 kubectl 打造出一整套适合自己操作习惯的工具体系。如果这些插件都不符合要求，大可以自己开发一款插件，这样做可以大大提升工作效率，将自己从重复的劳动中解放出来！这也是笔者不常加班的原因之一。