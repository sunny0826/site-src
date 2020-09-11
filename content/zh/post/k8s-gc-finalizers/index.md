---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "熟悉又陌生的 k8s 字段：finalizers"
subtitle: ""
summary: "metadata 中熟悉又陌生的字段 finalizers"
authors: ["guoxudong"]
tags: ["kubernetes","operator"]
categories: ["kubernetes"]
date: 2020-09-11T08:47:51+08:00
lastmod: 2020-09-11T08:47:51+08:00
draft: false
type: blog
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65gy1gimpo3u78mj21qi15otcv.jpg"
---
## 前言

经常操作 Kubernetes 集群的同学肯定对 `finalizers` 字段不陌生，每当删除 namespace 或 pod 等一些 Kubernetes 资源时，有时资源状态会卡在 `Terminating`，很长时间无法删除，甚至有时增加 `--force` flag 之后还是无法正常删除。这时就需要 `edit` 该资源，将 `finalizers` 字段设置为 []，之后 Kubernetes 资源就正常删除了。

这是一个比较常见的操作，但是当有人问 `finalizers` 字段的作用是什么的时候，我是懵逼的，我甚至不知道这个熟悉又陌生的单词怎么读！那么这篇文章就来探索一下 `finalizers` 这个字段到底是做什么的，在实践中应该怎么应用这个字段。（另外，这个单词读作 **['faɪnəlaɪzər]**）

## Finalizers

Finalizers 字段属于 Kubernetes GC 垃圾收集器，是一种删除拦截机制，能够让控制器实现异步的删除前（Pre-delete）回调。其存在于任何一个资源对象的 [Meta](https://github.com/kubernetes/apimachinery/blob/master/pkg/apis/meta/v1/types.go#L246) 中，在 k8s 源码中声明为 `[]string`，该 Slice 的内容为需要执行的拦截器名称。

对带有 Finalizer 的对象的第一个删除请求会为其 `metadata.deletionTimestamp` 设置一个值，但不会真的删除对象。一旦此值被设置，finalizers 列表中的值就**只能**被移除。

当 `metadata.deletionTimestamp` 字段被设置时，负责监测该对象的各个控制器会通过**轮询**对该对象的更新请求来执行它们所要处理的所有 Finalizer。 当所有 Finalizer 都被执行过，资源被删除。

`metadata.deletionGracePeriodSeconds` 的取值控制对更新的轮询周期。

每个控制器要负责将其 Finalizer 从列表中去除。

每执行完一个就从 `finalizers` 中移除一个，直到 `finalizers` 为空，之后其宿主资源才会被真正的删除。

```go
// DeletionTimestamp is RFC 3339 date and time at which this resource will be deleted. This
// field is set by the server when a graceful deletion is requested by the user, and is not
// directly settable by a client. The resource is expected to be deleted (no longer visible
// from resource lists, and not reachable by name) after the time in this field, once the
// finalizers list is empty. As long as the finalizers list contains items, deletion is blocked.
// Once the deletionTimestamp is set, this value may not be unset or be set further into the
// future, although it may be shortened or the resource may be deleted prior to this time.
// For example, a user may request that a pod is deleted in 30 seconds. The Kubelet will react
// by sending a graceful termination signal to the containers in the pod. After that 30 seconds,
// the Kubelet will send a hard termination signal (SIGKILL) to the container and after cleanup,
// remove the pod from the API. In the presence of network partitions, this object may still
// exist after this timestamp, until an administrator or automated process can determine the
// resource is fully terminated.
// If not set, graceful deletion of the object has not been requested.
//
// Populated by the system when a graceful deletion is requested.
// Read-only.
// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
// +optional
DeletionTimestamp *Time `json:"deletionTimestamp,omitempty" protobuf:"bytes,9,opt,name=deletionTimestamp"`

// Number of seconds allowed for this object to gracefully terminate before
// it will be removed from the system. Only set when deletionTimestamp is also set.
// May only be shortened.
// Read-only.
// +optional
DeletionGracePeriodSeconds *int64 `json:"deletionGracePeriodSeconds,omitempty" protobuf:"varint,10,opt,name=deletionGracePeriodSeconds"`

// Must be empty before the object is deleted from the registry. Each entry
// is an identifier for the responsible component that will remove the entry
// from the list. If the deletionTimestamp of the object is non-nil, entries
// in this list can only be removed.
// Finalizers may be processed and removed in any order.  Order is NOT enforced
// because it introduces significant risk of stuck finalizers.
// finalizers is a shared field, any actor with permission can reorder it.
// If the finalizer list is processed in order, then this can lead to a situation
// in which the component responsible for the first finalizer in the list is
// waiting for a signal (field value, external system, or other) produced by a
// component responsible for a finalizer later in the list, resulting in a deadlock.
// Without enforced ordering finalizers are free to order amongst themselves and
// are not vulnerable to ordering changes in the list.
// +optional
// +patchStrategy=merge
Finalizers []string `json:"finalizers,omitempty" patchStrategy:"merge" protobuf:"bytes,14,rep,name=finalizers"`
```

## 在 Operator 中的应用

知道了 Finalizers 是什么，那么当然也要知道怎么用 Finalizers 了。在实际开发 Operator 时，删除前（Pre-delete）回调是一个比较常见的功能，用于处理一些在资源删除前需要处理的逻辑，如：关联资源释放、释放资源通知、相关数据清理，甚至是阻止资源删除。一般 Finalizers 的处理也是会 `Reconcile` 中实现的，下面就使用 [chaosblade-operator](https://github.com/chaosblade-io/chaosblade-operator) 中的源码，简单介绍一些 Finalizers 的使用方式。

首先要了解的是 ChaosBlade-Operator 的工作原理：每个实验都会以 CR 的形式部署到 k8s 集群中，之后由 `chaosblade-operator` 来操作以 DaemonSet 形式部署 `chaosblade-tool` 对具体资源进行混沌实验。停止实验只需删除对应 CR 即可，在删除 CR 时，首先会执行一遍实验恢复逻辑，之后才会将 CR 删除。但如果恢复实验失败，则会将 CR 的 `Phase` 设置为 `Destroying`，而在 `Reconcile` 中观测到 `Phase` 状态为 `Destroying` 或者 `metadata.deletionTimestamp` 不为空时，就会不会移除 `finalizers` 中的拦截器名称，阻止该 CR 被删除。

这样设计的目的是为了在实验恢复失败后，让用户去主动查看实验恢复失败原因，如果是一些意外原因导致的实验恢复失败，及时去处理。在确认原因后，可使用 CLI 工具增加 `--force-remove` 进去强制删除，项目维护者在 [Issue#368](https://github.com/chaosblade-io/chaosblade/issues/368) 中也就这个设计给出了解答。

```go
// pkg/controller/chaosblade/controller.go 部分源码
...
const chaosbladeFinalizer = "finalizer.chaosblade.io"
...
func (r *ReconcileChaosBlade) Reconcile(request reconcile.Request) (reconcile.Result, error) {
    reqLogger := logrus.WithField("Request.Name", request.Name)
    forget := reconcile.Result{}
    // Fetch the RC instance
    cb := &v1alpha1.ChaosBlade{}
    err := r.client.Get(context.TODO(), request.NamespacedName, cb)
    if err != nil {
      return forget, nil
    }
    if len(cb.Spec.Experiments) == 0 {
      return forget, nil
    }

    // Destroyed->delete
    // Remove the Finalizer if the CR object status is destroyed to delete it
    if cb.Status.Phase == v1alpha1.ClusterPhaseDestroyed {
      cb.SetFinalizers(remove(cb.GetFinalizers(), chaosbladeFinalizer))
      err := r.client.Update(context.TODO(), cb)
      if err != nil {
        reqLogger.WithError(err).Errorln("remove chaosblade finalizer failed at destroyed phase")
      }
      return forget, nil
    }
    if cb.Status.Phase == v1alpha1.ClusterPhaseDestroying || cb.GetDeletionTimestamp() != nil {
      err := r.finalizeChaosBlade(reqLogger, cb)
      if err != nil {
        reqLogger.WithError(err).Errorln("finalize chaosblade failed at destroying phase")
      }
      return forget, nil
    }
    ...
    return forget, nil
}
```

如果 `Phase` 状态为 `Destroyed`，则从 Finalizers 中移除 `finalizer.chaosblade.io`，之后正常删除 CR。

## 结语

在实际工作中，像 Finalizers 这样的东西太多了，很多平时挂在嘴边的东西，深究起来我们可能对其并不了解，甚至原本的理解就是错误的。在今后的文章中，除了各种实践干货，笔者还会将更多的精力投注到基本原理、底层实现、源码剖析中，更聚焦于技术本身，在不重复造轮子的基础上，学习和了解更多产品背后的代码设计和实现原理。最后在分享一句**弗兰西斯·培根**的话：

**“人生如同道路。最近的捷径通常是最坏的路。”**

## 参考

- [使用 CustomResourceDefinition 扩展 Kubernetes API - kubernetes.io](https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#finalizers)