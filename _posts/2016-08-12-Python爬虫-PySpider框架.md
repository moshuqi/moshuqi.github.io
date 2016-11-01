---
layout: post
title:  Python爬虫-pyspider框架
category: "Python"
---

**pyspider** 是一个用python实现的功能强大的网络爬虫系统，能在浏览器界面上进行脚本的编写，功能的调度和爬取结果的实时查看，后端使用常用的数据库进行爬取结果的存储，还能定时设置任务与任务优先级等。

本篇文章只是对这个框架使用的大体介绍，更多详细信息可见[官方文档](http://docs.pyspider.org/en/latest/)。

<h3>安装</h3>

首先是环境的搭建，网上推荐的各种安装命令，如：

	pip install pyspider
	
但是因为各种权限的问题，博主安装报错了，于是采用了更为简单粗暴的方式，直接把源码下下来run。

**pyspider**的[源码地址](https://github.com/binux/pyspider)，直接download或者git clone都行，下载完成后，进入文件夹目录。

系统默认用的Python是2.7版本，自己另外装了个3.4的，源码用python3跑起来。

先进行安装，在pyspider的路径下敲命令：

	python3 setup.py install
	
一堆的打印，完了之后没什么错误提示就是安装完成了。

接下来跑起来：

	python3 run.py 
	
运行结果如下图所示

![image1]({{ site.url }}/assets/pysp/1.jpg)

可以看到webui运行在**5000**端口处，在浏览器打开[127.0.0.1:5000](http://127.0.0.1:5000)或者[localhost:5000](http://localhost:5000)，便能看到框架的UI界面，如下图

![image1]({{ site.url }}/assets/pysp/2.jpg)

这样pyspider就算是跑起来了。有的文章会提到需要安装**phantomjs**，这个暂时用不上，先忽略。


<h3>开始</h3>

拿这个网页来做例子：[www.reeoo.com](http://www.reeoo.com)，爬取上面的数据。

![image1]({{ site.url }}/assets/pysp/3.jpg)





