---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "如何使 Grafana as code"
subtitle: ""
summary: "见多了 Infrastructure as code，今天我们来聊聊 Grafana as code。"
authors: ["guoxudong"]
tags: ["grafana","翻译"]
categories: ["翻译"]
date: 2020-08-19T11:51:17+08:00
lastmod: 2020-08-19T11:51:17+08:00
draft: false
type: blog
image:
  url: "https://tvax3.sinaimg.cn/large/ad5fbf65ly1ghw47grdiuj21he0tyjw8.jpg"
---
## 前言

Grafana Dashboard 可以做很多事情，但您知道其实是可以通过代码来配置管理 Grafana Dashboard 的吗？本文是 Grafana Labs 软件开发工程师 Malcolm Holmes 和 Inuits 的开源顾问 Julien Pivotto 在 FOSDEM 2020 上的 topic 演讲记录。 演讲中，两人讨论了如何使用代码来管理您的 Grafana 实例，并介绍了一些使用 [Jsonnet](http://jsonnet.org/) 的秘诀和技巧以及 [Grafonnet](https://github.com/grafana/grafonnet-libs)（一个用于生成 Grafana Dashboard 的 [Jsonnet](http://jsonnet.org/) 库）。

## 铺天盖地的 Dashboard

Pivotto 在演讲中首先肯定了 Grafana 丰富的 UI，创建一个漂亮的 Dashboard 非常简单。因此用户会创建大量的 Dashboard：当拥有的 Dashboard 越多，也就能越快地了解基础设施的情况。

这一切看起来都很棒，直到一段时间后，因为习惯和目标的冲突不得不需要去维护这套 Dashboard 时；或者处理不同的 graph 使用不同的颜色表示错误而引发的混乱时；又或者需要修改 50 个 Dashboard 才能使用 Grafana 的一个新功能时。这些时候，确保 Dashboard 正常工作且好看不再是一个简单的过程了。

从开发到运维，看到相同的 Dashboard 并对 Dashboard 具有相同的理解是很重要的，但是实现这个目标却很困难，这也就是您需要 Grafana as code 的原因。

Grafana Dashboard 可以通过很多方式创建：通过 Grafana UI、通过 Grafana REST API、Terraform，甚至可以直接将数据推送到 Grafana 数据库。

不仅如此，现在 Grafana 还支持开箱即用的文件配置，这意味着您可以对 Grafana 说，“嘿，请进入指定的目录，检查里面所有的 JSON 文件，并将它们作为 Dashboard 加载到 Grafana 中。”而当更新那些文件时，Grafana 会自动读取它们并更新 Dashboard，这真的很棒，您可以对文件进行编码并使  Dashboard 内容与的文件配置保持一致。

Grafana Dashboard 面板中的所有内容均为 JSON，非常简单易懂。但 JSON 本身“并不够好”，很难使用传统的模板系统为其制作模板。

![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghw6c59vh8j22bc1avwlg.jpg)

## Jsonnet

“对此我们能做些什么？” Holmes 在演讲开始时提出了这个问题。

之后他提出了一种使用 JSON 更好的办法：一种名为 **Jsonnet** 编程语言，其也可用于将资源部署到 Kubernetes，Jsonnet 脚本的输出结果就是 JSON。

“Jsonnet 具有许多种语言功能，这使得生成 JSON 以及与他人合作生成 JSON 成为一种乐趣。” Holmes 补充到。

![](https://tva2.sinaimg.cn/large/ad5fbf65gy1ghw6vj3mn3j22bc1av450.jpg)

上面这个示例：

- 定义一个局部变量，稍后引用该变量。
- 不必使用引号即可获得“不错的语法糖”。
- 双冒号语法意味着 `hidden` 可以在程序的其他地方使用，但不会最终出现在最终的 JSON 输出中。

## 关于语言

Holmes 还强调了 Jsonnet 三个与众不同之处。

### Functions

![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghw76mjmj8j22bc1av0yt.jpg)

在这个简化的示例中，定义了一个名为 `dashboard()` 的函数，包含两个参数：`title` 和 `uid`。Jsonnet 可以将很长的 JSON 内容封装在非常简单的命令中。

但在上面的示例中，`schemaVersion` 字段是写死的，如果想使用 `schemaVersion:22` 而不是函数中提供的 `schemaVersion:20`，`dashboard()`函数就没法解决这个问题的。

### Patches

而 Jsonnet 的 patches 功能，可以解决这个问题。在调用 Jsonnet 函数时可以为其添加 JSON 代码段，从而达到添加/覆盖指定字段的目的。

如下所示，`schemaVersion` 字段值被覆盖了：

![](https://tva1.sinaimg.cn/large/ad5fbf65gy1ghw7lembckj22bc1av0y8.jpg)

这些代码“功能非常强大”，其使您拥有了拓展更多内容的能力。

### Imports

Jsonnet 不仅可以创建函数，还可以将写好的函数 Import 到文件中。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1ghw8jfq0s1j22bc1avjyc.jpg)

在上述示例中，事先已经写好了一个函数，并将该函数放入名为 `dashboard.libsonnet` 的文件中。然后在 `main.jsonnet` 文件中，将该 Dashboard 文件加载到名为 `dashboard` 的局部变量中，并调用 `new()` 方法。然后添加`schemaVersion: 22` 字段，获得与之前相同的结果。

通过 Import 我们可以复用大量代码，最终生成更多的 Dashboard。

Jsonnet 还有有一个名为 Jsonnet bundler 的工具，有点类似于 Golang 的 vendor，它可以从 GitHub 或者类似的地方获取 Jsonnet 库，因此您可以与其他人分享 Dashboard 和资源，这使得协作更加有效。

### 现成的轮子

这是两个现成的库：

- https://github.com/grafana/grafonnet-lib
- https://github.com/grafana/jsonnet-libs/tree/master/grafana-builder

其中最突出的是 Grafonnet，也是 Holmes 和 Pivotto 在演示中使用的，这是一个非常简洁的库，提供了：创建 Dashboard ，创建 panel，创建 single stat panel 等基本功能。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1ghw9403k2zj22bc1avk2w.jpg)

## 演示

然后他们进行了快速演示。

{{< blocks/bilibili src="//player.bilibili.com/player.html?aid=796818511&bvid=BV18C4y1t7rZ&cid=226548812&page=1" >}}

## 未来

Holmes 说，在 Grafana Lab 内部已经有不少关于如何能让 Grafana 实例作为代码被管理得更好的讨论。我们相信这很有用，讨论已经带来了很多点子。比如，如果 Grafana 本身带有原生 Jsonnet 功能，那么就可以不用运行 Jsonnet 来生成 JSON，而是只要使用 Grafana 本身的能力就可以了。

我们可以预先对应用程序进行包装，其中既嵌入了代码又包含 Dashboard 和监视的配置。这样的话， Dashboard（及其附带的所有内容）的运行方式将与其余代码相同。这是特别有用的，因为人们喜欢把 Dashboard 放在 CI/CD pipline 的版本控制和集成中，就像他们其他的代码一样。

Grafonnet 是一个开源项目，在未来一年左右的时间里，Grafonnet 将会快速的发展。

{{% pageinfo color="primary" %}}
原文地址：[https://grafana.com/blog/2020/02/26/how-to-configure-grafana-as-code/](https://grafana.com/blog/2020/02/26/how-to-configure-grafana-as-code/)
{{% /pageinfo %}}
