---
title: "阿里云 ACK 挂载 NAS 数据卷"
date: 2019-07-08T15:09:56+08:00
draft: false
type: blog
banner: "https://tva2.sinaimg.cn/large/ad5fbf65gy1g4siutx8nrj21mf15oaby.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "记录在阿里云购买、配置、挂载 NAS 数据卷到 Kubernetes 集群，由于官方文档没有及时更新，可以看做是对官方文档的补充。"
tags: ["阿里云","kubernetes","容器"]
categories: ["kubernetes"]
keywords: ["阿里云","kubernetes","容器"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3j8edws4j21y013ete4.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

今天接到一个将 NAS 数据卷挂载到 Kubernetes 集群的需求，需要将一个 NAS 数据卷挂载到集群中。这一很简单的操作由于好久没有操作了，去翻看了一下官方文档，发现官方文档还在停留在去年7月份...为了防止之后还有相似情况的发生，这里将所有操作做一个简单记录。

## 购买存储包（创建文件系统）

在挂载 NAS 之前，首先要先购买 NAS 文件存储，这里推荐购买存储包，100G 的 SSD 急速型一年只需1400多，而容量型只要279，对于我这种只有少量 NAS 存储需求的人来说是是靠谱的，因为我只需要5G的左右的存储空间，SSD 急速型 NAS 一年只要18块，完美。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4sglwrx0gj22wa09gae4.jpg)

选择想要创建 NAS 所在 VPC 和 区域

## 添加挂载点

- 点击添加挂载点
![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4sgp0dos2j22ky0iowkr.jpg)

- 选择 VPC 网络、交换机和权限组
![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4sgpwqrgoj20xu0vowib.jpg)

## Linux 挂载 NAS 数据卷

在挂载点创建成功后，就可以将 NAS 数据卷挂载到 Linux 系统，这里以 CentOS 为例：

### 安装 NFS 客户端

如果 Linux 系统要挂载 NAS ，首先需要安装 NFS 客户端
```bash
sudo yum install nfs-utils
```

### 挂载 NFS 文件系统

这里阿里云早就进行了优化，点击创建的文件系统，页面上就可以 copy 挂载命令。页面提供了挂载地址的 copy 和挂载命令的 copy 功能。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4sh2i33wnj22w40yyn55.jpg)

挂载命令：

```bash
sudo mount -t nfs -o vers=4,minorversion=0,noresvport xxxxx.cn-shanghai.nas.aliyuncs.com:/ /mnt
```

### 查看挂载结果

直接在挂载数据卷所在服务上执行命令：

```bash
df -h
```

就可以看到结果：

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4sh6xwyt8j20lj0850tq.jpg)

## Kubernetes 集群挂载 NAS 数据卷

K8S 的持久数据卷挂载大同小异，流程都是：__创建PV__ -> __创建PVC__ -> __使用PVC__

下面就简单介绍在阿里云上的操作：

### 创建存储卷（PV）

首先要创建存储卷，选择 __容器服务__ -> __存储卷__ -> __创建__

这里要注意的是：__挂载点域名使用上面面的挂载地址__

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4shuiiyyqj20hc0hp0tz.jpg)

### 创建存储声明（PVC）

__选择 NAS__ -> __已有存储卷__ 

选择刚才创建的存储卷

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4shv5vs1kj20hx0bvt9g.jpg)

### 使用PVC

使用的方法这里就不做详细介绍了，相关文章也比较多，这里就只记录 Deployment 中使用的 yaml 片段：

```yaml
...
volumeMounts:
- mountPath: /data      # 挂载路径
    name: volume-nas-test
...
volumes:
- name: volume-nas-test
persistentVolumeClaim:
    claimName: nas-test     # PVC 名称
...
```

## 结语

这里只是做一个简单的记录，仅适用于阿里云 ACK 容器服务，同时也是 ACK 的一个简单应用。由于不经常对数据卷进行操作，这里做简单的记录，防止以后使用还要再看一遍文档。
