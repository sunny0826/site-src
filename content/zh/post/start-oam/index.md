---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "以应用为中心：开放应用模型（OAM）初探"
subtitle: ""
summary: "本文通过一个简单的示例，介绍开放应用模型（OAM）是如何实现以应用为中心，管理 Kubernetes 的。"
authors: ["guoxudong"]
tags: ["OAM","kubernetes"]
categories: ["OAM"]
date: 2020-06-28T14:53:23+08:00
lastmod: 2020-06-28T14:53:23+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1gg82nyjdvmj20s8089764.jpg"
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
## 前言

不久前，Kubernetes 也迎来了他 6 岁的生日，在这 6 年中，从孵化之初的三足鼎立，到后来的一统天下，Kubernetes 成为容器编排领域的事实标准已经有段时间了。在这期间，云原生的概念开始深入人心，越来越的公司组织和开发者开始接受、了解、实践云原生。如今，已有无数的应用以容器的形式运行在各种版本 Kubernetes 中了。

## 应用管理之惑

然而我们慢慢发现，随着应用和服务数量、使用场景以及承载业务的增加，Kubernetes 资源越来越难以管理。比如，有时候可能多个运维人员重复为一个 Deployment 配置了多个 Service 或 Ingress，而在一个 namespace 中动辄就有上百个 Service，在这些 Service 中找到那些重复、无效、甚至错误的 Service 可不是一件容易的事情。

