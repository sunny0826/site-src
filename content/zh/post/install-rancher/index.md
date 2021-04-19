---
title: "阿里云部署rancher2.1采坑记"
date: 2018-11-29T18:28:13+08:00
draft: false
type: blog
tags: ["容器","阿里云","rancher"]
categories: ["问题解决"]
banner: "http://tva2.sinaimg.cn/large/ad5fbf65ly1g0t1au03u7j21qf15ogr2.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "近期由于公司需要将部署在ucloud上的rancher迁移到阿里云上，所以将部署到阿里云的图中遇到的问题和踩到的坑在这里进行记录。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3ilt28ofj21qf15ogr2.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
>近期由于公司需要将部署在ucloud上的rancher迁移到阿里云上，所以将部署到阿里云的图中遇到的问题和踩到的坑在这里进行记录。

# 无法删除namespace

在安装新环境的rancher之前，需要将kubernetes集群中cattle-system ns下面的cluster-agent和node-agent干掉，这里我选择直接删除cattle-system这个命名空间

```bash
kubectl delete ns cattle-system
```

然而问题来了，在删除命名空间之后，这个命名空间并没有立刻被删除，而是一直处于Terminating状态，这里我专门写了一篇文章解决这个问题，这里就不再赘述

# 阿里云证书配置

由于之前使用的ucloud的机器进行测试，使用默认自签名证书并没有使用SSL证书，所以在配置证书这里遇到的许多的问题

首先根据官方文档使用权威CA机构颁发的证书，这里使用的是本公司自己的证书

获取证书方法：
![image](/images/source/jinrussl.png)

点击下载证书，选择nginx证书下载
![image](/images/source/zhengshu.png)

之后将下载的证书上传到rancher所在服务器，并配置好数据卷挂载

将下面代码的挂载地址指向证书文件，运行代码

```bash
$ docker run -d --restart=unless-stopped \
-p 80:80 -p 443:443 \
-v /root/var/log/auditlog:/var/log/auditlog \
-e AUDIT_LEVEL=3 \
-v /etc/your_certificate_directory/fullchain.pem:/etc/rancher/ssl/cert.pem \
-v /etc/your_certificate_directory/privkey.pem:/etc/rancher/ssl/key.pem \
rancher/rancher:latest --no-cacerts
```

之后会自动冲dockerhub上拉取最新的rancher进行进行安装，之后使用命令

```bash
docker ps
```

查看容器是否在运行，如果运行正常，则后端的配置就完成了

划重点：这是是在后端配置了证书，所以在阿里云的配置上要使用四层TCP监听

这个地方可是坑了我许久，我一直在前端配置https七层监听，导致一直无法正常访问，一度已经到了怀疑人生的地步=。=

之后就是简单的阿里云SLB配置四层TCP监听，这里也就不再赘述了

# k8s集群导入rancher

前后端都准备就绪，现在就可以访问rancher了，访问rancher根据页面提示进行基本配置，登录后选择添加集群

选择导入现有集群
![image](/images/source/add.png)

为集群创建一个rancher中的名称，然后根据提示将命令拷贝到k8s集群所在宿主机执行即可，注意：这里由于配置了证书，所以选择有证书，不绕过证书的那个命令执行，之后就可看到集群数据导入中
![image](/images/source/wating.png)

等待几秒即可开心的使用rancher了！

# 关于rancher部署后访问集群api超时问题

经过排查，原因是阿里云在容器服务对外连接处设置了TLS双向认证，导致rancher的外网ip经常性的被拦截，导致超时

解决办法：

对k8s集群中rancher的cattle-cluster-agent传递内网参数，将其配置为内网连接，就可以正常访问了

```bash
kubectl -n cattle-system patch deployments cattle-cluster-agent --patch '{
    "spec": {
        "template": {
                "spec": {
                    "hostAliases": [{
                                    "hostnames":["rancher.keking.cn"],  #rancher的域名
                                    "ip": "10.0.0.219"  #rancher部署地址
                                    }]
                        }
                    }
            }
}'
```