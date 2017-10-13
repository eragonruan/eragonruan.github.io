---
layout:     post
title:      "GTX1070 tensorflow配置"
subtitle:   " \"ubuntu14.04+gtx1070\""
description: "ubuntu14.04 gtx1070 gtx1080 tensorflow deep learning install 配置 安装"
date:       2017-05-04 21:06:00
author:     "slade"
header-img: "img/post-bg-002.jpg"
catalog: true
tags:
    - ubuntu
    - tensorflow
---

# 准备工作

- 首先下载英伟达驱动，[下载地址](http://www.geforce.cn/drivers)。根据自己的系统以及显卡型号选择驱动，笔者下载的是NVIDIA-Linux-x86_64-367.57.run
- 下载CUDA，[下载地址](https://developer.nvidia.com/cuda-toolkit)。笔者选择的是CUDA8.0，**目前1070和1080好像只支持8.0**。其他显卡的也可以选择CUDA7.5.
- 下载cudnn，[下载地址](https://developer.nvidia.com/cudnn)。要填写一个小问卷，很快的。根据自己选择的CUDA版本选择cudnn版本，笔者选择的是cudnn v5.1 for CUDA8.0
- 下载tensorflow，[下载地址](https://github.com/tensorflow/tensorflow)。这里我们选择安装编译好的二进制包，这是最简单的方法。在下载页面如下图位置，选择Linux-GPU：Python 3.5
![img1](http://slade-ruan.me/img/in-post/install_tensorflow/in-post-0504-01.png)

------

# 安装显卡驱动

如果是新安装的Ubuntu系统，那么直接安装按以下步骤安装即可。如果曾经装过显卡驱动，请先卸载旧版驱动。
- `Ctrl+Alt+F1`进tty，如果黑屏可以修改grub文件。具体做做法如下
```shell
sudo gedit /etc/default/grub    #用gedit打开grub文件
```
将GRUB_CMDLINE_LINUX_DEFAULT的值修改为 **quiet splash nomodeset**
```shell
sudo update-grub #更新grub
```
重启电脑(可能会发现系统分辨率变了，不要紧，安装好驱动就会恢复正常)

- 进入tty之后，输入用户名和密码。

- 
```shell
sudo service lightdm stop    #关闭图形界面
cd DL    #切换到你放驱动的文件夹，笔者的在DL文件夹
sudo chmod 755 NVIDIA-Linux-x86_64-367.57.run    #获取权限，根据自己驱动名修改
sudo ./NVIDIA-Linux-x86_64-367.57.run    #如果提示pre-install失败可以忽略
sudo service lightdm start    #开启图形界面
```

- 测试一下，`Ctrl+Alt+t`打开terminal
```shell
cat /proc/driver/nvidia/version
```
如果出现下面的界面
![img2](http://slade-ruan.me/img/in-post/install_tensorflow/in-post-0504-02.png)
那么说明驱动安装成功了，可以看到笔者安装的版本是367.57

------

# 安装CUDA

- 首先安装相关的依赖库
```shell
sudo apt-get install freeglut3-dev
sudo apt-get install build-essential
sudo apt-get install libx11-dev
sudo apt-get install libxmu-dev
sudo apt-get install libxi-dev
sudo apt-get install libglu1-mesa
sudo apt-get install libglu1-mesa-dev
```
         
- 切换到放cuda安装包的文件夹
```shell
sudo sh cuda_8.0.44_linux.run --override
```

- 这时候会出现一大堆协议，一直按space就可以，最后accept

- 需要特别注意：安装过程中问你要不要Install NVIDIA　～～　drive一定要`选ｎ`，不然之前装的显卡驱动就白装了。这里主要是因为cuda提示安装的驱动版本比较低。

- 其他都选yes，默认文件夹不用去改它。

- 安装完毕之后设置一下环境变量。
将下面两行写到~/.bashrc文件中
```shell
sudo gedit ~/.bashrc
```
```shell
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda8.0-/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

- 测试一下，切换到cuda_sample文件夹
```shell
cd ～/NVIDIA_CUDA-8.0_Samples    #切换到sample文件夹
make    #编译
cd bin/x86_64/linux/release    #切换到release目录
./deviceQuery    #运行一下实例
```
如果出现下面的图，说明安装成功了。
![img3](http://slade-ruan.me/img/in-post/install_tensorflow/in-post-0504-03.png)

------

# 安装Cudnn

- 
```shell
cd  DL    #切换目录，根据实际情况更改
tar xvzf cudnn-8.0-linux-x64-v5.1.tgz    #解压
sudo cp cuda/include/cudnn.h /usr/local/cuda/include    #复制头文件
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```
这样子就可以了。

------

# 安装Anaconda

anaconda集成了python的100多个库，比较方便。
先去官网[下载](https://www.continuum.io/downloads),下载好后，切换到下载目录
```shell
bash Anaconda3-4.2.0-Linux-x86_64.sh
```
中间会问你要不要修改环境变量，选y
安装完成记得执行一下 source ~/.bashrc 才会生效
测试一下，在命令行输入python，出现下图说明安装正确。
![img4](http://slade-ruan.me/img/in-post/install_tensorflow/in-post-0504-04.png)

------

# 安装tensorflow

- 首先执行下面两句，开启GPU加速
```shell
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
export CUDA_HOME=/usr/local/cuda
```

- 接着直接pip安装即可
```shell
pip install tensorflow-0.11.0rc1-cp35-cp35m-linux_x86_64.whl
```
这就安装好了。你没看错。如果是自己折腾从源码编译不要太麻烦

- 测试一下
```shell
cd anaconda3/lib/python3.5/site-packages/tensorflow/models/image/mnist
python convolutional.py
```
![img5](http://slade-ruan.me/img/in-post/install_tensorflow/in-post-0504-05.png)
5.4ms,比CPU不要快太多。就这么愉快的结束了。真正的学习过程才刚刚开始~

------
