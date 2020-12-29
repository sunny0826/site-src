---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "K8S 资源可视化利器：Kubectl-Graph"
subtitle: ""
summary: "介绍一款可视化 kubernetes resource 关系的 Kubectl 插件：kubectl-graph"
authors: ["guoxudong"]
tags: ["Kubernetes","Kubernetes Plugin","Krew"]
categories: ["Kubernetes Plugin"]
date: 2020-12-29T10:19:18+08:00
lastmod: 2020-12-29T10:19:18+08:00
draft: false
type: blog
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65gy1gm4uslnnn4j21y013e0ue.jpg"
---
## 前言

最近接手了一个规模比较大的集群，光是整理集群中的资源就使人头昏眼花，虽然我自认 `kubectl` 使用的已经十分熟练，但是上千个 kubernetes resource 看下来还是不堪重负。在不能为集群安装任何其他工具的情况下，可以改造的就只有我自己的 client 端，也就是 `kubectl` 了。本文就介绍一个有趣的 kubectl 插件：`kubectl-graph`。

## krew

要介绍 kubectl 的 plugin 机制，首先要介绍的就是 [krew](https://krew.sigs.k8s.io/) 。 krew 是 kubernetes [CLI SIG](https://github.com/kubernetes/community/blob/master/sig-cli/README.md#cli-special-interest-group) 项目，是用来管理 kubectl 插件的工具，作用类似于 yum 和 brew，可以用来搜索、安装和管理 kubectl 插件。

## kubectl-graph

kubectl-graph 是一款可视化 kubernetes resource 及资源间关系的 kubectl 插件，可以将集群中的资源以关系图的方式进行展示。

目前支持两种展示方法：
- [Graphviz](https://graphviz.org/)
- [Neo4j](https://neo4j.com/)

## 前期准备

除了 `kubectl`，由于需要进行绘图，所以还需安装上面两种展示方式的依赖。

### Graphviz

安装 Graphviz 用来生成关系图，需要使用 `dot` CLI 工具，并将图像输出为 SVG 格式：

```bash
$ brew install graphviz
```

### Neo4j

Neo4j 是一个高性能的 NoSQL 图形数据库，它将结构化数据存储在网络上而不是表中，很适合用来展示 kubernetes resource 之间的关系，但 Neo4j 的依赖较多，需要一点时间来安装。

#### 安装 Java

Neo4j 依赖 Java 环境，如果本机上没有安装 Java，请先前往 http://www.java.com 下载并安装。

#### 安装 cypher-shell

因为需要连接到 Neo4j 数据库，所以要安装 `cypher-shell` CLI：

```bash
$ brew install cypher-shell
```

#### 安装 Neo4j Desktop（可选）

接下来就是 Neo4j 本身的安装，我这里使用了 `Neo4j Desktop`，使用和管理起来比较方便，也是使用 `brew` 安装：

```bash
$ brew install --cask neo4j
```

安装好后，运行 `Neo4j Desktop`，完成设置即可

![设置 neo4j](https://tva4.sinaimg.cn/large/ad5fbf65gy1gm4ngqfkvzj21z41kwgqi.jpg)

#### 使用 docker 运行 Neo4j（可选）

当然，如果你感觉安装 `Neo4j Desktop` 比较麻烦，也可以使用 docker 运行 Neo4j：

```bash
$ docker run --rm -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=none neo4j
```

只不过后续查看关系图时，需要使用浏览器访问 http://localhost:7474 来查看结果。

### 安装 kubectl-graph

插件的安装方式比较简单，如果你使用的是 `kubectl 1.19` 之前的版本：

```bash
$ kubectl-krew install graph
```

使用 `kubectl 1.19` 之后的版本：

```bash
$ kubectl krew install graph
```

## 使用方式

安装完成后，就可以开始绘制 kubernetes resource 关系图了。

### Graphviz

使用 `kubectl graph` 命令获取 `kubec-system` 中正在运行的 pod，并通过管道传递给 `dot`：

```bash
$ kubectl graph pods --field-selector status.phase=Running -n kube-system | dot -T svg -o pods.svg
```

查看 `pods.svg` ，资源果然很多：

![pods.svg](https://tva1.sinaimg.cn/large/ad5fbf65gy1gm4nxnytzkj22d41zq19w.jpg)

### Neo4j

Neo4j 可以展示更为丰富且美观的关系图。在导入 kubernetes resource 之前，需要创建一个 Neo4j 数据库：

![创建 neo4j 数据库](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gm4o4b56mzj21z41kw46d.jpg)

数据库创建好后，点击 `Start` 运行并点击 `Open` 打开 `Neo4j Browser`：

![打开数据库](https://tva3.sinaimg.cn/large/ad5fbf65gy1gm4o605br2j20ow0fkjs1.jpg)

执行命令将 kubernetes resource 导入 Neo4j：

```bash
kubectl graph all -n kube-system -o cypher | cypher-shell -u neo4j -p <your-pass>
```

{{% alert title="注意" color="warning" %}}
这里的 `-u` 需要输入 `neo4j` 而不是你创建的数据库名称，`Neo4j Browser` 上也有提示：
![](https://tva3.sinaimg.cn/large/ad5fbf65gy1gm4o9rsqtzj21ve0m440w.jpg)
{{% /alert %}}

之后就可以在 Neo4j 上查看了，输入查询语句：`MATCH (n) RETURN n`：

![关系图](https://tva4.sinaimg.cn/large/ad5fbf65gy1gm4ofe5jiwj22761midzp.jpg)

这时一个美观的 kubernetes resource 关系图就出现了。

## 结语

kubectl 还有很多好用且有趣的 plugin，后续笔者会介绍如何开发一个 kubectl plugin 并分享更多有趣的 plugin。