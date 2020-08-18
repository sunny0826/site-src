---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "使用 Golang 和 HTML5 开发一个 MacOS App"
subtitle: ""
summary: "本篇文章将介绍如何使用 Go 语言 和 HTML5 来开发一个 MacOS App。"
authors: ["guoxudong"]
tags: ["Mac","Go","kustomize"]
categories: ["Go"]
date: 2020-08-18T09:23:28+08:00
lastmod: 2020-08-18T09:23:28+08:00
draft: false
type: blog
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65gy1ghuq4vr97sj21qi15odgp.jpg"
---
## 前言

Go语言（也称为Golang）是 google 在 2009 年推出的一种编译型编程语言。相对于其他编程语言，golang 具有编写并发程序或网络交互简单、数据类型丰富、编译速度快等特点，比较适合于高性能、高并发场景。Go 语言一直在网络编程、云平台开发、分布式系统等领域占据着重要的地位，尤其在云原生领域，杀手级项目 Docker 和 Kubernetes 都是采用 Go 语言开发的。而在其他领域，比如桌面应用开发，也有一些框架可以使用，本篇文章就来介绍如何使用 Go 语言 和 HTML5 来开发一个 MacOS App。

## 框架选择

这里我选用了 [echo](https://github.com/labstack/echo) 作为 web 框架，当然也可以选择其他的 web 框架，选择 echo 只不过因为其比较轻量。要做桌面应用，还需要一个 GUI 框架来构建应用，这里我选择的是 [Lorca](https://github.com/zserge/lorca)，使用 Lorca 可以用 Go 编写 HTML5 桌面程序，依赖 Chrome 进行 UI 渲染，但却不需要把 Chrome 打包到应用中，也就是说使用应用的电脑，需要安装 Chrome。

### lorca

echo 的使用方式中规中矩，没有什么需要介绍的。这里简要介绍一下 lorca，其的使用方法和原理都很简单，可以将其看做是一个浏览器，可在其上运行 web 应用，lorca 可直接将 web 应用包装成桌面应用。这里提供一个简单的示例：

```go
ui, _ := lorca.New("", "", 480, 320)
defer ui.Close()

// Bind Go function to be available in JS. Go function may be long-running and
// blocking - in JS it's represented with a Promise.
ui.Bind("add", func(a, b int) int { return a + b })

// Call JS function from Go. Functions may be asynchronous, i.e. return promises
n := ui.Eval(`Math.random()`).Float()
fmt.Println(n)

// Call JS that calls Go and so on and so on...
m := ui.Eval(`add(2, 3)`).Int()
fmt.Println(m)

// Wait for the browser window to be closed
<-ui.Done()
```

## 制作 MacOS App

在完成基本的编码后，接下来的工作才是重点：将应用包装成一个 MacOS APP。

### 制作图标

一个 MacOS APP 首先需要一个图标，这里请选择一个 1024 X 1024 分辨率，背景透明的 PNG 图片。这里假设该图片名为 `logo.png`：

- 新建一个名为 `tmp.iconset` 的临时目录，用于存放不同大小的临时图片
- 执行如下命令，将原图转为不同大小的图片并放入临时目录

```bash
$ sips -z 16 16     logo.png --out tmp.iconset/icon_16x16.png
$ sips -z 32 32     logo.png --out tmp.iconset/icon_16x16@2x.png
$ sips -z 32 32     logo.png --out tmp.iconset/icon_32x32.png
$ sips -z 64 64     logo.png --out tmp.iconset/icon_32x32@2x.png
$ sips -z 128 128   logo.png --out tmp.iconset/icon_128x128.png
$ sips -z 256 256   logo.png --out tmp.iconset/icon_128x128@2x.png
$ sips -z 256 256   logo.png --out tmp.iconset/icon_256x256.png
$ sips -z 512 512   logo.png --out tmp.iconset/icon_256x256@2x.png
$ sips -z 512 512   logo.png --out tmp.iconset/icon_512x512.png
$ sips -z 1024 1024   logo.png --out tmp.iconset/icon_512x512@2x.png
```
- 使用 [iconutil](https://developer.apple.com/library/content/documentation/GraphicsAnimation/Conceptual/HighResolutionOSX/Optimizing/Optimizing.html#//apple_ref/doc/uid/TP40012302-CH7-SW2) 生成图标

```bash
$ iconutil -c icns tmp.iconset -o icon.icns
```

`icon.icns` 就是制作好的 MacOS App 图标。

### 制作 .app bundle

macOS 上安装的可运行程序是一个 `.app` 的目录，里面包含了应用的二进制文件、资源文件以及清单文件。其的目录结构为（也可以通过”右键-显示包内容“来查看 `.app` 文件内容）：

```bash
$ tree Kustomize.app
Kustomize.app
└── Contents
    ├── Info.plist
    ├── MacOS
    │   └── kustomize
    └── Resources
        ├── assets
        │   ├── css
        │   │   ├── page.css
        │   │   ├── prism.css
        │   │   └── weui.min.css
        │   ├── images
        │   │   └── favicon.ico
        │   └── js
        │       ├── jquery.min.js
        │       ├── prism.js
        │       └── weui.min.js
        ├── icon.icns
        └── views
            ├── copyreght.html
            ├── footer.html
            ├── header.html
            ├── index.html
            └── yaml.html
```

可以看到：

- `Info.plist` 为清单文件，存储应用信息
- `MacOS` 中存放二进制可执行文件
- `Resources` 存放静态资源文件和图标

### Info.plist 文件

这是一个清单文件，根据自己应用的内容对齐进行修改，更多内容可以参考 [trayhost](https://github.com/shurcooL/trayhost) 项目的说明。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleExecutable</key>
	<string>kustomize</string>
	<key>CFBundleIconFile</key>
	<string>icon.icns</string>
	<key>CFBundleIdentifier</key>
	<string>io.guoxudong.kustomize-remote-observer</string>
	<key>NSHighResolutionCapable</key>
  <true/>
	<key>LSUIElement</key>
  <string>1</string>
</dict>
</plist>
```

### 使用脚本构建 App

上面的这些只不过是介绍一下原理及手动修改方式，实际应用中可以使用脚本来完成这些工作。使用如下脚本，可以一键完成：

- `.app` 应用的构建
- go 应用的打包
- 清单文件的生成
- 静态资源的拷贝

```shell
#!/bin/sh

APP="Kustomize.app"
mkdir -p $APP/Contents/{MacOS,Resources}
go build -o $APP/Contents/MacOS/kustomize
cat > $APP/Contents/Info.plist << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleExecutable</key>
	<string>kustomize</string>
	<key>CFBundleIconFile</key>
	<string>icon.icns</string>
	<key>CFBundleIdentifier</key>
	<string>io.guoxudong.kustomize-remote-observer</string>
	<key>NSHighResolutionCapable</key>
  <true/>
	<key>LSUIElement</key>
  <string>1</string>
</dict>
</plist>
EOF
cp icons/icon.icns $APP/Contents/Resources/icon.icns
cp -r assets $APP/Contents/Resources/assets
cp -r views $APP/Contents/Resources/views
find $APP
```

{{% alert title="注意" color="warning" %}}
在 MacOS 中，当您运行 App bundle 时，进程的工作目录是根目录（`/`），而不是 `Contents/Resources` 目录。如果需要从 `Resources` 加载资源，则需要进行如下更改：

```go
ep, err := os.Executable()
if err != nil {
	log.Fatalln("os.Executable:", err)
}
err = os.Chdir(filepath.Join(filepath.Dir(ep), "..", "Resources"))
if err != nil {
	log.Fatalln("os.Chdir:", err)
}
```
{{% /alert %}}

### 制作 DMG 文件

DMG 文件用于分发应用程序，将 `.app` 文件压缩制成镜像，可以很方便的通过拖拽的形式完成安装。

#### 制作模板

制作 DMG 文件首先需要制作模板。打开`磁盘工具 - 文件 - 新建映象 - 空白映象`（或直接按 `⌘N`）创建一个新的磁盘镜像。给它取个名字，设置足够的空间空间，分区选择`CD/DVD`。

![新建模板](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghuwoma7w4j20bd09odig.jpg)

制作好后，打开该镜像，进行文件夹视图定制（按`⌘J`），选择展示图标的大小及背景图片，这里可以隐藏工具栏

![文件夹视图定制](https://tva3.sinaimg.cn/large/ad5fbf65gy1ghuwwafpr1j20uy0jhb29.jpg)

右键`应用程序`选择制作替身，将替身移动到镜像中

![制作替身](https://tva1.sinaimg.cn/large/ad5fbf65gy1ghuwthtnxqj20bh05xwey.jpg)

将打包好的 app 加入到 DMG 镜像中就完成了 DMG 模板的定制

![定制好的视图](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghuwzoih10j20lo0cqtof.jpg)

#### 转换 DMG 文件

目前的 DMG 模板文件还没有经过压缩并且是可写的状态，这样是不能作为程序发布的，所以这里需要对模板进行转换。

![转换](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghux3e65ngj20ct04676t.jpg)

打开 `磁盘工具 - 映象 - 转换`，然后选择压缩后存储的目录就完成了最后一步 DMG 文件的转换。

![转换成功](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghux55kyzdj20pn0fqgt6.jpg)

现在点开 DMG 文件，将应用拖动到应用程序中，就可以在启动台中看到我们的应用程序了！

![启动台](https://tvax1.sinaimg.cn/large/ad5fbf65gy1ghux7e0g5tj20hn0fq7e7.jpg)

#### 自动化

上面只是展示了如何手动制作 DMG 镜像，实际使用当然是要将这些步骤自动化的。我将这部分内容做成了一个 go 脚本，原理其实就是使用 `hdiutil` 这个命令行工具，有兴趣的同学可以文末找到项目地址，`Makefile` 中有详细构建的命令。

## 项目展示

我使用 Go + HTML5 制作了一个 `Kustomize Remote` 的项目，可以从远程 kustomize 项目中获取配置，并 build 成 yaml 文件，UI样式为微信风格，支持 public 和 private 项目。

![kustomize-remote-observer](https://tva4.sinaimg.cn/large/ad5fbf65gy1ghuxhffoe1j20dc0h8wf1.jpg)

![yaml result](https://tva1.sinaimg.cn/large/ad5fbf65gy1ghuxk4zxv2j20dc0h83zl.jpg)

{{% pageinfo color="primary" %}}
项目地址：[https://github.com/sunny0826/kustomize-remote-observer](https://github.com/sunny0826/kustomize-remote-observer)

也可以直接在 [release 页面](https://github.com/sunny0826/kustomize-remote-observer/releases) 下载 DMG 文件安装试用，只需 Mac 上有 Chrome 即可。
{{% /pageinfo %}}

## 结语

Go 语言一直在网络编程、云平台开发、分布式系统等领域占据着重要的地位，但是像桌面应用或者机器学习这样的领域，同样也能做出不错的效果。作为一门受欢迎的编程语言 Golang 已经有十多年的历史了，相信它在将来还能在更多的领域焕发生机，创造辉煌。
