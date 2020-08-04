---
title: "使用 Velero 进行集群备份与迁移"
date: 2019-11-13T09:13:22+08:00
draft: false
type: blog
banner: "https://tvax2.sinaimg.cn/large/ad5fbf65gy1g8wfan6eq1j21qi15o0yu.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本文介绍了使用 Velero 来进行 k8s 集群资源进行备份和迁移。"
tags: ["阿里云","kubernetes","velero"]
categories: ["kubernetes"]
keywords: ["阿里云","kubernetes","velero"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3ia8txjdj21qi15o0yu.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## 前言

在近日的一个风和日丽的下午，正在快乐的写 bug 时，突然间钉钉就被 call 爆了，原来是 k8s 测试集群的一个 namespace 突然不见了。这个 namespace 里面有 60 多个服务，瞬间全部没有了……虽然得益于我们的 CI/CD 系统，这些服务很快都重新部署并正常运行了，但是如果在生产环境，那后果就是不可想象的了。在排查这个问题发生的原因的同时，集群资源的灾备和恢复功能就提上日程了，这时 Velero 就出现了。

## Velero

[Velero](https://github.com/vmware-tanzu/velero) 是 VMWare 开源的 k8s 集群备份、迁移工具。可以帮助我们完成 k8s 的例行备份工作，以便在出现上面问题的时候可以快速进行恢复。同时也提供了集群迁移功能，可以将 k8s 资源迁移到其他 k8s 集群的功能。Velero 将集群资源保存在对象存储中，默认情况下可以使用 [AWS](https://velero.io/docs/v1.1.0/aws-config)、[Azure](https://velero.io/docs/v1.1.0/azure-config)、[GCP](https://velero.io/docs/v1.1.0/gcp-config) 的对象存储，同时也给出了插件功能用来拓展其他平台的存储，这里我们用到的就是阿里云的对象存储 OSS，阿里云也提供了 Velero 的插件，用于将备份存储到 OSS 中。下面我就介绍一下如何在阿里云容器服务 ACK 使用 Velero 完成备份和迁移。

> Velero 地址：https://github.com/vmware-tanzu/velero

> ACK 插件地址：https://github.com/AliyunContainerService/velero-plugin

### 下载 Velero 客户端

Velero 由客户端和服务端组成，服务器部署在目标 k8s 集群上，而客户端则是运行在本地的命令行工具。

- 前往 [Velero 的 Release 页面](https://github.com/vmware-tanzu/velero/releases) 下载客户端，直接在 GitHub 上下载即可
- 解压 release 包
- 将 release 包中的二进制文件 `velero` 移动到 `$PATH` 中的某个目录下
- 执行 `velero -h` 测试

### 创建 OSS bucket

创建一个 OSS bucket 用于存储备份文件，这里也可以用已有的 bucket，之后会在 bucket 中创建 `backups`、`metadata`、`restores`三个目录，这里建议在已有的 bucket 中创建一个子目录用于存储备份文件。

创建 OSS 的时候一定要选对区域，要和 ACK 集群在同一个区域，存储类型和读写权限选择**标准存储**和**私有**：

![image](https://tva3.sinaimg.cn/wap720/ad5fbf65gy1g8w7t8c4xbj21021d8thq.jpg)

### 创建阿里云 RAM 用户

这里需要创建一个阿里云 RAM 的用户，用于操作 OSS 以及 ACK 资源。

- 新建权限策略

    ![image](https://tvax4.sinaimg.cn/large/ad5fbf65gy1g8w80cjiv2j21uo18cag8.jpg)

    策略内容：

    ```json
    {
        "Version": "1",
        "Statement": [
            {
                "Action": [
                    "ecs:DescribeSnapshots",
                    "ecs:CreateSnapshot",
                    "ecs:DeleteSnapshot",
                    "ecs:DescribeDisks",
                    "ecs:CreateDisk",
                    "ecs:Addtags",
                    "oss:PutObject",
                    "oss:GetObject",
                    "oss:DeleteObject",
                    "oss:GetBucket",
                    "oss:ListObjects"
                ],
                "Resource": [
                    "*"
                ],
                "Effect": "Allow"
            }
        ]
    }
    ```
- 新建用户

    在新建用户的时候要选择 `编程访问`，来获取 `AccessKeyID` 和 `AccessKeySecret`，这里请创建一个新用于用于备份，不要使用老用户的 AK 和 AS。

    ![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1g8w8h4ek4uj21h40ue785.jpg)

### 部署服务端

- 拉取 [Velero 插件](https://github.com/AliyunContainerService/velero-plugin) 到本地

    ```bash
    git clone https://github.com/AliyunContainerService/velero-plugin
    ```

- 配置修改

    1. 修改 `install/credentials-velero` 文件，将新建用户中获得的 `AccessKeyID` 和 `AccessKeySecret` 填入，这里的 OSS EndPoint 为之前 OSS 的访问域名（**注：这里需要选择外网访问的 EndPoint。**）：

        ![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1g8w8xd1sgzj21c20cm75z.jpg)

        ```bash
        ALIBABA_CLOUD_ACCESS_KEY_ID=<ALIBABA_CLOUD_ACCESS_KEY_ID>
        ALIBABA_CLOUD_ACCESS_KEY_SECRET=<ALIBABA_CLOUD_ACCESS_KEY_SECRET>
        ALIBABA_CLOUD_OSS_ENDPOINT=<ALIBABA_CLOUD_OSS_ENDPOINT>
        ```

    2. 修改 `install/01-velero.yaml`，将 OSS 配置填入：

        ```yaml
        ---
        apiVersion: velero.io/v1
        kind: BackupStorageLocation
        metadata:
        labels:
            component: velero
        name: default
        namespace: velero
        spec:
        config: {}
        objectStorage:
            bucket: <ALIBABA_CLOUD_OSS_BUCKET>  # OSS bucket 名称
            prefix: <OSS_PREFIX>    # bucket 子目录
        provider: alibabacloud
        ---
        apiVersion: velero.io/v1
        kind: VolumeSnapshotLocation
        metadata:
        labels:
            component: velero
        name: default
        namespace: velero
        spec:
        config:
            region: <REGION>    # 地域，如果是华东2（上海），则为 cn-shanghai
        provider: alibabacloud
        ```

    3. k8s 部署 Velero 服务

        ```bash
        # 新建 namespace
        kubectl create namespace velero
        # 部署 credentials-velero 的 secret
        kubectl create secret generic cloud-credentials --namespace velero --from-file cloud=install/credentials-velero
        # 部署 CRD
        kubectl apply -f install/00-crds.yaml
        # 部署 Velero
        kubectl apply -f install/01-velero.yaml
        ```

    4. 测试 Velero 状态

        ```bash
        $ velero version
        Client:
            Version: v1.1.0
            Git commit: a357f21aec6b39a8244dd23e469cc4519f1fe608
        Server:
            Version: v1.1.0
        ```

        可以看到 Velero 的客户端和服务端已经部署成功。

    5. 服务端清理

        在完成测试或者需要重新安装时，执行如下命令进行清理：

        ```bash
        kubectl delete namespace/velero clusterrolebinding/velero
        kubectl delete crds -l component=velero
        ```

### 备份测试

`velero-plugin` 项目中已经给出 `example` 用于测试备份。

- 部署测试服务
```bash
kubectl apply -f examples/base.yaml
```
- 对 `nginx-example` 所在的 namespace 进行备份
```bash
velero backup create nginx-backup --include-namespaces nginx-example --wait
```
- 模拟 namespace 被误删
```bash
kubectl delete namespaces nginx-example
```
- 使用 Velero 进行恢复
```bash
velero restore create --from-backup nginx-backup --wait
```

### 集群迁移

迁移方法同备份，在备份后切换集群，在新集群恢复备份即可。

### 高级用法

- 定时备份

    对集群资源进行定时备份，则可在发生意外的情况下，进行恢复（默认情况下，备份保留 30 天）。

    ```bash
    # 每日1点进行备份
    velero create schedule <SCHEDULE NAME> --schedule="0 1 * * *"
    # 每日1点进行备份，备份保留48小时
    velero create schedule <SCHEDULE NAME> --schedule="0 1 * * *" --ttl 48h
    # 每6小时进行一次备份
    velero create schedule <SCHEDULE NAME> --schedule="@every 6h"
    # 每日对 web namespace 进行一次备份
    velero create schedule <SCHEDULE NAME> --schedule="@every 24h" --include-namespaces web
    ```

    定时备份的名称为：`<SCHEDULE NAME>-<TIMESTAMP>`，恢复命令为：`velero restore create --from-backup <SCHEDULE NAME>-<TIMESTAMP>`。

- 备份删除

    直接执行命令进行删除
    
    ```bash
    velero delete backups <BACKUP_NAME>
    ```
    
- 备份资源查看

    备份查看

    ```bash
    velero backup get
    ```

    查看定时备份

    ```bash
    velero schedule get
    ```

    查看可恢复备份

    ```bash
    velero restore get
    ```

- 备份排除项目

    可为资源添加指定标签，添加标签的资源在备份的时候被排除。

    ```bash
    # 添加标签
    kubectl label -n <ITEM_NAMESPACE> <RESOURCE>/<NAME> velero.io/exclude-from-backup=true
    # 为 default namespace 添加标签
    kubectl label -n default namespace/default velero.io/exclude-from-backup=true
    ```

### 问题汇总

#### 时区问题

进行定时备份时，发现备份使用的事 UTC 时间，并不是本地时间，经过排查后发现是 `velero` 镜像的时区问题，在调整后就会正常定时备份了，这里我重新调整了时区，直接调整镜像就好，修改 `install/01-velero.yaml` 文件，将镜像替换为 `registry-vpc.cn-shanghai.aliyuncs.com/keking/velero:latest`。

```yaml
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: velero
  namespace: velero
spec:
  replicas: 1
  selector:
    matchLabels:
      deploy: velero
  template:
    metadata:
      annotations:
        prometheus.io/path: /metrics
        prometheus.io/port: "8085"
        prometheus.io/scrape: "true"
      labels:
        component: velero
        deploy: velero
    spec:
      serviceAccountName: velero
      containers:
      - name: velero
        # sync from gcr.io/heptio-images/velero:latest
        image: registry-vpc.cn-shanghai.aliyuncs.com/keking/velero:latest   # 修复时区后的镜像
        imagePullPolicy: IfNotPresent
        command:
          - /velero
        args:
          - server
          - --default-volume-snapshot-locations=alibabacloud:default
        env:
          - name: VELERO_SCRATCH_DIR
            value: /scratch
          - name: ALIBABA_CLOUD_CREDENTIALS_FILE
            value: /credentials/cloud
        volumeMounts:
          - mountPath: /plugins
            name: plugins
          - mountPath: /scratch
            name: scratch
          - mountPath: /credentials
            name: cloud-credentials
      initContainers:
      - image: registry.cn-hangzhou.aliyuncs.com/acs/velero-plugin-alibabacloud:v1.2
        imagePullPolicy: IfNotPresent
        name: velero-plugin-alibabacloud
        volumeMounts:
        - mountPath: /target
          name: plugins
      volumes:
        - emptyDir: {}
          name: plugins
        - emptyDir: {}
          name: scratch
        - name: cloud-credentials
          secret:
            secretName: cloud-credentials

```

#### 版本问题

截止发稿时，Velero 已经发布了 v1.2.0 版本，目前 ACK 的 Velero 的插件还未升级。

## 结语

近日正好有 k8s 集群服务迁移服务的需求，使用 Velero 完成了服务的迁移，同时也每日进行集群资源备份，其能力可以满足容器服务的灾备和迁移场景，实测可用，现已运行在所有的 k8s 集群。
