---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "AWS CDK | IaC 何必只用 Yaml"
subtitle: ""
summary: "使用 AWS CDK 破解 IaC 的 Yaml 困局。"
authors: ["guoxudong"]
tags: ["AWS", "AWS CDK", "IaC", "基础设施即代码"]
categories: ["IaC"]
date: 2021-01-21T10:43:11+08:00
lastmod: 2021-01-21T10:43:11+08:00
draft: false
type: blog
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65gy1gmvgjyxvb6j20rs0ijgn0.jpg"
---
## 前言

近年来基础设施即代码（IaC）的方式被越来越多的开发者和管理者所采用，各大公有云都提供了使用 IaC 管理自己云资源的方式，如 AWS 的 CloudFormation、阿里云的 ROS 等，而第三方的 Terraform 也有各大公有云的 Provider。越来越多像我一样的云资源运维和管理者开始采用 IaC 的方式对云资源进行创建、运维和管理。 

## IaC 管理之惑

但在实际使用中，IaC 其实并没有看上去的那么美丽。

### Imperative IaC vs. Declarative IaC

Imperative 和 Declarative 也就是**命令式**和**声明式**的 IaC，他们的不同点在于命令式的 IaC 是由代码编写者来确定如何达到自己想要目的的，如：我需要一个创建 VPC，就需要编写代码或命令来完成这个创建 VPC 的动作，直接操作公有云的 OpenAPI 和 CLI 工具就是这种方式；而声明式的 IaC 则是由代码编写者定义了系统期望的状态，并不需要关心云平台如何去实现我的这个要求，这其中就以 kubernetes 的 Yaml 配置为代表。两相对比声明式的 IaC 显然更容易上手。

### 这样就够了吗？

虽然声明式的 IaC 看上去更简单且高效，但事实并非如此。和我一样主要工作是管理和运维 kubernetes 集群的同学，常常自称为 YAML 工程师，原因就是我们日常工作需要管理和维护数量庞大的 YAML 文件，小到一个微服务，大到一整套云环境，大部分情况都是采用 YAML 或 JSON 格式的配置文件进行管理，我们手中的 YAML 越来越多，而 YAML 文件的可读性并没有那么友好。**“YAML 地狱”** 这个词形容的就非常贴切。

### 如何破解 YAML 地狱？

其实这个问题早就引起了开发者的广泛讨论，为了解决这个问题很多项目都做出了尝试，如 Helm 这样采用 template 的方式，或 kustomize 这样采用 overlay 的方式对 YAML 进行抽象和简化。 

目前比较受欢迎的还有一种方式，就是采用常规编程语言通过代码来生成声明式的配置，然后再基于声明式的配置进行部署，这样既不会重复造轮子，同时常规编程语言的可读性、代码量以及编写的难易程度都比直接编写 Yaml 文件要简单的多。比如我之前介绍过的 `Grabana` 就是采用这种模式，使用 Golang 来生成 Grafana Dashboard 配置并部署，详见：[《Grabana：使用 Golang 或 Yaml 生成 Grafana Dashboard》](./grabana-create-grafana-dashboard)。 

这种方式融合了 Imperative 和 Declarative 的优点是一个非常不错的选择。

## AWS CDK

AWS Cloud Development Kit（AWS CDK） 是 AWS 开发的一种开源软件开发框架，可以使用 Python 或 Typescript 之类的编程语言，利用函数快速构建代码框架，快速的定义云资源，并且还提供了一系列默认选项，使得代码量进一步降低。

### 支持的语言

AWS CDK 目前支持的语言有：
- Typescript
- JavaScript
- Python
- Java
- C#

AWS CDK 还提供了十分完善的脚手架工具，以 Python 为例，只需新建目录，并在目录中执行如下命令，即可拉起一套的 CDK Python 代码：

```bash
cdk init app --language python
```

之后只需在 `app/app_stack.py` 中编写相应代码即可，非常方便。

### 原理

AWS CDK 将 Imperative 和 Declarative 进行了结合，通过编程语言生成 CloudFormation 的 template，之后再由 CloudFormation 生成对应的 Stack，最终在 AWS 上完成云资源的创建和变更。

