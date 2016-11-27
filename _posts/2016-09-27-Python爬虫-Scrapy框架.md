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
	
创建完成后，当前目录下出现对应的项目文件夹，可以看到生成的`Comics`文件结构为：

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
	
每个文件对应的具体功能可查阅官方文档，本篇实现对这些文件涉及不多，所以按下不表。

### 创建Spider类

创建一个用来实现具体爬取功能的类，我们所有的处理实现都会在这个类中进行，它必须为 `scrapy.Spider` 的子类。

在 `Comics/spiders` 文件路径下创建 `comics.py` 文件。

`comics.py` 的具体实现：

	#coding:utf-8
	
	import scrapy
	
	class Comics(scrapy.Spider):
	
		name = "comics"
	
		def start_requests(self):
			urls = ['http://www.xeall.com/shenshi']
			for url in urls:
				yield scrapy.Request(url=url, callback=self.parse)
	
		def parse(self, response):
			self.log(response.body);
    
自定义的类为`scrapy.Spider`的子类，其中的`name`属性为该爬虫的唯一标识，作为scrapy爬取命令的参数。其他方法的属性后续再解释。

### 运行

创建好自定义的类后，切换到`Comics`路径下，运行命令，启动爬虫任务开始爬取网页。

	scrapy crawl comics
	
打印的结果为爬虫运行过程中的信息，和目标爬取网页的html源码。

	2016-11-26 22:04:35 [scrapy] INFO: Scrapy 1.2.1 started (bot: Comics)
	2016-11-26 22:04:35 [scrapy] INFO: Overridden settings: {'ROBOTSTXT_OBEY': True, 'BOT_NAME': 'Comics', 'NEWSPIDER_MODULE': 'Comics.spiders', 'SPIDER_MODULES': ['Comics.spiders']}
	2016-11-26 22:04:35 [scrapy] INFO: Enabled extensions:
	['scrapy.extensions.corestats.CoreStats',
	 'scrapy.extensions.telnet.TelnetConsole',
	 'scrapy.extensions.logstats.LogStats']
	 ...
	 
此时，一个基本的爬虫创建完成了，下面是具体过程的实现。


## 爬取漫画图片

### 起始地址

爬虫的起始地址为：

	http://www.xeall.com/shenshi
	
我们主要的关注点在于页面中间的漫画列表，列表下方有显示页数的控件。如下图所示

![image](/images/posts/scrapy/1.jpg)

爬虫的主要任务是爬取列表中每一部漫画的图片，爬取完当前页后，进入下一页漫画列表继续爬取漫画，依次不断循环直至所有漫画爬取完毕。

起始地址的`url`我们放在了`start_requests`函数的`urls`数组中。其中`start_requests`是重载了父类的方法，爬虫任务开始时会执行到这个方法。

`start_requests`方法中主要的执行在这一行代码：请求指定的`url`，请求完成后调用对应的回调函数`self.parse`

	scrapy.Request(url=url, callback=self.parse)
	
对于之前的代码其实还有另一种实现方式：

	#coding:utf-8
	
	import scrapy
	
	class Comics(scrapy.Spider):
	
		name = "comics"
		start_urls = ['http://www.xeall.com/shenshi']
	
		def parse(self, response):
			self.log(response.body);
			
`start_urls`是框架中提供的属性，为一个包含目标网页url的数组，设置了`start_urls`的值后，不需要重载`start_requests`方法，爬虫也会依次爬取`start_urls`中的地址，并在请求完成后自动调用`parse`作为回调方法。

不过为了在过程中方便调式其它的回调函数，demo中还是使用了前一种实现方式。

### 爬取漫画url

从起始网页开始，首先我们要爬取到每一部漫画的url。

#### 当前页漫画列表

起始页为漫画列表的第一页，我们要从当前页中提取出所需信息，动过实现回调`parse`方法。

在开头导入`BeautifulSoup`库

	from bs4 import BeautifulSoup
	
请求返回的**html**源码用来给`BeautifulSoup`初始化。

	def parse(self, response):
		content = response.body;
		soup = BeautifulSoup(content, "html5lib")
		
初始化指定了`html5lib`解释器，若没安装这里会报错。BeautifulSoup初始化时若不提供指定解释器，则会自动使用自认为匹配的最佳解释器，这里有个坑，对于目标网页的源码使用默认最佳解释器为`lxml`，此时解析出的结果会有问题，而导致无法进行接下来的数据提取。所以当发现有时候提取结果又问题时，打印`soup`看看是否正确。

查看html源码可知，页面中显示漫画列表的部分为类名为`listcon`的`ul`标签，通过`listcon`类能唯一确认对应的标签

![image](/images/posts/scrapy/2.jpg)

提取包含漫画列表的标签

	listcon_tag = soup.find('ul', class_='listcon')
	
上面的`find`方法意为寻找`class`为`listcon`的`ul`标签，返回的是对应标签的所有内容。

在列表标签中查找所有拥有`href`属性的`a`标签，这些`a`标签即为每部漫画对应的信息。

	com_a_list = listcon_tag.find_all('a', attrs={'href': True})
	
然后将每部漫画的`href`属性合成完整能访问的url地址，保存在一个数组中。
	
	comics_url_list = []
	base = 'http://www.xeall.com'
		for tag_a in com_a_list:
			url = base + tag_a['href']
			comics_url_list.append(url)
				
此时`comics_url_list`数组即包含当前页每部漫画的url。

#### 下一页列表

### 爬取漫画图片

#### 当前页图片

#### 下一页图片