上面描述的只是运维人员内部可能存在的冲突，更多的冲突来自开发与运维人员之间，由于各自关注的角度不同，出现了对 Deployment 配置权的争夺，他们各自关心的字段不尽相同，但同时还要面对同一份 `deployment.yaml`，这就是冲突的根源。我们的做法是使用 kustomize 将一份 `deployment.yaml` 分成不同的 [overlays](https://kubernetes-sigs.github.io/kustomize/api-reference/glossary/#overlay)，将开发和运维关注的字段分开管理，而这只是缓兵之计，依旧没有一个统一的配置文件来描述整个应用，比如这个应用由几个 Deployment、Service、 Ingress 组成，一个新手如果想要查看一个资源相关的其他资源，只能通过 label 和“相似”的名称去找或者猜。而这样做显然是很危险的，这也是为什么我不敢轻易清理生产环境中无用的 Service 和 ConfigMap 的原因，你永远也想不到有什么地方可能引用了他们。

相对标准 Kubernetes 资源，Operator 的管理难度就更大了，各式各样的 Operator 存在于我的 Kubernetes 集群中，`kubectl get crd` 命令输出的结果更是长的可怕。

而开放应用模型（OAM）可能是助我脱离苦海的一味良药。

## 开放应用模型（OAM）

OAM 是阿里云与 Azure 在 2019 年末联合推出的标准化云原生应用管理模型。相比于传统 PaaS 封闭、不能同“以 Operator 为基础的云原生生态”衔接的现状，基于 OAM 和 Kubernetes 构建的现代云原生应用管理平台，本质上是一个 **“以应用为中心”** 的 Kubernetes ，保证了这个应用平台在能够无缝接入整个云原生生态。同时，OAM 可以进一步屏蔽掉容器基础设施的复杂性和差异性，为平台的使用者带来低心智负担的、标准化的、一致的应用管理与交付体验。

所谓 “应用模型”，其实是一个专门用来对云原生应用本身和它所需运维能力进行定义与描述的标准开源规范。所以对于 Kubernetes 来说，OAM 即是一个标准的“应用定义”项目（类比已经不再活跃的 Kubernetes Application CRD 项目），同时也是一个专注于封装、组织和管理 Kubernetes 中各种 “运维能力”、以及连接 “运维能力” 与 “应用” 的平台层项目。而通过 “定义应用” 和 “组织管理应用的运维能力” 这两大核心功能，我们可以构建一个更容易管理、维护和发展的云原生平台。

以下是 OAM 的一些基本概念：

### Component

在 OAM 中，**Component（组件）** 就是一个完全面向业务研发人员设计的、用于定义应用程序而不必考虑其运维详细信息的载体。一个应用程序包含一个或多个 Component 。例如，一个网站应用可以由一个 Java web 组件和一个数据库组件组成。

OAM 中的 Component 包含两个部分：

- 工作负载描述 —— 如何运行此 Component，以及它的运行内容，实际上就是一个完整的 K8s CR；
- 可重写参数列表 —— 研发通过这个字段表示该 Component 的哪些字段后续可以被运维或者系统覆盖。

### Trait

在 OAM 中，我们通过 **Trait（运维特征）** 来描述和构建具备可发现性和可管理性的平台层能力。

Trait 是与 Component 绑定的，一个 Component 可以绑定多个 Trait，从而把运维能力也加入到应用描述中，方便底层基础设施统一管理。

### Application Configuration

最终，通过引用 Component 名称并对它绑定 Trait ，运维人员就可以使用 **ApplicationConfiguration（应用配置）** 来实例化应用程序。ApplicationConfiguration 的主要功能，就是让应用运维人员（或系统）了解和使用业务研发人员传达的信息，然后自由的为 Component 组合绑定不同的运维能力以相应实现其最终的运维目的。

下面这张图很好的描述了 OAM 架构的使用场景，开发与运维的**关注点分离**，而最终都由一份 `ApplicationConfiguration` 来描述整个应用：

![image](https://tvax4.sinaimg.cn/large/ad5fbf65ly1gg82h3v1o1j20jg0bg77i.jpg)

## 上手实践

上面只是对 OAM 进行了简单的介绍，由于篇幅有限，如 Scope 这样的概念并没有进行介绍，更多内容欢迎加入 [OAM 社区](https://oam.dev/)。

下面就以一个简单的示例，开启我们的 OAM 之旅：

### 前提条件

本示例为官方示例，使用 OAM 部署一个 `nginx` 应用，该应用包含 Deployment、Service 和 Ingress。

* Kubernetes 集群
* [Helm 3](https://helm.sh/docs/intro/)

### 安装控制端

#### 安装 Crossplane 和 OAM

注意，这里的 `crossplane-oam-sample` 是官方维护的一个 crossplane 示例，只是用作开发和演示，并不是生产可用，关于 crossplane 的更多内容，请见[项目官网](https://crossplane.io/)。

```bash
$ helm repo add oam https://oam-dev.github.io/crossplane-oam-sample/archives/
$ kubectl create namespace oam-system
$ helm install crossplane --namespace oam-system oam/crossplane-oam
```

这里如果由于墙的原因无法拉取 `gcr.io/kubebuilder/kube-rbac-proxy:v0.4.1` 镜像，导致 `crossplane-oam-localstack` 无法启动的话，可以使用我提供的替代镜像 `guoxudongdocker/kube-rbac-proxy:v0.4.1`。

#### 拉取示例仓库

```bash
$ git clone https://github.com/oam-dev/catalog.git
# 进入示例
$ cd catalog/traits/ingresstrait
```

#### 部署 CRD 并启动 controller

```bash
# 部署 CRD
$ make install
~/go/bin/controller-gen "crd:trivialVersions=true" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases
kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/ingresstraits.core.oam.dev created
# 启动 IngressTrait controller
$ go run main.go
I0629 11:15:22.035708     802 request.go:621] Throttling request took 1.000526734s, request: GET:https://192.168.4.210:6443/apis/apiregistration.k8s.io/v1?timeout=32s
2020-06-29T11:15:22.088+0800    INFO    controller-runtime.metrics      metrics server is starting to listen    {"addr": ":8080"}
2020-06-29T11:15:22.089+0800    INFO    setup   starting manager
2020-06-29T11:15:22.089+0800    INFO    controller-runtime.manager      starting metrics server {"path": "/metrics"}
2020-06-29T11:15:22.089+0800    INFO    controller-runtime.controller   Starting EventSource    {"controller": "ingresstrait", "source": "kind source: /, Kind="}
2020-06-29T11:15:22.193+0800    INFO    controller-runtime.controller   Starting Controller     {"controller": "ingresstrait"}
2020-06-29T11:15:22.193+0800    INFO    controller-runtime.controller   Starting workers        {"controller": "ingresstrait", "worker count": 1}
```

由于这里只是简单演示，没有将 IngressTrait controller 打包成镜像，而是在本地运行 controller，所以需要 go 环境。

### 部署应用

#### 配置 RBAC

使用命令：`kubectl apply -f rbac.yaml`，配置 RBAC。这里需要注意的是官方 IngressTrait 的 sample 示例中并没有 `rbac.yaml`，需要我们自己配置，否则的话会在部署时由于权限原因无法拉起 Deployment。

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-clusterrole-poc
rules:
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - "*"

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: component-deployment-workload-poc
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deployment-clusterrole-poc
subjects:
  - kind: ServiceAccount
    name: crossplane-oam              # Remember to use the actual ServiceAccount name
    namespace: oam-system             # Remember to use the actual ServiceAccount namespace

```

#### 部署 Component

使用 `kubectl apply -f sample_component.yaml ` 命令部署 Component，该 Component 中的 workload 为 Deployment。

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: example-deploy
spec:
  workload:
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: web
    spec:
      selector:
        matchLabels:
          app: test
      template:
        metadata:
          labels:
            app: test
        spec:
          containers:
            - name: nginx
              image: nginx:1.17
              ports:
                - containerPort: 80
                  name: web
```

#### 部署 ApplicationConfiguration

可以看到这个 ApplicationConfiguration 中包含一个 Component，而 Component 中又绑定了 一个 IngressTrait 类型的 Trait，由于这只是一个简单示例，所有只有一个 Component 和一个 Trait，在实际的生产环境中，一个 ApplicationConfiguration 可由多个 Component 组成，一个 Component 又可绑定多个 Trait 为其提供诸如流量管控、弹性伸缩等运维特性。

使用命令：`kubectl apply -f sample_application_config.yaml`

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: example-appconfig
spec:
  components:
    - componentName: example-deploy
      traits:
        - trait:
            apiVersion: core.oam.dev/v1alpha2
            kind: IngressTrait
            metadata:
              name: example-ingress-trait
            spec:
                rules:
                  - host: nginx.oam.com
                    paths:
                      - path: /
                        backend:
                          serviceName: deploy-test
                          servicePort: 8080
```

#### 检查结果

可以看到 Deployment、Service 和 Ingress 已经部署成功：

```bash
$ kubectl get deploy,svc,ing
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web   1/1     1            1           8m29s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/deploy-test   ClusterIP   10.43.170.228   <none>        8080/TCP   8m29s

NAME                                       HOSTS           ADDRESS                       PORTS   AGE
ingress.extensions/example-ingress-trait   nginx.oam.com   192.168.1.129,192.168.4.210   80      8m29s
```

访问服务：

```bash
$ curl -H "Host: nginx.oam.com"  http://192.168.1.129
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

应用的整体结构如下图所示：

![OMA](https://tva4.sinaimg.cn/large/ad5fbf65gy1gg94rq5fyij20ef0drdgg.jpg)

## 结语

通过上面这个简单的示例，可以看出如果遵循 OAM 模型来划分应用，我们可以从 ApplicationConfiguration 入手，看到应用中都包含哪些组件（Component），同时又可以看到每个组件都有哪些运维特性（Trait）来支持这个组件，逐层的查看每个模块的描述和配置，最终全面了解这个应用，而不用像现在这样使用 label 和 name，漫无目的的靠运气来理清整个架构，真正的做到**以应用为中心**。

OAM 的本质是将云原生应用定义中的研发、运维关注点分离，资源对象进行进一步抽象，化繁为简，包罗万象。

## 参考

* [深度解读！阿里统一应用管理架构升级的教训与实践 - CSDN](https://mp.weixin.qq.com/s/rRaHl5a5PU9Xg5psMservA?from=timeline&isappinstalled=0&scene=2&clicktime=1588769747&enterid=1588769747)
* [oam-dev/catalog - github.com](https://github.com/oam-dev/catalog)
