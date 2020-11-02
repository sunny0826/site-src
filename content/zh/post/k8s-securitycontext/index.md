---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "熟悉又陌生的 k8s 字段：SecurityContext"
subtitle: ""
summary: "pod 和 containers 中熟悉又陌生的字段 SecurityContext"
authors: ["guoxudong"]
tags: []
categories: []
date: 2020-10-28T11:54:04+08:00
lastmod: 2020-10-28T11:54:04+08:00
draft: true
type: blog
image:
  url: ""
---
## 前言

如果要投票在 Kubernetes 中很重要，但又最容易被初学者忽略的字段，那么我一定投给 `SecurityContext`。Security Context（安全上下文）从名字上就可以知道这个字段和安全有关，那么它是如何控制容器安全的？又是如何实现的？本文我们就来探索一下 `SecurityContext` 这个字段。

## SecurityContext

安全上下文（Security Context）定义 Pod 或 Container 的特权与访问控制设置。安全上下文包括但不限于：

* 自主访问控制（Discretionary Access Control）：基于`用户 ID（UID）`和`组 ID（GID）`来判定对对象（例如文件）的访问权限。
* 安全性增强的 Linux（SELinux）：为对象赋予安全性标签。
* 以特权模式或者非特权模式运行。
* Linux 权能：为进程赋予 root 用户的部分特权而非全部特权。
* AppArmor：使用程序框架来限制个别程序的权能。
* Seccomp：过滤进程的系统调用。
* AllowPrivilegeEscalation：控制进程是否可以获得超出其父进程的特权。 此布尔值直接控制是否为容器进程设置 `no_new_privs` 标志。当容器以特权模式运行或者具有 `CAP_SYS_ADMIN` 权能时，AllowPrivilegeEscalation 总是为 `true`。
* readOnlyRootFilesystem：以只读方式加载容器的根文件系统。

SecurityContext 可以应用于 Container 和 Pod 维度：

* 在 Pod 上设置的安全性配置会应用到 Pod 中所有 Container 上，并且会还会影响 Volume
* 在 Container 上设置的安全性配置仅适用于该容器本身，不会影响到其他容器以及 Pod 的 Volume

## 应用场景

上面的介绍来自官方文档，理解起来并不轻松，但总体思想是确定的：**遵循权限最小化原则，除了业务运行所必需的系统权限，去除不必的其他权限**。下面是笔者总结出的一些常见应用场景来帮助理解 `SecurityContext`。

### 自定义对象访问权限

也就是上文中提到的**自主访问控制（Discretionary Access Control）**，通过设置 UID 和 GID 来达到限制容器权限的目的。

```yaml
...
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
...
```

如上这个配置，可以看到 Pod 中所有容器内的进程都是已用户 `uid=1000` 和主组 `gid=3000`来运行的，而创建的任何文件的属组都是 `groups=2000`。通过这样的方法，可以达到让用户的进程不使用默认的 Root （`uid=0,gid=0`）权限的目的。

### 禁止容器以 Root 身份运行

如果 `runAsNonRoot` 字段配置为 `true`，kubelet 在启动容器时会进行检查，如果以 UID 为 0 运行，则禁止容器启动，该 Pod 的 STATUS 变为 `CreateContainerConfigError`，并生成 Warning 类型事件 `Error: container has runAsNonRoot and image will run as root`。只有当设置 `runAsUser` 或者镜像本身就不是以 Root 身份运行时，该 Pod 才能正常启动。

### 系统 Capabilities 的控制

在某些时候，用户进程需要使用到 Root 用户的某些特权，但是又不想将全部特权都放给容器，这里就用到了 Linux Capabilities，用户可以在在 Container 的 `securityContext` 字段中添加 `capabilities` 字段。在 `capabilities` 字段中，可以添加或移除容器的 Capabilities。

{{% alert title="注意" color="warning" %}}
Linux Capabilities 的定义的形式为 `CAP_XXX`。但是你在 Container 字段使用时，需要将名称中的 `CAP_` 部分去掉。例如，要添加 `CAP_SYS_TIME`，可在 `capabilities` 列表中添加 `SYS_TIME`。
{{% /alert %}}

### 只读访问根文件系统

使用 `readOnlyRootFilesystem` 字段，可以配阻止对容器根目录的写入，也十分的实用。

### SELinux

可以通过 SELinux 的策略配置控制用户，进程等对文件等访问控制。若要给 Container 设置 SELinux 标签，可以在 Pod 或 Container 清单的 `securityContext` 中包含的 `seLinuxOptions` 字段进行设置。

{{% alert title="注意" color="warning" %}}
要指定 SELinux，需要在宿主操作系统中装载 SELinux 安全性模块。
{{% /alert %}}

## 源码分析

## 结语
