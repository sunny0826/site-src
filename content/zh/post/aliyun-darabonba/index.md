---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "听说，阿里云给它的 OpenAPI 开发了一套编程语言"
subtitle: ""
summary: "这回是 OpenAPI as code 了"
authors: ["guoxudong"]
tags: ["阿里云"]
categories: ["阿里云"]
date: 2020-09-01T11:49:12+08:00
lastmod: 2020-09-01T11:49:12+08:00
draft: false
type: blog
image:
  url: "https://tvax3.sinaimg.cn/large/ad5fbf65gy1gic33rn8o2j21qq15ogsv.jpg"
---
## OpenAPI

熟悉公有云的同学对 OpenAPI 都不会陌生，OpenAPI 可以称之为公有云与用户之间的一座桥梁。直接使用公有云的大多是技术人员，而对于技术人员，尤其是开发者来说，往往并不满足于只使用 Web UI 界面来与公有云交互，尤其是当使用的公有云产品日益增多时。由于 Web UI 是面向全体用户设计，并不能满足用户的全部需求。这时 OpenAPI 就出现了，用户通过 OpenAPI 将自己的系统直接对接公有云，并根据自己的使用场景和需求进行设计，开发出一套满足自己需求的公有云管理系统或流程，这样既提高了用户本身的自动化水平，还降低了误操作带来的风险。

## 背景

最早的 OpenAPI 往往是云厂商开放出来的一系列 RESTful API 接口，用户需要根据接口要求自己封装认证方法、传入参数，但是由于部分 OpenAPI 并不是 RESTful 风格、产品升级导致的接口参数变化、文档更新不及时等问题，导致云厂商开始寻求新的解决办法。SDK 就是一种解决办法，通过云厂商自己封装的 SDK，可以提高用户体验并屏蔽部分直接使用 OpenAPI 带来的麻烦，但是随着云产品的增加，需要开发的 SDK 越来越多，并且由于 SDK 往往是多语言的，云厂商需要投入大量人手来维护这些 SDK，导致某些产品由于人力资源有限并没有提供 SDK 或者 SDK 语言不全。

在这种背景下，阿里云的同学提出了一种新办法：他们重新定义了一门 DSL 语言 Darabonba 来描述各种各样的 OpenAPI，就如题目所说给 OpenAPI 开发了一种编程语言，从某种程度上来说，可以称之为：**OpenAPI as code**。

## Darabonba

Darabonba(原名 TeaDSL)，是一种 OpenAPI 应用的领域特定语言。可以利用它为任意风格的接口生成多语言的 SDK、代码示例、测试用例、接口编排等。

