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

<<<<<<< HEAD
=======


>>>>>>> 88c1cf83a4027139ae1522812cc632b5f6f72556

<h4>BeautifulSoup 对象初始化</h4>

将一段文档传入 **BeautifulSoup** 的构造方法，就能得到一个文档对象。如下代码所示，文档通过请求url获取：

	#coding:utf-8
	
	from bs4 import BeautifulSoup
	import urllib2
	
	url = 'http://reeoo.com'
	request = urllib2.Request(url)
	response = urllib2.urlopen(request, timeout=20)
	
	content = response.read()
	soup = BeautifulSoup(content, 'html.parser')
	
request 请求没有做异常处理，这里暂时先忽略。***BeautifulSoup*** 构造方法的第二个参数为文档解析器，若不传入该参数，BeautifulSoup会自行选择最合适的解析器来解析文档，不过会有警告提示。

也可以通过文件句柄来初始化，可先将HTML的源码保存到本地同级目录 **reo.html**，然后将文件名作为参数：

	soup = BeautifulSoup(open('reo.html'))

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

![image1]({{ site.url }}/assets/BS4/2.jpg)

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

*find_all(name , attrs , recursive , string , ** kwargs)*

<h5>name 参数</h5>

查找所有名字为 *name* 的tag

	soup.find_all('title')
	# [<title>Reeoo - web design inspiration and website gallery</title>]
	
	soup.find_all('footer')
	# [<footer id="footer">\n<div class="box">\n<p> ... </div>\n</footer>]
	
<h5>keyword 参数</h5>

如果指定参数的名字不是内置的参数名（name , attrs , recursive , string），则将该参数当成tag的属性进行搜索，不指定tag的话则默认为对所有tag进行搜索。

如，搜索所有 ***id*** 值为 *footer* 的标签

	soup.find_all(id='footer')
	# [<footer id="footer">\n<div class="box">\n<p> ... </div>\n</footer>]
	
加上标签的参数

	soup.find_all('footer', id='footer')
	# [<footer id="footer">\n<div class="box">\n<p> ... </div>\n</footer>]
	
	# 没有id值为'footer'的div标签，所以结果返回为空
	soup.find_all('div', id='footer')
	# []
	
获取所有缩略图的 *div* 标签，缩略图用 *class* 为 *thumb* 标记

	soup.find_all('div', class_='thumb')
	
这里需要注意一点，因为 **class** 为Python的保留关键字，所以作为参数时加上了下划线，为“**class_**”。

指定名字的属性参数值可以包括：***字符串***、***正则表达式***、***列表***、***True/False***。

<h5>True/False</h5>

是否存在指定的属性。

搜索所有带有 *target* 属性的标签

	soup.find_all(target=True)


搜索所有不带 *target* 属性的标签（仔细观察会发现，搜索结果还是会有带 *target* 的标签，那是不带 *target* 标签的子标签，这里需要注意一下。）

	soup.find_all(target=False)

可以指定多个参数作为过滤条件，例如页面缩略图部分的标签如下所示：

	...

	<li>
		<div class="thumb">
			<a href="http://reeoo.com/aim-creative-studios"><img class="lazy" src="http://reeoo.com/assets/white.gif" data-original="http://media.reeoo.com/AIM Creative Studios.png!page" width="300" height="200" alt="AIM Creative Studios" tile="AIM Creative Studios"></a>
		</div>
		<div class="title">
			<a href="http://reeoo.com/aim-creative-studios">AIM Creative Studios</a>
		</div>
	</li>
	
	...

搜索 ***src*** 属性中包含 *reeoo* 字符串，并且 ***class*** 为 *lazy* 的标签：

	soup.find_all(src=re.compile("reeoo.com"), class_='lazy')
	
搜索结果即为所有的缩略图 ***img*** 标签。

有些属性不能作为参数使用，如 ***data-**** 属性。在上面的例子中，*data-original* 不能作为参数使用，运行起来会报错，*SyntaxError: keyword can't be an expression*。

