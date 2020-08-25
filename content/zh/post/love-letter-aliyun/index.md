---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "云中谁寄锦书来，免费生成一封七夕情书吧"
subtitle: ""
summary: "在七夕节制作一封云上情书吧"
authors: ["guoxudong"]
tags: ["云效","阿里云"]
categories: ["阿里云"]
date: 2020-08-25T16:19:09+08:00
lastmod: 2020-08-25T16:19:09+08:00
draft: false
type: blog
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65gy1gi34sbradjj20ge091wh0.jpg"
---
## 前言

今天是七夕节，但对我来时只是”平凡“的一天，上午认识的一个阿里云的小姐姐给我发来一个可疑链接，说是有好玩的活动，推荐我参加一下。于是在午休时间，闲来无事的我点开了这个链接，没想到事情并不简单。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1gi34yex8kgj20al05qmxm.jpg)

## 云效 DevOps

点开之后，原来是使用云效 DevOps 来体验发布一个为朋友、爱人定制的“情书”。年初时，我和云效的开发人员有聊过，他们对云效 DevOps 非常有信心，相信会做出一个和原来的云效完全不同的更好用的产品，正好借着这个机会，我来免费体验一把云效 DevOps。

![](https://tva4.sinaimg.cn/large/ad5fbf65gy1gi352mu9k4j212w0q0wmi.jpg)

### 整体感觉

UI 的整体感觉非常棒，完全不像是“阿里云”的 UI，风格简洁清爽。云效 DevOps 将整个 DevOps 生命周期都做了出来，从代码仓库，静态代码扫描，CI/CD，到文档和任务的管理都有，还改善了之前只能使用阿里云代码仓库的缺点，可以使用自建代码仓库，同时还支持云有云、公有云和混合云的部署方式。同时**云效还为小微企业提供了扶持计划，30人以下团队可以免费使用！**

![活动页](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi359oqgh9j21h20q97a5.jpg)

这里就不过多介绍云效，有兴趣的朋友可以上阿里云官方，搜索**云效**就能了解更多内容了。

## 制作情书

与其说是是制作，不如说是修改制作好的代码，将自己想说的话放进去，然后在通过云效一键发布。

### 克隆七夕代码

新用户注册好之后选择`导入代码库`-`URL导入`，贴入示例代码库地址 `https://code.aliyun.com/yunxiao2020/letter.git` 点击 `确定` 即可完成示例代码的导入。

![云效 DevOps](https://tva2.sinaimg.cn/large/ad5fbf65gy1gi35ntu6avj212m0k2dgv.jpg)

### 修改情书内容

编辑代码修改情书内容，地址`app/service/data.js`：

![修改内容](https://tva3.sinaimg.cn/large/ad5fbf65gy1gi35q45ru3j21hc0pfmy3.jpg)

可以修改以下字段来实现内容的定制：

- `theme`：情书模板，提供了爱人、朋友、同事三种模板，如下所示：

![爱人、朋友、同事](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi35sch68xj210g0jb0xy.jpg)

- `from`：寄信人，上学时候写过情书的同学都懂~
- `To`：收信人，同上
- `avatar`：寄信人头像，我这里使用的是图床的 URL，没有图床的 github 上的图片地址也是可以的
- `question`：开信问题：设置一个只有你的他/她知道答案的问题，防止被别人看到你的真心话
- `answer`：上面问题的答案
- `text`: 情书正文，说出你想和他/她说的话吧！

现在情书内容就写完了，点击`保存`-`确定`，将代码提交到 `master` 分支。

![保存](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gi35y8l234j20fe0880sw.jpg)

### 配置流水线

代码写好了，现在就可以使用流水线将情书的 H5 发布出去了。点击左上角九宫格，选择 `流水线` - `新建流水线`，选择模板：`其他` - `云效七夕活动`

![选择模板](https://tva3.sinaimg.cn/large/ad5fbf65gy1gi362uhnv8j21hc0pfmye.jpg)

之后就可以配置流水线了，点击 `添加代码源`，选择 `云效Codeup` 代码源，选择刚刚克隆的 `letter` 代码库，选择 `master` 分支，开启 `代码源触发`

![image](https://tvax1.sinaimg.cn/large/ad5fbf65gy1gi36917of6j21h70q7wie.jpg)

配置 Docker 镜像构建，点击 `Docker 镜像构建` - `镜像构建并推送至自定义镜像仓库`，填入以下内容：

- 镜像仓库地址: `registry.cn-hangzhou.aliyuncs.com/yunxiao-letter/yunxiao-letter:${BUILD_JOB_ID}`
- 用户名:`yunxiao-letter@1515906102291199`
- 密码: `yunxiao2020`

配置 Kubernetes 发布，点击 `Kubernetes 发布` - `Kubectl发布` - `新建连接` - `自定义集群`

之后进入 `https://research.devops.aliyun.com/kube.config.yml` 页面，将页面配置文件复制到集群配置文件中。

![连接 Kubernetes](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gi36dhtzfxj21070jgt9j.jpg)

配置好连接后，填写其他配置：

- 命名空间填: `yunxiao`
- YAML路径填: `deployment.yml`
- 新建变量1    选择`上游输出`，`YUNXIAO_LETTER_IMAGE`=`镜像仓库地址`
- 新建变量2    选择`自定义`，`PIPELINE_ID`=`${PIPELINE_ID}`

![其他配置](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi36f8xxffj20w70icdgb.jpg)

完成所有配置后，点击 `保存并运行` - `运行`：

![](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi36hajgyuj20fd08s746.jpg)

### 修复问题

这里为了让用户体验质量卡点 `JavaScript 单元测试` 功能，他们还在项目中埋了一个小坑，让首次构建失败，原因是：测试通过率小于100%

![质量卡点](https://tva1.sinaimg.cn/large/ad5fbf65gy1gi36jr1kwjj21090a7t9h.jpg)

查看报错信息，发现是 `expect` 值被设置为 `400`： 

![报错信息](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gi36k3n45mj212g0e4tad.jpg)

进入对应的单元测试文件，修改代码：

![修复问题](https://tva2.sinaimg.cn/large/ad5fbf65gy1gi36md5twaj21h40pzn10.jpg)

修改完成后，点击 `保存` - `提交`，由于之前设置了流水线 `提交源代码触发`，流水线在提交后会自动触发，并发布成功：

![发布成功](https://tva4.sinaimg.cn/large/ad5fbf65gy1gi36npelasj210109l3zc.jpg)

点击`预览`或者扫面二维码就可以看到你的情书了。

## 结语

通过这次七夕活动，体验了一把云效 DevOps，整体来说用户体验很好，同时还提供了30人一下团队免费使用的政策，可以为小团队省出不少搭建和开发 DevOps 环境和流程的时间和经历，非常推荐大家都来尝试一下，制作一封云上情书送给你的他/她吧。

{{% pageinfo color="primary" %}}
活动地址：[https://developer.aliyun.com/adc/series/devops/?spm=a2c6h.12883283.1362932.3.2785201ctOUs0C&accounttraceid=fd6b3040ada34768aa78f84a9f645c46kouc](https://developer.aliyun.com/adc/series/devops/?spm=a2c6h.12883283.1362932.3.2785201ctOUs0C&accounttraceid=fd6b3040ada34768aa78f84a9f645c46kouc)
{{% /pageinfo %}}

## 参考

- [云效DevOps七夕云中密书 - developer.aliyun.com](https://developer.aliyun.com/adc/scenario/exp/8464960ac980400d95ff092b95e1a97e)