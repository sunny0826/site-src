---
title: "使用 Kustomize 帮你管理 kubernetes 应用（二）： Kustomize 的使用方法"
date: 2019-04-19T16:05:02+08:00
draft: false
type: blog
banner: "http://wx4.sinaimg.cn/large/ad5fbf65gy1g2816bavnxj21qi15oacw.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "本篇为系列文章第二篇，手把手教你使用 Kustomize 的两种方式。"
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tva3.sinaimg.cn/large/ad5fbf65ly1ge3j4h35xpj21qi15oacw.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
本文介绍使用和维护 Kustomize 的方法及步骤。

## 定制配置
在这个工作流方式中，所有的配置文件（ YAML 资源）都为用户所有，存在于私有 repo 中。其他人是无法使用的。

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g2813d1ia7j20qo0f0dgk.jpg)

1. 创建一个目录用于版本控制

    我们希望将一个名为 ***ldap*** 的 Kubernetes 集群应用的配置保存在自己的 repo 中。
    这里使用 ```git``` 进行版本控制。

    ```bash
    git init ~/ldap
    ```

2. 创建一个 ```base```

    ```bash
    mkdir -p ~/ldap/base
    ```
    在这个目录中创建并提交 ```kustomization.yaml``` 文件和一组资源，例如 ```deployment.yaml``` ```service.yaml``` 等。

3. 创建 ```overlays```

    ```bash
    mkdir -p ~/ldap/overlays/staging
    mkdir -p ~/ldap/overlays/production
    ```
    每个目录都需要一个 ```kustomization.yaml``` 文件以及一个或多个 ```patch``` ，例如 ```healthcheck_patch.yaml``` ```memorylimit_patch.yaml``` 等。。

    ```staging``` 目录可能会使用一个 ```patch``` ，用于在 ```configmap``` 增加一个实验配置。

    ```production``` 目录则可能会在 ```deployment``` 中增加在副本数。

4. 生成 ```variants```

    运行 ```kustomize``` ，将生成的配置用于 kubernetes 应用部署

    ```bash
    kustomize build ~/ldap/overlays/staging | kubectl apply -f -
    kustomize build ~/ldap/overlays/production | kubectl apply -f -
    ```

    在 kubernetes 1.14 版本， ```kustomize``` 已经集成到 ```kubectl``` 命令中，成为了其一个子命令，可使用 ```kubectl``` 来进行部署

    ```bash
    kubectl apply -k ~/ldap/overlays/staging
    kubectl apply -k ~/ldap/overlays/production
    ```

## 使用现成的配置
在这个工作流方式中，可从别人的 repo 中 fork kustomize 配置，并根据自己的需求来配置。

![](http://wx4.sinaimg.cn/large/ad5fbf65gy1g281xyfebej20qo0f0dgr.jpg)

1. 通过 fork/modify/rebase 等方式获得配置

2. 将其克隆为你自己的 ```base```

    在这个 ```bash``` 目录维护在一个 repo 中，在这个例子使用 ```ladp``` 的 repo

    ```bash
    mkdir ~/ldap
    git clone https://github.com/$USER/ldap ~/ldap/base
    cd ~/ldap/base
    git remote add upstream git@github.com:$USER/ldap
    ```

3. 创建 ```overlays```

    如上面的案例一样，创建并完善 ```overlays``` 目录中的内容

    ```bash
    mkdir -p ~/ldap/overlays/staging
    mkdir -p ~/ldap/overlays/production
    ```
    用户可以将 ```overlays``` 维护在不同的 repo 中

4. 生成 ```variants```

    ```bash
    kustomize build ~/ldap/overlays/staging | kubectl apply -f -
    kustomize build ~/ldap/overlays/production | kubectl apply -f -
    ```

    在 kubernetes 1.14 版本， ```kustomize``` 已经集成到 ```kubectl``` 命令中，成为了其一个子命令，可使用 ```kubectl``` 来进行部署

    ```bash
    kubectl apply -k ~/ldap/overlays/staging
    kubectl apply -k ~/ldap/overlays/production
    ```

5. （可选）更新 ```base```
    用户可以定期从上游 repo 中 `rebase` 他们的 `base` 以保证及时更新

    ```bash
    cd ~/ldap/base
    git fetch upstream
    git rebase upstream/master
    ```

## 参考
- [kustomize workflows - github.com](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/workflows.md)