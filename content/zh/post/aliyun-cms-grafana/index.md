---
title: "使用 Grafana 展示阿里云监控指标"
date: 2019-11-07T11:08:36+08:00
draft: false
type: blog
banner: "https://tvax1.sinaimg.cn/large/ad5fbf65gy1g8piqjask7j21y014swo7.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
# translator: "郭旭东"
# translatorlink: "https://github.com/sunny0826"
# originallink: ""
summary: "本文介绍使用 Grafana 展示阿里云监控指标的方法，并提供了使用 helm chart 一键部署包含阿里云监控 dashboard 的 Grafana-Server。"
tags: ["阿里云","kubernetes","grafana"]
categories: ["kubernetes"]
keywords: ["阿里云","kubernetes","grafana"]
image:
  url: "https://tva2.sinaimg.cn/large/ad5fbf65ly1ge3i6sry8nj21y014swo7.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---

## 前言

对于阿里云用户来说，阿里云监控是一个很不错的产品，首先它在配额内使用是免费的！免费的！免费的！重要的事情说三遍。他的功能类似于 zabbix，但是比 zabbix 提供了更多的监控项，基本上在云上使用的资源都可以通过云监控来实时监控。而它提供的开箱即用方式，天然集成云资源，并提供多种告警方式，免去了监控与告警系统搭建与维护的繁琐，并且减少了资源的消耗，比购买 ECS 自己搭建 zabbix 要少消耗很多资源。同时阿里云监控和阿里云其他服务一样，也提供了比较完整的 OpenApi 以及各种语言的 sdk，可以基于阿里云的 OpenApi 将其与自己的系统集成。我们之前也是这么做的，但是随着监控项的增加，以及经常需要在办公场地监控投屏的专项监控页，光凭我们的运维开发工程师使用 vue 写速度明显跟不上，而且页面的美观程度也差很多。

### 手写前端 VS Grafana

