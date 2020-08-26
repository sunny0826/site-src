---
title: "centos7安装指定版本的docker"
date: 2018-08-14T20:05:21+08:00
draft: false
type: blog
tags: ["容器", "Linux"]
categories: ["部署安装"]
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0t18xk06xj21qq15ods6.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "在使用CentOS7，并使用荫安装搬运工的时候，往往不希望安装最新版本的搬运工，而是希望安装与自己熟悉或者当前业务环境需要的版本，例如目前Kubernetes支持的最新搬运工版本为v17.03，所以就产生了安装指定版本 docker 的需求。"
keywords: ["容器", "Linux"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3ikxkvspj21qq15ods6.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# 前言
>在使用**centos7**，并使用荫安装搬运工的时候，往往不希望安装最新版本的搬运工，而是希望安装与自己熟悉或者当前业务环境需要的版本，例如目前Kubernetes支持的最新搬运工版本为v17.03，所以就产生了安装指定版本码头工人的需求。

# 安装步骤

```bash
# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加Docker软件包源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#关闭测试版本list（只显示稳定版）
sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test

# 更新yum包索引
yum makecache fast

#NO.1 直接安装Docker CE （will always install the highest  possible version，可能不符合你的需求）
yum install docker-ce

#NO.2 指定版本安装
yum list docker-ce --showduplicates|sort -r 
#找到需要安装的
yum install docker-ce-17.09.0.ce -y
#启动docker
systemctl start docker & systemctl enable docker
```

# 采坑指南
>当然本着万事皆有坑的原则，这里也是有坑的，在安装中也是会遇到如下的问题

在执行一下命令的时候：

```bash
yum install docker-ce-17.03.0.ce -y
```

会出现如下的报错：

```bash
--> Finished Dependency Resolution
Error: Package: docker-ce-17.03.0.ce-1.el7.centos.x86_64 (docker-ce-stable)
        Requires: docker-ce-selinux >= 17.03.0.ce-1.el7.centos
        Available: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)
            docker-ce-selinux = 17.03.0.ce-1.el7.centos
        Available: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)
            docker-ce-selinux = 17.03.1.ce-1.el7.centos
        Available: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)
            docker-ce-selinux = 17.03.2.ce-1.el7.centos
You could try using --skip-broken to work around the problem
You could try running: rpm -Va --nofiles --nodigest
```

在出现这个问题之后，需要执行以下命令：

```bash
#要先安装docker-ce-selinux-17.03.2.ce，否则安装docker-ce会报错
yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm

#然后再安装 docker-ce-17.03.2.ce，就能正常安装
yum install docker-ce-17.03.2.ce-1.el7.centos
```