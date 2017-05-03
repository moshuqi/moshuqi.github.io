---
layout: post
title:  【Android OTA】用nodejs搭建服务器
category: "Android"
---

OTA，Over-the-Air的简写，OTA升级就是通过GPRS、3G、无线网络下载升级补丁升级，不用通过有线连接来升级。

Android的应用或是系统，可以通过这种方式进行版本的更新升级。OTA具体原理自行google，或者参考[这篇文章](https://source.android.com/devices/tech/ota/)。本文和接下来的两篇文章主要介绍的是具体的实现过程。


### OTA升级大致过程

1.	设备向服务器进行版本更新检测请求
2.	服务器将更新信息返回到设备端
3.	设备通过返回信息下载指定的更新文件
4.	下载完成后设备安装升级

Android单个应用和整个系统的升级方式存在差异，接下来两篇文章会分别介绍实现。

本文重点实现升级过程中的第二步，搭建一个服务器Demo，以方便后续的测试工作。服务器用的nodejs，使用较为简单，没接触过的也可以跟着下面的步骤将服务器搭建在本地运行起来。

## OTA服务器搭建

### 安装nodejs

用的是Mac系统，安装命令

	brew install node
	
查看是否安装完成

	$ node -v
	v6.2.0
	
**npm** 是专门管理nodejs包的工具，用来方便地安装第三方模块，安装nodejs时应该也默认同时安装了npm，可以命令查看

	$ npm -v
	3.8.9
	
### 运行

安装完成后，先实现个简单的Demo，只需简单几行代码便可在本地运行起一个服务器。

先创建一个文件夹 **Server**，在文件夹充创建文件 **SimpleServer.js**

用文本编辑工具打开**SimpleServer.js**，代码实现：

	var http = require('http');

	var server = http.createServer(function(req, res) {
		res.end('Hello!');
	}).listen(3001);
	console.log('Server listening at port: 3001');
	
然后打开命令行工具，**cd** 到 **Server** 目录下，敲入命令运行服务器：

	node SimpleServer.js
	
命令行打印输出：

	Server listening at port: 3001
	
此时服务器开始监听来自本地端口**3001**的请求。

打开浏览器，地址栏输入 **http://localhost:3001** ，可以看到浏览器界面返回文本 **Hello!**。即一个最简单的服务器。

### Express框架

上边实现的服务器对任何请求都返回*Hello！*文本，现实中服务器当然没那么简单。OTA服务器在接到请求时需要返回对应信息，并提供更新文件的下载接口。

**Express** 是一种保持最低程度规模的灵活 Node.js Web 应用程序框架，为 Web 和移动应用程序提供一组强大的功能。**Express** 提供的各种 HTTP 实用程序方法和中间件能帮助我们方便的处理网络请求。[具体可参考官方文档](http://expressjs.com/zh-cn/)

#### 使用Express应用生成器快速搭建框架

使用以下命令安装 express：

	npm install express-generator -g
	
输入如下命令，在当前目录下创建名为 **ota** 的 Express 应用

	express --view=pug ota
	
看结果打印，在ota目录下自动创建了对应文件

![image1](/images/posts/OtaServer/1.jpg)

**cd** 到 **ota** 目录下，安装需要的依赖

	cd ota/
	npm install
	
	
完成后，在node_modules目录下会下载好项目需要的第三方依赖模块
	
![image2](/images/posts/OtaServer/2.jpg)

项目的运行文件为 **bin/www**，打开查看源文件，可看到服务器运行端口默认情况下为 **3000**

![image3](/images/posts/OtaServer/3.jpg)

命令行运行服务器

	node bin/www 
	
浏览器打开网址 [http://localhost:3000](http://localhost:3000)

Express框架默认生成的界面

![image4](/images/posts/OtaServer/4.jpg)

同时可以看到访问服务器时命令行的打印信息

![image5](/images/posts/OtaServer/5.jpg)