![Darabonba 设计理念](https://tva3.sinaimg.cn/large/ad5fbf65ly1giba4r6z4aj20rs0ci400.jpg)

在笔者看来，操作云产品的功能是开发者的目的，而 OpenAPI 是实现这个目的的手段，SDK 则简化了这种手段，Darabonba 的作用则降低了开发 SDK 的成本，并提高了开发 SDK 的速度，对云厂商的效率会有非常明显的提升。同时 Darabonba 也为使用多种编程语言的团队提供了一条捷径，原先需要每种编程语言都要出人来参加 OpenAPI 的集成，现在只需要公有云维护团队出一名或几名同学即可完成全语言 SDK 的生成。而如果公司本身有需要开发大量的 OpenAPI，甚至可以直接使用 Darabonba，开发符合自己系统 OpenAPI 的工具，[Darabonba](https://github.com/aliyun/darabonba) 目前已经开源，使用 [Apache-2.0 LICENSE](https://github.com/aliyun/darabonba/blob/master/LICENSE)。

### 组件

Darabonba 目前支持：**Java**、**C#**、**TypeScript**、**PHP**、**Golang** 和 **Python** 代码的生成，除了解析器和多语言生成器，还提供了：

- [VS Code 插件](https://github.com/aliyun/darabonba-vscode)：提供语法高亮、代码提示、代码格式化、语法检查等功能。
- [CLI](https://github.com/aliyun/darabonba-vscode)：命令行工具，快速在本地拉起一个 Darabonba 项目。
- [Darabonba 模块仓库](https://darabonba.api.aliyun.com/module)：模块仓库，提供 Darabonba 模块的发布和下载。

### Darabonba 语言优势

- 更宽泛的风格支持：支持 RESTful 风格的 OpenAPI，及其他所有基于 HTTP 协议的 OpenAPI。对非 RESTful 风格的 OpenAPI 更友好。
- 编程逻辑化：将 OpenAPI 从元数据定义走向编程化，封装复杂的 HTTP 处理过程为简单的方法接口。
- 运行时事务性支持：支持配置或设置 OpenAPI 的幂等、重试、超时、退避，将复杂的 OpenAPI 调用过程收敛在方法中。

## 快速上手

下图可以看到完整的 Darabonba 运行流程，现在我们就来快速制作一套 Code Sample 吧。

![](https://tva3.sinaimg.cn/large/ad5fbf65ly1gibatrt4puj21ao1e8tet.jpg)

### 安装

阿里云提供了Darabonba 的 Web UI 界面，但是在网页上不好调试，我们选择本地安装 CLI 命令行工具。

```shell
# Darabonba CLI 是由 Node.js 开发的，使用 npm 来安装
$ npm install -g @darabonba/cli
```

### 构建 Darabonba 模块

我们假设要创建一个模块为 `sample_ecs`，用来生成查询 ECS 信息的 SDK 代码。首先创建一个目录：

```shell
$ mkdir sample_ecs
$ cd sample_ecs
```

初始化模块，输入相关信息：

```shell
$ dara init
package scope: guoxudong.io
package name: sample_ecs
package version: 0.0.1
main entry: main.dara
```

完成初始化后，会新建 2 个文件：

- `Darafile`：包描述文件
- `main.dara`：入口文件

### 安装 VS Code 插件

打开 VS Code，按 `F1` 或 `Ctrl + Shift + P` 打开命令面板，选择 Install Extension 并输入 `darabonba`。

或启动VS Code 快速打开（`Ctrl + P`），粘贴以下命令，然后按 Enter。

```shell
ext install darabonba.darabonba
```

之后就可以使用语法高亮、代码提示、代码格式化、语法检查等功能了。

![代码高亮](https://tva2.sinaimg.cn/large/ad5fbf65ly1gibbkgjhc5j20q80l3wh7.jpg)

### 安装依赖模块

首先需要设置依赖仓库地址：

```shell
$ dara config set registry https://darabonba.api.aliyun.com
```

之后就是将依赖写入 `Darafile` 中：

```json
{
  "scope": "guoxudong.io",
  "name": "sample_ecs",
  "version": "0.0.1",
  "main": "main.dara",
  "libraries": {
    "Console": "darabonba:Console:*",
    "ECS": "alibabacloud:Ecs20140526:*",
    "RPC": "alibabacloud:RPC:*",
    "Util": "darabonba:Util:*"
  },
  "java": {
    "package": "aliyun.com.alibabacloud.sample"
  },
  "csharp": {
    "namespace": "Alibabacloud.Sample"
  },
  "php": {
    "package": "Alibabacloud.Sample"
  },
  "python": {
    "package": "alibabacloud_sample"
  }
}

```

这里引入了4个模块：

- `Console`：打印输出模块
- `ECS`：ECS 模块
- `RPC`：RPC Client 模块
- `Util`：工具模块

{{% alert title="Warning" color="warning" %}}
值得注意的的是 `libraries` 中内容的 key，就是 `.dara` 文件中 import 导入依赖的名称，所以这里可以将 key 修改为好理解的值，然后 import 相应值就可以了。
{{% /alert %}}

修改完 `Darafile` 之后，安装这些依赖：

```shell
$ dara install
```

之后就可以看到多了一个 `.libraries.json` 文件和一个 `libraries` 目录，需要的所有依赖模块就都已经安装好了。

### 查看模块内容

更多的模块，可在[模块仓库](https://darabonba.api.aliyun.com/module)中搜索。这里以 ECS 模块为例

![ECS 模块](https://tva2.sinaimg.cn/large/ad5fbf65gy1gic1orwhjoj21h70q3gr4.jpg)

可以在 `Detail` 中看到所有可以调用的接口，通过还可以点击其他 tab 可以查看版本、安装方式等内容：

![ECS 模块](https://tvax2.sinaimg.cn/large/ad5fbf65ly1gibbruwd1zj21at0pqadt.jpg)

也可通过命令单独安装模块：

```shell
$ dara install alibabacloud:Ecs20140526
```

### 代码编写

现在就可以编写 Darabonba 代码了，Darabonba 代码的整体风格偏向于 Java，不是很难懂，这里贴上一段简单的代码：

```java
import ECS;
import RPC;
import Util;
import Console;

/**
* Initialization  初始化公共请求参数
*/
static function Initialization(regionId: string)throws : ECS{

    var config = new RPC.Config{};
    // 您的AccessKey ID
    config.accessKeyId = "<accessKeyId>";
    // 您的AccessKey Secret
    config.accessKeySecret = "<accessKeySecret>";
    // 您的可用区ID
    config.regionId = regionId;

    return new ECS(config);
}

/**
* DescribeZones    查询一个阿里云地域下的可用区
*/
static async function DescribeZones(client: ECS, regionId: string)throws: void{
    var req = new ECS.DescribeZonesRequest{};
    // 可用区所在的地域ID。您可以调用DescribeRegions查看最新的阿里云地域列表
    req.regionId = regionId;
    // 根据汉语、英语和日语筛选返回结果。更多详情，请参见RFC7231
    // 取值范围：
    // zh-CN
    // en-US
    // ja
    // 默认值：zh-CN。
    req.acceptLanguage = "zh-CN";
    var resp = client.describeZones(req);
    Console.log("--------------------查询地域下的可用区--------------------");
    Console.log(Util.toJSONString(resp));
}


static async function main(args: [string]) throws: void {
    // 可用区域Id （请自行配置）
    var regionId = "<regionId>";

    var client = Initialization(regionId);

    // 查询阿里云地域下的可用区
    DescribeZones(client, regionId)
}
```

这个代码注释很完善，这里就不多讲解了，本示例在官网有[完整示例](https://api.aliyun.com/?spm=a2c6h.17640777.J_1935739830.2.8f9a54e1V4j8vN#/codesample)，有兴趣的同学可以研究一下。而 Darabonba 的文档，可以在 [Github](https://github.com/aliyun/darabonba) 上找到。

### 代码生成

现在就可以生成代码了，下面以生成 Python 代码为例，执行命令：

```shell
$ dara codegen python ./tmp
```

命令执行成功后，就可以看到 Python 代码已经生成了：

![image](https://tva4.sinaimg.cn/large/ad5fbf65gy1gic21suu35j208a05h3yh.jpg)

如果代码还没有写完，想检查是否有语法错误，可以使用 `check` 命令检查：

```shell
$ dara check main.dara
Check success !
```

到这我们的代码就生成成功了，但是这还不是结束，我们需要去测试一下生成的代码能否正常运行，在实践中就出现过代码生成成功，但是运行报错的情况。

{{% alert title="Warning" color="warning" %}}
如果生成的是 Python 代码，这里推荐使用 `Python 3.6`，经测 3.8 版本不支持 sdk 的一些语法。
{{% /alert %}}

同样的，也可以在 [OpenAPI Explorer Code Sample](https://darabonba.api.aliyun.com/sample)，通过 Web UI 来生成代码，除了调试速度比较慢之外，其余体验都十分不错。

![Code Sample](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gic2hxwdtaj21ha0qb441.jpg)

## Code Sample 全民赛码

最近阿里云还推出了这么一个比赛，看了下奖品有机械键盘、无人机、双肩包和内推资格，有兴趣的同学可以关注一下，还是挺好玩的：[传送门](https://developer.aliyun.com/topic/codesample/active1?spm=dara_code_sample.home.0.0.2ee614e5L9uDCw)

![](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gic2ef0fxgj21o00hyh65.jpg)

阿里云开放平台携手开发者社区、内容设计部，联合举办“OpenAPI 开发者挑战赛第三期—— CodeSample 全民赛码 ”，面向数万开发者，招募阿里云 OpenAPI 示例代码（CodeSample）。无论您是入门开发，或是运维大神，无论是利用 OpenAPI 解决一个轻量场景，或是满足一个小功能，通通到碗里来！

## 结语

在这个项目叫 TeaDSL 的时候笔者就开始关注 Darabonba 了，由于笔者是 OpenAPI SDK 的重度使用用户，之前开发的 devops 平台以及 [cms-grafana-builder](https://github.com/sunny0826/cms-grafana-builder) 项目都大量使用了阿里云 SDK。在4月份看到朴灵的[《TeaDSL：支持任意 OpenAPI 网关的多语言 SDK 方案》](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247495266&idx=1&sn=64177bf3fc7f4068c14733dc77b5383f&chksm=e92ad36dde5d5a7bf6a0ddd821a3cdee6ee063eb221623c6c8333cf68787b3118bd0a8653f17&scene=21#wechat_redirect)时，认为其只是解决云厂商 OpenAPI 开发的多语言困局，提升研发效率，和 OpenAPI 的使用者关系不大。但是在这次进行深入研究之后发现，Darabonba 甚至可以用来生成自己系统的 OpenAPI 多语言 SDK，并不是只能用于生成阿里云的 SDK，非常的惊艳。
