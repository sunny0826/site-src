---
title: "炫酷的终端软件 eDEX-UI"
date: 2019-04-29T11:55:47+08:00
draft: false
type: blog
banner: "https://wx2.sinaimg.cn/large/ad5fbf65gy1g2jfg79sy3j21hc0tztfr.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "eDEX-UI 是一个全屏且跨平台、可定制的终端模拟器，具有先进的监控和触摸屏支持。它的外观类似科幻的计算机界面。在保持未来感的外观和感觉的同时，它努力保持一定的功能水平并可用于现实场景，其更大的目标是将科幻用户体验纳入主流。"
tags: ["工具"]
categories: ["程序员趣闻"]
keywords: ["eDEX-UI","工具"]
image:
  url: "https://tvax4.sinaimg.cn/large/ad5fbf65ly1ge3ih2v4pkj21hc0tznfr.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
马上就是五一假期了，而且今年的五一假期有4天！想必大家已经安排好是在家写代码还是出门去冒险了。不过在五一假期之前，我这里推荐一个好玩的又好用的软件给大家。

想必大部分朋友和我一样在上周去看了复联4，其中钢铁侠战衣及设备各种炫酷又极具科技感的操作界面一定让你记忆犹新。很多朋友可能和我一样，都希望拥有一套这样的操作界面，这样不管是工作还是学习都会变得有趣而高效（主要是炫酷）。其实很早以前我就尝试写过，但是由于技术有限，写出来的工具都不是很符合我的要求，渐渐的也就都废弃了。而今天要介绍的这个软件，完全符合我的要求，高端大气上档次，并且还是开源的。

## eDEX-UI

[eDEX-UI](https://github.com/GitSquared/edex-ui) 是一个全屏且跨平台、可定制的终端模拟器，具有先进的监控和触摸屏支持。它的外观类似科幻的计算机界面。在保持未来感的外观和感觉的同时，它努力保持一定的功能水平并可用于现实场景，其更大的目标是将科幻用户体验纳入主流。

### 特性

- 功能齐全的终端仿真器，带有选项卡、颜色、模拟鼠标，并支持 curses 和类似 curses的应用程序。
- 实时系统（CPU、RAM、进程）和网络（GeoIP、活动连接、传输速率）监控。
- 完全支持触摸屏，包括屏幕键盘。
- 具备跟随终端 CWD（当前工作目录）的目录查看器。
- 包括主题、屏幕键盘布局、CSS 注入等在内的高级自定义。
- 由才华横溢的声音设计师制作的可选音效，可实现最佳的好莱坞黑客氛围。

### 显示

![image](https://yqfile.alicdn.com/b959597643a41c4b83e697307877082124c360d4.png)

这里我使用了 `tron-disrupted` 主题，还有多种主题可以选择

可以看到这里的界面十分炫酷，可以为有些乏味的 shell 操作增添一抹乐趣

### 配置

eDEX-UI 可以通过 `settings.json` 文件进行配置，配置包括执行的 shell 类型、工作目录、键盘类型、主题等

`settings.json` 在 Mac 系统中，存放在 `/Users/guoxudong/Library/Application Support/eDEX-UI` 中，默认的工作目录也是这个路径

![image](https://wx3.sinaimg.cn/large/ad5fbf65gy1g2jflhunukj21h30tck0r.jpg)

这里可以看到我选择使用 `zsh` 和 `tron-disrupted` 主题，并将工作目录改为了我的用户空间

## 局限

- 目前看来该软件的全平台支持是不错的，同时还支持触摸屏操作，但是目前还未测试在 pad 上使用，测试之后会在后续文章中补充
- CPU 占用过高，该软件 CPU 占用很高，如果是配置一般的电脑不建议让其作为终端常驻，偶尔拿出来玩玩即可