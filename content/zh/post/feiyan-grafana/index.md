---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 Grafana 展示肺炎疫情动态"
subtitle: "开发 Grafana DataSource 和 Grafana Dashboard"
summary: "开发 Grafana Dashboard 展示新型肺炎疫情动态。"
authors: ["guoxudong"]
tags: ["肺炎疫情"]
categories: ["肺炎疫情"]
date: 2020-02-14T10:12:52+08:00
lastmod: 2020-02-14T10:12:52+08:00
featured: false
draft: false
type: blog

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3ihjkwh8j23402c0npp.jpg"
  caption: ""
  focal_point: "Center"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
[Grafana]: https://grafana.com/
[SimpleJson]: https://grafana.com/grafana/plugins/grafana-simple-json-datasource/installation

## 前言

新型冠状病毒疫情汹涌而来，全国各地严防死守，而疫情的实时数据也通过不同的渠道，如微信城市服务的疫情动态订阅、支付宝的疫情实时追踪、新浪新闻的疫情实时动态等等，各种平台纷纷将疫情的实时动态进行展示，确保人们可以第一时间了解疫情的发展情况。

而无论是哪一家的数据推送和展示，都是面向大众的，并不能个性化的展示我们最关心的那些数据，所以这时就需要自制一个疫情动态展示的 Dashboard 了。

说到 Dashboard，第一个联想到的当然就是 [Grafana] 了，[Grafana] 是自2014年以来推出的多平台开源分析和交互式可视化软件。连接支持的数据源，它会提供 Web 图表的展示以及报警。终端用户可以通过插件进行拓展，从而使用交互式的查询及展示复杂的监控仪表盘。

## 项目准备

明确目标，我们这里需要定制一个 Dashboard 用于展示疫情动态，由于我目前在上海，需要展示全国和上海的确诊、疑似、治愈和死亡病例数；同时还需要一个病例发展曲线，用来观察疫情发展趋势；各省区情况已经上海各区情况也是需要的。

[Grafana] 只是一个展示数据的工具，首先需要的是数据源，目前市面上并没有可以直接用于 Grafana 的疫情数据源，这里我们需要：

- 需要一个 [Grafana]，无论是在你的笔记本电脑上，还是在你的 K8S 集群中（这里推荐使用 docker 进行运行 Grafana，如果部署在 K8S 集群中，那更好）。

- 安装 [SimpleJson] 插件，它可以将 json 格式的数据，用作 Grafana 的数据源。


## 开发数据源

