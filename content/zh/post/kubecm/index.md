---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Kubecm：管理你的 kubeconfig"
subtitle: "帮助你管理杂乱无章的 kubeconfig"
summary: "介绍一款小工具：kubecm，帮助你管理杂乱无章的 kubeconfig。 "
authors: ["guoxudong"]
tags: ["kubernetes"]
categories: ["kubernetes"]
date: 2019-12-09T10:07:46+08:00
lastmod: 2019-12-09T10:07:46+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3ixa9p7oj21ae0sekjm.jpg"
  caption: ""
  focal_point: "TopLeft"
  preview_only: false

---
## 前言

该项目脱胎于 [mergeKubeConfig](https://github.com/sunny0826/mergeKubeConfig) 项目，最早写该项目的目的是在一堆杂乱无章的 kubeconfig 中自由的切换。随着需要操作的 Kubernetes 集群越来越多，在不同的集群之间切换也越来越麻烦，而操作 Kubernetes 集群的本质不过是通过 `kubeconfig` 访问 Kubernetes 集群的 API Server，以操作 Kubernetes 的各种资源，而 `kubeconfig` 不过是一个 yaml 文件，用来保存访问集群的密钥，最早的 [mergeKubeConfig](https://github.com/sunny0826/mergeKubeConfig) 不过是一个操作 yaml 文件的 Python 脚本。而随着 golang 学习的深入，也就动了重写这个项目的念头，就这样 [kubecm](https://github.com/sunny0826/kubecm) 诞生了。

## kubecm

[kubecm](https://github.com/sunny0826/kubecm) 由 golang 编写，支持 `Mac` `Linux` 和 `windows` 平台，`delete` `rename` `switch` 提供比较实用的交互式的操作，目前的功能包括：

- add ：添加新的 `kubeconfig` 到 `$HOME/.kube/config` 中
- completion ：命令行自动补全功能
- delete：删除已有的 `context` ，提供交互式和指定删除两种方式
- merge：将指定目录中的 `kubeconfig` 合并为一个 `kubeconfig` 文件
- rename：重名指定的 `context`，提供交互式和指定重命名两种方式
- switch：交互式切换 `context`

## 安装

[kubecm](https://github.com/sunny0826/kubecm) 支持 `Mac` `Linux` 和 `windows` 平台，安装方式也比较简单：

#### MacOS

{{% alert note %}}
使用 `brew` 或者直接下载二进制可执行文件
{{% /alert %}}

```bash
brew install sunny0826/tap/kubecm
```

#### Linux

{{% alert note %}}
下载二进制可执行文件
{{% /alert %}}

```bash
# linux x86_64
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v${VERSION}/kubecm_${VERSION}_Linux_x86_64.tar.gz
tar -zxvf kubecm.tar.gz kubecm
cd kubecm
sudo mv kubecm /usr/local/bin/
```

#### Windows

{{% alert note %}}
下载二进制可执行文件，并将文件移动到 `$PATH` 中即可
{{% /alert %}}

## 命令行自动补全

{{% alert note %}}
[kubecm](https://github.com/sunny0826/kubecm) 提供了和 [kubectl](https://github.com/kubernetes/kubectl) 一样的 completion 命令行自动补全功能（支持 bash/zsh）
{{% /alert %}}

以 `zsh` 为例，在 `$HOME/.zshrc` 中添加

```vim
source <(kubecm completion zsh)
```

然后使用 `source` 命令，使其生效

```zsh
source $HOME/.zshrc
```

之后，在输入 `kubecm` 后按 <kbd>tab</kbd> 键，就可以看到命令行自动补全的内容

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g9qa0yy3bvj21co0f2hdt.jpg)

## 操作 kubeconfig

{{% alert note %}}
[kubecm](https://github.com/sunny0826/kubecm) 可以实现 `kubeconfig` 的查看、添加、删除、合并、重命名和切换
{{% /alert %}}

#### 查看

```bash
# 查看 $HOME/.kube/config 中所有的 context
kubecm
```

#### 添加

```bash
# 添加 example.yaml 到 $HOME/.kube/config.yaml，该方式不会覆盖源 kubeconfig，只会在当前目录中生成一个 config.yaml 文件
kubecm add -f example.yaml

# 功能同上，但是会将 example.yaml 中的 context 命名为 test
kubecm add -f example.yaml -n test

# 添加 -c 会覆盖源 kubeconfig
kubecm add -f example.yaml -c
```

#### 删除

```bash
# 交互式删除
kubecm delete
# 删除指定 context
kubecm delete my-context
```

#### 合并

```bash
# 合并 test 目录中的 kubeconfig,该方式不会覆盖源 kubeconfig，只会在当前目录中生成一个 config.yaml 文件
kubecm merge -f test 

# 添加 -c 会覆盖源 kubeconfig
kubecm merge -f test -c
```

#### 重命名

```bash
# 交互式重命名
kubecm rename
# 将 dev 重命名为 test
kubecm rename -o dev -n test
# 重命名 current-context 为 dev
kubecm rename -n dev -c
```

#### 切换默认 namespace

```bash
# 交互式切换 namespace
kubecm namespace
# 或者
kubecm ns
# 切换默认 namespace 为 kube-system
kubecm ns kube-system
```

## 效果展示

![](Interaction.gif)

## 视频介绍

{{< blocks/bilibili src="//player.bilibili.com/player.html?aid=88259938&cid=150776221&page=1" >}}

## 结语

[kubecm](https://github.com/sunny0826/kubecm) 项目的初衷为学习 golang 并熟悉 client-go 的使用，随着使用的深入，断断续续增加了不少功能，开发出了一个看上去还算正规的项目。总的来说都是根据自己的喜好来开发的业余项目，欢迎各位通过 [ISSUE](https://github.com/sunny0826/kubecm/issues/new) 来进行交流和讨论。
