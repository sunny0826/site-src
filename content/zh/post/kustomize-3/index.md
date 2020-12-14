---
title: "使用 Kustomize 帮你管理 kubernetes 应用（三）：将 Kustomize 应用于 CI/CD"
date: 2019-05-06T16:46:28+08:00
draft: false
type: blog
banner: "https://wx4.sinaimg.cn/large/ad5fbf65gy1g2smcy97ooj21qi15o7bb.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本篇为系列文章第三篇，使用 jenkins 发布一个简单的使用 flask 写的 web 项目，来演示在 CI/CD 流程中 Kustomize 的简单使用。"
tags: ["kubernetes", "kustomize", "工具"]
categories: ["kustomize"]
keywords: ["kubernetes", "kustomize", "工具"]
image:
  url: "https://tva1.sinaimg.cn/large/ad5fbf65ly1ge3j5cgy7cj21qi15o7bb.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

首先明确软件版本，我这里使用的是 ```Jenkins ver. 2.121.3``` ，这个版本比较老，其上安装 Kubernetes 插件所使用 ```kubectl``` 版本也比较老，**无法使用** Kustomize 的 yaml 文件需要的 ```apiVersion: apps/v1``` ，直接使用生成 ``deploy.yaml`` 文件会报错，所以这里选择了自己构建一个包含 ```kubectl``` 和 ```kustomize``` 的镜像，在镜像中使用 Kustomize 生成所需 yaml 文件并在 Kubernetes 上部署。

