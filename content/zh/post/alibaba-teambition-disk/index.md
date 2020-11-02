---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "试用阿里网盘内测版-不限速、无广告、隐私安全我全都要"
subtitle: ""
summary: "阿里 Teambition 网盘体验实录"
authors: ["guoxudong"]
tags: ["阿里云","网盘"]
categories: ["阿里云"]
date: 2020-11-02T09:26:27+08:00
lastmod: 2020-11-02T09:26:27+08:00
draft: false
type: blog
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65gy1gkatai1vr8j20gq07wn0f.jpg"
---
## 前言

Teambition 网盘自开放内测申请以来已经过去了数月，上周五终于收到了心心念念内测码，于是立马上手体验。Teambition 网盘宣称是不限速、不打扰、够安全、易于协作的网盘，属于阿里巴巴工作学习套件 Teambition，提供能够满足日常需求的超大存储空间，有趣、好用的协作等功能。

从官网可以看出 Teambition 网盘准确的抓到了个人网盘用户的痛点：**上传下载限速**、**各种广告弹窗**、**隐私安全**以及**操作困难**，而作为工作学习套件的一部分还增加了协作功能，除了分享资源，还提供了各种协作功能，大大提升协作效率。

## 优势

目前 Teambition 网盘还处于 Beta 内测版，申请地址：[https://survey.aliyun.com/apps/zhiliao/jqBinngVQ](https://survey.aliyun.com/apps/zhiliao/jqBinngVQ)

### 速度

经实测 Teambition 网盘的上传下载速度都能达到本地带宽的最大值，上传下载速度都非常丝滑。

![上传速度](https://tvax1.sinaimg.cn/large/ad5fbf65ly1gkamt4yh6tj20b604p74d.jpg)

<br>

Teambition 网盘背靠阿里巴巴集团，由阿里云提供资源支持，所以 Teambition 可以以极低成本拿到海量的资源，这也就是他们敢于宣传资源上传下载不限速的原因。

上传下载速度应该是所有功能的核心，也是和友商比较最多的一点。作为一名云资源管理者和维护者，我深知存储和网络带宽消耗成本的巨大，企业为了保证各种图片、视频、文件资源的可用性，会花费不菲的成本来采购 OSS 或 S3 这样的对象存储，这些资源的上传下载自然不会限速，甚至还有配套的加速服务。

所以对于网盘的限速从情感上是可以接受的，因为这些资源成本巨大，但是如果限速到**几k、几十k/秒**，那这样的策略就不是基于成本的考虑，而是垄断后的特权了。

### 容量

内测版开放了 2T 的免费容量，这个大小基本满足了个人用户的使用需求，后续应该会开放购买更大的容量。

![网盘容量](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gkasv8u9fzj208n03xglh.jpg)

### 页面

网盘页面简洁明了，功能一目了然，可以识别众多文件格式。确实也和承诺中的一样，没有广告推送和弹窗，整体保持简洁清新的风格。

![网盘页面](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gkamyvsv50j21h60q7dj9.jpg)

### 协作

除了一般网盘的功能外，Teambition 网盘还突出了协作的功能。

#### 讨论功能

![讨论](https://tvax1.sinaimg.cn/large/ad5fbf65ly1gkanbqdngbj20dd09dq3w.jpg)

参与协作的用户可以使用不同的颜色，将想要讨论的对象**圈出来**，这样就可以在同一幅图片或者视频中进行多组讨论，不用添加多余的描述。这个功能在视频中效果会更加明显，可以逐帧**圈**出修改对象并发表意见（剪辑师、设计师狂怒）。

![圈出讨论](https://tva1.sinaimg.cn/large/ad5fbf65gy1gkandlm1cfj21780ppqil.jpg)

你觉得是狗可爱还是猫可爱呢？：）

#### 图像识别

文档中并没有提到这个功能，是我在测试时偶然发现的，Teambition 网盘会自动识别你上传图像的内容，并在详情中的 **“包含的事物”** 中展示，如下图：

![图像识别](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gkanoe8od2j214l0jo1gt.jpg)

#### 分享

目前分享功能是关闭的，原因是内测期间有用户利用 Teambition 网盘分享违规文件，所以暂时无法体验 Teambition 网盘的分享功能。不过从 **[「分享」功能升级说明](https://thoughts.teambition.com/sharespace/5f72e44becf9290016f85c8c/docs/5f99203decf9290016f85ce3)** 可以看到目前已经开启了安全升级，同时还增加了更多功能。

多种分享权限：

* 限时分享有效期增加更细致的时间颗粒度（7天、永久、自定义）
* 分享支持增加添加密码
* 批量分享支持画册、缩略图两种模式查看文件

粒度更细的分享管理：

* 支持查看分享文件的浏览量和被下载量
* 支持查看分享文件的有效期
* 分享中的文件支持随时取消

## 不足

Teambition 网盘的核心功能确实直击网盘使用的痛点，但还处于 Beta 内测版，有很多不足。

* 目前只有 web 端和手机端可以使用，微信小程序、桌面端（Mac 和 Windows）、平板设备端还未开放
* 手机端目前还不能删除文件，删除文件要在 web 端操作，而且手机端网盘的加载速度也很慢，且多次出现加载不出来的情况
* 分享功能暂时不可用
* 暂时还没有与各种设备打通，还没有手机相册/内容备份等功能

比较遗憾的是，并没有找到更多关于隐私安全保障的介绍，相信后续会放出更多的相关内容。

## 结语

总的来说，Teambition 网盘抓住了网盘使用的痛点，并以此收获了众多的关注与好评。但要走的路还很长，在海量用户使用下网盘本身的可用性、对各种设备的支持、对违规内容的识别和管控都将是 Teambition 网盘面临的挑战。

往长远了看 Teambition 网盘的出现给了用户更多的选择，同时打破了目前网盘市场的局面，逼着“一家独大”的同行去改善自己的产品和服务，从用户的角度这都是好事情。