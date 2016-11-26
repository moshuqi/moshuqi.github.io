---
layout: post
title:  Python爬虫-用Scrapy框架实现漫画的爬取
category: "Python"
---


在之前一篇[抓取漫画图片的文章](https://moshuqi.github.io/2016/06/15/Python爬虫抓取漫画图片/)里，通过实现一个简单的Python程序，遍历所有漫画的url，对请求所返回的html源码进行正则表达式分析，来提取到需要的数据。

本篇文章，通过 **scrapy** 框架来实现相同的功能。**scrapy** 是一个为了爬取网站数据，提取结构性数据而编写的应用框架。关于框架使用的更多详情可浏览[官方文档](https://doc.scrapy.org/en/1.2/)，本篇文章展示的是爬取漫画图片的大体实现过程。


## scrapy环境配置

### 安装

首先是 **scrapy** 的安装，博主用的是Mac系统，直接运行命令行：

	pip install Scrapy
	
对于html节点信息的提取使用了 **Beautiful Soup** 库，大概的用法可见[之前的一篇文章](https://moshuqi.github.io/2016/07/20/Python%E7%88%AC%E8%99%AB%E5%BA%93-BeautifulSoup/)，直接通过命令安装：

	pip install beautifulsoup4
	
对于目标网页的 **Beautiful Soup** 对象初始化需要用到 **html5lib** 解释器，安装的命令：

	pip install html5lib
	
安装完成后，直接在命令行运行命令：

	scrapy

可以看到如下输出结果，这时候证明scrapy安装完成了。

	Scrapy 1.2.1 - no active project

	Usage:
	  scrapy <command> [options] [args]
	
	Available commands:
	  bench         Run quick benchmark test
	  commands      
	  fetch         Fetch a URL using the Scrapy downloader
	  genspider     Generate new spider using pre-defined templates
	  runspider     Run a self-contained spider (without creating a project)
	  settings      Get settings values
	  ...
	
### 项目创建

通过命令行在当前路径下创建一个名为 **Comics** 的项目

	scrapy startproject Comics
	
创建完成后，当前目录下出现对应的项目文件夹，可以看到 **Comics** 文件结构为：

	|____Comics
	| |______init__.py
	| |______pycache__
	| |____items.py
	| |____pipelines.py
	| |____settings.py
	| |____spiders
	| | |______init__.py
	| | |______pycache__
	|____scrapy.cfg
	
Ps. 打印当前文件结构命令为： 

	find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'

### 运行

## 爬取漫画图片

### 起始地址

### 爬取漫画url

#### 当前页漫画列表

#### 下一页列表

### 爬取漫画图片

#### 当前页图片

#### 下一页图片