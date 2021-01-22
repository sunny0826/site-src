---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 AWS CDK Python 从零开始构建 EKS 集群"
subtitle: ""
summary: "使用 AWS CDK Python 版本从零开始构建一个 EKS 集群。"
authors: ["guoxudong"]
tags: ["AWS", "AWS CDK", "IaC", "基础设施即代码"]
categories: ["IaC"]
date: 2021-01-22T10:07:01+08:00
lastmod: 2021-01-22T10:07:01+08:00
draft: false
type: blog
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65gy1gmwl7ox518j20rs0kut9l.jpg"
---
## 前言

上篇文章[《AWS CDK | IaC 何必只用 Yaml》](../aws-cdk-introduction) 笔者介绍了 AWS CDK 的概念和基本使用方法，本篇文章就来使用 CDK 在 AWS 从零开始构建一个全新的 KES 集群，实际感受一下使用 AWS CDK 创建和管理云资源的简单和便捷。

## 资源清单

本文中，笔者会创建以下资源：

- 创建一个 EKS 集群
- 为 EKS master 配置一个 IAM Role 
- 创建一个 VPC（包含子网和 NAT）
- 为 EKS 创建一个 Node Group 用来管理一组 Worker 节点
- 为 EKS 创建一个 Auto Scaling Group 用来管理弹性伸缩节点

## Show me the code

本文代码已全部上传 GitHub，配合代码阅读本文体验更佳。

{{% alert title="Github 地址" color="primary" %}}
https://github.com/sunny0826/aws-cdk-eks
{{% /alert %}}

### 安装 AWS CDK

首先其实需要有 `AWS CLI` 命令行工具，并配置了 `AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY` 和 `AWS_DEFAULT_REGION`，这里就不做详细介绍了，同时还要安装 `Node.js` 10.3.0 以上的版本。

使用 `npm` 安装：

```bash
$ npm install -g aws-cdk
```

安装完成后，检查 `AWS CDK` 版本：

```bash
$ cdk --version
```

### 创建 APP

`AWS CDK` 安装完成后，就可以开始创建项目了。

新建一个目录：

```bash
$ mkdir aws-cdk-eks
$ cd aws-cdk-eks
```

初始化项目：

```bash
$ cdk init app --language python
$ source .venv/bin/activate
$ python -m pip install -r requirements.txt
```

这里就会生成一个 Python 项目，目录结构如下如下：

```bash
$ tree
.
├── README.md
├── app.py
├── cdk.json
├── cdk_python
│   ├── __init__.py
│   └── cdk_python_stack.py   # 主要文件
├── requirements.txt
├── setup.py
└── source.bat
```

之后的代码就是写在 `cdk_python_stack.py` 中。


### Codeing

接下来就是写代码时间了。

#### 创建 VPC

首先 EKS 需要一个 VPC，这里有三种方式：

- 使用 `default` VPC
- 指定一个已有 VPC
- 新建一个 VPC

直接使用 `default` VPC:

```python
vpc = ec2.Vpc.from_lookup(self, id='Vpc', is_default=True)
```

指定现有 VPC：

```python
vpc = ec2.Vpc.from_lookup(self, id='Vpc', vpc_id='vpc-0417e46d')
```

新建 VPC：

```python
vpc = ec2.Vpc(self,
    'eks-vpc', # VPC id
    cidr='10.3.0.0/16', # CIDR
    max_azs=3,  # 跨3个AZ
    nat_gateways=1 # 新建一个 NAT Gateway
    )
```

还有很多其他参数可以配置，这里用不到直接使用默认值。

#### 创建 IAM

这里需要给 k8s 的 master 创建一个  IAM Role，这样我们才能对 EKS 进行管理。

```python 
eks_master_role = iam.Role(self, 'EksMasterRole',
    role_name='EksAdminRole',
    assumed_by=iam.AccountRootPrincipal()
    )
```

#### 创建 EKS

VPC 和 IAM 都已经准备好了，现在可以创建 EKS 集群了。

```python
cluster = eks.Cluster(self, 
    'Cluster',  # 集群 id
    vpc=vpc,  # 指定 VPC
    version=eks.KubernetesVersion.V1_18,  # K8S 版本
    masters_role=eks_master_role, # naster 的 IAM Role
    default_capacity=0  # 这里不需要 worker 节点，后面采用 MNG 或 ASG 来管理
    )
```

可以看到先前定义的 `vpc` 和 `eks_master_role` 都作为参数被传给了 `cluster`，而 `default_capacity` 是定义默认 worker 节点的，下面我们会采用 MNG 和 ASG 来管理 worker 节点，所以这里设置为 0.

#### 为 EKS 添加 MNG

`cluster` 定义好后，相当于 K8S 的 master 节点已经配置完成，接下来就是 worker 节点的配置。EKS 可以使用 MNG 和 ASG 来管理 worker 节点。

```python
cluster.add_nodegroup_capacity(
    'MNG',  # MNG id
    capacity_type=eks.CapacityType.SPOT,  # 节点类型
    desired_size=2, # 节点数量
    instance_types=[  # 节点规格
        ec2.InstanceType('t3.large'),
        ec2.InstanceType('m5.large'),
        ec2.InstanceType('c5.large'),
    ]),
```

#### 为 EKS 添加 ASG

```python 
cluster.add_auto_scaling_group_capacity(
  'ASGNG', # ASG id
  instance_type=[  # 节点规格
        ec2.InstanceType('t3.large'),
        ec2.InstanceType('m5.large'),
        ec2.InstanceType('c5.large'),
  ],
  desired_capacity=2 # 节点数量
  )
```

当然 MNG 和 ASG 都可以设置 `max_size` 和 `min_size`，也就是可以实现节点级别的弹性伸缩，但是目前测试下来只有 ASG 可以将配置的资源 TAG 带入 EC2 的配置，而 MNG 需要通过定制 `launch_template_spec` 的方式才能实现。如果对这方面没有要求的话推荐使用 MNG。

到这里代码就写好了，只有几十行代码，下面我们就是检查和部署了。

### Bootstrap

如果是第一次使用 `AWS CDK` 需要先执行 `cdk bootstrap` 命令，这个命令会在 S3 创建一个名为 `cdktoolkit-XXX` 的 bucket 用来存放 CDK 配置。

### 检查

执行 `cdk diff` 命令，这时就会打印出一系列列表，告诉你会有哪些资源变化，大致内容如下图。

![cdk diff](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gmwkeqhlblj23qq1i27wh.jpg)

可以执行 `cdk synth` 命令用来查看生成的 AWS CloudFormation template，笔者统计了一下生成 AWS CloudFormation template 的行数，这几十行代码居然生成了 **1156** 行的 CloudFormation 配置！

### 部署

在检查无误后就可以开始部署了，执行命令 `cdk deploy` 并输入 `y` 确认，之后可以看到部署的进度条。如果部署中间出现错误， CDK 会自动进行回滚，之前创建和修改的资源都会被恢复原样，可以放心使用。

### 销毁

在完成测试后，执行命令 `cdk destroy` 对创建的资源进行释放。

## 结语

非常感谢来自 AWS 的 [@pahud](https://github.com/pahud) 同学的指导和帮助，总体来说 Python 版本的 CDK 使用起来比较方便，但文档和源码中的说明略有不足。`AWS CDK` 的核心引擎其实是使用 Typescript 编写，其他语言的版本都是采用 [JSii](https://github.com/aws/jsii) 通过 TypeScript 转化而来。如果要深入使用，这里还是推荐使用 Typescript 的版本（其实我已经换成 Typescript 来写了），难度不是很大，值得一试。