## 软件版本
| 软件 | 版本 |
| --- | --- |
| Jenkins | [2.121.3](https://jenkins.io/) |
| kubectl | [v1.14.1](https://kubernetes.io/docs/tasks/tools/install-kubectl/) |
| kustomize | [v2.0.3](https://github.com/kubernetes-sigs/kustomize/releases) |

## 前期准备

- Jenkins ：本篇使用 Jenkins 演示 CI/CD ，安装 Jenkins 就不在赘述，可以使用多种方法安装 Jenkins ，详细方法见[官网](https://jenkins.io)。同时。 CI/CD 的工具有很多，这里为了省事使用笔者现有的 Jenkins 进行演示，**不推荐**使用同笔者一样的版本，请使用较新的版本；同时也可以使用其他 CI/CD 工具，这里推荐使用 [drone](https://drone.io/)。如果有更好的方案，欢迎交流，可以在[关于](https://blog.maoxianplay.com/contact/)中找到我的联系方式。
- ```kubectl``` & ```kustomize``` ：上文中提到了由于 Jenkins 版本比较老，所以这里笔者自己制作了包含二者的 docker 镜像，已上传 dockerhub ，需要自取： [```guoxudongdocker/kubectl```](https://hub.docker.com/r/guoxudongdocker/kubectl)
- Web 应用：这里使用 flask 写了一个简单的 web 应用，用于演示，同样以上传 dockerhub [```guoxudongdocker/flask-python```](https://hub.docker.com/r/guoxudongdocker/flask-python)

## 目录结构

首先看一下目录结构，目录中包括 ```Dockerfile``` 、 ```Jenkinsfile``` 、 Kustomize 要使用的 ```deploy``` 目录以及 web 应用目录。

```bush
.
├── Dockerfile
├── Jenkinsfile
├── app
│   ├── main.py
│   └── uwsgi.ini
└── deploy
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── dev
        │   ├── healthcheck_patch.yaml
        │   ├── kustomization.yaml
        │   └── memorylimit_patch.yaml
        └── prod
            ├── healthcheck_patch.yaml
            ├── kustomization.yaml
            └── memorylimit_patch.yaml
```

这里可以看到 overlays 总共有两个子目录 `dev` 和 `prod` ，分别代表不同环境，在不同的环境中，应用不同的配置。

## Jenkins 配置

Jenkins 的配置相对简单，只需要新建一个 pipeline 类型的 job

![WX20190506-180159](https://wx4.sinaimg.cn/large/ad5fbf65gy1g2rr57oixbj20tn0ogq6v.jpg)

增加参数化构建，**注**：参数化构建需要安装 Jenkins 插件

![WX20190506-180918](https://ws4.sinaimg.cn/large/ad5fbf65gy1g2rrcb5ic9j21470q7mz8.jpg)

然后配置代码仓库即可

![WX20190507-094958](https://ws3.sinaimg.cn/large/ad5fbf65gy1g2sij1xlb2j214w0nw0uw.jpg)

## Pipeline 

```groovy
podTemplate(label: 'jnlp-slave', cloud: 'kubernetes',
  containers: [
    containerTemplate(
        name: 'jnlp',
        image: 'guoxudongdocker/jenkins-slave',
        alwaysPullImage: true
    ),
    containerTemplate(name: 'kubectl', image: 'guoxudongdocker/kubectl:v1.14.1', command: 'cat', ttyEnabled: true),
  ],
  nodeSelector:'ci=jenkins',
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    hostPathVolume(mountPath: '/usr/bin/docker', hostPath: '/usr/bin/docker'),
    hostPathVolume(mountPath: '/usr/local/jdk', hostPath: '/usr/local/jdk'),
    hostPathVolume(mountPath: '/usr/local/maven', hostPath: '/usr/local/maven'),
    secretVolume(mountPath: '/home/jenkins/.kube', secretName: 'devops-ctl'),
  ],
)
{
    node("jnlp-slave"){
        stage('Git Checkout'){
            git branch: '${branch}', url: 'https://github.com/sunny0826/flask-python.git'
        }
        stage('Build and Push Image'){
            withCredentials([usernamePassword(credentialsId: 'docker-register', passwordVariable: 'dockerPassword', usernameVariable: 'dockerUser')]) {
                sh '''
                docker login -u ${dockerUser} -p ${dockerPassword}
                docker build -t guoxudongdocker/flask-python:${Tag} .
                docker push guoxudongdocker/flask-python:${Tag}
                '''
            }
        }
        stage('Deploy to K8s'){
            if ('true' == "${deploy}") {
                container('kubectl') {
                    sh '''
                    cd deploy/base
                    kustomize edit set image guoxudongdocker/flask-python:${Tag}
                    '''
                    echo "部署到 Kubernetes"
                    if ('prod' == "${ENV}") {
                        sh '''
                        # kustomize build deploy/overlays/prod | kubectl apply -f -
                        kubectl applt -k deploy/overlays/prod
                        '''
                    }else {
                        sh '''
                        # kustomize build deploy/overlays/dev | kubectl apply -f -
                        kubectl applt -k deploy/overlays/dev
                        '''
                    }	
                }
            }else{
                echo "跳过Deploy to K8s"
            }

        }
    }
}


```

这里要注意几点：

- 拉取 git 中的代码需要在 jenkins 中配置凭据。
- 笔者的 jenkins 部署在 Kubernetes 上，要操作集群的话，需要将 kubeconfig 以 Secret 的形式挂载到 jenkins 所在 namespace。
- `jenkins-slave` 需要 Java 环境运行，所以要将宿主机的 `jdk` 挂载到 `jenkins-slave` 中。
- 同样的，宿主机中需要事先安装 `docker`。
- `docker-register` 为 dockerhub 的登录凭证，需要在 jenkins 中添加相应的凭证。

## 演示

image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

### 开始构建

这里选择环境、分支，填入版本即可开始构建，**注意：**这里的版本将已 tag 的形式标记 docker 镜像。

![WX20190507-095142](https://ws2.sinaimg.cn/large/ad5fbf65gy1g2sikst7tuj20ob0evabw.jpg)

这里就可以看到构建成功了

![WX20190507-103721](https://ws2.sinaimg.cn/large/ad5fbf65ly1g2sjw9w22ej20v80km0w3.jpg)

### 查看结果

这里为了方便（其实就是懒），我就不给这个服务添加 ingress 来从外部访问了，这里使用 [KT](https://yq.aliyun.com/articles/690519) 打通本地和 k8s 集群网络来进行调试。

>为了简化在Kubernetes下进行联调测试的复杂度，云效在SSH隧道网络的基础上并结合Kubernetes特性构建了一款面向开发者的辅助工具kt

这里看到这个服务正常启动了

![WX20190507-104154](https://ws2.sinaimg.cn/large/ad5fbf65ly1g2sk11dnzxj20av027jrn.jpg)

### 发布新版本

更新 web 服务并提交

![WX20190507-104936](https://ws4.sinaimg.cn/large/ad5fbf65gy1g2sk94v1c5j209702vwej.jpg)


按照上面步骤在 jenkins 中重新构建，当然也可以配置钩子，每次代码提交后自动构建

### 查看查看新版本

同上面一样，在构建成功后查看服务是否更新

![WX20190507-105539](https://wx4.sinaimg.cn/large/ad5fbf65gy1g2skfczaz4j20by01smx7.jpg)

可以看到，版本已经更新了

### 发布生产环境

这里模拟一下发布生产环境，假设生产环境是在 `devops-prod` 的 namespace 中，这里只做演示之用，真正的生产环境中，可能存在不止一个 k8s 集群，这时需要修改 Jenkinsfile 中的 `secretVolume` 来挂载不同 k8s 的 kubeconfig 来达到发布到不同集群的目的。当然，一般发布生产环境只需选择测试通过的镜像来发布即可，不需要在进行构建打包。

![WX20190507-110730](https://ws3.sinaimg.cn/large/ad5fbf65gy1g2skrnbjyuj20fc0bjmxp.jpg)

### 查看生产版本

![WX20190507-110850](https://tva2.sinaimg.cn/large/ad5fbf65ly1g2skt3rp4yj20aq010glj.jpg)

### 总结

上面的这些步骤简单的演示了使用 jenkins 进行 CI/CD 的流程，流程十分简单，这里仅供参考

## Kustomize 的作用

那么， Kustomize 在整个流程中又扮演了一个什么角色呢？

### 更新镜像

在 `jenkinsfile` 中可以看到， kustomize 更新了基础配置的镜像版本，这里我们之前一直是使用 `sed -i "s/#Tag/${Tag}/g" deploy.yaml` 来进行替换了，但是不同环境存在比较多的差异，需要替换的越来越多，导致 Jekninsfile 也越来越臃肿和难以维护。 kustomize 解决了这个问题。

```bash
kustomize edit set image guoxudongdocker/flask-python:${Tag}
```

### 环境区分

上面也提到了，不同的环境我们存在这许多差异，虽然看上去大致类似，但是很多细节都需要修改。这时 kustomize 就起到了很大的作用，不同环境相同的配置都放在 `base` 中，而差异就可以在 `overlays` 中实现。

```bash
.
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── dev
    │   ├── healthcheck_patch.yaml
    │   ├── kustomization.yaml
    │   └── memorylimit_patch.yaml
    └── prod
        ├── healthcheck_patch.yaml
        ├── kustomization.yaml
        └── memorylimit_patch.yaml
```
可以看到， `base` 中维护了项目共同的基础配置，如果有镜像版本等基础配置需要修改，可以使用 `kustomize edit set image ...` 来直接修改基础配置，而真正不同环境，或者不同使用情况的配置则在 `overlays` 中 以 patch 的形式添加配置。这里我的配置是 prod 环境部署的副本为2，同时给到的资源也更多，详情可以在 [Github](https://github.com/sunny0826/flask-python) 上查看。

### 与 kubectl 的集成

在 jenkinsfile 中可以看到

```bash
# kustomize build deploy/overlays/dev | kubectl apply -f -
kubectl apply -k deploy/overlays/dev
```

这两条命令的执行效果是一样的，在 `kubectl v1.14.0` 以上的版本中，已经集成了 kustomize ，可以直接使用 `kubectl` 进行部署。

## 结语

这里只是对 kustomize 在 CI/CD 中简单应用的展示，只是一种比较简单和基础的使用，真正的 CI 流程要比这个复杂的多，这里只是为了演示 kustomize 的使用而临时搭建的。而 kustomize 还有很多黑科技的用法，将会在后续的文章中介绍。