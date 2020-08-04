---
title: "Docker容器启动退出解决方案"
date: 2018-09-27T19:27:03+08:00
draft: false
type: blog
tags: ["容器", "Linux"]
categories: ["问题解决"]
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0t14jxg0jj21qq15ojvp.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "解决容器启动退出问题。"
keywords: ["容器","Linux"]
image:
  url: "https://tva4.sinaimg.cn/large/ad5fbf65ly1ge3if5zqkzj21qq15ojvp.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# 现象
启动docker容器 

```bash
docker run –name [CONTAINER_NAME] [CONTAINER_ID] 
```

查看容器运行状态 

```bash    
docker ps -a 
```

发现刚刚启动的mydocker容器已经退出

# 原因
docker容器的主线程（dockfile中CMD执行的命令）结束，容器会退出

# 解决办法
1. 可以使用交互式启动

	```bash
	docker run -i [CONTAINER_NAME or CONTAINER_ID]
	```

2. 上面的不太友好，建议使用后台模式和tty选项

	```bash
	docker run -dit [CONTAINER_NAME or CONTAINER_ID]
	```

3. Docker 容器在后台以守护态（Daemonized）形式运行，可以通过添加 -d 参数来实现

	```bash
	$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
	```

4. 在脚本最后一行添加**tail -f /dev/null**，这个命令永远完成不了，所以该脚本一直不会执行完，所以该容器永远不会退出。

>**TIPs**:退出时，使用```[ctrl + D]```，这样会结束docker当前线程，容器结束，可以使用 ```[ctrl + P]``` ```[ctrl + Q]``` 退出而不终止容器运行

>如下命令，会在指定容器中执行指定命令， ```[ctrl+D]``` 退出后不会终止容器运行

>docker默认会把容器内部pid=1的作为默认的程序
