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
# 在wsl2的ubuntu20.04中docker安装

## 1.概念

* 镜像文件

  >image

* 容器实例

  > 一个容器运行一种服务，当我们需要时就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

* 仓库

  > 存放一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候再从仓库中拉下来

## 2.在wsl2的ubuntu 20.04上安装docker

* 因为wsl2已经完整使用了linux内核了，此种方式和先前在linux安装docker类似，步骤如下：

> curl -fsSL https://get.docker.com -o get-docker.sh
> sudo sh get-docker.sh
> sudo service docker start

**注意**：不同于完全linux虚拟机方式，WLS2下通过apt install docker-ce命令安装的docker无法启动，因为WSL2方式的ubuntu里面没有systemd。上述官方get-docker.sh安装的docker，dockerd进程是用ubuntu传统的init方式而非systemd启动的。

* 安装完成后需要配置国内镜像加速及日志

> sudo vi /etc/docker/daemon.json
>
> {
>   "registry-mirrors": [
>       "https://registry.docker-cn.com",
>       "http://hub-mirror.c.163.com",
>       "https://docker.mirrors.ustc.edu.cn"
>   ],
>   "log-driver":"json-file",
>   "log-opts": {"max-size":"500m", "max-file":"3"}
> }

## 3.docker服务的systemd指令和sysvinit指令

wsl2 没有systemd指令，而是sysvinit指令，双方对照表为：

|          Systemd 命令           | Sysvinit 命令                |
| :-----------------------------: | ---------------------------- |
|  systemctl start service_name   | service service_name start   |
|   systemctl stop service_name   | service service_name stop    |
| systemctl restart service_name  | service service_name restart |
|  systemctl status service_name  | service service_name status  |
|  systemctl enable service_name  | chkconfig service_name on    |
| systemctl disnable service_name | chkconfig service_name off   |

## 4.测试搜索镜像运行容器

>sudo service docker start
>sudo docker search hello-world
>sudo docker run hello-world

**注意**：直接运行sudo docker run hello-world会出现如下报错

> Unable to find image 'hello-world:latest' locally

此时不要慌，由于我们配置了国内镜像加速，会自动在远程库中拉取此镜像。

当你看到

>Hello from Docker!
>This message shows that your installation appears to be working correctly.

恭喜你已经成功在wsl2的ubuntu20.04中安装了docker！！！

参考文献：

> https://blog.csdn.net/gongdiwudu/article/details/119048832
>
> https://www.cnblogs.com/airscrat/p/15428950.html