---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 iTerm2 打造美观高效的 Mac 终端"
subtitle: ""
summary: "快速配置 Iterm2 + oh my zsh + powerlevel10k"
authors: ["guoxudong"]
tags: ["工具"]
categories: ["工具","Mac"]
date: 2021-01-04T14:14:02+08:00
lastmod: 2021-01-04T14:14:02+08:00
draft: false
type: blog
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65gy1gmbpun02u3j20rs0ik0tn.jpg"
---
## 前言

最近换了一台新电脑，开发环境和软件都需要重新安装和配置，正好借着这个机会，介绍一下 macOS 终端神器 iTerm2 的安装配置，并推荐一些插件和好用的工具。

## iTerm2

iTerm2 是默认终端的替代品，也是目前 macOS 下最好用的终端工具，集颜值和效率于一身。

### 安装

直接前往 [iTerm2 官网](http://www.iterm2.com/) 下载即可，下载完成后解压并双击安装。

![iTerm2 官网](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gmbncotndlj21km1gu7hm.jpg)

### 设置热键

为了快速唤出 iterm2 终端，这里推荐使用热键进行唤出。

![设置热键](https://tva1.sinaimg.cn/large/ad5fbf65gy1gmbnhmj7w8j21k20y87el.jpg)

### 设置 Status bar

iterm2 提供了很多 Status bar，可在在终端页面显示更多关于本机的信息，如：CPU、内存、电池电量等。

![配置 Status bar](https://tva1.sinaimg.cn/large/ad5fbf65gy1gmbnkvtjloj21ey0tojwj.jpg)

点击 `Configure Status bar` 进入配置页面，这里将想要的 Status bar 拖入下面的方框即可。这里还推荐选择 `Auto-Rainbow`，这样 Status bar 就是以彩色的形式展示了。

![选择 Status bar](https://tva4.sinaimg.cn/large/ad5fbf65gy1gmbnn70borj21eu0skdjz.jpg)

### 配色

选择一个自己喜欢的配色方案。

![选择配色方案](https://tva4.sinaimg.cn/large/ad5fbf65gy1gmbnqt1lhej21fg0qqn26.jpg)

### 光标选择

这里提供了三种光标可供选择：`_`、`|`、`[]`。

![光标选择](https://tva1.sinaimg.cn/large/ad5fbf65gy1gmbnsyh5rqj21g00qu79h.jpg)

### 窗口设置

这里可以设置窗口透明度、背景图片、行列数以及风格等。

![窗口设置](https://tva2.sinaimg.cn/large/ad5fbf65gy1gmbnw0mzfej21fi0wmagb.jpg)

### 迁移配置

如果你已经有配置好的 iterm2，可以将配置导出，迁移到新 Mac 上。

![导出配置](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gmboqetam4j21fi17gkjl.jpg)

之后在新 Mac 上导入即可。

![导入配置](https://tva4.sinaimg.cn/large/ad5fbf65gy1gmborddsylj21es176e81.jpg)

## oh my zsh

在设置好 iterm2 之后，就需要安装 [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)。Oh My Zsh 是一款社区驱动的命令行工具，它基于 zsh 命令行，提供了主题配置，插件机制，大大提高了可玩性及使用效率。

### 安装

可以使用 `curl` 和 `wget` 安装：

```bash
# curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# wget
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### powerlevel10k

[powerlevel10k](https://github.com/romkatv/powerlevel10k) 是一款 zsh 主题，是 [powerlevel9k](https://github.com/Powerlevel9k/powerlevel9k) 的升级版，强调快速、高效和开箱即用。powerlevel10k 免去了之前 powerlevel9k 比较繁琐的安装方式，如安装字体，配置样式、修改主题等一系列繁琐的操作，开箱即用，非常简单。

#### 安装 

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

#### 配置

在代码拉取成功后，执行命令 `source ~/.zshrc` 会自动安装字体文件，无需任何其他操作。

之后执行命令，即可开始配置：

```bash
p10k configure
```

这里会进行交互式的配置，只需根据提示进行选择即可。

![交互式的配置](https://tva2.sinaimg.cn/large/ad5fbf65gy1gmap0z93zdg20ok0l60xf.gif)

### 插件

oh my zsh 还提供了多种好用的插件，这里介绍两款好用的插件。

#### 语法高亮

可以在命令行高亮显示语法，效果如下：

![语法高亮](https://tvax4.sinaimg.cn/large/ad5fbf65gy1gmbodvj2sej20lu0isdj6.jpg)

安装方式：

```bash
# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git  ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
```

#### 命令自动补全

可以根据您的历史记录和完成情况给输入的命令提供建议，效果如下：

![命令自动补全](https://tvax3.sinaimg.cn/large/ad5fbf65gy1gmbojbb38wj20sy044wel.jpg)

安装方式

```bash
# zsh-autosuggestion
git clone https://github.com/zsh-users/zsh-autosuggestions.git  ~/.oh-my-zsh/plugins/zsh-autosuggestions
```

#### 插件配置

安装好之后，需要修改 `.zshrc`：

```bash
# .zshrc
...
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
...
```

修改完成后，执行命令，完成设置：

```bash
source ~/.zshrc
```

更多插件，详见：https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins

## 结语

iTerm2 其实还有很多好玩的配置，由于篇幅有限这里就不过多介绍了，感兴趣的朋友可以登录官网查看官方文档。