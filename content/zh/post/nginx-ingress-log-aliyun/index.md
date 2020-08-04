---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "修改 Nginx Ingress 日志打印格式"
subtitle: "结合阿里云日志服务统计系统访问日志"
summary: "修改 nginx-ingress 日志，并结合阿里云日志服务制作系统访问日志统计图表。"
authors: ["guoxudong"]
tags: ["ingress","阿里云","日志服务"]
categories: ["Kubernetes"]
date: 2020-03-02T15:29:16+08:00
lastmod: 2020-03-02T15:29:16+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3ja0bo02j21qi15owib.jpg"
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

最近接到一个需求，需要展示 ingress 上面的访问日志，由于我们的业务系统都部署在 Kubernetes 上面，通过 ingress 进行访问，所以这里的访问日志，其实就是我们全部业务系统的访问日志。

日志采集方面，阿里云天生就提供了 nginx-ingress 日志和采集和展示，本身提供很多不错的基于 ingress 日志数据的图表与分析。如果你使用的是阿里云 ACK 容器服务，那么极端推荐使用，配置方法见官方文档：https://help.aliyun.com/document_detail/86532.html。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1gcfmo5d410j21970nzwjg.jpg)

让人头秃的是，我们这次不但要采集 ingress 日志上比较常规的 `url` `client_ip` `method` `status` 等字段，还要采集我们系统在 `Request Headers` 里面自定义的参数，这些参数是默认的 ingress 并不展示的，所以需要我们进行调整。

## 开始

首先明确需要调整的组件：

- `nginx-ingress` 的 ConfigMap：用于打印自定义日志字段
- `AliyunLogConfig`：这个是阿里云日志服务的 CRD 扩展，需要在这个里面加入新增的字段名和修改后的正则表达式
- 在日志服务控制台，添加新增字段的指定字段查询
- 新增展示仪表盘

### 调整 ingress 日志输出

我们 ingress 组件使用的是 `nginx-ingress-container`，这里要调整日志输出格式，老规矩，直接官方文档：https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/。

从文档可见，只需要调整 `ingress-nginx` 的 ConfigMap `nginx-configuration` data 中的 `log-format-upstream` 字段即可。

{{% alert note %}}
知识点：

官方文档里面给的说明不是很详细，没有提到 `Request Headers` 里自定义的字段应该怎么表示（也有可能是我眼瘸没看见），但经过我多次试验发现，`Request Headers` 里的字段在 `log-format-upstream` 中应该使用 `$http_{your field}` 表示，比如 `$http_cookie`；而带 `-` 的字段则需要将 `-` 改为 `_`，并且使用小写，比如 `app-Id` 就应使用 `$http_app_id` 表示。
{{% /alert %}}

修改 ConfigMap，`ingress-controller` 将进行热更新，看到如下日志，就证明配置已完成更新，接下来就可以看到你自定义字段的值已经打印出来了。

```go
I0302 08:20:58.393365 9 controller.go:200] Backend successfully reloaded.
```

### 调整阿里云日志组件配置

{{% alert note %}}
执行下面的步骤请确保已经按照[官方文档](https://help.aliyun.com/document_detail/86532.html)正确部署阿里云日志服务在您的 K8S 集群之后，并且已达到要求的版本。
{{% /alert %}}

日志已经成功打印了，接下来就是调整日志采集的字段了，这里只需要调整日志服务 CRD 的扩展配置即可。

```bash
$ kubectl edit AliyunLogConfig k8s-nginx-ingress
```
在修改配置之前，推荐先去 https://regex101.com/ 验证正则表达式是否正确，将调整过的正则表达式和 `ingress-controller` 打印的日志贴入下图指定位置，就可以看出正则表达式是否正确。

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gcfo9lxuc6j21gv0juwka.jpg)

然后将添加的字段名称（这个名称将作为 key 在日志服务中展示，可以与 header 中的字段不同）和正则表达式贴入如下 CRD 中。

```yaml
apiVersion: log.alibabacloud.com/v1alpha1
kind: AliyunLogConfig
metadata:
  # your config name, must be unique in you k8s cluster
  name: k8s-nginx-ingress
spec:
  # logstore name to upload log
  logstore: nginx-ingress
  # product code, only for k8s nginx ingress
  productCode: k8s-nginx-ingress
  # logtail config detail
  logtailConfig:
    inputType: plugin
    # logtail config name, should be same with [metadata.name]
    configName: k8s-nginx-ingress
    inputDetail:
      plugin:
        inputs:
        - type: service_docker_stdout
          detail:
            IncludeLabel:
              io.kubernetes.container.name: nginx-ingress-controller
            Stderr: false
            Stdout: true
        processors:
        - type: processor_regex
          detail:
            KeepSource: false
            Keys:
            - client_ip
            - x_forward_for
            - remote_user
            - time
            - method
            - url
            - version
            - status
            - body_bytes_sent
            - http_referer
            - http_user_agent
            - request_length
            - request_time
            - proxy_upstream_name
            - upstream_addr
            - upstream_response_length
            - upstream_response_time
            - upstream_status
            - req_id
            - host
            - #需要添加的字段名称
            - ...
            NoKeyError: true
            NoMatchError: true
            Regex: #修改后的正则表达式
            SourceKey: content
```

### 日志控制台新增字段

如果上面的操作无误的话，日志服务中就会展示您添加的字段了，如果配置有误，所有的自定义字段都会不显示，只会显示保留字段名称。

添加指定字段查询，就可以快速查看添加的字段了。

![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1gcfohy9fv4j21460gxtc6.jpg)

### 新增展示仪表盘

日志既然已经取到了，那么展示就很容易了，直接在查询栏中输入分析语句，日志服务支持 SQL 聚合日志，并直接生成统计图表，点击添加到仪表盘可以就可以添加到现有仪表盘或者新建一个仪表盘。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1gcfos33c23j219a0nuae3.jpg)


## 成果

之后进行一些微调，添加过滤栏，由于这里统计的是登录用户，你甚至都可以添加一个词云来看看哪些用于使用系统比较频繁。当然，想添加什么都看您的喜好，日志在你手里，想怎么分析都可以。

![image](https://tva4.sinaimg.cn/large/ad5fbf65gy1gcfowk10tjj21970ns79i.jpg)

## 结语

本次实现的功能并不是什么高深的功能，只不过是一个简单的访问日志记录和展示，相信每个系统其实都有一套这种功能。但是这种实现方式在我看来优点更多：

- 无代码：全程没有写一行代码，如果有的话，也就是业务需要统一 `Request Headers` 里面的字段。
- 配置简单：只需要修改 nginx ConfigMap 中的一个字段，并在 CRD 中添加字段名称和正在表达式，唯一的难度可能就是正则表达式。
- 配置快：整体的配置时间很短，加上查文档和调整图表也不过半天的时间，肯定比 `提需求-评估-开发-测试-验收` 全流程走一遍，前端后端撕一遍要快的多的多的多。
- 高度定制：可以根据自己的喜好，随意定制图表。

> 最近发现阿里云日志服务是一个宝藏产品，从安全到 k8s 业务，从成本控制到疫情动态，日志服务真的就是把所有没有前端开发资源的服务都帮了一把。
> --- 摘自本人朋友圈
