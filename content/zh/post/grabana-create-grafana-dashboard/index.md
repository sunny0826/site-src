---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Grabana：使用 Golang 或 Yaml 生成 Grafana Dashboard"
subtitle: ""
summary: "使用 Golang 或 Yaml 生成 Grafana Dashboard"
authors: ["guoxudong"]
tags: ["grafana","Go"]
categories: ["工具"]
date: 2020-08-26T09:35:23+08:00
lastmod: 2020-08-26T09:35:23+08:00
draft: false
type: blog
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65gy1gi48o1j7l6j21he0tydq9.jpg"
---
## 前言

在之前的一篇文章[《如何使 Grafana as code》](./how-to-configure-grafana-as-code)中介绍了使用 [Jsonnet](http://jsonnet.org/) 实现 Grafana as code，通过代码来批量、动态、可复用的生成 Grafana Dashboard。但毕竟 `Jsonnet` 是一门小众的编程语言，可用文档不多且示例较少，那么有没有使用我们熟悉的编程语言来生成 Grafana Dashboard 的办法呢？答案是肯定的，本篇文章就介绍一款用于生成 Grafana Dashboard 的 Golang 库：[Grabana](https://github.com/K-Phoen/grabana)

## Grabana

Grabana 提供了一种面向开发人员友好的创建 Grafana Dashboard 的方式，也就是俗称的 Grafana as code。

不止于此，Grabana 还支持使用 yaml 文件来生成 Dashboard。并且完全不需要像 Jsonnet 那样先生成 json 配置，再将配置导入 Grafana，而是直接基于写好的代码或者 yaml 文件，通过封装好的 Grafana API 直接将 Dashboard 发布到指定 Grafana 中，省去了繁琐的操作，实现了完全的自动化。

### Dashboard as code

使用 Golang 可以通过如下方式构建 Dashboard 配置：

```go
builder := dashboard.New(
    "Awesome dashboard",
    dashboard.AutoRefresh("5s"),
    dashboard.Tags([]string{"generated"}),
    dashboard.VariableAsInterval(
        "interval",
        interval.Values([]string{"30s", "1m", "5m", "10m", "30m", "1h", "6h", "12h"}),
    ),
    dashboard.Row(
        "Prometheus",
        row.WithGraph(
            "HTTP Rate",
            graph.DataSource("prometheus-default"),
            graph.WithPrometheusTarget(
                "rate(prometheus_http_requests_total[30s])",
                prometheus.Legend("{{handler}} - {{ code }}"),
            ),
        ),
    ),
)
```

配置构建好之后，只需调用 Grabana 的 `client`，传入 Grafana 的地址，以及事先创建的 Grafana API Key 即可一键发布 Dashboard。

{{% alert title="Tips" color="primary" %}}
API Key 的创建方法 `Configuration` - `API Keys` - `Add API Keys` 如下图：

![新建 API Key](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi41u4pq30j21h10pn76o.jpg)

![获得 API Key](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gi41y39kz1j20li0biaaz.jpg)

{{% /alert %}}

创建好 API Key 之后，将其填入 `grabana.WithAPIToken()` 中即可，创建/更新 Dashboard 代码如下：

```go
ctx := context.Background()
client := grabana.NewClient(&http.Client{}, grafanaHost, grabana.WithAPIToken("such secret, much wow"))

// create the folder holding the dashboard for the service
folder, err := client.FindOrCreateFolder(ctx, "Test Folder")
if err != nil {
    fmt.Printf("Could not find or create folder: %s\n", err)
    os.Exit(1)
}

if _, err := client.UpsertDashboard(ctx, folder, builder); err != nil {
    fmt.Printf("Could not create dashboard: %s\n", err)
    os.Exit(1)
}
```

当然官方还提供了一个比较完整的 [example](https://github.com/K-Phoen/grabana/blob/master/cmd/example)，直接使用 `go run main.go` 即可体验一键创建 Dashboard。

### Dashboard as YAML

Grabana 的特别之处还在于他还提供了使用 yaml 创建 Dashboard 的方式，作为一名资深 yaml 工程师，每当看到 yaml 都会感到格外的亲切。

同样的 Dashboard ，yaml 配置如下：

```yaml
# dashboard.yaml
title: Awesome dashboard

editable: true
tags: [generated]
auto_refresh: 5s

variables:
  - interval:
      name: interval
      label: Interval
      values: ["30s", "1m", "5m", "10m", "30m", "1h", "6h", "12h"]

rows:
  - name: Prometheus
    panels:
      - graph:
          title: HTTP Rate
          height: 400px
          datasource: prometheus-default
          targets:
            - prometheus:
                query: "rate(promhttp_metric_handler_requests_total[$interval])"
                legend: "{{handler}} - {{ code }}"
```

目前官方还没有提供类似 `grabana apply -f dashboard.yaml` 这样的 CLI 命令来发布 Dashboard，还是要使用 Golang 代码才能将其发布，代码如下：

```go
content, err := ioutil.ReadFile("dashboard.yaml")
if err != nil {
    fmt.Fprintf(os.Stderr, "Could not read file: %s\n", err)
    os.Exit(1)
}

dashboard, err := decoder.UnmarshalYAML(bytes.NewBuffer(content))
if err != nil {
    fmt.Fprintf(os.Stderr, "Could not parse file: %s\n", err)
    os.Exit(1)
}

ctx := context.Background()
client := grabana.NewClient(&http.Client{}, grafanaHost, grabana.WithAPIToken("such secret, much wow"))

// create the folder holding the dashboard for the service
folder, err := client.FindOrCreateFolder(ctx, "Test Folder")
if err != nil {
    fmt.Printf("Could not find or create folder: %s\n", err)
    os.Exit(1)
}

if _, err := client.UpsertDashboard(ctx, folder, dashboard); err != nil {
    fmt.Printf("Could not create dashboard: %s\n", err)
    os.Exit(1)
}
```

同样也可以找到比较完整的 [example](https://github.com/K-Phoen/grabana/tree/master/cmd/yaml)，这些示例都可以在官方 GitHub 仓库中找到，有兴趣的同学可以看一下。

## 结语

总的来说，这是一个挺有意思的项目，使用 Golang 代码或 yaml 文件来生成 Grafana Dashboard，方便易用不繁琐。美中不足的是，使用 yaml 生成 Dashboard 并没有完全脱离 Golang 代码。就像笔者上文中提到的，其实可以将项目包装成一个 CLI 工具，使用类似 `grabana apply -f dashboard.yaml` 的方式来发布 yaml 配置可能会更好，并且实现起来也并不困难：）。

生成 Grafana Dashboard 其实还有很多其他语言的实现方式，比如使用 Python 实现的 [grafanalib](https://github.com/weaveworks/grafanalib)，与 Grabana 相比 grafanalib 的来头更大，贡献者和 star 数也更多，有兴趣的朋友可以关注一下，这里就不展开详细介绍了。