这种方法完美的绕过了 CloudFormation 配置本身的复杂性和较差的可读性，用户可以选择一个自己熟悉的编程语言，以代码的形式来对基础资源进行编排，同时还有很多默认选项，为不想关心太多细节的开发者提供了便利。

比如只使用这样一行代码，就能创建一个全新的 VPC：

```python
class CdkPythonStack(core.Stack):
    def __init__(self, scope: core.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        ...
        vpc = ec2.Vpc(self,
                      'eks-vpc',
                      cidr='10.3.0.0/16',
                      max_azs=3,
                      nat_gateways=1
                      )
        ...
```

### CLI Command

AWS CDK 还提供了一些命令来帮助开发者完成代码构建、资源检查和部署等功能。

#### init

`cdk init` 帮助开发者生成一个指定语言的 cdk 项目，目前支持 5 种语言。

#### diff

`cdk diff` 是最常用的一个命令了，会帮助用户检查当前 Stack 和 云上资源的不同，并作出标记：

```bash
$ cdk diff
Stack HelloCdkStack
IAM Statement Changes
┌───┬────────────────────────┬────────┬──────────────┬───────────┬───────────┐
│   │ Resource               │ Effect │ Action       │ Principal │ Condition │
├───┼────────────────────────┼────────┼──────────────┼───────────┼───────────┤
│ + │ ${MyFirstBucket.Arn}/* │ Allow  │ s3:GetObject │ *         │           │
└───┴────────────────────────┴────────┴──────────────┴───────────┴───────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[+] AWS::S3::BucketPolicy MyFirstBucket/Policy MyFirstBucketPolicy3243DEFD
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 ├─ [~] DeletionPolicy
 │   ├─ [-] Retain
 │   └─ [+] Delete
 └─ [~] UpdateReplacePolicy
     ├─ [-] Retain
     └─ [+] Delete
```

#### ls

`cdk ls` 可以列出当前 app 所有的 stack。

#### synth

前面说到了 CDK 会生成 CloudFormation template， `cdk synth` 就是会生成这样一个 template 方便用户检查。

```bash
$ cdk synth
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.XX.X,@aws-cdk/aws-events=1.XX.X,@aws-cdk/aws-iam=1.XX.X,@aws-cdk/aws-kms=1.XX.X,@aws-cdk/aws-s3=1.XX.X,@aws-cdk/cdk-assets-schema=1.XX.X,@aws-cdk/cloud-assembly-schema=1.XX.X,@aws-cdk/core=1.XX.X,@aws-cdk/cx-api=1.XX.X,@aws-cdk/region-info=1.XX.X,jsii-runtime=node.js/vXX.XX.X
```

#### deploy

在检查无误后，就可以进行部署了，使用 `cdk deploy` 命令，就会开始部署 CloudFormation，可以看到实时进度，如果遇到问题，也会进行回滚。

```bash
$ cdk deploy
HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...
 1/1 | 8:39:43 AM | UPDATE_COMPLETE      | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketB8884501)
 1/1 | 8:39:44 AM | UPDATE_COMPLETE_CLEA | AWS::CloudFormation::Stack | HelloCdkStack
 2/1 | 8:39:45 AM | UPDATE_COMPLETE      | AWS::CloudFormation::Stack | HelloCdkStack

 ✅  HelloCdkStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT:stack/HelloCdkStack/ID
```

#### destroy

在体验完后，可以使用 `cdk destroy` 对 CloudFormation 以及 CloudFormation 创建的资源进行清理和回收。

## 结语

如果你是 AWS 用户，推荐可以尝试使用 AWS CDK，无论是使用体验还是开发速度都十分突出，只需不到 100 行的代码，就可以生成 上千行 CloudFormation 配置，随着基础设施越来越复杂，使用 AWS CDK 获得的红利也会越多。后续我也会出一篇使用 AWS CDK Python 从 0 开始创建 EKS 集群的文章，感兴趣的同学可以关注一下。

如果你不是 AWS 用户，但是也想采用这种方式创建和维护你的基础资源，也可以关注一下 [pulumi](https://github.com/pulumi/pulumi) 项目，这是一个开源项目，其支持包括 AWS、Azure、Google Cloud 和阿里云。后续我同样会出一篇相关内容的文章，敬请期待。