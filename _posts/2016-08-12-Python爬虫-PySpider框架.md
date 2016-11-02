---
layout: post
title:  Python爬虫-pyspider框架的使用
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

<h4>新建任务</h4>

第一次跑起来的时候因为没有任务，界面的列表为空，右边有个**Create**按钮，点击新建任务。

![image1]({{ site.url }}/assets/pysp/4.jpg)

+	Project Name：任务的名字，可以任意填
+	Start URL(s)：爬取任务开始的地址，这里我们填目标网址的url

填写完成后，点击**Create**，便创建成功并跳转到了另一个界面，如下图所示

![image1]({{ site.url }}/assets/pysp/5.jpg)

界面右边区域自动生成了初始默认的代码：

	#!/usr/bin/env python
	# -*- encoding: utf-8 -*-
	# Created on 2016-11-02 09:27:35
	# Project: reo
	
	from pyspider.libs.base_handler import *
	
	
	class Handler(BaseHandler):
	    crawl_config = {
	    }
	
	    @every(minutes=24 * 60)
	    def on_start(self):
	        self.crawl('http://www.reeoo.com', callback=self.index_page)
	
	    @config(age=10 * 24 * 60 * 60)
	    def index_page(self, response):
	        for each in response.doc('a[href^="http"]').items():
	            self.crawl(each.attr.href, callback=self.detail_page)
	
	    @config(priority=2)
	    def detail_page(self, response):
	        return {
	            "url": response.url,
	            "title": response.doc('title').text(),
	        }


00


	#!/usr/bin/env python
	# -*- encoding: utf-8 -*-
	# Created on 2016-10-25 09:30:00
	# Project: test
	
	from pyspider.libs.base_handler import *
	import pymongo
	
	class Handler(BaseHandler):
	    crawl_config = {
	    }
	
	    @every(minutes=24 * 60)
	    def on_start(self):
	        self.crawl('http://www.reeoo.com', callback=self.index_page)
	
	    @config(age=10 * 24 * 60 * 60)
	    def index_page(self, response):
	        for each in response.doc('div[class="thumb"]').items():
	            detail_url = each('a').attr.href
	            print (detail_url)
	            self.crawl(detail_url, callback=self.detail_page)
	
	    @config(priority=2)
	    def detail_page(self, response):
	        header = response.doc('body > article > section > header')
	        title = header('h1').text()
	        # print (title)
	        
	        tags = []
	        for each in header.items('a'):
	            tags.append(each.text())
	        # print (tags)
	        
	        content = response.doc('div[id="post_content"]')
	        description = content('blockquote > p').text()
	        # print (description)
	        
	        website_url = content('a').attr.href
	        # print (website_url)
	        
	        image_url_list = []
	        for each in content.items('img[data-src]'):
	            image_url_list.append(each.attr('data-src'))
	        # print (image_url_list)
	        
	        return {
	            "title": title,
	            "tags": tags,
	            "description": description,
	            "image_url_list": image_url_list,
	            "website_url": website_url,
	        }
	    
	    def on_result(self, result):
	        print ('------------------')
	        print (result)
	        print ('------------------')
	        
	        client = pymongo.MongoClient(host='127.0.0.1', port=27017)
	        db = client['pyspyspider_projectdb']
	        coll = db['website']
	        
	        data = {
	            'title': result['title'],
	            'tags': result['tags'],
	            'description': result['description'],
	            'website_url': result['website_url'],
	            'image_url_list': result['image_url_list']
	        }
	        
	        data_id = coll.insert(data)
	        print (data_id)
        
        
        
        
        
        






