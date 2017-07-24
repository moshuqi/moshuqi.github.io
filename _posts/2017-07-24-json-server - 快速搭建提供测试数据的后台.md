---
layout: post
title:  json-server - 快速搭建提供测试数据的后台
category: "nodejs"
---

### 安装

	sudo npm install -g json-server
	
### 创建数据

db.json

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

### 运行

	json-server --watch db.json
	
![image1](/images/posts/JsonServer/1.png)

修改端口号

json-server --watch db.json --port 8888


访问 http://localhost:3000/markbooks

![image2](/images/posts/JsonServer/2.png)

访问 http://localhost:3000/user

![image3](/images/posts/JsonServer/3.png)

http://localhost:3000/db

![image4](/images/posts/JsonServer/4.png)

通过参数请求

http://localhost:3000/markbooks?category=nodejs

![image5](/images/posts/JsonServer/5.png)

分页请求

http://localhost:3000/markbooks?category=nodejs&_page=1&_limit=2

![image6](/images/posts/JsonServer/6.png)

post

curl -d "category=iOS" -d "title=iOS mark" "http://127.0.0.1:3000/markbooks"

	{
      "category": "iOS",
      "title": "iOS mark",
      "id": 6
    }

自定义路由

	{
	  "/api/markbooks/:category": "/markbooks?category=:category"
	}

json-server db.json --routes routes.json

![image7](/images/posts/JsonServer/7.png)


