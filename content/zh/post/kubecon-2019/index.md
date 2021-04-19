---
title: "Kubecon 2019 见闻：云原生未来可期"
date: 2019-07-02T10:18:18+08:00
draft: false
type: blog
banner: "http://tva2.sinaimg.cn/large/ad5fbf65gy1g4k6mh797pj21900u07it.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "2019年6月24-26日，KubeCon + CloudNativeCon + Open Source Summit大会在上海世博中心举行。本次大会规模空前，预计有超过40个国家，3500多名云原生、开源领域的开发者参加，门票更是早早售罄。作为一名云原生应用的使用者与开发者，我也报名参与了这次大会。"
tags: ["云原生"]
categories: ["程序员趣闻"]
keywords: ["kubecon"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3iy1axydj21900u07it.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
## 前言

2019年6月24-26日，KubeCon + CloudNativeCon + Open Source Summit大会在上海世博中心举行。本次大会规模空前，预计有超过40个国家，3500多名云原生、开源领域的开发者参加，门票更是早早售罄。作为一名云原生应用的使用者与开发者，我也报名参与了这次大会。

6月的上海已经入梅，潮湿的空气对于已经在上海生活好多年的我还是会感受到不适，但是这些也无法阻碍 KubeCon 带给我的兴奋与激动，况且这次是在家门口举行，3站地铁就能到达了世博中心。

参与本次大会不仅仅是因为可以接触到最新的 Kubernetes & Cloud Native 实践，更是因为可以与很多神交已久的朋友会面，同时也可以与很多业界大牛面对面的交流，获取宝贵的经验。

![48124131751_3fc63103d5_o](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4lbp87794j22bc1jknpe.jpg)

## 国内云原生开源力量强劲

会议第一天，先后参加了蚂蚁金服组织的 SOFAStack Workshop 和由 CNCF, VMware, 阿里云和 PingCAP 联合主办的中国原创CNCF项目社区沙龙，同时在午休时候短暂的旁听了华为主办的 Apache ServiceComb Meetup 。

作为 [ServiceMesher](http://servicemesher.com) 社区的一员，与会议的组织者在社区中都很熟悉了，虽然没有注册 SOFAStack Workshop 会议，但也很自然的混进去了。作为金融级分布式框架 SOFAStack ，是用于快速构建金融级分布式架构的一套中间件，也是建立于蚂蚁金服海量金融场景锤炼出来的最佳实践。而此次的 Workshop 更是经过了精心的准备，准备了充足的材料，并在蚂蚁同学的帮助了使用 SOFAStack 实践了 __使用 SOFAStack 快速构建微服务__ 、 __SOFABoot 动态模块实践__ 、 __使用 Seata 保障支付一致性__ 、 __基于 Serverless 轻松构建云上应用__ 和 __使用 CloudMesh 轻松实践 Service Mesh__ 五个demo。在 Service Mesh 方面，蚂蚁的同学异常的活跃，是 Istio 中文文档翻译和 [ServiceMesher](http://servicemesher.com) 社区的主要组织者。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4lcu8mpmsj21480tokjl.jpg)
<center>《未来架构-从服务化到云原生》作者敖小剑老师现场签名</center>

午饭后短暂旁听了华为主办的 Apache ServiceComb Meetup ，虽然时间短暂，只有不到1个小时，但是对于国内运作开源项目，尤其是 Apache 项目有个更深的理解，同时让我联想到了之前在 [《从开源小白到 Apache Member ，阿里工程师的成长笔记》](https://developer.aliyun.com/article/704943) 阿里巴巴技术专家望陶成为的文章，中国的开源软件和开发者在开源领域起到越来越重要的作用。

![IMG_2470](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4lcxc767gj23402c07wi.jpg)

下午参加了由 CNCF, VMware, 阿里云和 PingCAP 联合主办的中国原创CNCF项目社区沙龙，聆听了 [Harbor](https://github.com/goharbor/harbor) 、 [Dragonfly](https://github.com/dragonflyoss/Dragonfly) 、 [TiKV](https://github.com/tikv/tikv) 三个中国原创 CNCF 项目的分享，同时 李响、Dan Kohn 等大佬也在会上发言表达了对国内 CNCF 开源项目的肯定及期待。而在会上也结识了阿里云容器镜像服务的开发小哥，作为阿里云的资深用户与开发小哥进行了交流，了解一些容器镜像服务方面的新功能，同时也反应了在使用中遇到的问题，总的来说收获颇丰。

![IMG_2472](https://ws3.sinaimg.cn/large/ad5fbf65gy1g4lfret268j23402c0kjm.jpg)

## 现场惊现 Linux 及 Git 创始人 Linus Torvalds

大会第二天的 Keynote 一直是 Linux 基金会宣传其理念，愿景以及赞助商进行市场宣传的重要活动。而当天最让人激动的就是 Linus 在 Keynote 后的一次嘉宾谈话，毫不夸张的说，我的工作就是 Linus 给的。而为了看到活的 Linus ，很多人一大早就在 red hall 的门前排起了长队，由于没有看好座位的分布，我只是找到了一个比较偏的位置，但是还是可以看的比较清楚。 Linus 本人还是十分幽默的，在谈话中提到了 Linux 5.1-rc6 的 release 计划，同时还询问现成有多少人是从事内核开发的，不过现场举手的同学并不多。

Keynote 上还提到了中国在整个云原生运动中的巨大贡献，中国的 K8s contributors 已经在全球所有贡献者中排名第二，超过 10% 的 CNCF 会员来自中国，26%的 Kubernetes 的认证供应商来自中国，同时也公布了蚂蚁金服作为黄金会员加入 CNCF。

![48125038821_66fcf00e96_o](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4lghq7jo9j20xc0m87o8.jpg)

平均每小时一个的分组会议，我的日程安排的满满的，但就这样还是出现了由于到晚了无法进入分组会议室的情况，注意这里不是因为到晚了不让进，而是进去都没有站的地方！可见人气之旺！让我有了像是上学时候穿梭在教学楼，赶人气高的选修课的错觉。

而在赞助商展示区也有不少的收获，与rancher、kong、jenkins等开源软件的开发者进行了交流，同时也获得了不少小礼品。最大的收获就是在阿里云展台与张磊大神的合影。

![IMG_2536](https://tva2.sinaimg.cn/large/ad5fbf65gy1g4lh0fv4x5j23402c0qv7.jpg)
<center>与张磊大神的合影</center>

## ServiceMesher 社区的壮大

平时都是在网上与社区的朋友们进行交流，讨论技术，交流经验。而 KubeCon 就变成了一场网友面基大会，见到了很多有过交流和帮助过我的朋友，包括 Jimmy 、 小剑 、秀龙老哥...等等，同时也认识了不少新朋友。与去年11月的 KubeCon 相比，社区的朋友越来越多，在短短半年内 Service Mesh 相关书籍已经出版了4本，而作者都是社区成员，可见社区的活跃。

![](http://tva2.sinaimg.cn/large/ad5fbf65gy1g4k6mh797pj21900u07it.jpg)

## 写在最后

正如 KubeCon 第三天 Keynote 上，阿里云智能容器平台负责人丁宇的话：__Kubernetes 正当时，云原生未来可期__ 。在 KubeCon 上看到云原生及开源软件的发展速度迅猛，各大厂商也都在最近几年组建了自己的开源团队，在使用开源软件获取便利的同时也在回馈社区，这也是让竞争对手共同为一款开源软件进行贡献的原因。相信随着开源运动在国内的深入，将会出现越来越多中国原创的开源项目，也会有更多的开发者加入到开源项目中，在贡献的同时提升自己的技术水平。
