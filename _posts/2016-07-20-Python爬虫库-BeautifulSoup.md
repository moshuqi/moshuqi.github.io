---
layout: post
title:  Python爬虫库-Beautiful Soup的使用
category: "Python"
---

**Beautiful Soup** 是一个可以从HTML或XML文件中提取数据的Python库，简单来说，它能将HTML的标签文件解析成树形结构，然后方便地获取到指定标签的对应属性。

如在上一篇文章[通过爬虫爬取漫画图片](https://moshuqi.github.io/python/2016/06/15/Python爬虫抓取漫画图片.HTML)，获取信息纯粹用正则表达式进行处理，这种方式即复杂，代码的可阅读性也低。通过**Beautiful Soup**库，我们可以将指定的class或id值作为参数，来直接获取到对应标签的相关数据，这样的处理方式简洁明了。

当前最新的 **Beautiful Soup** 版本为4.4.0，Beautiful Soup 3 当前已停止维护。

**Beautiful Soup 4** 可用于 Python2.7 和 Python3.0，本文示例使用的Python版本为2.7。

博主使用的是Mac系统，直接通过命令安装库：

	sudo easy_install beautifulsoup4

安装完成后，尝试包含库运行：

	from bs4 import BeautifulSoup
	
若没有报错，则说明库已正常安装完成。



<h3>开始</h3>

本文会通过这个网页[http://reeoo.com](http://reeoo.com)来进行示例讲解，如下图所示

![image1]({{ site.url }}/assets/BS4/1.jpg)

最后会实现一个demo，爬取页面上所有缩略图对应的标题、跳转url和图片地址。


<h4>BeautifulSoup 对象初始化</h4>

将一段文档传入 **BeautifulSoup** 的构造方法，就能得到一个文档对象。如下代码所示，文档通过请求url获取：

	#coding:utf-8
	
	from bs4 import BeautifulSoup
	import urllib2
	
	url = 'http://reeoo.com'
	request = urllib2.Request(url)
	response = urllib2.urlopen(request, timeout=20)
	
	content = response.read()
	soup = BeautifulSoup(content, 'HTML.parser')
	
request 请求没有做异常处理，这里暂时先忽略。***BeautifulSoup*** 构造方法的第二个参数为文档解析器，若不传入该参数，BeautifulSoup会自行选择最合适的解析器来解析文档，不过会有警告提示。

也可以通过文件句柄来初始化，可先将HTML的源码保存到本地同级目录 **reo.HTML**，然后将文件名作为参数：

	soup = BeautifulSoup(open('reo.HTML'))

可以打印 **soup**，输出内容和HTML文本无二致，此时它为一个复杂的树形结构，每个节点都是Python对象。

Ps. 接下来示例代码中所用到的 **soup** 都为该soup。

<h4>Tag</h4>

Tag对象与HTML原生文档中的标签相同，可以直接通过对应名字获取

	tag = soup.title
	print tag

打印结果：
	
	<title>Reeoo - web design inspiration and website gallery</title>
	
<h4>Name</h4>

通过**Tag**对象的*name*属性，可以获取到标签的名称

	print tag.name
	# title
	
<h4>Attributes</h4>

一个tag可能包含很多属性，如id、class等，操作tag属性的方式与字典相同。

例如网页中包含缩略图区域的标签 **article**
	
	...
	<article class="box">
		<div id="main">
		<ul id="list">
			<li id="sponsor"><div class="sponsor_tips"></div>
				<script async type="text/javascript" src="//cdn.carbonads.com/carbon.js?zoneid=1696&serve=CVYD42T&placement=reeoocom" id="_carbonads_js"></script>
			</li>
	...	
			
获取它 *class* 属性的值

	tag = soup.article
	c = tag['class']
	
	print c		
	# [u'box']

也可以直接通过 ***.attrs*** 获取所有的属性

	tag = soup.article
	attrs = tag.attrs
	
	print attrs
	# {u'class': [u'box']}

ps. 因为class属于多值属性，所以它的值为数组。


<h4>tag中的字符串</h4>

通过 ***string*** 方法获取标签中包含的字符串

	tag = soup.title
	s = tag.string
	
	print s
	# Reeoo - web design inspiration and website gallery

<h3>文档树的遍历</h3>

一个Tag可能包含多个字符串或其它的Tag，这些都是这个Tag的子节点。Beautiful Soup提供了许多操作和遍历子节点的属性。

<h4>子节点</h4>

通过**Tag**的 *name* 可以获取到对应标签，多次调用这个方法，可以获取到子节点中对应的标签。

如下图：

![image1]({{ site.url }}/assets/BS4/1.jpg)

我们希望获取到 **article** 标签中的 **li**

	tag = soup.article.div.ul.li
	print tag
	
打印结果：

	<li id="sponsor"><div class="sponsor_tips"></div>
	<script async="" id="_carbonads_js" src="//cdn.carbonads.com/carbon.js?zoneid=1696&amp;serve=CVYD42T&amp;placement=reeoocom" type="text/javascript"></script>
	</li>
	
也可以把中间的一些节点省略，结果也一致

	tag = soup.article.li
	
通过 **.** 属性只能获取到第一个tag，若想获取到所有的 *li* 标签，可以通过 *find_all()* 方法

	ls = soup.article.div.ul.find_all('li')
	
获取到的是包含所有*li*标签的列表。

tag的 **.contents** 属性可以将tag的子节点以列表的方式输出:
	
	tag = soup.article.div.ul
	contents = tag.contents

打印 *contents* 可以看到列表中不仅包含了 *li* 标签内容，还包括了换行符 *'\n'*

过tag的 **.children** 生成器,可以对tag的子节点进行循环

	tag = soup.article.div.ul
	children = tag.children
	print children
	
	for child in children:
		print child

可以看到 **children** 的类型为 *<listiterator object at 0x109cb1850>*

*.contents* 和 *.children* 属性仅包含tag的直接子节点，若要遍历子节点的子节点，可以通过 *.descendants* 属性，方法与前两者类似，这里不列出来了。

<h4>父节点</h4> 

通过 **.parent** 属性来获取某个元素的父节点，article 的 父节点为 body。

	tag = soup.article
	print tag.parent.name
	# body

或者通过 **.parents** 属性遍历所有的父辈节点。

	tag = soup.article
	for p in tag.parents:
		print p.name

<h4>兄弟节点</h4> 

**.next_sibling** 和 **.previous_sibling** 属性用来插叙兄弟节点，使用方式与其他的节点类似。

<h3>文档树的搜索</h3>

对树形结构的文档进行特定的搜索是爬虫抓取过程中最常用的操作。

<h4>find_all()</h4>


















