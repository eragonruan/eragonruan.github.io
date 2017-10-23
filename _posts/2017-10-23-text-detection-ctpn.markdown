---
layout:     post
title:      "论文阅读与实现--CTPN"
subtitle:   "detecting text in natural image with connectionist text proposal network"
description:	"场景文本检测，ctpn，detecting text in natural image with connectionist text proposal network，scene text detection"
date:       2017-10-22 14:00:00
author:     "slade"
header-img: "img/post-bg-004.jpg"
catalog: true
tags:
    - paper
    - tensorflow
    - deep learning
---

# 简介

本文将对CTPN这篇文章的思路做一个详细的介绍，同时对代码进行解读。  
论文地址：[arxiv](https://arxiv.org/pdf/1609.03605.pdf)  
作者github地址：[github](https://github.com/tianzhi0549/CTPN)  
tensorflow版本地址：[tensorflow](https://github.com/eragonruan/text-detection-ctpn)  
作者提供的版本使用的caffe，没有提供训练的代码，但是有一个online的demo  

------

# 论文的关键idea

- 文本检测的其中一个难点就在于文本行的长度变化是非常剧烈的。因此如果是采用基于faster rcnn等通用物体检测框架的算法都会面临一个问题？怎么生成好的text proposal？这个问题实际上是比较难解决的。因此在这篇文章中作者提供了另外一个思路，检测一个一个小的，固定宽度的文本段，然后再后处理部分再将这些小的文本段连接起来，得到文本行。检测到的文本段的示意图如下图所示。  
![img1](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-01.png)
- 具体的说，作者的基本想法就是去预测文本的竖直方向上的位置，水平方向的位置不预测。因此作者提出了一个vertical anchor的方法。与faster rcnn中的anchor类似，但是不同的是，vertical anchor的宽度都是固定好的了，论文中的大小是16个像素。而高度则从11像素到273像素变化，总共10个anchor.  
- 同时，对于水平的文本行，其中的每一个文本段之间都是有联系的，因此作者采用了CNN+RNN的一种网络结构，检测结果更加鲁棒。


------

# pipeline

整个算法的流程主要有以下几个步骤：（参见下图）  
- 首先，使用VGG16作为base net提取特征，得到conv5_3的特征作为feature map，大小是W×H×C
- 然后在这个feature map上做滑窗，窗口大小是3×3。也就是每个窗口都能得到一个长度为3×3×C的特征向量。这个特征向量将用来预测和10个anchor之间的偏移距离，也就是说每一个窗口中心都会预测出10个text propsoal。
- 将上一步得到的特征输入到一个双向的LSTM中，得到长度为W×256的输出，然后接一个512的全连接层，准备输出。
- 输出层部分主要有三个输出。2k个vertical coordinate，因为一个anchor用的是中心位置的高（y坐标）和矩形框的高度两个值表示的，所以一个用2k个输出。（注意这里输出的是相对anchor的偏移）。2k个score，因为预测了k个text proposal，所以有2k个分数，text和non-text各有一个分数。k个side-refinement，这部分主要是用来精修文本行的两个端点的，表示的是每个proposal的水平平移量。
- 这是会得到密集预测的text proposal，所以会使用一个标准的非极大值抑制算法来滤除多余的box。
- 最后使用基于图的文本行构造算法，将得到的一个一个的文本段合并成文本行。
![img2](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-02.png)

------

# 一些细节

## vertical anchor

- k个anchor的设置如下：宽度都是16像素，高度从11~273像素变化（每次乘以1.4）
- 预测的k个vertical coordinate的坐标如下：
![img3](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-03.png)
- 与真值IoU大于0.7的anchor作为正样本，与真值IoU最大的那个anchor也定义为正样本，这个时候不考虑IoU大小有没有到0.7，这样做有助于检测出小文本。
- 与真值IoU小于0.5的anchor定义为负样本。
- 只保留score大于0.7的proposal

## BLSTM
- 文章使用了双向的LSTM，每个LSTM有128个隐层
- 加了RNN之后，整个检测将更加鲁棒，具体效果见下图。上图是加了RNN，下图是没有RNN的结果。
![img4](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-04.png)

## 训练
- 对于每一张训练图片，总共抽取128个样本，64正64负，如果正样本不够就用负样本补齐。这个和faster rcnn的做法是一样的。
- 训练图片都将短边放缩到600像素。

# 代码注解
reimplement的github[地址](https://github.com/eragonruan/text-detection-ctpn)

## 准备训练数据
- 正如上文所提的，这个网络预测的是一些固定宽度的text proposal，所以真值也应该按照这样来标注。但是一般数据库给的都是整个文本行或者单词级别的标注。因此需要把这些标注转换成一系列固定宽度的box。代码在prepare_training_data这个文件夹。
- 整个repo是基于RBG大神的faster rcnn改的，所以根据他的输入要求。要再将数据转换为voc的标注形式，这部分代码也在prepare_training_data这个文件夹。


## 一些预测结果
- 这个repo没有实现文章中提到的side refinement部分。
- 以身份证检测做了个例子，但是模型是可以应用到各种水平文本检测的应用中的。
- 部分检测的结果如下所示：
![img5](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-05.jpg)
![img6](http://slade-ruan.me/img/in-post/ctpn/in-post-1022-06.jpg)

