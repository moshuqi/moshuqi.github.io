---
layout: post
title:  Python爬虫库-Beautiful Soup的使用
category: "Python"
---

**Beautiful Soup** 是一个可以从HTML或XML文件中提取数据的Python库，简单来说，它能将html的标签文件解析成树形结构，然后方便地获取到指定标签的对应属性。

如在上一篇文章[通过爬虫爬取漫画图片](https://moshuqi.github.io/python/2016/06/15/Python爬虫抓取漫画图片.html)，获取信息纯粹用正则表达式进行处理，这种方式即复杂，代码的可阅读性也低。通过**Beautiful Soup**库，我们可以将指定的class或id值作为参数，来直接获取到对应标签的相关数据，这样的处理方式简洁明了。

当前最新的 **Beautiful Soup** 版本为4.4.0，Beautiful Soup 3 当前已停止维护。

**Beautiful Soup 4** 可用于 python2.7 和 python3.0，本文示例使用的python版本为2.7。

博主使用的是Mac系统，直接通过命令安装库：

	sudo easy_install beautifulsoup4

安装完成后，尝试包含库运行：

	from bs4 import BeautifulSoup
	
若没有报错，则说明库已正常安装完成。

