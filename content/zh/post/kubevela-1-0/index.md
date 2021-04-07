---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "让云原生应用的交付变得更简单 | KubeVela v1.0 正式发布"
subtitle: ""
summary: "Kubevela v1.0 正式发布"
authors: ["guoxudong"]
tags: ["OAM","kubevela"]
categories: ["OAM"]
date: 2021-04-06T09:40:41+08:00
lastmod: 2021-04-06T09:40:41+08:00
draft: false
type: blog
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65gy1gpa21vzq1zj20p00angrh.jpg"
---
## 背景

在去年的 KubeCon+CloudNativeCon 2020 北美峰会上，来自 CNCF 应用交付领域小组（CNCF SIG App Delivery) 与 Open Application Model (OAM) 社区，以及阿里云、Azure 的 OAM 项目维护者正式宣布了 KubeVela 开源项目的发布。笔者有幸作为初创成员参与到 kubeVela 的早期开发中，见证了 KubeVela 的诞生及高速发展。就在3月份的最后一天，KubeVela 迎来的 1.0 版本。

## KubeVela 与 OAM

从 KubeVela 诞生的第一天起，就有很多朋友询问 KubeVela 和 OAM 的关系。OAM 是一种**标准模型**，不同的团队可以基于这个标准模型开发出不同的实现；而 KubeVela 则是 OAM 标准模型的实现，是**一个简单易用且高度可扩展的应用管理平台与核心引擎**。

## KubeVela 给 Kubernetes 插上翅膀

Kubernetes 本身十分灵活且功能丰富，但正是由于其灵活多变，导致了复杂度及管理难度的直线上升。在实践中，如没有一个统一的标准，全面使用 Kubernetes 一段时间后，集群将变的十分复杂且难以维护，并且随着人员变动或核心成员的离开，集群很可能会陷入无人敢动，无法维护的境地。

为了规避这种情况，各个公司的平台团队会基于 Kubernetes 开发自己的 PaaS 平台，平台团队通过“限制” Kubernetes 的能力，只放出有限的字段供业务团队使用，也就是基于自己的使用场景定制化开发一个上层平台，这样只是重复的造轮子且极大的限制了 Kubernetes 本身的拓展性。

![传统 PaaS 的能力困境](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gp9ucbg6zbj21b60qi12v.jpg)

KubeVela 的出现很好的解决了这些问题。

### 高度拓展 Kubernetes，打破传统 PaaS 的限制

OAM 的出现提供了一种可拓展、方便快捷的将 Kubernetes 的能力进行组装与合成的能力。在原生 Kubernetes 中，搭建一个简单的 web 应用至少需要一个 Deployment 和一个 Service，它们之间通过 labels 进行绑定，所以在 Deployment 的描述文件中，无法看到是哪个 Service 绑定了它，同理 service 也无法在描述文件中看到它绑定了哪个 Deployment，而这还只是最简单的场景，如果 Deployment 还挂载了 PVC、ConfigMap 和 Secrets 呢？如果 Service 还绑定了多个 Ingress 呢？

所以传统基于 Kubernetes 的 PaaS 平台会根据平台本身的某种垂直场景来制定一些内部的标准并提供部分 Kubernetes 的能力。而 KubeVela 打破了这些限制，KubeVela 为 Kubernetes 增加了一层抽象，平台团队只需定制自己的 `ComponentDefinition` 及 `Traits` 即可将 Kubernetes 的能力开放给应用研发团队，开发团队只需要编写一个 docker-compose 风格应用描述文件 `Appfile` 即可，不需要接触和学习任何 Kubernetes 层的相关细节。同时社区也会有很多现成的模板供用户使用，每个模板都像插件一样可以轻松的“安装”与“卸载”。

![](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gpa43rkaq9j21em0qadll.jpg)

而作为应用的管理和维护者，可以轻松的从 `Application` 这个 CR 中查看应用都包含哪些组件与运维特性，极大的提升了应用的可描述性及可维护性，非常方便新人接手运维以及问题排查。让云原生真正走进 **“以应用为中心”** 的时代。

### 多种模式，纳管全部 Kubernetes 资源

此次 KubeVela 的 v1.0 版本较之 v0.X 版本最大的亮点除了 API 版本升级至 `v1beta1`，标志着 API Resource 基本稳定以外，最大的亮点就是支持 CUE 、Helm 和原生 Kubernetes 资源模板三种应用抽象模式。

