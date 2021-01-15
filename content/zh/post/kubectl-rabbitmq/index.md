---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 kubectl-rabbitmq 部署和运维 K8S 上的 RabbitMQ 集群"
subtitle: ""
summary: "推荐一个 kubectl 插件：kubectl-rabbitmq"
authors: ["guoxudong"]
tags: ["Kubernetes","Kubernetes Plugin","Krew"]
categories: ["Kubernetes Plugin"]
date: 2021-01-15T15:34:10+08:00
lastmod: 2021-01-15T15:34:10+08:00
draft: false
type: blog
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65gy1gmohurr6m0j21qi15owkb.jpg"
---
## 前言

最近接到一个在 K8S 中部署一个 RabbitMQ 集群的任务，既然是部署在 K8S 集群中，首选的当然是 RabbitMQ Operator 了。不过在浏览官方文档时，意外的官方也有开发一个 kubectl-rabbitmq 的插件来帮助部署和运维 RabbitMQ Operator，在试用后发现体验意外的不错。那么本文我们就使用 kubectl-rabbitmq 来部署一个 RabbitMQ 集群吧！

## 插件安装

安装插件前需要安装 [krew](https://krew.sigs.k8s.io/)，也就是 kubectl 的插件管理工具，krew 的安装这里就不做详细说明了。

```shell
& kubectl krew install rabbitmq
```

安装完成后就可以使用 `kubectl rabbitmq` 来部署和管理 RabbitMQ 集群了。


### 安装 RabbitMQ Operator

使用 `kubectl-rabbitmq` 安装 RabbitMQ Operator 非常简单，只需一行命令：

```shell
$ kubectl rabbitmq install-cluster-operator
namespace/rabbitmq-system created
customresourcedefinition.apiextensions.k8s.io/rabbitmqclusters.rabbitmq.com created
serviceaccount/rabbitmq-cluster-operator created
role.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-role created
clusterrole.rbac.authorization.k8s.io/rabbitmq-cluster-operator-role created
rolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/rabbitmq-cluster-operator-rolebinding created
deployment.apps/rabbitmq-cluster-operator created
```

之后就可以看到 `rabbitmq-cluster-operator` 已经不再在 namespace： `rabbitmq-system` 中了：

```shell
$ kubectl get pod -n rabbitmq-system
NAME                                         READY   STATUS    RESTARTS   AGE
rabbitmq-cluster-operator-7bbbb8d559-k8zwm   1/1     Running   0          35m
```

### 创建 RabbitMQ 集群

创建 RabbitMQ 集群同样简单，也是一行命令：

```shell
$ kubectl rabbitmq create <rabbitmq-cluster-name> --replicas 1 --service ClusterIP --image rabbitmq:3.8.9-management
```

这里除了 `<rabbitmq-cluster-name>` 以外，其余参数均为可选项，内容为 RabbitMQ Operator 的 CR 文件。此处的原理也比较简单，只是生成了一份 CR 配置。更多参数请参考[官方文档](https://www.rabbitmq.com/kubernetes/operator/using-operator.html)。

```shell
$ kubectl get RabbitmqCluster
NAME            AGE   STATUS
test-rabbitmq   54s
```

{{% alert title="注意" color="warning" %}}
请确保你集群有可用的 `StorageClass`，因为默认情况下 RabbitMQ Operator 创建的 RabbitMQ 集群会为每个实例使用 `StorageClass` 分配一个 10G 的 PVC
{{% /alert %}}

### 查看集群中所有 RabbitMQ

可以使用 `list` 命令查看集群中所有使用 RabbitMQ Operator 创建的 RabbitMQ 集群：

```shell
$ kubectl rabbitmq list
NAME            AGE   STATUS
test-rabbitmq   10m
```

### 查看指定 RabbitMQ 的所有资源

使用 `get` 命令可以轻松查看指定 RabbitMQ 集群的全部资源：

```shell
$ kubectl rabbitmq get test-rabbitmq
NAME                         READY   STATUS    RESTARTS   AGE
pod/test-rabbitmq-server-0   1/1     Running   0          11m

NAME                                   DATA   AGE
configmap/test-rabbitmq-plugins-conf   1      11m
configmap/test-rabbitmq-server-conf    2      11m

NAME                                    READY   AGE
statefulset.apps/test-rabbitmq-server   1/1     11m

NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)              AGE
service/test-rabbitmq         ClusterIP   10.96.84.122   <none>        15672/TCP,5672/TCP   11m
service/test-rabbitmq-nodes   ClusterIP   None           <none>        4369/TCP,25672/TCP   11m

NAME                                 TYPE     DATA   AGE
secret/test-rabbitmq-default-user    Opaque   3      11m
secret/test-rabbitmq-erlang-cookie   Opaque   1      11m
```

### 打开 RabbitMQ UI

一般情况下，我们访问 ClusterIP 类型的 svc 都要使用到 `pord-forward` 的方式，`kubectl-rabbitmq` 则封装了该方法，使用 `manage` 命令即可马上弹出 UI 界面：

```shell
$ kubectl rabbitmq manage test-rabbitmq
Forwarding from 127.0.0.1:15672 -> 15672
Forwarding from [::1]:15672 -> 15672
Handling connection for 15672
Handling connection for 15672
```

### 获取默认用户

既然已经可以访问 UI 界面了，那么下一步肯定是获取默认用户名/密码，使用 `secrets` 命令即可：

```shell
$ kubectl rabbitmq secrets test-rabbitmq
username: sLSVWbiixZSB_XKL6SoI9kB_Pdefe477
password: zm0oV3UraiadH9E2NNot6Igq8woeEyRi
```

当然也可以直接查看 `secrets` 资源：

```shell
# user
kubectl -n NAMESPACE get secret INSTANCE-default-user -o jsonpath="{.data.username}" | base64 --decode
# pass
kubectl -n NAMESPACE get secret INSTANCE-default-user -o jsonpath="{.data.password}" | base64 --decode
```

现在就顺利登陆 UI 界面了

![RabbitMQ UI](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gmoh3scl4pj23ra1aidox.jpg)


### 监控 RabbitMQ

使用 `observe` 可以在终端观察指定 RabbitMQ 节点的监控信息，如下命令是查看 `test-rabbitmq-server-0` 的监控信息：

```shell
$ kubectl rabbitmq observe test-rabbitmq 0
```

![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gmoh8s8qe0j21rc1a87lq.jpg)

### 验证 RabbitMQ

为了验证 RabbitMQ 是否正常运行，使用 `perf-test` 来进行：

```shell
$ kubectl rabbitmq perf-test test-rabbitmq --rate 100
service/perf-test created
pod/perf-test created
```

这里可以看到拉起了 `perf-test` 的 pod 和 svc，查看其日志：

```shell
$ kubectl logs -f perf-test
id: test-091555-328, starting consumer #0
id: test-091555-328, starting consumer #0, channel #0
id: test-091555-328, starting producer #0
id: test-091555-328, starting producer #0, channel #0
id: test-091555-328, time: 1.000s, sent: 15259 msg/s, received: 9462 msg/s, min/median/75th/95th/99th consumer latency: 5337/140173/167205/239702/259102 µs
id: test-091555-328, time: 2.000s, sent: 44288 msg/s, received: 13328 msg/s, min/median/75th/95th/99th consumer latency: 281875/625383/726050/825489/844442 µs
id: test-091555-328, time: 3.000s, sent: 35377 msg/s, received: 16624 msg/s, min/median/75th/95th/99th consumer latency: 869436/1176394/1463207/1500775/1509740 µs
id: test-091555-328, time: 4.001s, sent: 15490 msg/s, received: 21770 msg/s, min/median/75th/95th/99th consumer latency: 1404599/1549405/1735201/1917333/1953431 µs
id: test-091555-328, time: 5.001s, sent: 17367 msg/s, received: 18243 msg/s, min/median/75th/95th/99th consumer latency: 1967969/2297884/2502092/2649344/2678139 µs
id: test-091555-328, time: 6.005s, sent: 16679 msg/s, received: 17953 msg/s, min/median/75th/95th/99th consumer latency: 2484404/2998748/3165893/3294687/3326600 µs
id: test-091555-328, time: 7.006s, sent: 16729 msg/s, received: 18375 msg/s, min/median/75th/95th/99th consumer latency: 2393535/2753560/2990014/3194592/3224623 µs
id: test-091555-328, time: 8.006s, sent: 20648 msg/s, received: 17156 msg/s, min/median/75th/95th/99th consumer latency: 2472263/2861278/3079786/3256128/3313167 µs

```

在 UI 上也可以看到监控发生了变化。

![](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gmohfwtciuj23p21g8qfb.jpg)

验证完成后删除 `perf-test`:

```shell
$ kubectl delete pod,svc perf-test
```

### 删除 RabbitMQ 集群

完成测试后，使用 `delete` 即可删除 RabbitMQ 集群。

```shell
$ kubectl rabbitmq delete test-rabbitmq
rabbitmqcluster.rabbitmq.com "test-rabbitmq" deleted
```

## 总结

以上的这些功能，使用 `kubectl` 和 yaml 文件也可以完成同样的效果，只不过有些操作比较繁琐。`kubectl rabbitmq` 简化了很多操作，在实际管理和维护 RabbitMQ 集群时，是一个非常不错的工具。

