---
layout: post
title:  json-server - 快速搭建提供测试数据的后台
category: "nodejs"
---

App开发的过程中，有时需要对一些数据进行网络请求，开发前期服务器相应的API没有实现，此时想测试App代码中网络请求的流程是否有问题，可以通过 **json-server** 快速搭建一个简单的服务器来进行测试，只需预先定义好一个包含相关返回数据的 **json** 文件，即可对服务器进行常规的请求。

### 安装

先保证系统已经安装了nodejs，其中自带npm工具，运行命令行，在全局安装 **json-server**

	sudo npm install -g json-server
	
确认是否安装完成，显示当前的版本

	json-server -v
	
### 创建数据

新建文件 **db.json**，文件内容如下，假设我们用一些数据来保存书签相关的信息。

	{
	  "markbooks": [
	    {
	      "id": 1,
	      "title": "nodejs的express使用介绍",
	      "link": "http://www.cnblogs.com/mq0036/p/5243312.html",
	      "category": "nodejs"
	    },
	    {
	      "id": 2,
	      "title": "Nodejs基础中间件Connect",
	      "link": "http://www.tuicool.com/articles/emeuie",
	      "category": "nodejs"
	    },
	    {
	      "id": 3,
	      "title": "[译]Node.js安全清单",
	      "link": "https://segmentfault.com/a/1190000003860400",
	      "category": "nodejs"
	    },
	    {
	      "id": 4,
	      "title": "iOS下音视频通信的实现-基于WebRTC",
	      "link": "http://www.cocoachina.com/ios/20170306/18837.html",
	      "category": "iOS"
	    },
	    {
	      "id": 5,
	      "title": "iOS 成长之路",
	      "link": "http://www.jianshu.com/p/1e1eb5cac79d",
	      "category": "iOS"
	    }
	  ],
	  "user": {
	    "id": 1,
	    "name": "msq",
	    "desc": "-_-!!!!"
	  }
	}
	
**PS.** `id` 字段在 **POST** 请求时需要用到，若不存在该字段会抛出异常。如果只用来响应 **GET** 请求，可以不加 `id` 字段。

### 运行

#### 运行服务器

命令行输入命令启动服务器

	json-server --watch db.json
	
服务器运行成功后，命令行窗口显示相关的信息。*--watch* 参数监听db文件内容的修改并实时生效。
	
![image1](/images/posts/JsonServer/1.png)

默认运行的端口号是3000，也可以通过在运行命令时加上参数修改端口号

	json-server --watch db.json --port 8888

#### GET请求

访问markbooks的信息 ***http://localhost:3000/markbooks***

![image2](/images/posts/JsonServer/2.png)

访问user的信息 ***http://localhost:3000/user***

![image3](/images/posts/JsonServer/3.png)

也可以访问json文件所有信息 ***http://localhost:3000/db***

![image4](/images/posts/JsonServer/4.png)

通过参数请求，如访问指定类别的数据 ***http://localhost:3000/markbooks?category=nodejs***

![image5](/images/posts/JsonServer/5.png)

json-server还自带了一些参数来实现一些效果，如分页请求，获取第1页的数据，每页最多限制2个数据

***http://localhost:3000/markbooks?category=nodejs&_page=1&_limit=2***

![image6](/images/posts/JsonServer/6.png)

#### POST请求

**POST**请求可以用来创建新数据，如给markbooks数组中新添加一个数据，设置**category**和**title**，**link**字段留空。

使用了**curl**工具来产生一个post请求

	curl -d "category=iOS" -d "title=iOS mark" "http://127.0.0.1:3000/markbooks"
	
请求完成后db.json文件中增加了字段，其中 `id` 字段为系统自动添加上的字段。

	{
      "category": "iOS",
      "title": "iOS mark",
      "id": 6
    }
    

也可以通过**POST**修改当前字段

	curl -d "name=New Name" "http://127.0.0.1:3000/user"
	
修改之后**db.json**文件自动进行了更新，原来的`user`字段变成了

	"user": {
	    "name": "New Name"
	  }
	


#### 自定义路由

还可以自定义路由，将新的路由映射到原来的路由上。需要注意，每个路由必须以 `/` 开头。

创建路由定义文件 **routes.json**，*category* 为参数

	{
	  "/api/markbooks/:category": "/markbooks?category=:category"
	}
	
将自定义路由文件作为参数运行服务器

	json-server db.json --routes routes.json
	
此时访问 ***http://localhost:3000/api/markbooks/nodejs***

![image7](/images/posts/JsonServer/7.png)

### 后记

其他的具体内容可参考[json-server官方文档](https://github.com/typicode/json-server)，**json-server** 还提供了实现某些功能的现成接口。当然，大部分情况下，写个简单的json文件来作为数据请求的返回足够用来测试了。

完。


