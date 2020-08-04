---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "如何参与开源项目"
subtitle: ""
summary: "本文根据作者参与开源项目的经验，介绍了如何参与到开源项目中贡献自己的力量。"
authors: ["guoxudong"]
tags: ["开源"]
categories: ["开源"]
date: 2020-05-20T11:12:59+08:00
lastmod: 2020-05-20T11:12:59+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65gy1gf12hx32qgj20m40bmq2z.jpg"
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

这篇文章的起因是朋友的一个疑问：如何参与开源项目？搜索了一下网上类似的文章，大多都是讲解如何操作 GitHub 来给开源项目贡献代码、开源协议有哪些以及开源项目的一些介绍。而开源项目作为开源思想的产物，最难的从来都不是贡献代码，而参与的方式也不只有贡献代码一种（虽然贡献代码是最直接的）。下面就根据我的经验，介绍一下如何参与到开源项目中。

## 心理建设

在和不同的小伙伴聊过之后，发现大家都有一个同样的问题：很多同学都觉得参与开源项目是技术大牛的事情，我们这种技术水平一般的，只需学习怎么使用就行了。

那么这就引出了小标题**心理建设**的重要性，一个开放的项目（伪开源不算），是可以接纳任何可行有益的建议的，只要是对项目有利的贡献，都会得到认可（前提是你能说服别人）。会有一些技术大牛维护这些项目，但不代表任何参与该项目人都是技术大牛，就如之前 4 岁小女孩可以给 Linux 内核贡献提交，只要你的提交可以提升项目质量，哪怕是一个符号的修改都会得到肯定。社区是开放的，任何人都可以参与进来；社区又是严谨的，只要有错误，任何人都可以修改它，并不是“大神”们的专利。

## 如何开始

开始前，首先要明确你想要做的内容，除了贡献代码以外，bug 的发现、新功能的建议、文档的补充、测试用例的完善，甚至是错别字的纠正，这些都是参与开源项目的方式。正如上文所说，一个开放的社区，是不会拒绝任何可以提升项目质量的行为的。

而代码贡献方面，如果有志于贡献高质量的代码、修复 bug 或贡献新功能，在开始时，可以打开 ISSUE，里面有一些打着 `good first issue` Label 的 ISSUE，这些 ISSUE 通常会使一些小功能的开发或者 bug 的修复，你可以通过完成这个 ISSUE 来踏出你贡献代码的第一步。当然，在该 ISSUE 中的交流时必不可少的，这样可以帮助你更详细的了解该 ISSUE 要解决的问题，从而在开发中少走弯路。

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gf0ylqtijgj20sx0bjabu.jpg)

## 贡献规范

每个开源项目都会有自己的**贡献指南（Contributing Guidelines）**，在参与项目之前，请先**阅读该指南**，一般该指南中都会有诸如 Slack channel 或邮件列表这样的沟通工具的链接，或者钉钉、微信这样即时交流工具的二维码（当然最简单的是在 ISSUE 中交流，并且这也是最直接也最容易得到回复的沟通方式）。

一般来说，**不要在 fork 代码的 `master` 分支上做任何修改，该分支用来和上游代码库保持同步！** 一个开发分支对应一个功能点，并且对应一个 PR，一个 PR 对应一个 ISSUE，最好不要将多个功能写在一个 PR 里，这样会增加项目维护者 review 的难度。

## GitHub 操作

下面就是一些在 Github 上的操作规范。

**fork 项目**

要参与项目贡献，首先需要 fork 项目代码，在项目页面点击 `fork` 按钮，将其 fork 到自己的仓库中：

![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gf05j7h9uzj20ty07ywfd.jpg)

**本地配置**

在项目 fork 好之后，将其拉取到本地：

```bash
$ git clone https://github.com/sunny0826/kubernetes.git
```

为了保持与上游仓库代码一致，添加上游仓库：

```bash
$ git remote add upstream https://github.com/kubernetes/kubernetes.git
```

上游仓库添加好之后，之后都可以通过以下命令来使本地仓库与上游仓库保持同步：

```bash
# 切换分支
$ git checkout master
# 更新上游代码
$ git fetch upstream
# 合并代码到 master 分支
$ git merge upstream/master
# 上传代码
$ git push
```

**新建开发分支**

在同步上游仓库之后，就可以新建分支，添加自己的修改了：

```bash
$ git checkout -b xxx
```

**提交 PR**

在开发完成后，就可以 push 代码，然后提交 PR 了。一般需要在提交 PR 时，将关联的 ISSUE 标出，并说明该 PR 解决了什么问题。

而一些大的开源组织会有一些其他要求，比如 CNCF 的项目，在提交 PR 后 [k8s-ci-robot](https://github.com/k8s-ci-robot) 会检测你是否签署了 Linux 基金会的[贡献许可协议](https://identity.linuxfoundation.org/projects/cncf)（如果没有，就会要求你先签署一下该协议），同时还会做一些其他操作，比如根据内容打一些标签、做一些简单的测试，确保代码无冲突，并给你分配 reviewer（当然也可以使用 `/assign` 命令自己指定）。

**Review**

在提交 PR 后，就会有人来对你提交的代码进行 review。当然，这个人并不会立即出现，因为大部分开源项目的维护者都不是全职的，并且如果该项目的维护者在国外，还需要考虑时差问题。

reviewer 会对你提交的内容进行一些评论，可能是需要更改的点，或者是需要增加一些相关的单元测试，这个过程将一直持续，直到这些内容达到合并的标准，当看到 `/lgtm` 时，恭喜你，你的代码通过 review 被合并了。

**删除开发分支**

PR 被成功合并后，就可以对之前开发的分支进行清理了，因为在 review 中，会提交多个 commit，而合并一般会将这些 commit 压缩为一个 commit 然后合并到 `master` 分支，这就导致了 commit 信息的不一致，这也是为什么在前文要求不要使用 `master` 分支的原因，如果使用 `master` 分支，在提交几次 PR 后，就会多出很多很多的 commit...

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gf07ejzdglj20mb04fdg6.jpg)

清理本地分支：

```bash
$ git branch -d xxx
```

这种方式就是通过分支管理，让 `master` 分支始终与上游仓库的 commit 信息一致，而从 `master` 分支 checkout 出开发分支，在开发分支内容合并入上游之后，只需同步 `master` 分支内容，然后重复上面的步骤就可以开始新的开发了。

## 总结

以上就是这些年我参与开源项目的一些经验，是对网上一些文章的补充，其实不同开源组织的贡献方式不尽相同，在参与之前一定要先阅读**贡献指南（Contributing Guidelines）**，这样会少走很多弯路。还有就是在 ISSUE 中的交流请尽量使用英语，哪怕你知道和你交流的是一名可以读懂汉语的人，这样做的原因是为了让其他不懂汉语的社区成员可以读懂你们交流，从而参与进来，而不是让人以为你们在“密谋”着什么。
