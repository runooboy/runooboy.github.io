---
title: 安装配置TensorFLow
date: 2019-07-01 13:42:57
category:
- 人工智能
---
本文目前主要是记录windows平台下安装配置TensorFLow的流程，mac和linux暂时还没有尝试安装 - -！
<!-- more -->

### 安装Anaconda和PyCharm
这两个软件不必多说，在官网找到并下载并安装即可，安装路径是可以自己选择的，另外Anaconda带的python环境路径可以配置到Path中，那下次可以直接在cmd中使用python。

### 安装配置TensorFlow
接下来就进入正文了，此文目前只介绍cpu版的TensorFlow安装配置：首先在[pypi](https://pypi.org/project/tensorflow/#files)上下载所需的TensorFLow文件。

下载好了后将TensorFLow文件放入Anaconda下的Script文件夹下，使用pip install命令安装TensorFlow：

``` bash
$ pip install tensorflow-1.13.1-cp37-cp37m-win_amd64.whl(自己下载好的文件)
```

这时TensorFlow环境就已经差不多配置好了，只需要在PyCharm中设置Interpreter路径为Anaconda下的python.exe即可。
