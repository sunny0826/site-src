---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "告别手写 Helm Chart README"
subtitle: "一键生成 Helm Chart README 文件"
summary: "helm-docs 可以根据 charts 内容自动生成 markdown 文件。"
authors: ["guoxudong"]
tags: ["helm","kubernetes"]
categories: ["kubernetes"]
date: 2020-05-08T11:20:01+08:00
lastmod: 2020-05-08T11:20:01+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65gy1gekvrbpjl9j21qn15odjn.jpg"
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

随着云原生应用的普及，Helm 的作用也日益凸显，越来越多的云原生应用以 Helm Chart 的形式发布，可以说现在如果没有一个 Helm Chart 都不好意思说自己是云原生应用。

一个好的应用必定有一套好的文档，文档的质量往往和代码的质量成正比。而 Helm Chart 中的 `README.md` 文件就承担了文档的作用，该文件会介绍这个 Helm Chart 的基本信息、使用方式以及参数配置等，用户可以通过该文档的指引，配置符合自己需求的参数，最终完成云原生应用的部署。

但这也给云原生应用的开发者提出了挑战，开发者不但需要把 `value.yaml` 和 `Chart.yaml` 等文件的参数以 Markdown 的形式搬运到 `README.md` 文件中，同时还要将参数的默认值，以及介绍填入表格中。但如果参数出现了变动，往往无法及时更新文档。这就导致了用户明明根据文档配置了参数，但是部署的效果就是无法达到预期。

## Helm-docs

helm-docs 可以根据 charts 内容自动生成 markdown 文件。该文件会包含有关 charts 的元数据，以及 `value.yaml` 中的参数，同时还可以引用子模板（默认为 `README.md.gotmpl`），进一步定制生成的内容。

### 安装

[helm-docs](https://github.com/norwoodj/helm-docs) 使用 golang 开发，支持多平台：

**MacOS**

可以使用 homebrew 安装：

```bash
brew install norwoodj/tap/helm-docs
```

**下载可执行文件**

到 [release](https://github.com/norwoodj/helm-docs/releases) 页面下载对应平台的可执行文件。

### 快速开始

**直接使用可执行文件**

使用方法也很简单，直接进入到 Chart 所在目录，执行命令：

```bash
helm-docs
# 或者
helm-docs --dry-run # 不生成 README.md 文件，而是将生成的内容打印到控制台
```

**使用 docker**

如果不想安装可执行文件，也可以使用 docker，将 Chart 目录挂载到 docker 镜像中，实现相同的效果：

```bash
docker run -v "$(pwd):/helm-docs" jnorwood/helm-docs:latest
# 或者
docker run -v "$(pwd):/helm-docs" jnorwood/helm-docs:latest --dry-run
```

### 进阶实践

下面就以我的开源项目 [cms-grafana-builder](https://github.com/sunny0826/cms-grafana-builder) 为例，讲解 helm-docs 的一些进阶使用。

**添加参数说明**

helm-docs 可以通过 `value.yaml` 中的注释生成参数说明，注释格式如下所示，`--` 后的内容会自动填充到 Chart Values 的 Description 中：

```yaml
# access_key_id -- Aliyun Access Key Id.
access_key_id: ""
# access_secret -- Aliyun Access Secret.
access_secret: ""
# region_id -- Aliyun Region Id.
region_id: "cn-shanghai"
# password -- Grafana admin password.
password: "admin"

image:
  # image.repository -- Image source repository name.
  repository: grafana/grafana
  # image.pullPolicy -- Image pull policy.
  pullPolicy: IfNotPresent
```

**自定义模板**

可以新建 `README.md.gotmpl` 模板来进一步定制 `README.md` 的输出样式。

`README.md.gotmpl` 文件的内容如下，可以在模板中插入 Markdown 来充实 `README.md` 的内容，以及改变展示内容的顺序：

```golang
{{ template "chart.header" . }}
{{ template "chart.description" . }}

{{ template "chart.versionLine" . }}

{{ template "chart.sourceLinkLine" . }}

## Introduction

This chart helps you run a grafana server that include aliyun cms dashboard.


{{ template "chart.requirementsSection" . }}

{{ template "chart.valuesSection" . }}
```

更多内容和示例，详见 https://github.com/norwoodj/helm-docs

## 总结

helm-docs 可以帮助很多像我这样需要维护多个 Helm Chart 的开发者，在更新完或新建 Chart 以后，使用 `helm-docs` 来自动生成 `README.md` 文件，无需逐个寻找和修改，甚至将其集成到 CI 流水线中，自动生成最新的 `README.md`，保证文档和代码的一致。
