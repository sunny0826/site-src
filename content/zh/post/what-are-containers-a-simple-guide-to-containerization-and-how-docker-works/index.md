---
title: "什么的容器？Docker 工作原理及容器化简易指南"
date: 2019-04-20T19:54:50+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65gy1g2adn11fe3j20rs0ijju6.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "Docker 非常棒！ 它使软件开发者无需担心配置和依赖性，在任何地方打包，发送和运行他们的应用程序。而在与 kubernetes 相结合后，它使应用集群部署和管理变得更方便。这使得 Docker 深受软件开发者的喜爱，越来越多的开发者开始使用 Docker。"
tags: ["翻译","Docker","容器"]
categories: ["翻译"]
keywords: ["翻译","Docker","容器"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3jdcoeujj20rs0ijju6.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
**Docker 非常棒！** 它使软件开发者无需担心配置和依赖性，在任何地方打包，发送和运行他们的应用程序。而在与 kubernetes 相结合后，它使应用集群部署和管理变得更方便。这使得 Docker 深受软件开发者的喜爱，越来越多的开发者开始使用 Docker。

那么 Docker 到底是什么？

它是构建、测试、部署和发布**容器化**应用的**平台**。称其为平台是因为 Docker 其实是一套用于管理与容器相关的所有事物的工具。作为 Docker 的核心，接下来我们将深入探讨容器。 

## 什么是容器？
容器提供了在计算机上的隔离环境中安装和运行应用程序的方法。在容器内运行的应用程序仅可使用于为该容器分配的资源，例如：CPU，内存，磁盘，进程空间，用户，网络，卷等。在使用有限的容器资源的同时，并不与其他容器冲突。您可以将容器视为简易计算机上运行应用程序的隔离沙箱。

这个概念听起来很熟悉，有些类似于虚拟机。但它们有一个关键的区别：容器使用的一种非常不同的，轻量的技术来实现资源隔离。容器利用了底层 Linux 内核的功能，而不是虚拟机采用的  [hypervisor](https://en.wikipedia.org/wiki/Hypervisor ) 的方法。换句话说，容器调用 Linux 命令来分配和隔离出一组资源，然后在此空间中运行您的应用程序。我们快速来看下两个这样的功能：

1. **namespaces**

    简单的讲就是，[Linux namespace](http://man7.org/linux/man-pages/man7/namespaces.7.html) 允许用户在独立进程之间隔离 CPU 等资源。进程的访问权限及可见性仅限于其所在的 namespaces 。因此，用户无需担心在一个 namespace 内运行的进程与在另一个 namespace 内运行的进程冲突。甚至可以同一台机器上的不同容器中运行具有相同 PID 的进程。同样的，两个不同容器中的应用程序可以使用相同的端口。

2. **cgroups**

    [cgroups](http://man7.org/linux/man-pages/man7/cgroups.7.html) 允许对可用资源设置限制和约束。例如，您可以在一台拥有 16G 内存的计算机上创建一个 namespace ，限制其内部进程可用内存为 1GB。

到这，您可能已经猜到 Docker 的工作原理了。当您请求 Docker 运行容器时，Docker 会在您的计算机上设置一个资源隔离的环境。然后 Docker 会将打包的应用程序和关联的文件复制到 namespace 内的文件系统中，此时环境的配置就完成了。之后 Docker 会执行您指定的命令运行应用程序。

简而言之，Docker 通过使用 Linux namespace 和 cgroup（以及其他一些命令）来协调配置容器，将应用程序文件复制到为容器分配的磁盘，然后运行启动命令。Docker 还附带了许多其他用于管理容器的工具，例如：列出正在运行的容器，停止容器，发布容器镜像等许多其他工具。

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g2a8h1rc6lj211a0rcjsu.jpg)

与虚拟机相比，容器更轻量且速度更快，因为它利用了 Linux 底层操作系统在隔离的环境中运行。虚拟机的 hypervisor 创建了一个非常牢固的边界，以防止应用程序突破它，而[容器的边界不那么强大](https://sysdig.com/blog/container-isolation-gone-wrong/)。另一个区别是，由于 namespace 和 cgroups 功能仅在 Linux 上可用，因此容器无法在其他操作系统上运行。此时您可能想知道 Docker 如何在 macOS 或 Windows 上运行？ Docker 实际上使用了一个技巧，并在非 Linux 操作系统上安装 Linux 虚拟机，然后在虚拟机内运行容器。

让我们利用目前为止学到的所有内容，从头开始创建和运行 Docker 容器。如果你还没有将 Docker 安装在你的机器上，可以参考[这里](https://docs.docker.com/install/)安装 Docker 。在这个示例中，我们将创建一个 Docker 容器，下载一个用 C语言 写的 Web 服务，编译并运行它，然后使用浏览器访问这个 Web 服务。

我们将从所有 Docker 项目开始的地方：创建一个 ```Dockerfile``` 开始。此文件描述了如何创建用于运行容器的 docker 镜像。既然我们还没有聊到镜像，那么让我们看一下[镜像的官方定义](https://docs.docker.com/get-started/#images-and-containers)：

> 镜像是一个可执行包，其包含运行应用程序所需的代码、运行时、库、环境变量和配置文件，容器是镜像的运行时实例。

简单的讲，当你要求 Docker 运行一个容器时，你必须给它一个包含如下内容的镜像：

1. 包含应用程序及其所有依赖的**文件系统快照**。
2. 容器启动时的运行命令。

在 Docker 的世界，使用别人的镜像作为基础镜像来创建自己的镜像是十分普遍的。例如，官方 reds Docker 镜像就是基于 Debian 文件系统快照（[rootfs tarball](http://www.ethernetresearch.com/geekzone/building-linux-rootfs-from-scratch/)），并安装在其上配置 Redis。

在我们的示例中，我们选择 [Alpine Linux](https://hub.docker.com/_/alpine) 为基础镜像。当您在 Docker 中看到 “alpine” 时，它通常意味着一个精简的基本镜像。 Alpine Linux 镜像大小只有约为5 MB！

在您的计算机创建一个新目录（例如 ```dockerprj``` ），然后新建一个 ```Dockerfile``` 文件。

```bash
umermansoor:dockerprj$ touch Dockerfile
```
将如下内容粘贴到 ```Dockerfile```：

```docker
# Use Alpine Linux rootfs tarball to base our image on
FROM alpine:3.9 

# Set the working directory to be '/home'
WORKDIR '/home'

# Setup our application on container's file system
RUN wget http://www.cs.cmu.edu/afs/cs/academic/class/15213-s00/www/class28/tiny.c \
  && apk add build-base \
  && gcc tiny.c -o tiny \
  && echo 'Hello World' >> index.html

# Start the web server. This is container's entry point
CMD ["./tiny", "8082"]

# Expose port 8082
EXPOSE 8082 
```

这个 ```Dockerfile``` 包含创建镜像的内容说明。我们创建镜像基于 Alpine Linux（[rootfs tarball](http://www.ethernetresearch.com/geekzone/building-linux-rootfs-from-scratch/)），并将工作目录设置为 ```/home``` 。接下来下载，编译并创建了一个用C编写的简单 Web 服务器的可执行文件，然后指定在运行容器时要执行的命令，并将容器端口8082暴露给主机。

现在，我们就可以构建镜像了。在 ```Dockerfile``` 的同级目录运行 ```docker build``` 命令：

```bash
umermansoor:dockerprj$ docker build -t codeahoydocker .
```

如果这个命令成功了，您将看到：

```bash
Successfully tagged codeahoydocker:latest
```

此时我们的镜像就创建成功了，该镜像主要包括：

1. 文件系统快照（Alpine Linux 和 我们安装的 Web 服务）
2. 启动命令（```./tiny 8092```）

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g2aakgpe16j20zo0bqjt5.jpg)

既然成功构建了镜像，那么我们可以使用如下命令运行容器。

```bash
umermansoor:dockerprj$ docker run -p 8082:8082 codeahoydocker:latest
```

让我们了解下这里发生了什么。

通过 ```docker run``` 命令，我们请求 Docker 基于 ```codeahoydocker:latest``` 镜像创建和启动一个容器。```-p 8082:8082``` 将本地的8082端口映射到容器的8082端口（容器内的 Web 服务器正在监听8082端口上的连接）。打开你的浏览器并访问 localhost:8082/index.html 。你将可以看到 ***Hello World*** 信息。

![image](http://wx4.sinaimg.cn/large/ad5fbf65gy1g2aazadeamj20yo0rcq5e.jpg)

最后我想补充一点，虽然 Docker 非常棒，而且对于大多数项目来说它是一个不错的选择，但我们并非处处都要使用它。在我的工作中，Docker 与 Kubernetes 结合使用，可以非常轻松地部署和管理后端微服务，我们不必为每个服务配置新的运行环境。另一方面，对于性能密集型应用程序，Docker 可能不是最佳选择。我经手的其中一个项目必须处理来自移动游戏客户端的 TCP 长连接（每台机器1000个），这时 Docker 网络出现了很多问题，导致无法将它用于该项目。

希望上面这些内容有用。

> 这篇文章由 [Umer Mansoor](https://www.linkedin.com/in/umansoor) 撰写，可以在 [Facebook](https://www.facebook.com/codeahoy) 或 [Twitter](https://twitter.com/codeahoy) 上关注并留下评论。

原文地址： https://codeahoy.com/2019/04/12/what-are-containers-a-simple-guide-to-containerization-and-how-docker-works/