CUE 是一门强大的 DSL 语言，其专为大规模配置而设计，借助 CUE 用户可以定制非常复杂的模板，十分适合用来定义抽象模板。关于 CUE 的更多内容见[官方文档](https://kubevela.io/zh/docs/cue/basic)，这里不做详细介绍。

而在 v1.0 版本中最大的惊喜则是支持纳管 Helm 为 `ComponentDefinition`。在实际场景中，常常会用到 Helm 来部署第三方应用，但 Helm chart 本身是一个黑盒，如果有一些定制需求则需要手动去修改 Helm chart 的 template，十分的痛苦。而 KubeVela 1.0 版本提供了纳管 Helm chart 的功能，通过如下代码即可将一个 `elasticsearch` 的 Helm chart 定义为 `ComponentDefinition`：

```yaml
apiVersion: core.oam.dev/v1beta1
kind: ComponentDefinition
metadata:
  name: elasticsearch-chart
  annotations:
    definition.oam.dev/description: helm chart for elasticsearch
spec:
  workload:
    definition:
      apiVersion: apps/v1
      kind: StatefulSet
  schematic:
    helm:
      release:
        chart:
          spec:
            chart: "elasticsearch"
            version: "7.11.1"
      repository:
        url: "https://helm.elastic.co/"
```

使用同样的方式定义一个 kibana 的 `ComponentDefinition`，即可轻松的描述一个 `elasticsearch` + `kibana` 的应用：

```yaml
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: elasticsearch
  namespace: default
spec:
  components:
    - name: elasticsearch-web
      type: elasticsearch-chart
      properties: 
        imageTag: "7.11.1"
        replicas: 1
        volumeClaimTemplate:
          accessModes: [ "ReadWriteOnce" ]
          resources:
            requests:
              storage: 20Gi
        fullnameOverride: "elasticsearch-web"
    - name: kibana-test
      type: kibana-chart
      properties: 
        fullnameOverride: "kibana-web"
        elasticsearchHosts: "http://elasticsearch-web:9200"
        imageTag: "7.11.1"
      traits:
        - type: ingress
          properties:
            domain: kibana.guoxudong.io
            http:
              "/": 5610
```

Helm chart 中所有 value 值都可以在 `properties` 进行定义，同时还可以为 components 绑定已经定义好的 Trait（运维特性），无需修改 Helm chart 本身的 template，非常方便！

如果只是开放一些简单的参数，则可以使用原生 Kubernetes 资源模板，虽然没有 CUE 那么灵活，但是可以快速上手且无缝对接 Trait 的能力，非常适合一些简单的场景。

同时在 1.0 版本，所有的抽象定义都会自动生成 `Open-API-v3` 架构 JSON 格式的表单数据，方便前端进行集成。无论是 CUE、Helm 还是原生 Kubernetes 资源模板，都会已生成一个名为 `schema-<your-definition-name>` 的 ConfigMap，其中的 key  `openapi-v3-json-schema` 的值就是 JSON 格式的参数，可以非常方便生成一个前端表单供平台和应用团队使用，效果如下：

![](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gp9zsaw9cxj21cn0ebdhd.jpg)

查看 ConfigMap 内容：

```shell
$ kubectl get configmaps schema-webservice  -n vela-system  -o jsonpath="{.data.openapi-v3-json-schema}" | jq .
{
  "properties": {
    "addRevisionLabel": {
      "default": false,
      "description": "If addRevisionLabel is true, the appRevision label will be added to the underlying pods",
      "title": "addRevisionLabel",
      "type": "boolean"
    },
    "cmd": {
      "description": "Commands to run in the container",
      "items": {
        "type": "string"
      },
      "title": "cmd",
      "type": "array"
    },
    "cpu": {
      "description": "Number of CPU units for the service, like `0.5` (0.5 CPU core), `1` (1 CPU core)",
      "title": "cpu",
      "type": "string"
    },
    "env": {
      "description": "Define arguments by using environment variables",
      "items": {
        "properties": {
          "name": {
            "description": "Environment variable name",
            "title": "name",
            "type": "string"
          },
          "value": {
            "description": "The value of the environment variable",
            "title": "value",
            "type": "string"
          },
          "valueFrom": {
            "description": "Specifies a source the value of this var should come from",
            "properties": {
              "secretKeyRef": {
                "description": "Selects a key of a secret in the pod's namespace",
                "properties": {
                  "key": {
                    "description": "The key of the secret to select from. Must be a valid secret key",
                    "title": "key",
                    "type": "string"
                  },
                  "name": {
                    "description": "The name of the secret in the pod's namespace to select from",
                    "title": "name",
                    "type": "string"
                  }
                },
                "required": [
                  "name",
                  "key"
                ],
                "title": "secretKeyRef",
                "type": "object"
              }
            },
            "required": [
              "secretKeyRef"
            ],
            "title": "valueFrom",
            "type": "object"
          }
        },
        "required": [
          "name"
        ],
        "type": "object"
      },
      "title": "env",
      "type": "array"
    },
    "image": {
      "description": "Which image would you like to use for your service",
      "title": "image",
      "type": "string"
    },
    "port": {
      "default": 80,
      "description": "Which port do you want customer traffic sent to",
      "title": "port",
      "type": "integer"
    }
  },
  "required": [
    "addRevisionLabel",
    "image",
    "port"
  ],
  "type": "object"
}
```

基于该功能，通过一条命令，即可在本地浏览器查看 `Components Types` 和 `Traits` 的文档，效果如下：

```shell
$ vela show webservice --web
```
![CLI 打开文档页面](https://tva4.sinaimg.cn/large/ad5fbf65gy1gpax5vem5hj22qq202tmu.jpg)


### 关注点分离，实现真正的 DevOps

关注点分离是 OAM 和 KubeVela 用来解决最终用户与平台团队所面临的困境的方式。

对于**最终用户（应用开发团队）**，KubeVela 是一个简单、易用且高拓展的云原生应用管理工具，其可以让开发者以极低的心智负担和上手成本在 Kubernetes 上部署应用，只需一行命令 `vela up` 即可拉起一整套云原生应用。

对于**平台团队**，KubeVela 是一个可以任意扩展的云原生平台内核，平台工程师可以轻松的将 Kubernetes 生态中的能力，通过 KubeVela 以类似插件的形象注入到 Kubernetes 集群中，简单高效且易于维护。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1gp9zu2k1iaj20q00ds0yt.jpg)

KubeVela 的出现，终结了应用开发团队和平台开发团队之间的”灰色地带“，大家对各自关注点有了清楚的认知，降低了团队之间的沟通成本及扯皮风险，真正的实现了 DevOps 的理念。

## 功能展望

除了上述的功能外，KubeVela v1.0 还提供声明和使用云资源，支持蓝绿和金丝雀发布，使用 Service Mesh 实现多版本多集群部署的能力，以及一些性能的优化、功能的加强及 bug 的修复等。

在之后的版本中，KubeVela 还会将 Terraform 集成到核心模板引擎中，以提供使用多种云资源的能力，并会完善各种功能，并**在合适的时候将项目捐献给 CNCF**。

## 关于社区

KubeVela 诞生于 OAM 社区，甚至 KubeVela 这个名字都是由社区用户投票产生的，是真正由社区发起的开源项目。笔者作为社区成员从项目成立之初就参与其中，除了部分功能及代码外，还先后为 KubeVela 制作了两版官方网站并成为了 KubeVela 网站项目 [kubevela.io](https://github.com/oam-dev/kubevela.io) 的 maintainer。

KubeVela 社区是一个非常开放的社区，目前还有大量的新功能在规划和实现中，欢迎大家的贡献、使用和反馈。想要参与的同学也可以在项目 [issues](https://github.com/oam-dev/kubevela/issues) 中找到 `good first issue` 标签的 issue 完成并提交你的第一个 PR。

## 总结

发布至今 KubeVela 从单一的 CLI 命令行工具，发展成具备强大功能的应用管理平台与核心引擎。在笔者看来，现在的 KubeVela 就像是一套可以随意定制的乐高，用户可以根据自己的需要自由的将所需能力进行组装，这里每一块拼装的部分都可以根据用户喜好进行定制，且这种定制是低成本、可复用的。

站在一个应用管理者和维护者的角度，使用了 KubeVela 以后，通过一份描述文件，就可以清楚的看到一个应用到底由几个 workload 组成，每个 workload 都绑定了什么运维能力，大大降低了运维人员的心智负担及重复劳动时间。

在 v1.0 版本我们也为 KubeVela 构建了新的官网，更多内容[详见官网](https://kubevela.io)。KubeVela 旨在构建一个面向未来的云原生 PaaS 架构，将横向可扩展性和以应用为中心这些最佳实践带给社区用户，推动甚至引领云原生社区在应用层的发展。

- KubeVela 官网：https://kubevela.io
- KubeVela 项目地址：https://github.com/oam-dev/kubevela

## 参考资料

- [KubeVela - The Extensible App Platform Based on Open Application Model and Kubernetes](https://kubevela.io/blog/kubevela-the-extensible-app-platform-based-on-open-application-model-and-kubernetes/)
- [KubeVela - Release v1.0.0](https://github.com/oam-dev/kubevela/releases/tag/v1.0.0)