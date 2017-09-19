---
layout:     post
title:      "Ubuntu windows双系统远程切换"
subtitle:   "双系统远程切换"
date:       2017-09-19 20:13:00
author:     "slade"
header-img: "img/post-bg-004.jpg"
catalog: true
tags:
    - ubuntu
    - 杂
---

平时在宿舍或者在家需要用到实验室的电脑，远程的时候切换系统是个麻烦的事情。还要担心实验室断电之后，电脑关机了。没人帮忙开机。所以有了此文。


# 远程唤醒电脑
如果你的主板支持定时开机，那么这个问题就比较简单。以笔者的Z170主板举例，开机按F2进入BIOS设置，选择Power management。开启resume by alarm功能。自行设定时间即可。如果主板不支持，还有一个方法使用网卡的网络唤醒功能，但是前提是你的电脑需要接着网线连接到路由器上，并且路由器要有一个广域网的固定ip供你访问。如果上述条件都满足，那就可以通过在家访问路由器，从而唤醒你的电脑。
------
# 定时关机
Windows运行时间过程容易出现各种问题，因此在没有跑程序的时候，每天半夜自动重启一次。具体做法如下：
控制面板->系统和安全->管理工具->计划任务->操作->创建基本任务->名称描述自己定->触发器每天或自定->选择启动程序->选择C:\Windows\System32\shutdown.exe->添加参数 -s
------
# 双系统切换
笔者的电脑使用GRUB引导双系统，开机的时候会停留在GRUB页面等待选择系统。由于人不在电脑前面，所以切换系统就很麻烦。废话不多说，方法如下：
- 首先修改GRUB设置，修改默认启动项
```shell
sudo gedit /etc/default/grub 
```
- 然后将GRUB_DEFAULT这一项修改为saved，接着保存更新
```shell
sudo update-grub 
```
- 紧接着设置默认启动项，设置成你常用的那个系统
```shell
sudo grub-set-default 0
```
笔者常用的Ubuntu, 其对应的启动项是第0个。这个顺序可以在开机时候的GRUB页面上看到，从上到下从0开始编号就行，笔者的Ubuntu在第一行，所以序号是0
- 这样就设置好了默认系统。平时自动开机之后将会进入Ubuntu系统。这时候如果想进windows怎么办？还是首先先进Ubuntu，进入之后打开Terminal。输入
```shell
sudo grub-reboot 2
sudo reboot
```
- 这个命令仅在本次重启有效，所以重启之后会进入Windows。但是下次再重启还是会进默认的Ubuntu。
------
