---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubenav: 使用手机管理你的 K8S 集群"
subtitle: ""
summary: "移动端 K8S 多集群管理利器：Kubenav"
authors: ["guoxudong"]
tags: ["Kubernetes"]
categories: ["Kubernetes"]
date: 2021-04-12T16:57:26+08:00
lastmod: 2021-04-12T16:57:26+08:00
draft: false
type: blog
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65gy1gph2vhtwssj229616swxv.jpg"
---
## 背景

相信广大 SRE/运维工程师都和笔者一样，以防出现紧急情况，无论是双休日还是过年过节，笔记本电脑常伴左右。笔者也有过在休假时，业务系统出现大规模故障，蹲在日本街头拿手机远程指挥同事排查问题的经历。那段经历除了给茶余饭后增加一些谈资以外，更多的是在手边没有可以快速接入排查问题的设备时无尽的焦虑。而 Kubenav 的出现很大程度可以缓解这个焦虑。

## Kubenav

Kubenav 号称是装在口袋里的 Kubernetes 集群导航，其提供了移动、桌面和 web 端 APP 用来查看和管理 Kubernetes 集群。

该应用使用 [Ionic Framework](https://ionicframework.com/)  和 [Capacitor](https://capacitor.ionicframework.com/) 开发。应用的前端部分使用 TypeScript 和 React 组件实现功能。后端部分使用 [Go mobile](https://github.com/golang/go/wiki/Mobile) 与 Kubernetes API Server 和 Cloud Providers 进行通信，实现了 kubenav 移动端和桌面端实现近 100% 的代码共享。

### 主要功能

Kubenav 基本上就是一个 Kubernetes Dashboard 的增强版，在其上可以非常方便的查看 Kubernetes 的各种资源，不止移动端，在桌面端和 web 端也十分好用。

- **支持移动、桌面和 web 端**：移动端支持安卓和 IOS，桌面端支持 Windows、Linux 和 Mac，由于几乎 100% 的代码共享，各端具有相同的使用体验。
- **管理 Kubernetes 资源**：可以管理所有的 Kubernetes 资源，包括 CRD 资源。
- **多集群管理**：支持多集群管理，除了通过 kubeconfig 导入集群之外，还提供了 Google GCP、AWS EKS、Azure AKS、DigitalOcean 等集群导入方式。
- **搜索与筛选**：可以全局搜索和筛选 Kubernetes 资源，无需选择固定 Namespace。
- **Logs 日志查看**、**Exec 终端操作**、**Port-Forwarding** 等操作点击即可使用。

### 增强功能

除了 Kubernetes 资源的基本查看和操作外，Kubenav 还提供了插件功能，目前有 Helm、Prometheus、Elasticsearch、Jaeger 四种插件，其中笔者主要体验了 Helm 和 Prometheus 插件的使用。

#### Helm

![Helm 插件](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gphviddn2cj21lc0xnk23.jpg)

Helm 插件 是Kubenav 默认开启的一个插件，通过插件页面可以很轻松的查看到 Helm Chart 的全部信息，包括**配置**、**状态**、**历史信息**和 **Value 值**。其中最实用的就是 Value 的展示，可以直观的看到该 Chart 在部署时 `--set` 的 Value 值，免去了调试时 `helm get values` 的麻烦。

#### Prometheus

![Prometheus 插件](https://tva3.sinaimg.cn/large/ad5fbf65gy1gphvoqpvxjj21lc0x0qb8.jpg)

Prometheus 插件则是需要事先在集群中安装 Prometheus，并在 Kubenav 的 **Setting** -> **General** 中手动开启 Prometheus 插件，之后就可以在集群中通过 ConfigMap 来在 Kubenav 中展示各种 Dashboard 了，可以理解为一个简化版的 Grafana，ConfigMap 内容如下：

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  # Name of the ConfigMap. The name is used as reference in the "kubenav.io/prometheus-dashboards" annotation.
  name: nginx-ingress-dashboard
  # Dashboards namespace, which is configured in the settings via "Dashboards Namespace" or via the "--plugin.prometheus.dashboards-namespace" command-line flag.
  namespace: kubenav
  labels:
    # Required label, so that kubenav can found the dashboard.
    kubenav.io/prometheus-dashboard: "true"
data:
  # Title of the dashboard.
  title: "NGINX Ingress Controller"
  # Description of the dashboard.
  description: "Dashboard for NGINX Ingress Controller Metrics"
  # Array of variables.
  variables: |
    [
      {
        "name": "Namespace",
        "label": "controller_namespace",
        "query": "nginx_ingress_controller_config_hash",
        "allowAll": true
      },
      {
        "name": "ControllerClass",
        "label": "controller_class",
        "query": "nginx_ingress_controller_config_hash{namespace=~\"{{ .Namespace }}\"}",
        "allowAll": true
      },
      {
        "name": "Controller",
        "label": "controller_pod",
        "query": "nginx_ingress_controller_config_hash{namespace=~\"{{ .Namespace }}\",controller_class=~\"{{ .ControllerClass }}\"}",
        "allowAll": true
      },
      {
        "name": "Ingress",
        "label": "ingress",
        "query": "nginx_ingress_controller_requests{namespace=~\"{{ .Namespace }}\",controller_class=~\"{{ .ControllerClass }}\", controller_pod=~\"{{ .Controller }}\"}",
        "allowAll": true
      }
    ]
  # Array of charts.
  charts: |
    [
      {
        "title": "Controller Request Volume",
        "unit": "ops",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "4",
          "lg": "4",
          "xl": "4"
        },
        "type": "singlestat",
        "queries": [
          {
            "label": "Request Volume",
            "query": "round(sum(irate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m])), 0.001)"
          }
        ]
      },
      {
        "title": "Controller Connections",
        "unit": "",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "4",
          "lg": "4",
          "xl": "4"
        },
        "type": "singlestat",
        "queries": [
          {
            "label": "Controller Connections",
            "query": "sum(avg_over_time(nginx_ingress_controller_nginx_process_connections{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m]))"
          }
        ]
      },
      {
        "title": "Controller Success Rate",
        "unit": "%",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "4",
          "lg": "4",
          "xl": "4"
        },
        "type": "singlestat",
        "queries": [
          {
            "label": "Controller Success Rate",
            "query": "sum(rate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\",status!~\"[4-5].*\"}[2m])) / sum(rate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m])) * 100"
          }
        ]
      },
      {
        "title": "Ingress Request Volume",
        "unit": "reqps",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "6",
          "xl": "6"
        },
        "type": "area",
        "queries": [
          {
            "label": "{{ .ingress }}",
            "query": "round(sum(irate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\",ingress=~\"{{ .Ingress }}\"}[2m])) by (ingress), 0.001)"
          }
        ]
      },
      {
        "title": "Ingress Success Rate",
        "unit": "%",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "6",
          "xl": "6"
        },
        "type": "area",
        "queries": [
          {
            "label": "{{ .ingress }}",
            "query": "sum(rate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\",ingress=~\"{{ .Ingress }}\",status!~\"[4-5].*\"}[2m])) by (ingress) / sum(rate(nginx_ingress_controller_requests{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\",ingress=~\"{{ .Ingress }}\"}[2m])) by (ingress) * 100"
          }
        ]
      },
      {
        "title": "Network I/O Pressure",
        "unit": "MB/s",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "4",
          "xl": "4"
        },
        "type": "area",
        "queries": [
          {
            "label": "Received",
            "query": "sum (irate (nginx_ingress_controller_request_size_sum{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m])) / 1024 / 1024"
          },
          {
            "label": "Sent",
            "query": "- sum (irate (nginx_ingress_controller_response_size_sum{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m])) / 1024 / 1024"
          }
        ]
      },
      {
        "title": "Average Memory Usage",
        "unit": "MiB",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "4",
          "xl": "4"
        },
        "type": "area",
        "queries": [
          {
            "label": "NGINX",
            "query": "avg(nginx_ingress_controller_nginx_process_resident_memory_bytes{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}) / 1024 / 1024"
          }
        ]
      },
      {
        "title": "Average CPU Usage",
        "unit": "Cores",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "4",
          "xl": "4"
        },
        "type": "area",
        "queries": [
          {
            "label": "NGINX",
            "query": "sum (rate (nginx_ingress_controller_nginx_process_cpu_seconds_total{controller_pod=~\"{{ .Controller }}\",controller_class=~\"{{ .ControllerClass }}\",namespace=~\"{{ .Namespace }}\"}[2m]))"
          }
        ]
      },
      {
        "title": "Ingress Certificate Expiry",
        "unit": "Days",
        "size": {
          "xs": "12",
          "sm": "12",
          "md": "12",
          "lg": "12",
          "xl": "12"
        },
        "type": "area",
        "queries": [
          {
            "label": "{{ .host }}",
            "query": "(avg(nginx_ingress_controller_ssl_expire_time_seconds{namespace=~\"{{ .Namespace }}\"}) by (host) - time()) / 60 / 60 / 24"
          }
        ]
      }
    ]
```

将该 ConfigMap 部署到 `kubenav` Namespace 中，就可以在 Kubenav 中看到 `NGINX Ingress Controller` 的 Dashboard 了，官方也提供了一些示例 Dashboard，这些 Dashboard 可以在 [kubenav/deploy](https://github.com/kubenav/deploy/tree/master/dashboards) 中找到。

## 总结

![iPad 端](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gphvzznisbj22nw1vcx73.jpg)

除了手机端，Kubenav 在 iPad 端有着更好的表现力，十分适合像笔者这样的 iPad 重度使用者。无论是问题排查还是使用的灵活度都有了大大的提升，结合 AWS Console 和阿里云 APP，基本上除写代码以外的大部分工作都可以在移动端完成，妈妈再也不用担心我蹲在日本街头拿手机远程指挥同事排查问题了！