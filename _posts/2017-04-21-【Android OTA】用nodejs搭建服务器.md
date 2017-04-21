---
layout: post
title:  【Android OTA】用nodejs搭建服务器
category: "Android"
---

OTA，Over-the-Air的简写，OTA升级就是通过GPRS、3G、无线网络下载升级补丁升级，不用通过有线连接来升级。

Android的应用或是系统，可以通过这种方式进行版本的更新升级。OTA具体原理自行google，或者参考[这篇文章](https://source.android.com/devices/tech/ota/)。本文和接下来的两篇文章主要介绍的是具体的实现过程。


###OTA升级大致过程

1.	设备向服务器进行版本更新检测请求
2.	服务器将更新信息返回到设备端
3.	设备通过返回信息下载指定的更新文件
4.	下载完成后设备安装升级

Android单个应用和整个系统的升级方式存在差异，接下来两篇文章会分别介绍实现。

本文重点实现升级过程中的第二步，搭建一个服务器Demo，以方便后续的测试工作。服务器用的nodejs，使用较为简单，没接触过的也可以跟着下面的步骤将服务器搭建在本地运行起来。

###安装nodejs

用的是Mac系统，安装命令

	brew install node
	
查看是否安装完成

	$ node -v
	v6.2.0