<h5>attrs 参数</h5>

定义一个字典参数来搜索对应属性的tag，一定程度上能解决上面提到的不能将某些属性作为参数的问题。

例如，搜索包含 ***data-original*** 属性的标签

	print soup.find_all(attrs={'data-original': True})
	
搜索 ***data-original*** 属性中包含 *reeoo.com* 字符串的标签
	
	soup.find_all(attrs={'data-original': re.compile("reeoo.com")})
	
搜索 ***data-original*** 属性为指定值的标签
	
	soup.find_all(attrs={'data-original': 'http://media.reeoo.com/Bersi Serlini Franciacorta.png!page'})


<h5>string 参数</h5>

和 ***name*** 参数类似，针对文档中的字符串内容。

搜索包含 *Reeoo* 字符串的标签：

	soup.find_all(string=re.compile("Reeoo"))
	
打印搜索结果可看到包含3个元素，分别是对应标签里的内容，具体见下图所示


![image1]({{ site.url }}/assets/BS4/3.jpg)

![image1]({{ site.url }}/assets/BS4/4.jpg)

![image1]({{ site.url }}/assets/BS4/5.jpg)

<h5>limit 参数</h5>

*find_all()* 返回的是整个文档的搜索结果，如果文档内容较多则搜索过程耗时过长，加上 ***limit*** 限制，当结果到达 *limit* 值时停止搜索并返回结果。

搜索 *class* 为 *thumb* 的 **div** 标签，只搜索3个

	soup.find_all('div', class_='thumb', limit=3)
	
打印结果为一个包含3个元素的列表，实际满足结果的标签在文档里不止3个。

<h5>recursive 参数</h5>

*find_all()* 会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 ***recursive***=**False**。


<h4>find()</h4>

*find(name , attrs , recursive , string , ** kwargs)*

***find()*** 方法和 ***find_all()*** 方法的参数使用基本一致，只是 ***find()*** 的搜索方法只会返回第一个满足要求的结果，等价于 ***find_all()*** 方法并将*limit*设置为1。

	soup.find_all('div', class_='thumb', limit=1)
	soup.find('div', class_='thumb')

搜索结果一致，唯一的区别是 ***find_all()*** 返回的是一个数组，***find()*** 返回的是一个元素。

当没有搜索到满足条件的标签时，***find()*** 返回 **None**， 而 ***find_all()*** 返回一个空的列表。

<h4>CSS选择器</h4>

**Tag** 或 **BeautifulSoup** 对象通过 ***select()*** 方法中传入字符串参数, 即可使用CSS选择器的语法找到tag。

语义和CSS一致，搜索 **article** 标签下的 **ul** 标签中的 **li** 标签

	print soup.select('article ul li')
	
通过类名查找，两行代码的结果一致，搜索 **class** 为 *thumb* 的标签

	soup.select('.thumb')
	soup.select('[class~=thumb]')
	

通过id查找，搜索 **id** 为 *sponsor* 的标签

	soup.select('#sponsor')
	
通过是否存在某个属性来查找，搜索具有 **id** 属性的 **li** 标签

	soup.select('li[id]')
	

通过属性的值来查找查找，搜索 **id** 为 *sponsor* 的 **li** 标签

	soup.select('li[id="sponsor"]')
	

 
<h3>其他</h3>

其他的搜索方法还有：

**find_parents()** 和 **find_parent()**

**find_next_siblings()** 和 **find_next_sibling()**

**find_previous_siblings()** 和 **find_previous_sibling()**

...

参数的作用和 ***find_all()***、***find()*** 差别不大，这里就不再列举使用方式了。这两个方法基本已经能满足绝大部分的查询需求。


还有一些方法涉及文档树的修改。对于爬虫来说大部分工作只是检索页面的信息，很少需要对页面源码做改动，所以这部分的内容也不再列举。

具体详细信息可直接参考**Beautiful Soup**库的[官方说明文档](http://beautifulsoup.readthedocs.io/zh_CN/latest/)

【完】。 ：）