手写前端虽然可定制化程度更高，但是需要消耗大量精力进行调试，对于运维人员，哪怕是运维开发也是吃不消的（前端小哥哥和小姐姐是不会来帮你的，下图就是我去年拿 vue 写的伪 Grafana 展示页面，花费了大约一周时间在调整这些前端元素）。
![image](https://tva4.sinaimg.cn/large/ad5fbf65gy1g8pfrw1licj22ye1gg4qp.jpg)

Grafana 则标准化程度很高，展示也更加符合大众审美，某些定制化需求可以通过自定义 DataSource 或者 AJAX 插件的 iframe 模式完成。开发后端 DataSource 肯定就没有前端调整 css 那么痛苦和耗时了，整体配置开发一个这样的页面可能只消耗一人天就能完成。而在新产品上线时，构建一个专项监控展示页面速度就更快了，几分钟内就能完成。
![image](https://tva4.sinaimg.cn/large/ad5fbf65gy1g8pfvp0keej22yc1g2khm.jpg)
## 关于阿里云监控

云监控（CloudMonitor）是一项针对阿里云资源和互联网应用进行监控的服务。

云监控为云上用户提供开箱即用的企业级开放型一站式监控解决方案。涵盖 IT 设施基础监控，外网网络质量拨测监控，基于事件、自定义指标、日志的业务监控。为您全方位提供更高效、更全面、更省钱的监控服务。通过提供跨产品、跨地域的应用分组管理模型和报警模板，帮助您快速构建支持几十种云产品、管理数万实例的高效监控报警管理体系。通过提供 Dashboard，帮助您快速构建自定义业务监控大盘。使用云监控，不但可以帮助您提升您的系统服务可用时长，还可以降低企业 IT 运维监控成本。

云监控服务可用于收集获取阿里云资源的监控指标或用户自定义的监控指标，探测服务可用性，以及针对指标设置警报。使您全面了解阿里云上的资源使用情况、业务的运行状况和健康度，并及时收到异常报警做出反应，保证应用程序顺畅运行。

## 关于 Grafana

Grafana 是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。由于云监控的 Grafana 还没有支持告警，所以我们这里只用了 Grafana 的可视化功能，而告警本身就是云监控自带的，所以也不需要依赖 Grafana 来实现。而我们的 Prometheus 也使用了 Grafana 进行数据可视化，所以有现成的 Grafana-Server 使用。

## 阿里云监控对接 Grafana

首先 Grafana 服务的部署方式这里就不做介绍了，请使用较新版本的 Grafana，最好是 5.5.0+。后文中也有我开源的基于阿里云云监控的 Grafana 的 helm chart，可以使用 helm 安装，并会直接导入云监控的指标，这个会在后文中介绍。

### 安装阿里云监控插件

进入插件目录进行安装

```bash
cd /var/lib/grafana/plugins/
git clone https://github.com/aliyun/aliyun-cms-grafana.git 
service grafana-server restart
```

如果是使用 docker 或者部署在 k8s 集群，这里也可以使用环境变量在 Grafana 部署的时候进行安装

```yaml
...
spec:
  containers:
  - env:
    - name: GF_INSTALL_PLUGINS  # 多个插件请使用,隔开
      value: grafana-simple-json-datasource,https://github.com/aliyun/aliyun-cms-grafana/archive/master.zip;aliyun-cms-grafana
...
```

您也可以下载 aliyun-cms-grafana.zip 插件解压后，上传服务器的 Grafana 的 plugins 目录下，重启 grafana-server 即可。

### 配置云监控 DataSource

1. Grafana 启动后，进入 `Configuration` 页面，选择 `DataSource` Tab 页，单击右上方的`Add data source`，添加数据源。
2. 选中`CMS Grafana Service`，单击`select`。
    ![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1g8ph0ukr0pj21nm0jk76m.jpg)
3. 填写配置项，URL 根据云监控所在地域填写，并且填写阿里云账号的 accessKeyId 和 accessSecret，完成后单击`Save&Test`。
    ![image](https://tvax3.sinaimg.cn/large/ad5fbf65gy1g8ph4bg2bij218m194n9f.jpg)

### 创建 Dashboard

1. 单击 `Create` -> `Dashboard` -> `Add Query`
2. 配置图标，数据源选择之前添加的 `CMS Grafana Service`，然后文档中的配置项填入指标即可（这里要注意的是，云监控 API 给返回的只有实例 ID，并没有自定义的实例名称，这里需要手动将其填入 `Y - column describe` 中；而且只支持输入单个 Dimension，若输入多个，默认选第一个，由于这些问题才有了后续我开发的 `cms-grafana-builder` 的动机）。
    ![image](https://tva4.sinaimg.cn/large/ad5fbf65gy1g8phck0irbj22ye13in79.jpg)
3. 配置参考 [云产品监控项](https://help.aliyun.com/document_detail/28619.html)，
    ![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1g8phg832uvj21a40vo793.jpg)

## 使用 helm chart 的方式部署 Grafana

项目地址：https://github.com/sunny0826/cms-grafana-builder

### cms-grafana-builder

由于上文中的问题，我们需要手动选择每个实例 ID 到 Dimension 中，并且还要讲该实例的名称键入 `Y - column describe` 中，十分的繁琐，根本不可能大批量的输入。

这就是我开发这个 Grafana 指标参数生成器的原因，起初只是一个 python 脚本，用来将我们要监控的指标组装成一个 Grafana 可以使用 json 文件，之后结合 Grafana 的容器化部署方法，将其做成了一个 helm chart。可以在启动的时候自动将需要的参数生成，并且每日会对所有指标进行更新，这样就不用每次新购或者释放掉资源后还需要再跑一遍脚本。

### 部署

只需要将项目拉取下来运行 `helm install` 命令

```bash
helm install my-release kk-grafana-cms \
--namespace {your_namespace} \
--set access_key_id={your_access_key_id} \
--set access_secret={your_access_secret} \
--set region_id={your_aliyun_region_id} \
--set password={admin_password}
```

更多详情见 [github README](https://github.com/sunny0826/cms-grafana-builder)，欢迎提 issue 交流。

### 指标选择

在部署成功后，可修改 ConfigMap：`grafana-cms-metric`，然后修改对应的监控指标项。

### 效果

ECS:
![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1g8pi9toh3dj21gv0pldyf.jpg)

RDS:
![](https://tva2.sinaimg.cn/large/ad5fbf65gy1g8pi9o91ejj21h80q316p.jpg)

EIP:
![](https://tva4.sinaimg.cn/large/ad5fbf65gy1g8pi9i9if3j21h70q3aif.jpg)

Redis:
![](https://tvax1.sinaimg.cn/large/ad5fbf65gy1g8pi8ss733j21h30pz7b6.jpg)

## 后记

为了满足公司需求，后续还开发 DataSource 定制部分，用于公司监控大屏的展示，这部分是另一个项目，不在这个项目里，就不细说了，之后有机会总结后再进行分享。