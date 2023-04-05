---
title: Atlas踩坑记之调用摄像头实现yolov5(pytorch)模型推理
abbrlink: e3e8256a
date: 2023-04-05 20:48:03
tags:
  - study
categories:
  - 技能
---
# Atlas踩坑记之调用摄像头实现yolov5(pytorch)模型推理
## 0.写在前面
&emsp;上篇完成了Atlas200DK环境搭建已经成功一半了！接下来当然是要开发自己的应用啦。本次项目是要完成在Atlas上调用摄像头读取图片并用yolov5模型推理，实现目标检测。
## 1.硬件准备
&emsp;参考开发者套件的[硬件安装部分](https://www.hiascend.com/document/detail/zh/Atlas200DKDeveloperKit/1013/environment/atlased_04_0006.html)，要注意查看自己的主板型号，在板子上有刻印(暂时不知道怎么用命令行查找)。如果是IT21DMDA的主板需要自己买一条白色的树梅派摄像头排线，是IT21VDMB主板的话应该开发者套件里面会有两条黑色的线。然后再准备一个树梅派摄像头，我用的是Raspberry pi cammera V2。
## 2.样例部署-用yolov5模型推理图片
&emsp;在Ascend仓库的ModelZoo中有pytorch框架的yolov5[样例demo](https://gitee.com/ascend/modelzoo-GPL/tree/master/built-in/ACL_Pytorch/Yolov5_for_Pytorch),先根据此链接的readme完成基本功能，注意要安装第三方依赖库，注意pytorch版本尽量为1.10.1与其一致。
&emsp;这里遇到最多的坑就是没有onnx相关库，在将pytorch的.pt文件转化为.onnx模型时会出现各种报错，如下图所示。这些报错基本都是因为没有onnx相关库，千万不要以为是torch和cann版本不对应等其他原因去折腾torch的版本(没错我就是这么干的，还去cann文档找对应torch版本然后还编译安装，后来发现好像对应版本是在Ascend设备上训练pytorch模型用的，离线推理并不需要对应版本)。
![图片1](/pic/2023-03-30_01-50.png)
![图片2](/pic/2023-03-30_16-42.png)
![图片3](/pic/2023-03-31_23-17.png)
![图片4](/pic/2023-03-31_23-17_1.png)
&emsp;解决方法就是无论如何都要把onnx、onnxsim、onnx-simplifier都装上。
&emsp;最终我们需要能够完成在Ascend处理器上用yolov5推理图片并保存结果到.json文件里。
## 3.样例部署-调用摄像头
&emsp;这里需要用到华为sample仓里面的[人脸检测样例](https://gitee.com/ascend/samples/tree/master/python/level2_simple_inference/2_object_detection/object_detection_camera)。这是使用摄像头推理一个基于caffe的yolov3模型。这里基本没有什么坑，很顺利就能在电脑上看到你的人脸被检测出来了。
## 4.样例部署-用yolov5模型推理摄像头
&emsp;虽然我们已经实现了调用摄像头推理人脸识别的yolov3模型，以及用yolov5推理图片，但是仔细阅读这两个样例的代码后发现，这两套使用的python库有所不同，前者是调用的摄像头和推理的接口是在这个路径下
```ubuntu
PYTHONPATH=${THIRDPART_PATH}/acllite:$PYTHONPATH #设置pythonpath为固定目录
```
而后者的推理则是调用了CANN安装目录下的库
```ubuntu
INSTALL_DIR=${HOME}/Ascend/ascend-toolkit/latest
```
&emsp;且两者在图像预处理上有所不同，使得直接将样例2的模型放到样例3的推理代码中无法运行。
### 4.1我的解决思路：
&emsp;由于之前对图像格式等知识的欠缺，导致这里进度卡了很久。如果对图像有所了解应该能较快解决这个问题，首先我们需要学习ATC模型转换工具，他能帮助我们改变模型所需的输入格式和大小。
&emsp;在样例3的模型转换中，用到了.cfg文件该文件内容如下：
```ubuntu
aipp_op{
aipp_mode:static
crop:false
rbuv_swap_switch:true
input_format : YUV420SP_U8
load_start_pos_h : 0
load_start_pos_w : 0
src_image_size_w : 416
src_image_size_h : 416

csc_switch : true

matrix_r0c0 : 298
matrix_r0c1 : 516
matrix_r0c2 : 0
matrix_r1c0 : 298
matrix_r1c1 : -100
matrix_r1c2 : -208
matrix_r2c0 : 298
matrix_r2c1 : 0
matrix_r2c2 : 409
input_bias_0 : 16
input_bias_1 : 128
input_bias_2 : 128
mean_chn_0 : 0
mean_chn_1 : 0
mean_chn_2 : 0
min_chn_0 : 0.0
min_chn_1 : 0.0
min_chn_2 : 0.0

var_reci_chn_0 :0.003921568627451
var_reci_chn_1 :0.003921568627451
var_reci_chn_2 :0.003921568627451
}

```
&emsp;仔细阅读CANN文档后能够理解其含义。首先将整体模型的输入变成YUV420SP_U8格式，然后通过硬件将其转化为rbg格式再输入原模型。就是将原来需要rbg格式输入的yolov5.pt模型，转化成输入的格式为YUV420SP_U8的.om模型。
&emsp;那么YUV420SP_U8格式哪里来呢？官方给我们提供了dvpp工具，其输出就为YUV类型的图像，这样就能实现将摄像头读取的图像经过dvpp预处理后放入模型进行推理。
后续的后处理很容易能够参考样例2自行写出，主要功能就是指出模型结果对应的类别id。

