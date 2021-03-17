---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubectl Plugin 推荐（三）| 插件开发篇"
subtitle: ""
summary: "讲解如何快速开发一款自己的 Kubectl Plugin。"
authors: ["guoxudong"]
tags: ["Kubernetes","Kubernetes Plugin","Krew"]
date: 2021-03-17T09:54:18+08:00
lastmod: 2021-03-17T09:54:18+08:00
draft: false
type: blog
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65gy1gomurdt886j20p00an3zg.jpg"
---
## 前言

之前的两篇文章中笔者推荐了一些好用的 Kubectl Plugin。但在实践中那些插件不一定能满足全部需求，这时不妨动手开发一个，花费时间不多，但却能极高的提升工作效率和使用体验。

本篇文章就来讲解如何快速开发一款自己的 Kubectl Plugin。

## 简介

Kubectl Plugin 的开发流程和注意事项：

- 编写一个二进制可执行文件以 `kubectl-xxx` 命名
- 需要将可执行文件放在环境变量 `PATH` 中
- 之后就可以使用 `kubectl xxx` 来调用了
- 无法覆盖 `kubectl` 内置的命令

二进制可执行文件可以是任何语言开发，本文主要讲解使用 go 语言开发，官方推荐使用 [cli-runtime](https://github.com/kubernetes/cli-runtime/) 项目，可以参考 [sample-cli-plugin](https://github.com/kubernetes/sample-cli-plugin) 示例。

## 使用 GitHub Template

模板地址：https://github.com/replicatedhq/krew-plugin-template

推荐使用 GitHub Template 创建 Plugin 项目，该模板遵循最佳实践开发，并集成了一些简化开发流程的工具，使用 [GoReleaser](https://goreleaser.com/) 和 GitHub Action 进行自动化发布。该 template 也是官方文档推荐的 **非官方 GitHub Template**。


### 创建项目

进入 krew-plugin-template 项目，点击 `Use this template`

![使用模板](https://tva4.sinaimg.cn/large/ad5fbf65gy1gomp3m8xjqj22w81naaqb.jpg)

输入 `Repository name`，点击 `Create repository from template` 创建 repo

![创建项目](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gomp5bbcl2j21fk0x4wis.jpg)

### 配置项目

待项目创建完成后， clone 该项目并在本地通过终端进入该项目所在目录。

```bash
$ git clone https://github.com/sunny0826/kubectl-demo.git
$ cd kubectl-demo
```

使用 `make setup` 命令开始配置

![set up](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gomq1rr8vzj20y80eg0wp.jpg)

根据提示，填入 `GitHub Organization or Username`、`GitHub Repo Namek` 和 `Plugin Name` 后，会自动对项目进行配置。

### 项目结构

配置完成后，一个 Kubectl Plugin 项目的基本框架就完成了，结构如下：

```bash
.
├── LICENSE
├── Makefile
├── README.md
├── cmd
│   └── plugin
│       ├── cli
│       │   └── root.go   # cli 配置
│       └── main.go       # 项目入口
├── deploy
│   └── krew
│       └── plugin.yaml   # krew-index 配置
├── doc
│   └── USAGE.md
├── go.mod
├── go.sum
└── pkg
    ├── logger
    │   └── logger.go
    └── plugin
        └── plugin.go     # 业务逻辑代码
```

接下来就是业务逻辑的开发，主要的业务逻辑代码就写在 `pkg/plugn/plugin.go` 中；CLI 的相关配置，如 `flag` 和子命令的配置则在 `cmd/plugin/cli/root.go` 中。开发所需的 client-go 、k8s API 等资源也都已经集成在项目中，可直接使用。

### 构建测试

在业务逻辑完成后，需要先在本地进行测试。使用 `make bin` 命令可以将项目构建为可执行文件并用来测试，该命令会完成基础的 `fmt`、`vet` 测试并完成 `build`。

![make bin](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gomqjcj5krj21py0uiwnd.jpg)

### 创建 Release

在完成代码逻辑和测试后，就可以将其发布到 github release 了，GitHub Action 和 [GoReleaser](https://goreleaser.com/) 会自动完成 Release 相关操作，只需给项目打上 tag 并 push 代码，无需任何手动操作。

```bash
$ git tag v0.1.0 -m 'initial release'
$ git push --tags
```

>如果是首次 Release，会出现 Release 成功但是 GitHub Action 失败的情况，这里可以忽略，失败的是自动推送 PR 到 krew-index 的步骤。

## 推送到 krew-index

Release 完成后，就可以将已经开发好的 Kubectl Plugin 推送到 [krew-index](https://github.com/kubernetes-sigs/krew-index) 了，这样就可以使用 `kubectl krew install` 安装了。

### 首次推送

首次提交需要手动 fork krew-index 项目并提交 PR，拷贝 `deploy/krew/plugin.yaml` 中的内容，根据 release `checksums.txt` 内容补全不同平台可执行文件的 `sha256` 和 `description` 值，将其放入 `plugin` 目录并提交 PR。

### 自动推送

完成首次 PR 提交后，就可以使用 GitHub Action 自动提交 PR 到 krew-index 了，通过[ krew-release-bot](https://github.com/rajatjindal/krew-release-bot) 机器人自动提交的 PR 无需 review 自动 merge，大大降低了更新流程。

使用这个 GitHub Action，首先需要一份 `.krew.yaml` 配置文件，该项目作者提供了一个不错的工具，可以根据已经提交的 Kubectl Plugin 自动生成 `.krew.yaml` 内容，将生成的配置拷贝到 `.krew.yaml`，之后的 Release 成功后会自动提交 PR 到 krew-index。

![Krew Release Bot Helper](https://tva3.sinaimg.cn/large/ad5fbf65gy1gomu5iw2ngj22bk116afg.jpg)

工具地址：https://rajatjindal.com/tools/krew-release-bot-helper/

在 `<your-git-root>/.github/workflows/release.yml` 添加配置（GitHub Template 中已包含该配置）：

```yaml
name: release
on:
  push:
    tags:
    - 'v*.*.*'
jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@master
    - name: Setup Go
      uses: actions/setup-go@v1
      with:
        go-version: 1.13
    - name: GoReleaser
      uses: goreleaser/goreleaser-action@v1
      with:
        version: latest
        args: release --rm-dist
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    - name: Update new version in krew-index
      uses: rajatjindal/krew-release-bot@v0.0.38
```

之后每次 Release 都会自动推送更新 PR，效果如下：

![PR 自动合并](https://tva1.sinaimg.cn/large/ad5fbf65gy1gomu9xo7gij21ym1rgdu7.jpg)

## 注意事项

官方提供了[插件命名指南](https://krew.sigs.k8s.io/docs/developer-guide/develop/naming-guide/)，大致有以下内容：

- 使用小写字母和连字符，不要使用驼峰式命名
- 表意明确，独一无二
- 使用动词和资源类型命名，如 `open-svc`
- 如果是供应商插件，前缀请使用供应商，如 `gke-login`
- 不能包含 `kube` 前缀
- 避免资源缩写，如 `debug-ingress` 而不能是 `new-ing`

同时 `description` 要描述清楚，且每行不要操作 80 个字符，这样可以方便窄屏幕的用户使用。

## 结语

本篇为《Kubectl Plugin 推荐》系列最后一篇，授人以鱼不如授人以渔，开发一个 Kubectl Plugin 并不会花费很多时间，但根据实际使用情况，为 kubectl 增加更多的功能和拓展，大大提升工作效率和使用体验，非常值得一试。