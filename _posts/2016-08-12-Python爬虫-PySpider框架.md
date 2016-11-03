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


+	**on_start(self)**	程序的入口，当点击左侧绿色区域右上角的 **run** 按钮时首先会调用这个函数

+	**self.crawl(url, callback)**	pyspider库主要的API，用于创建一个爬取任务，**url** 为目标地址，这里为我们刚刚创建任务指定的起始地址，**callback** 为抓取到数据后的回调函数

+	**index_page(self, response)**	参数为 **Response** 对象，**response.doc** 为 **pyquery** 对象（[具体使用可见pyquery官方文档](https://pythonhosted.org/pyquery/)），pyquery和jQuery类似，主要用来方便地抓取返回的html文档中对应标签的数据

+	**detail_page(self, response)**		返回一个 **dict** 对象作为结果，结果会自动保存到默认的 **resultdb** 中，也可以通过重载方法来讲结果数据存储到指定的数据库，后面会再提到具体的实现

其他一些参数

+	**@every(minutes=24 * 60)**		通知 **scheduler**（框架的模块） 每天运行一次

+	**@config(age=10 * 24 * 60 * 60)**		设置任务的有效期限，在这个期限内目标爬取的网页被认为不会进行修改

+	**@config(priority=2)**		设定任务优先级


**Ps.** 需要注意的一个地方，前面跑的 **run.py** 不是下载的源码文件夹中的，而是在 `pyspider` 文件夹中的 **run.py**，如下图，可以看到有两个 **run.py** 文件，虽然两个都能跑起来，但我们用到的是圈出来的那个，否则不能通过 **--config** 配置。

![image1]({{ site.url }}/assets/pysp/6.jpg)

成功跑起来之后可以看到在当前文件夹中生成了一个 `data` 文件夹，生成的结果默认会保存到 **result.db** 中，爬取数据后可打开看里面保存了运行的结果。

![image1]({{ site.url }}/assets/pysp/7.jpg)

<h4>运行</h4>

点击左边绿色区域右上角的 **run** 按钮，运行之后页面下册的 **follows** 按钮出现红色角标

![image1]({{ site.url }}/assets/pysp/8.jpg)

选中 **follows** 按钮，看到 **index_page** 行，点击行右侧的运行按钮

![image1]({{ site.url }}/assets/pysp/9.jpg)

运行完成后显示如下图，即 `www.reeoo.com` 页面上所有的url

![image1]({{ site.url }}/assets/pysp/10.jpg)

此时我们可以任意选择一个结果运行，这时候调用的是 **detail_page** 方法，返回最终的结果。

结果为json格式的数据，这里我们保存的是网页的 `title` 和 `url`，见左侧黑色的区域

![image1]({{ site.url }}/assets/pysp/11.jpg)

回到主页面，此时看到任务列表显示了我们刚刚创建的任务，设置 **status** 为 `running`，然后点击 **Run** 按钮执行

![image1]({{ site.url }}/assets/pysp/12.jpg)

执行过程中可以看到整个过程的打印输出

![image1]({{ site.url }}/assets/pysp/13.jpg)

执行完成后，点击 **Results** 按钮，进入到爬取结果的页面

![image1]({{ site.url }}/assets/pysp/14.jpg)

![image1]({{ site.url }}/assets/pysp/15.jpg)

右上方的按钮选择将结果数据保存成对应的格式，例如：JSON格式的数据为：

![image1]({{ site.url }}/assets/pysp/16.jpg)

以上则为pyspider的基本使用方式。

<h3>自定义爬取指定数据</h3>

接下来我们通过自定义来抓取我们需要的数据，目标为抓取这个页面中，每个详情页内容的标题、标签、描述、图片的url、点击图片所跳转的url。

![image1]({{ site.url }}/assets/pysp/17.jpg)

![image1]({{ site.url }}/assets/pysp/18.jpg)

点击首页中的 **project name > reo**，跳转到脚本的编辑界面

![image1]({{ site.url }}/assets/pysp/19.jpg)

<h4>获取所有详情页面的url</h4>

**index_page(self, response)** 函数为获取到 `www.reeoo.com` 页面所有信息之后的回调，我们需要在该函数中对 **response** 进行处理，提取出详情页的url。

通过查看源码，可以发现 **class** 为 **thum** 的 **div** 标签里，所包含的 **a** 标签的 **href** 值即为我们需要提取的数据，如下图

![image1]({{ site.url }}/assets/pysp/20.jpg)

代码的实现

	def index_page(self, response):	
        for each in response.doc('div[class="thumb"]').items():
            detail_url = each('a').attr.href
            print (detail_url)
            self.crawl(detail_url, callback=self.detail_page)
            
**response.doc('div[class="thumb"]').items()** 返回的是所有 **class** 为 **thumb** 的 **div** 标签，可以通过循环 **for...in** 进行遍历。

**each('a').attr.href** 对于每个 **div** 标签，获取它的 **a** 标签的 **href** 属性。

可以将最终获取到的url打印，并传入 **crawl** 中进行下一步的抓取。

运行起来之后的结果如下图，中间的灰色区域为打印的结果

![image1]({{ site.url }}/assets/pysp/21.jpg)

注意左侧区域下方的几个按钮，可以展示当前所爬取页面的一些信息，**web** 按钮可以查看当前页面，**html** 显示当前页面的源码，**enable css selector helper** 可以通过选中当前页面的元素自动生成对应的 css 选择器方便的插入到脚本代码中，不过并不是总有效，在我们的demo中就是无效的~

![image1]({{ site.url }}/assets/pysp/21.jpg)

![image1]({{ site.url }}/assets/pysp/22.jpg)

<h4>抓取详情页中指定的信息</h4>












        
        
        
        
        
        






