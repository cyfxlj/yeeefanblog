---
title: Atlas200DK踩坑记之分设环境搭建
tags:
  - study
categories:
  - 技能
abbrlink: e08621db
date: 2023-03-19 09:50:36
---
# Atlas200DK踩坑记之分设环境搭建
## 0.写在前面
&emsp;本篇内容主要是配置Atlas200DK开发和运行分设环境（在主机ubuntu18.04中搭建MindStudio和cann开发环境，在atals200DK开发者套件中安装运行环境）。注意分设环境是有别与合设环境的，请各位小伙伴明确自己的开发方式再进行环境搭建。
&emsp;最近在做华为开发板Atlas200DK的一些项目工作，在搭建开发环境和运行环境时，踩了许多坑，前前后后重装系统3次，这里仅以此篇，记录我的踩坑记，避免下次出现意外忘记如何操作，也给大家在开发时带来一些参考。
## 1.ubuntu18.04系统安装
&emsp;首先我们为什么要选用ubuntu18.04系统，是因为在[cann的用户手册](https://www.hiascend.com/document/detail/zh/mindstudio/50RC3/releasenote/releasenote_000001.html)，CANN软件包支持的OS清单中Atlas200仅支持ubuntu18.04和ubuntu16.04版本，因此为了避免后续产生不必要的麻烦选用ubuntu18.04。
&emsp;由于我的电脑的显卡是NVIDIA的，在安装系统时发现直接选择Install ubuntu将会出现屏幕显示混乱或者进不了安装页面的现象，这时我们要在选择Install ubuntu界面选择到此选项，并点击'e'，进入edit mode，找到'quiet splash ---'将'---'换成'nomodeset'。不难看出这应该是由于系统在选择显卡上出现了一些问题。
&emsp;在安装好系统后，第一次重启仍然会出现上述问题，重复上一步骤进入系统，再在终端下执行下面的命令：
```ubuntu
sudo gedit /etc/default/grub
//找到下面内容
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
//将其替换为如下内容
GRUB_CMDLINE_LINUX_DEFAULT="nouveau.modeset=0"
//执行变更
sudo update-grub
```
&emsp;然后我们需要注意网上许多安装ubuntu的教程中分配空间时会将根目录/的空间分配的很小，但是我们今后在ubuntu上所装的软件，不出意外大部分都是装在根目录下的，因此！！！墙裂建议将根目录/的空间尽可能分配大一些。
## 2.配置开发环境
&emsp;由于本次教程是分设环境的搭建，因此我们将在pc机上安装开发环境。
### 2.1下载MindStudio和CANN对应版本安装包
&emsp;版本的对应关系非常重要，[MindStudio的用户手册](https://www.hiascend.com/document/detail/zh/mindstudio/50RC3/releasenote/releasenote_000001.html)上有个版本对应关系，我们要确定好MindStudio和CANN的版本，我这里选择的是MindStudio5.0.RC3，CANN6.0.RC1。直接点击进入下载。这里我们要在Ubuntu下下载，因此选择.tar.gz文件
![图片](/pic/Snipaste_2023-03-20_10-53-44.png)
接着下载CANN6.0.RC1的arrch64架构的tookit和nnrt包。tookie是要在MindStudio上导入的开发环境工具包，而nnrt包是要安装在开发板上的运行环境包。
![图片2](/pic/Snipaste_2023-03-20_11-01-27.png)
然后下载X86_64架构的tookit包
![图片3](/pic/Snipaste_2023-03-20_11-42-05.png)
## 2.2安装开发环境
为了方便省事我使用root用户安装在根目录下面，这就是为什么我强烈推荐多分配点空间给根目录。
1. 首先安装MindStudio，cd到安装包目录，打开root用户
```ubuntu
su - root
//如果第一次进入root需要设置密码
sudo passwd
//然后输入三次设置密码
//安装依赖环境
sudo apt-get install -y gcc g++ make cmake zlib1g-dev libbz2-dev libsqlite3-dev libssl-dev libffi-dev unzip pciutils net-tools libblas-dev gfortran libblas3 liblapack-dev openssh-server xterm firefox xdg-utils libdbus-glib-1-dev gdb
```
2. 安装python3.7.5
```ubuntu
wget https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz
//解压
tar -zxvf Python-3.7.5.tgz
//进入解压后的文件夹，执行配置、编译和安装命令：
cd Python-3.7.5
./configure --prefix=/usr/local/python3.7.5 --enable-loadable-sqlite-extensions --enable-shared
make
sudo make install
```
3. 设置python环境
输入命令
```ubuntu
sudo gedit ~/.bashrc
```
在最后一行将下面内容填入
```ubuntu
#用于设置Python3.7.5库文件路径
export LD_LIBRARY_PATH=/usr/local/python3.7.5/lib:$LD_LIBRARY_PATH
#如果用户环境存在多个Python3版本，则指定使用python3.7.5版本
export PATH=/usr/local/python3.7.5/bin:$PATH
```
完成之后检查一下版本python3 --version
安装如下python依赖：
```ubuntu
pip3 install attrs
pip3 install numpy
pip3 install decorator
pip3 install sympy
pip3 install cffi
pip3 install pyyaml
pip3 install pathlib2
pip3 install psutil
pip3 install protobuf
pip3 install scipy
pip3 install requests
pip3 install grpcio
pip3 install pylint
pip3 install absl-py
```
4. 安装MindStudio
```ubuntu
//{version}为自己对应的软件包版本
tar -zxvf MindStudio_{version}_linux.tar.gz
cd MindStudio/bin
./MindStudio.sh
```
5. 安装CANN
cd到下载CANN安装包得目录下面
```ubuntu
chmod +x 软件包名.run
./软件包名.run --install
```
&emsp;配置环境变量
```ubuntu
vi ~/.bashrc
source /usr/local/Ascend/ascend-toolkit/set_env.sh
source ~/.bashrc
```
&emsp;配置交叉编译环境，搭建分设环境时，用于编译的是X86架构的pc机，而用于运行的则是arm64架构的昇腾处理器。
```ubuntu
sudo apt-get install g++-aarch64-linux-gnu
```
至此开发环境搭建完毕。

&emsp;写到一半突然感觉自己写的太罗嗦了，这些其实在用户手册中都有，再结合[这个博主的文章](https://zhanghui-china.blog.csdn.net/article/details/124611486)就可以了，那就再浅浅介绍记录一下自己踩的一些坑吧。
&emsp;首先下载[官方例程](https://gitee.com/ascend/samples.git)时千万要注意不要像前面安装环境那样装在“/home/user/下载” 这个目录里面，不能出现中文，不然后面模型转换时无法加载模型文件。
&emsp;然后我们在运行例程时可能会需要一些其他的python包，我们打开终端用root用户装上即可。
&emsp;在最后我运行推理程序的时候出现了下面的错误
![图片4](/pic/QQ%E5%9B%BE%E7%89%8720230320123816.jpg)
可以看到在MindStudio中发送的远程命令为
```
cd /home/HwHiAiUser/.../out
/bin/bash ./main
```
为什么是/bin/bash ./main？而不是直接./main？
对此我目前只能在终端中ssh远程连接然后手动cd到main的目录再执行./main。
# 3.配置运行环境
&emsp;配置运行环境包括了制卡、网络配置、安装nnrt等操作，这里仅提供官方文档。
https://www.hiascend.com/document/detail/zh/Atlas200DKDeveloperKit/1013/environment/atlased_04_0003.html
&emsp;但是在装完测试demo时，可能会有如下报错：
![图片5](/pic/QQ图片20230327192629.jpg)
&emsp;我们参考这篇文章就可以完美解决了:[https://bbs.huaweicloud.com/blogs/344623](https://bbs.huaweicloud.com/blogs/344623)
# 4.合设环境搭建
&emsp;鉴于在MindStudio上开发具有一定的难度和门槛，这里还是介绍一下合设环境搭建。我们在配置运行环境时已经安装了nnrt。还需要在运行环境中安装CANN。这里我遇到的一个坑就是arm64架构的开发板中的换源问题。
&emsp;ubuntu有两种软件源：archive 源和 ports 源是根本完全不一样的，其涉及到处理 器的架构。如果两个源混用的话，会造成一些不可描述的错误和BUG。
&emsp;软件源[http://archive.ubuntu.com](http://archive.ubuntu.com)收录的架构为 AMD64 (x86_64) 和 Intel x86
&emsp;软件源[http://ports.ubuntu.com](http://ports.ubuntu.com)收录的架构为 arm64，armhf，PowerPC，ppc64el 和 s390x。
&emsp;因此我们需要用ports的软件源，我这里是从Ascend官方技术交流群的一个大佬那边copy过来的源：
```ubuntu
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
```
&emsp;记得最后要update。
&emsp;之后我们就可以随心所欲下载你想要的包了，包括opencv等。
