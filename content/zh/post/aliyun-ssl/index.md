---
title: "GitHub/Gitee 静态页托管页部署SSL证书"
date: 2019-08-23T09:36:55+08:00
draft: false
type: blog
banner: "https://tva2.sinaimg.cn/large/ad5fbf65gy1g69glqoddyj21jk15odie.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本文档介绍了在 Github/Gitee 的静态页托管Pages服务部署SSL证书，配置HTTPS安全访问的操作说明。"
tags: ["阿里云"]
categories: ["部署安装"]
keywords: ["https","ssl","阿里云"]
image:
  url: "https://tvax2.sinaimg.cn/large/ad5fbf65ly1ge3i9py7k6j21jk15odie.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

本文档介绍了在 [Github](https://pages.github.com/) / [Gitee](https://gitee.com/help/articles/4136) 的静态页托管Pages服务部署SSL证书，配置HTTPS安全访问的操作说明。

### Pages服务

Github/Gitee的Pages是一个免费的静态网页托管服务，您可以使用Github或码云Pages托管博客、项目官网等静态网页。常见的静态站点生成器有：Hugo、Jekyll、Hexo等，可以用来生成静态站点。默认情况下，托管的站点使用 `github.io` / `gitee.io` 域名来访问站点，同时也支持自定义域名，并配置强制使用HTTPS。

> 注意：如果要在 Gitee Pages 上配置自定义域名+HTTPS，则需要开启 Gitee Pages Pro 。

### Github Pages 服务部署SSL证书

#### 前提条件

- GitHub 仓库
- 开启 GitHub Pages

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69e503ukoj21ig0hwad9.jpg)

#### 证书签发

1. 购买证书后点击申请

    ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69ee2r500j22cc078t9z.jpg)

2. 证书申请

    如果该域名是由阿里云购买，则选择自动DNS验证，如果不是在阿里云购买的，可以选择手动验证。

    ![image](https://tva2.sinaimg.cn/bmiddle/ad5fbf65gy1g69egsu7fuj20ye0swwh3.jpg)

    ![image](https://tva2.sinaimg.cn/bmiddle/ad5fbf65gy1g69eo1wls7j20ya0r0418.jpg)

3. 证书签发

    证书通过申请后，会收到证书签发的邮件。

    ![image](https://tva2.sinaimg.cn/wap720/ad5fbf65gy1g69epoqw6uj21680cotaj.jpg)

#### 设置自定义域名

1. 解析域名

    在证书签发成功后，添加DNS解析，将绑定了SSL证书的域名解析到 `YourRepo.github.io` 。

    ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69evivrvqj21mi07it9g.jpg)

2. 配置域名

    解析之后将域名添加到 `Custom domain` 并且点击 `Save` ，Github会自动验证，出现`Your site is published at https://YourDomainName.com/`则证明解析成功。
    
    ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69esrcn2tj21a210wwk0.jpg)

### Gitee Pages Pro 服务部署SSL证书

#### 前提条件

- Gitee 仓库
- 开启 Gitee Pages Pro

> Gitee 需要开启 Gitee Pages Pro 服务才支持自定义域名+HTTPS。

#### 证书签发

证书签发同 Github Pages。这里介绍非阿里云购买的域名，进行证书申请。

1. 购买证书流程如上

2. 申请证书

    证书验证方式选择`手工DNS验证`。

3. 拷贝验证信息

    拷贝验证信息内的`记录值`。

    ![image](https://tva2.sinaimg.cn/bmiddle/ad5fbf65gy1g69eo1wls7j20ya0r0418.jpg)

4. 验证解析

    进入购买域名所在网站进行DNS解析，这里以[name.com](https://www.name.com/zh-cn/)为例：

    ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69fqad2euj221g0700tt.jpg)

    解析成功之后，返回阿里云SSL证书管理页面点击`验证`.

5. 证书签发

    签发成功后会收到签发成功的邮件。

#### 设置自定义域名

1. 解析域名

    进入域名所在网站，添加DNS解析记录，将绑定了SSL证书的域名解析到`gitee.gitee.io`

    ![image](https://ws3.sinaimg.cn/large/ad5fbf65gy1g69fyy5it5j21z606mjs9.jpg)

2. 配置域名

    1. 域名添加到`自定义域名`

        ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69g11wx0qj21a60xiq7m.jpg)

    2. 配置证书

        - 证书下载，选择 nginx 类型。

            ![image](https://tva2.sinaimg.cn/bmiddle/ad5fbf65gy1g69g3pua7xj20ne0v0jus.jpg)

        - gitee pages 配置证书，将证书文件与私钥文件贴入并提交。

            ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g69g64n1btj21bs0yogq8.jpg)

        - 勾选`强制使用HTTPS`，并保存。

### 验证

在Github/Gitee配置成功之后，您可在浏览器中输入 https://www.YourDomainName.com 验证证书安装结果。可以正常访问静态托管站点，并且浏览器地址栏显示绿色的小锁标识说明证书安装成功。
