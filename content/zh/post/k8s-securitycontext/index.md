---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "熟悉又陌生的 k8s 字段：SecurityContext"
subtitle: ""
summary: "pod 和 containers 中熟悉又陌生的字段 SecurityContext"
authors: ["guoxudong"]
tags: []
categories: []
date: 2020-11-04T11:54:04+08:00
lastmod: 2020-11-04T11:54:04+08:00
draft: false
type: blog
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1gkd7ceh6r0j21qi15otga.jpg"
---
## 前言

如果要投票在 Kubernetes 中很重要，但又最容易被初学者忽略的字段，那么我一定投给 `SecurityContext`。从 Security Context（安全上下文）的名字就可得知它和安全有关，那么它是如何控制容器安全的？又是如何实现的？本文我们就来探索一下 `SecurityContext` 这个字段。

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

更多关于 Capabilites 的内容可以在 [capabilities man page ](https://man7.org/linux/man-pages/man7/capabilities.7.html) 中查看。

{{% alert title="注意" color="warning" %}}
Linux Capabilities 的定义的形式为 `CAP_XXX`。但是你在 Container 字段使用时，需要将名称中的 `CAP_` 部分去掉。例如，要添加 `CAP_SYS_TIME`，可在 `capabilities` 列表中添加 `SYS_TIME`。
{{% /alert %}}

### 只读访问根文件系统

使用 `readOnlyRootFilesystem` 字段，可以配阻止对容器根目录的写入，也十分的实用。

### SELinux

可以通过 SELinux 的策略配置控制用户，进程等对文件等访问控制。若要给 Container 设置 SELinux 标签，可以在 Pod 或 Container 清单的 `securityContext` 中包含的 `seLinuxOptions` 字段进行设置。

```yaml
...
seLinuxOptions:
  level: "s0:c123,c456"
...
```

{{% alert title="注意" color="warning" %}}
要指定 SELinux，需要在宿主操作系统中装载 SELinux 安全性模块。
{{% /alert %}}

## 源码分析

`SecurityContext` 功能的实现更多是通过 runtime 来完成，kubelet 侧多是进行一些判断，将 `SecurityContext` 参数传递给 CRI。

### 禁止容器以 Root 身份运行

kubelet 在创建容器时，会调用 `generateContainerConfig()` 方法来获取容器的配置：

```go
// github/kubernetes/pkg/kubelet/kuberuntime/kuberuntime_container.go
func (m *kubeGenericRuntimeManager) generateContainerConfig(container *v1.Container, pod *v1.Pod, restartCount int, podIP, imageRef string, podIPs []string, nsTarget *kubecontainer.ContainerID) (*runtimeapi.ContainerConfig, func(), error) {
  ...
  // Verify RunAsNonRoot. Non-root verification only supports numeric user.
  if err := verifyRunAsNonRoot(pod, container, uid, username); err != nil {
    return nil, cleanupAction, err
  }
  ...
}
```

而验证 `RunAsNonRoot` 配置的就是 `verifyRunAsNonRoot()` 方法：

```go
// verifyRunAsNonRoot verifies RunAsNonRoot.
func verifyRunAsNonRoot(pod *v1.Pod, container *v1.Container, uid *int64, username string) error {
  effectiveSc := securitycontext.DetermineEffectiveSecurityContext(pod, container)
  // If the option is not set, or if running as root is allowed, return nil.
  if effectiveSc == nil || effectiveSc.RunAsNonRoot == nil || !*effectiveSc.RunAsNonRoot {
    return nil
  }

  if effectiveSc.RunAsUser != nil {
    if *effectiveSc.RunAsUser == 0 {
      return fmt.Errorf("container's runAsUser breaks non-root policy (pod: %q, container: %s)", format.Pod(pod), container.Name)
    }
    return nil
  }

  switch {
  case uid != nil && *uid == 0:
    return fmt.Errorf("container has runAsNonRoot and image will run as root (pod: %q, container: %s)", format.Pod(pod), container.Name)
  case uid == nil && len(username) > 0:
    return fmt.Errorf("container has runAsNonRoot and image has non-numeric user (%s), cannot verify user is non-root (pod: %q, container: %s)", username, format.Pod(pod), container.Name)
  default:
    return nil
  }
}
```

这里的抛错是不是很眼熟？如果 uid 为 0 则返回我们上面说到的 `container has runAsNonRoot and image will run as root` 错误，并且阻止容器创建。

### 获取有效 SecurityContext

`determineEffectiveSecurityContext()` 方法用来确定 Pod 和 Container 中有效的 SecurityContext。

```go
// github/kubernetes/pkg/kubelet/kuberuntime/security_context.go
func (m *kubeGenericRuntimeManager) determineEffectiveSecurityContext(pod *v1.Pod, container *v1.Container, uid *int64, username string) *runtimeapi.LinuxContainerSecurityContext {
  // 合成有效的 SecurityContext，如果 Pod 和 Container 都设置了，则 Container 优先
  effectiveSc := securitycontext.DetermineEffectiveSecurityContext(pod, container)
  synthesized := convertToRuntimeSecurityContext(effectiveSc)
  if synthesized == nil {
    synthesized = &runtimeapi.LinuxContainerSecurityContext{
      MaskedPaths:   securitycontext.ConvertToRuntimeMaskedPaths(effectiveSc.ProcMount),
      ReadonlyPaths: securitycontext.ConvertToRuntimeReadonlyPaths(effectiveSc.ProcMount),
    }
  }

  // 设置 SeccompProfilePath.
  synthesized.SeccompProfilePath = m.getSeccompProfile(pod.Annotations, container.Name, pod.Spec.SecurityContext, container.SecurityContext)

  // 设置 ApparmorProfile.
  synthesized.ApparmorProfile = apparmor.GetProfileNameFromPodAnnotations(pod.Annotations, container.Name)

  // 设置 RunAsUser.
  if synthesized.RunAsUser == nil {
    if uid != nil {
      synthesized.RunAsUser = &runtimeapi.Int64Value{Value: *uid}
    }
    synthesized.RunAsUsername = username
  }

  // 设置 namespace 选项和 Supplemental Groups
  synthesized.NamespaceOptions = namespacesForPod(pod)
  podSc := pod.Spec.SecurityContext
  if podSc != nil {
    if podSc.FSGroup != nil {
      synthesized.SupplementalGroups = append(synthesized.SupplementalGroups, int64(*podSc.FSGroup))
    }

    if podSc.SupplementalGroups != nil {
      for _, sg := range podSc.SupplementalGroups {
        synthesized.SupplementalGroups = append(synthesized.SupplementalGroups, int64(sg))
      }
    }
  }
  if groups := m.runtimeHelper.GetExtraSupplementalGroupsForPod(pod); len(groups) > 0 {
    synthesized.SupplementalGroups = append(synthesized.SupplementalGroups, groups...)
  }

  // 判断是否应该添加 no_new_privs 选项
  synthesized.NoNewPrivs = securitycontext.AddNoNewPrivileges(effectiveSc)

  // 将 ProcMountType 转换为指定或默认的 masked paths
  synthesized.MaskedPaths = securitycontext.ConvertToRuntimeMaskedPaths(effectiveSc.ProcMount)
  // 将 ProcMountType 转换为指定或默认的 readonly paths
  synthesized.ReadonlyPaths = securitycontext.ConvertToRuntimeReadonlyPaths(effectiveSc.ProcMount)

  return synthesized
}
```

筛选出有效的配置会被用来生成 Linux 容器配置。

### docker Seccomp 配置

如果 runtime 为 docker，在 `CreateContainer()` 中：

```go
// github/kubernetes/pkg/kubelet/dockershim/docker_container.go
func (ds *dockerService) CreateContainer(_ context.Context, r *runtimeapi.CreateContainerRequest) (*runtimeapi.CreateContainerResponse, error) {
  ...
  securityOpts, err := ds.getSecurityOpts(config.GetLinux().GetSecurityContext().GetSeccompProfilePath(), securityOptSeparator)
  if err != nil {
    return nil, fmt.Errorf("failed to generate security options for container %q: %v", config.Metadata.Name, err)
  }

  hc.SecurityOpt = append(hc.SecurityOpt, securityOpts...)
  ...
}
```

而 `getSecurityOpts()` 方法就是将 SecurityContext 中 seccomp 有关配置格式化：

```go
// github/kubernetes/pkg/kubelet/dockershim/helpers_linux.go
func (ds *dockerService) getSecurityOpts(seccompProfile string, separator rune) ([]string, error) {
  // Apply seccomp options.
  seccompSecurityOpts, err := getSeccompSecurityOpts(seccompProfile, separator)
  if err != nil {
    return nil, fmt.Errorf("failed to generate seccomp security options for container: %v", err)
  }

  return seccompSecurityOpts, nil
}
// getSeccompSecurityOpts gets container seccomp options from container seccomp profile.
// It is an experimental feature and may be promoted to official runtime api in the future.
func getSeccompSecurityOpts(seccompProfile string, separator rune) ([]string, error) {
  seccompOpts, err := getSeccompDockerOpts(seccompProfile)
  if err != nil {
    return nil, err
  }
  return fmtDockerOpts(seccompOpts, separator), nil
}
// fmtDockerOpts formats the docker security options using the given separator.
func fmtDockerOpts(opts []dockerOpt, sep rune) []string {
  fmtOpts := make([]string, len(opts))
  for i, opt := range opts {
    fmtOpts[i] = fmt.Sprintf("%s%c%s", opt.key, sep, opt.value)
  }
  return fmtOpts
}
```

最终通过 `createResp, createErr := ds.client.CreateContainer(createConfig)` 传递给 docker runtime client，用来创建容器。

## 结语

SecurityContext 相关内容更多是和 Linux 知识挂钩，内容比较庞杂且资料较少，花费了1周多的时间也没能写出令人满意的内容。由于精力有限，只能做一个阶段性的总结，希望今后有空能更深入的解析这部分内容。