数据源这里使用 Python Bottle 进行开发，当然你也可以选择 flask，都是一样的，我使用 Bottle 的原因是之前开发的 Grafana 数据源是使用 Bottle 开发的，这里直接拿来就可以用，调试配置甚至用于构建 docker 镜像的 `Dockerfile` 和用于部署 K8S 的 `deploy.yaml` 都有现成可以用的。使用 Python 开发 [Grafana] 数据源很简单，只有符合 [SimpleJson] 的格式要求即可。可以根据 [Oz Nahum Tiram](http://oz123.github.io/about.html) 的博文 [Visualize almost anything with Grafana and Python](http://oz123.github.io/writings/2019-06-16-Visualize-almost-anything-with-Grafana-and-Python/index.html) 来学习如果使用 Python 作为 [Grafana] 的数据源。

在对数据源的定制中，使用两种类型的的数据：

- `timeserie` 类型：

    用于展示全国（含港澳台）和上海地区的疫情实时动态，展示确诊、疑似、治愈和死亡数，并且展示较昨日增加的数量，绘制了【确诊/疑似】数和【治愈/死亡】数的对比曲线。

    这里只要将全国确诊数 `gntotal` 与 当前时间戳组合返回即可，其他指标也是这种方式。

    ```
    @app.post('/query')
    def query():
        print(request.json)
        body = []
        all_data = getDataSync()
        time_stamp = int(round(time.time() * 1000))
        for target in request.json['targets']:
        name = target['target']
        if name == 'gntotal':
            body.append({'target': 'gntotal', 'datapoints': [[all_data['gntotal'], time_stamp]]})
        body = dumps(body)
        return HTTPResponse(body=body, headers={'Content-Type': 'application/json'})
    ```

- `table` 类型：

    用于绘制中国各省确诊、疑似、治愈和死亡病例数表格，以及上海各区确诊、疑似、治愈和死亡病例数表格。

    取出数据中的名称以及确诊、疑似、治愈和死亡数，`append` 到 `rows` 中即可。

    ```
    @app.post('/query')
    def query():
        print(request.json)
        body = []
        all_data = getDataSync()
        sh_data = getShDataSync()
        if request.json['targets'][0]['type'] == 'table':
            rows = []
            for data in all_data['list']:
                row = [data['name'], data['value'], data['susNum'], data['cureNum'], data['deathNum']]
                rows.append(row)
            sh_rows = []
            for data in sh_data['city']:
                row = [data['name'], data['conNum'], data['susNum'], data['cureNum'], data['deathNum']]
                sh_rows.append(row)
            bodies = {'all': [{
                "columns": [
                    {"text": "省份", "type": "name"},
                    {"text": "确诊", " type": "conNum"},
                    {"text": "疑似", " type": "susNum"},
                    {"text": "治愈", "type": "cureNum"},
                    {"text": "死亡", "type": "deathNum"}
                ],
                "rows": rows,
                "type": "table"
            }],
                'sh': [{
                    "columns": [
                        {"text": "省份", "type": "name"},
                        {"text": "确诊", " type": "value"},
                        {"text": "疑似", " type": "susNum"},
                        {"text": "治愈", "type": "cureNum"},
                        {"text": "死亡", "type": "deathNum"}
                    ],
                    "rows": sh_rows,
                    "type": "table"
                }]}

            series = request.json['targets'][0]['target']
            body = dumps(bodies[series])
      return HTTPResponse(body=body, headers={'Content-Type': 'application/json'})
    ```

## 选择展示 Panel 类型

总的来说，使用了4种 Panel 进行展示：

- 展示病例数的展示块，使用 `Singlestat`
- 展示数据对比曲线，使用 `Graph`
- 展示表格，使用 `Table`
- 文字标题，使用 `Text`

## 配置数据源

### 病例数展示块：

这里只有一个值，所以要选择 `First`。

![image](https://tvax2.sinaimg.cn/large/ad5fbf65gy1gbvs6gmbzlj20x00ku0uk.jpg)

### 病例数发展趋势图：

这里将【确诊/疑似】和【治愈/死亡】数进行对比。

![image](https://tva2.sinaimg.cn/large/ad5fbf65gy1gbvs8hmuvoj21gu0iu41j.jpg)

### 数据表格：

![image](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gbvsa2pharj21b30igdi1.jpg)

## 效果

整体效果还可以，先已用作公司大屏展示疫情情况（这里我司用于展示屏幕较小，只不过是一个小米电视，故字体和展示块都做的大了一些）。

![](featured.png)

## 构建

将代码打包成为 docker 镜像，就可以运行在任意环境以及 K8S 集群了，镜像已上传 dockerhub 直接拉取镜像，开箱即食。

```
# Dockerfile
FROM python:3.7.3-alpine3.9

LABEL maintainer="sunnydog0826@gmail.com"

COPY . /app

RUN echo "https://mirrors.aliyun.com/alpine/v3.9/main/" > /etc/apk/repositories \
    && apk update \
    && apk add --no-cache gcc g++ python3-dev python-dev linux-headers libffi-dev openssl-dev make \
    && pip3 install -r /app/requestments.txt -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com

WORKDIR /app

ENTRYPOINT ["uwsgi","--ini","uwsgi.ini"]
```

## 运行

- 拉取镜像

```
docker pull guoxudongdocker/feiyan-datasource
```

- 运行镜像

```
docker run -d --name datasource -p 8088:3000 guoxudongdocker/feiyan-datasource
```

- 添加数据源

    选择 [SimpleJson] 类型的数据源，点击添加，填入数据源地址：

    ![datasource](https://tva4.sinaimg.cn/large/ad5fbf65gy1gbvsocijjuj20jj0lagot.jpg)

- 导入 Dashboard

    点击 `Upload.json file`，选择 `wuhan2020-grafana/dashboard.json`

    ![import](https://tva4.sinaimg.cn/large/ad5fbf65gy1gbvspqvaz0j20uh0iracw.jpg)

- 使用 K8S 部署（可选）

    ```
    kubectl apply -f deploy.yaml
    ```

## 结语

截止目前（2020年2月14日），病例数还在不断的增加，但是疑似病例数趋势开始下降，可以看出，目前新型肺炎的确诊速度增加了；治愈数也在不断的增加；上海地区和其他地区比起来，虽然有大批返工人员进入，但是并没有增加特别多的病例数，各个社区严防死守的效果初显；同时上海一直保持着死亡1人的情况，而且中国首例新型肺炎治愈的也在上海。总的来说只要大家注意预防，待在家中，多消毒，多通风，一定可以战胜疫情，度过难关。

导入 Dashboard 的 `json` 文件和部署 K8S 的 `yaml` 文件都可以在 GitHub 上找到。

项目地址：https://github.com/sunny0826/wuhan2020-grafana
