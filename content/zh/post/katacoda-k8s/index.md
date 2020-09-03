---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Katacoda：免费学习 Kubernetes 利器"
subtitle: "在浏览器终端上学习 Kubernetes"
summary: "本文介绍免费学习 Kubernetes 利器：Katacoda，Katacoda 是一个面向软件工程师的交互式学习和培训平台，可在浏览器中使用真实环境学习和测试新技术，帮助开发人员学习，并掌握最佳实践。"
authors: ["guoxudong"]
tags: ["Kubernetes"]
categories: ["Kubernetes"]
date: 2020-03-27T15:57:11+08:00
lastmod: 2020-03-27T15:57:11+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3iwpp1lvj20r80dmn3s.jpg"
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

最近 ServiceMesher 社区重启了[《Istio 服务网格进阶实战》](https://github.com/servicemesher/istio-handbook) 的编写，我也作为编委会成员参与其中。该书的实践项目都基于 Istio 1.5 版本以及 Katacoda 提供的 Kubernetes 环境完成。由于实践部分都要使用 Katacoda，介绍 Katacoda 这章需要先完成，为其他参与编写实践篇的作者提供参考。

## Katacoda

Katacoda 是一个面向软件工程师的交互式学习和培训平台，可在浏览器中使用真实环境学习和测试新技术，帮助开发人员学习，并掌握最佳实践。该平台于 2019 年 11 月被 O'Reilly 收购。

Katacoda 可以快速的提供一套完整的临时环境，并在使用后将其回收。用户可以根据设计好的引导步骤，通过浏览器上的终端界面操作一套完整的环境，一步步的学习和实践。尤其是在学习 Kubernetes 这种复杂的应用时，单单是创建一个集群就要花去不少时间，同时消耗的资源也令一些初学者望而生畏，Katacoda 的出现很好的解决了这些问题。课程设计者可以定制应用程序所需环境，并设计循序渐进的指导路径，旨在确保用户以最佳方式学习。

在 Katacoda 每个用户都可以免费的学习和创建课程，其中：

- Course：课程，可包含一系列的 scenarios。
    - 官方教程入口：[https://katacoda.com/scenario-examples/scenarios/create-course](https://katacoda.com/scenario-examples/scenarios/create-course)
    - 汉化教程入口：[https://katacoda.com/guoxudong/courses/katacoda-example/create-course](https://katacoda.com/guoxudong/courses/katacoda-example/create-course)
- Scenarios：场景、方案。
    - 官方教程入口：[https://katacoda.com/scenario-examples/scenarios/create-scenario-101](https://katacoda.com/scenario-examples/scenarios/create-scenario-101)
    - 汉化教程入口：[https://katacoda.com/guoxudong/courses/katacoda-example/katacoda-create-scenarios](https://katacoda.com/guoxudong/courses/katacoda-example/katacoda-create-scenarios)

## 使用 Katacoda 学习

Katacoda 提供了非常便利的学习方式，用户只需要打开相应课程，就可以跟着课程设计者的说明，按照设计好的步骤一步步完成学习。

- 介绍会标明课程的难度和需要的时间，帮助用户了解该课程的基本信息：
![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gd8k9b4jwoj21ha0q7wha.jpg)

- 进入课程，左侧是该步骤说明，右侧是一个已经准备好的终端，直接可以使用：
![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gd8kdmfr3ej21h90qeq8s.jpg)

- 之后就是跟着步骤说明，一步步的完成学习即可：
![image](https://tva3.sinaimg.cn/large/ad5fbf65gy1gd8kh1jcs1j21hb0q5do7.jpg)

## 创建课程

既然可以学习别人设计好的课程，那么也可以自己设计课程，以供用户学习。

### 新建仓库

Katacoda 需要注册账号登录，这里直接使用 GitHub 账号登录即可，毕竟之后创建的方案都是存放在 GitHub 上的。

这里推荐在页面新建仓库，访问 https://www.katacoda.com/teach/git-hosted-scenarios ，点击 `Automatically Create and Configure Github Repository` 按钮，Katacoda 会自动在您的 Github 中创建一个名为 `katacoda-scenarios` 的仓库，并自动为您配置 Webhook，每次更新该仓库时，都会自动更新您 Katacoda 中课程的内容。

![katacoda 新建仓库页面](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gd73rov21ij219q0pl42u.jpg)

创建完成后，就可以在您的 Github 上找到名为 `katacoda-scenarios` 的代码仓库。

### Scenarios

Scenarios 即为方案、场景，由一组 Markdown、bash 脚本和一个 JSON 文件组成，这些文件保存了该 Scenarios 的所有配置。

Katacoda 官方提供了 CLI 工具，帮助您创建 Scenarios。

#### 安装 CLI

通过 npm 命令安装 `npm i katacoda-cli --global`。

命令遵循语法的是 `$ katacoda COMMAND`

安装完成后，可以通过运行命令 `katacoda --help` 查看帮助信息。

#### 创建 Scenarios 目录

例如，要创建新的方案，可以通过运行命令 `katacoda scenarios:create`，CLI 将会提示一些信息，帮助您创建方案：

- **Friendly URL:** 此处可输入 `test-scenario`，该属性将确定 scenarios 文件夹的名称，以及用来访问他的 URL。因此，该属性不能包括空格，需要是小写字母等。例如，如果您的用户名是 test-username 并且您的方案称为 test-scenario（如建议的那样），用于在平台中指向该方案的URL将为 https://katacoda.com/test-username/scenarios/test-scenario/
- **Title:** 方案的标题，将会显示在简介上
- **Description:** 方案的描述，将会显示在简介上
- **Difficulty level:** 难度级别，将会显示在简介上
- **Estimated time:** 估计完成的时间，将会显示在简介上
- **Number of steps:** 方案的步骤数。CLI 将会为您的所有步骤创建文件
- **Image:** 确定适用于您的方案的基本软件。例如，如果您需要 docker，java，go 等作为前提条件。更多相关信息，请阅读 https://katacoda.com/docs/scenarios/environments
- **Layout:** 它将确定方案界面元素的配置。例如，如果您只想显示终端，或编辑器+终端等形式，更多相关信息，请阅读 https://katacoda.com/docs/scenarios/layouts

输入这些信息，CLI 将帮您创建一个文件夹，其中引入了 ***friendly URL*** 的名称，并将在该文件夹内创建方案所需的文件。

#### 编辑 Scenarios

Scenarios 目录创建好之后，可以看到目录的结构：
```bash
.
├── finish.md
├── index.json
├── intro.md
├── step1.md
├── step2.md
├── step3.md
├── step4.md
└── step5.md
```

- `index.json` ：文件中定义了标题、描述、步骤顺序、UI 布局以及所需环境，内容与您使用 CLI 工具创建时输入的是一致的，如果想对输入的内容进行修改，也可以在这里修改
- `intro.md`：介绍页，用来介绍您这个 Scenarios
- `finish.md` ：结束页
- `step1-setpN.md`：步骤介绍，数目与您使用 CLI 工具创建 Scenarios 时输入的数目相同

### 上传

将创建的 Scenarios 移动到之前创建的 git 项目中。

```bash
$ git add .
$ git commit -m "New Scenarios"
$ git push origin master
```
上传成功后，在 **Your Profile** 页面就可以看到您上传的课程。

## 总结

Katacoda 是一个面向软件工程师的交互式学习和培训平台，开发人员根据产品特色设计学习流程，方便用户的学习；学习者则无需关心环境的搭建与依赖的安装，通过开发人员设计的最佳实践来进行学习，快速又高效。**最重要的是，它是免费的！白嫖的东西又有谁不喜欢呢？**

同时也欢迎各位朋友一起参与到[《Istio 服务网格进阶实战》](https://github.com/servicemesher/istio-handbook) 的编撰中，和 ServiceMesher 社区的朋友一起完成这部开源书籍。
