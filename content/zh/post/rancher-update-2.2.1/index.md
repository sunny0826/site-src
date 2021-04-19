---
title: "单节点版rancher升级指南"
date: 2019-03-31T11:15:35+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65gy1g1lzzx3n3fj20k00m2gpd.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "Rancher 不仅可以在任何云提供商的任何地方部署 Kubernetes 集群，而且还将它们集中在集中式的身份验证和访问控制之下。由于它与资源的运行位置无关，因此您可以轻松地在不同的环境部署你的 kubernetes 集群并操作他们。 Rancher 不是将部署几个独立的 Kubernetes 集群，而是将它们统一为一个单独的托管Kubernetes Cloud。"
tags: ["rancher","容器","kubernetes"]
categories: ["部署安装"]
keywords: ["rancher","容器","kubernetes"]
image:
  url: "https://tvax3.sinaimg.cn/large/ad5fbf65ly1ge3jb8zw5hj20k00m27bc.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
> Rancher 不仅可以在任何云提供商的任何地方部署 Kubernetes 集群，而且还将它们集中在集中式的身份验证和访问控制之下。由于它与资源的运行位置无关，因此您可以轻松地在不同的环境部署你的 kubernetes 集群并操作他们。 Rancher 不是将部署几个独立的 Kubernetes 集群，而是将它们统一为一个单独的托管Kubernetes Cloud。

## 前言
目前我们使用的是 rancher 2.1.1版本，在去年 rancher 发布 ```v2.1.*``` 版本的时候做过一次升级，当时遇到了很多问题，虽然都一一解决，但是并没有有效的记录下来，这里在升级 ```v2.2.*``` 版本的时候做一个记录以便在今后升级的时候的提供参考作用。

## 升级前的准备
- 首先查看当前 rancher 版本，记下这个版本号后面需要使用。查看方式就是登陆 rancher 在左下角就可以看到当前版本号，我们这里使用的```v2.1.1```版本。
- 打开官方文档，这里推荐对照官方文档进行升级，一般官方文档都会及时更新并提供最佳升级方法，而一般的博客会因为其写作时间、使用版本、部署环境的不同有所偏差。官方文档： https://www.cnrancher.com/docs/rancher/v2.x/cn/upgrades/single-node-upgrade/

## 升级
1. 首先获取正在运行的 rancher 容器 ID,由以下命令可知 ```RANCHER_CONTAINER_ID``` 为 ```83167cb60134```

    ```bash
    $ docker ps
    CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              
    PORTS                                       NAMES
    83167cb60134        rancher/rancher:latest   "entrypoint.sh"     4 months ago        Up 4 months         0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   priceless_newton
    ```

2. 停止该容器

    ```bash
    $ docker stop {RANCHER_CONTAINER_ID}
    ```

3. 创建正在运行的 Rancher Server 容器的数据卷容器，将在升级中使用，这里命名为 ```rancher-data``` 容器。

    - 替换{RANCHER_CONTAINER_ID}为上一步中的容器ID。
    - 替换{RANCHER_CONTAINER_TAG}为你当前正在运行的Rancher版本，如上面的先决条件中所述。

        ```bash
        $ docker create --volumes-from {RANCHER_CONTAINER_ID} --name rancher-data rancher/rancher:{RANCHER_CONTAINER_TAG}
        ```

4. 备份 ```rancher-data``` 数据卷容器

    如果升级失败，可以通过此备份还原Rancher Server，容器命名:rancher-data-snapshot-<CURRENT_VERSION>.

    - 替换{RANCHER_CONTAINER_ID}为上一步中的容器ID。
    - 替换{CURRENT_VERSION}为当前安装的Rancher版本的标记。
    - 替换{RANCHER_CONTAINER_TAG}为当前正在运行的Rancher版本，如先决条件中所述 。

        ```bash
        $ docker create --volumes-from {RANCHER_CONTAINER_ID} --name rancher-data-snapshot-{CURRENT_VERSION} rancher/rancher:{RANCHER_CONTAINER_TAG}
        ```

5. 拉取Rancher的最新镜像,这里确保有外网，可能拉取到新的镜像，如果没有外网，这里就需要将镜像上传到私有镜像仓库，将拉取地址设置为私有镜像仓库即可

    ```bash
    $ docker pull rancher/rancher:latest
    ```

6. 通过 ```rancher-data``` 数据卷容器启动新的 Rancher Server 容器。

    **这里要注意到，我们这是使用的是独立容器+外部七层负载均衡，是通过阿里云SLB进行SSL证书认证，需要在启动的时候增加```--no-cacerts```**

    ```bash
    $ docker run -d --volumes-from rancher-data --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:latest --no-cacerts
    ```
    
    >升级过程会需要一定时间，不要在升级过程中终止升级，强制终止可能会导致数据库迁移错误。

    >升级 Rancher Server后， server 容器中的数据会保存到 ```rancher-data``` 容器中，以便将来升级。
7. 删除旧版本 Rancher Server 容器

    如果你只是停止以前的Rancher Server容器(并且不删除它),则旧版本容器可能随着主机重启后自动运行，导致容器端口冲突。

8. 升级成功

    访问 rancher 可以看到右下角版本已经完成更新。

    ![image](http://tva2.sinaimg.cn/large/ad5fbf65gy1g1lzcmucn6j20ck03qt8p.jpg)