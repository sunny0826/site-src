---
title: "Alpine Linux详解"
date: 2019-03-15T09:53:02+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g13fonf2emj21qi15o7k2.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "Alpine以其小巧、简单在docker容器中得到了广泛的应用。但是Alpine Linux使用了musl，可能和其他Linux发行版使用的glibc实现会有些不同。这里主要介绍了它的基础用法，但是足以满足日常运维需要。"
tags: ["Linux","Docker","容器"]
categories: ["系统"]
keywords: ["Linux","Docker","容器"]
image:
  url: "https://tvax3.sinaimg.cn/large/ad5fbf65ly1ge3ibi0d7yj21qi15o7k2.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
### 简介
>Small. Simple. Secure.Alpine Linux is a security-oriented, lightweight Linux distribution based on musl libc and busybox.

>Alpine Linux 是一个社区开发的面向安全应用的轻量级Linux发行版。 Alpine 的意思是“高山的”，它采用了musl libc和busybox以减小系统的体积和运行时资源消耗，同时还提供了自己的包管理工具apk。

### 适用环境
由于其小巧、安全、简单以及功能完备的特点，被广泛应用于众多Docker容器中。我司目前使用的基础镜像均是基于该系统，[dockerhub](https://hub.docker.com/_/alpine)上有提供各种语言的基础镜像.如：```node:8-alpine```、```python:3.6-alpine```，同时也可以基于alpine镜像制作符合自己需求的基础镜像。

### 简单的镜像构建示例

这里提供一个python3的基础镜像的```Dockerfile```，[get-pip.py](https://pip.pypa.io/en/latest/installing/)可在 https://pip.pypa.io/en/latest/installing/ 下载。

```docker
FROM alpine

MAINTAINER guoxudong@keking.cn

# 拷贝安装pip的脚本
COPY get-pip.py /get-pip.py

# 设置alpine的镜像地址为阿里云的地址
RUN echo "https://mirrors.aliyun.com/alpine/v3.6/main/" > /etc/apk/repositories \
    # 安装依赖包
    && apk update \
    && apk add --no-cache bash \
    # libevent-dev libxml2-dev  libffi libxml2 libxslt libxslt-dev  \
    python3 gcc g++ python3-dev python-dev linux-headers libffi-dev openssl-dev \
    # 由于通过apk安装的pip总是基于python2.7的版本，不符合项目要求，此处使用get-pip.py的方式
    #安装基于python3.6的pip
    && python3 /get-pip.py \
    # 删除不必要的脚本
    && cd .. \
    && rm -f /get-pip.py \
    # 此环境专用做运行django项目，因此移除不必要的工具，减少空间
    #    && pip uninstall -y pip setuptools wheel \
    # 最后清空apk安装时产生的无用文件
    && rm -rf /var/cache/apk/*
```

**对比**：同样版本的python，对比镜像大小，可见使用alpine的优势

```bash
~ docker images | grep python
python                                  3.4                 ccbffa0d70d9        2 months ago        922MB
alpine-python3                          latest              69e41b673a50        2 months ago        297MB
```

### apk包管理

- 镜像源配置

    官方镜像源列表：http://dl-cdn.alpinelinux.org/alpine/MIRRORS.txt
    
    >MIRRORS.txt中是当前Alpine官方提供的镜像源（Alpine安装的时候系统自动选择最佳镜像源）

    国内镜像源
    - 清华TUNA镜像源：https://mirror.tuna.tsinghua.edu.cn/alpine/
    - 中科大镜像源：http://mirrors.ustc.edu.cn/alpine/
    - 阿里云镜像源：http://mirrors.aliyun.com/alpine/

    镜像源配置

    这里推荐使用阿里云镜像源，由于公司应用都是部署在阿里云上，使用阿里云镜像源会快很多

    ```bash
    $ vi /etc/apk/repositories
    # 将这两行插入到repositories文件开头
    http://mirrors.aliyun.com/alpine/v3.9/main
    http://mirrors.aliyun.com/alpine/v3.9/community
    # 后面是原有的默认配置
    http://dl-cdn.alpinelinux.org/alpine/v3.8/main
    http://dl-cdn.alpinelinux.org/alpine/v3.8/community
    ```

- apk包管理命令

    这里介绍一些常用的操作apk包管理命令

    - ```apk --help```可以查看完整的包管理命令

    ```bash
    bash-4.3# apk --help
    apk-tools 2.10.0, compiled for x86_64.

    Installing and removing packages:
    add       Add PACKAGEs to 'world' and install (or upgrade) them, while ensuring that all dependencies are met
    del       Remove PACKAGEs from 'world' and uninstall them

    System maintenance:
    fix       Repair package or upgrade it without modifying main dependencies
    update    Update repository indexes from all remote repositories
    upgrade   Upgrade currently installed packages to match repositories
    cache     Download missing PACKAGEs to cache and/or delete unneeded files from cache

    Querying information about packages:
    info      Give detailed information about PACKAGEs or repositories
    list      List packages by PATTERN and other criteria
    dot       Generate graphviz graphs
    policy    Show repository policy for packages

    Repository maintenance:
    index     Create repository index file from FILEs
    fetch     Download PACKAGEs from global repositories to a local directory
    verify    Verify package integrity and signature
    manifest  Show checksums of package contents

    Use apk <command> --help for command-specific help.
    Use apk --help --verbose for a full command listing.

    This apk has coffee making abilities.
    ```

    - ```apk info``` 列出所有已安装的软件包
    - ```apk apk update``` 更新最新本地镜像源
    - ```apk upgrade``` 升级软件
    - ```apk search``` 搜索可用软件包，**搜索之前最好先更新镜像源**

    ```bash
    $ apk search #查找所以可用软件包
    $ apk search -v #查找所以可用软件包及其描述内容
    $ apk search -v 'acf*' #通过软件包名称查找软件包
    $ apk search -v -d 'docker' #通过描述文件查找特定的软件包
    ```

    - ```apk add``` 从仓库中安装最新软件包，并自动安装必须的依赖包,也可以从第三方仓库添加软件包
    
    ```bash
    $ apk add curl busybox-extras       #软件以空格分开这里，这里列举我们用的最多的curl和telnet
    bash-4.3# apk add --no-cache curl
    bash-4.3# apk add mongodb --update-cache --repository http://mirrors.ustc.edu.cn/alpine/v3.6/main/ --allow-untrusted    #从指定镜像源拉取
    ```

    安装指定版本软件包

    ```bash
    bash-4.3# apk add mongodb=4.0.5-r0
    bash-4.3# apk add 'mongodb<4.0.5'
    bash-4.3# apk add 'mongodb>4.0.5'
    ```

    升级指定软件包

    ```bash
    bash-4.3# apk add --upgrade busybox #升级指定软件包
    ```

    **注**：安装之前最好修改本地镜像源，更新镜像源，搜索软件包是否存在，选择合适岸本在进行安装。

    - ```apk del``` 卸载并删除指定软件包

### 结语
Alpine以其小巧、简单在docker容器中得到了广泛的应用。但是Alpine Linux使用了musl，可能和其他Linux发行版使用的glibc实现会有些不同。这里主要介绍了它的基础用法，但是足以满足日常运维需要。毕竟在kubernetes集群中操作容器内环境较直接在虚拟机或者物理机上操作更为复杂，由于缩减的容器的大小，导致和CentOS或Ubuntu相比缺少许多功能。而缺少的这些功能又不想在基础镜像中安装导致容器变大，这个时候就可以在容器运行后，根据实际需要安装即可。

### 参考文档
https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management