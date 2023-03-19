---
title: 在wsl2的ubuntu20.04中安装docker
date: 2022-09-15 00:46:08
tags:
	- study
	- docker
	- wsl2
categories:
	- 技能
---

# wsl2下docker常用命令

帮助启动类命令：

* 启动docker：sudo service docker start
* 停止docker：sudo service docker stop
* 重启docker：sudo service docker restart
* 查看docker状态：sudo service docker status
* 查看概要信息：sudo docker info
* 帮助：sudo docker help

镜像命令：

* sudo docker search 某个镜像名字
* sudo docker pull 某个镜像名字
* sudo docker system df 查看镜像/容器/数据所占的空间
* sudo docker rmi -f 强制删除image

容器命令：

* 新建+容器启动：

  > sudo docker run -it ubuntu /bin/bash （i表示交互式启动，t表示终端，/bin/bash是shell）
  >
  > sudo docker run -it --name=myu1 ubuntu bash （容器的名字--name）

* 列出当前所有容器：sudo docker ps

* 退出容器：ctrl+p+q退出但不停止，exit退出停止

* 前台交互式启动：sudo docker run -d redis:6.0.8

* 查看容器日志：sudo docker log + 容器ID

* 查看容器内运行的进程：sudo docker top + 容器ID

* 查看容器内部细节：sudo docker inspect + 容器ID

* 重新进入正在后台运行的容器：sudo docker exec -it 容器ID /bin/bash

* 

