---
layout: post
title:  易瘦跑步客户端
category: "iOS"
---

### 引用截图


### 项目文件结构

### 布局方式

大部分视图控件使用了xib做布局，没用storyboard做页面间的跳转，是因为大部分时间用笔记本做开发，小屏幕看storyboard一堆连在一起的界面体验太差。当时开发的时候还是15年年底，iOS9的reference也才刚出，项目要考虑兼容iOS8。

有些视图的布局，需要的运算很小，懒得专门新建个xib，就直接通过计算frame的方式来实现了。

有些使用了xib但需要做布局变化的地方，会通过引用布局约束的**NSLayoutConstraint**，使用代码来进行进行修改。

也有些地方尝试使用纯代码定义**NSLayoutConstraint**的方式来实现自动布局，感受就是，实现的代码量太大，后续回顾维护起来不方便。

抽点代码出来感受一下。。这段代码其实在xib里面就是给一个视图添加了一个上下左右边缘的约束和对应的间距，界面上点几下就能实现的事，换成代码会要用好多行。所以项目里的布局能用xib的基本全用xib了，我懒。

	...
	
	[self addConstraint:[NSLayoutConstraint constraintWithItem:_contentView
	                                                     attribute:NSLayoutAttributeTop
	                                                     relatedBy:NSLayoutRelationEqual
	                                                        toItem:self
	                                                     attribute:NSLayoutAttributeTop
	                                                    multiplier:1.0
	                                                      constant:0.0]];
	    
	    [self addConstraint:[NSLayoutConstraint constraintWithItem:_contentView
	                                                     attribute:NSLayoutAttributeBottom
	                                                     relatedBy:NSLayoutRelationEqual
	                                                        toItem:self
	                                                     attribute:NSLayoutAttributeBottom
	                                                    multiplier:1.0
	                                                      constant:0.0]];
	    
	    [self addConstraint:[NSLayoutConstraint constraintWithItem:_contentView
	                                                     attribute:NSLayoutAttributeLeading
	                                                     relatedBy:NSLayoutRelationEqual
	                                                        toItem:self
	                                                     attribute:NSLayoutAttributeLeading
	                                                    multiplier:1.0
	                                                      constant:0.0]];
	    
	    [self addConstraint:[NSLayoutConstraint constraintWithItem:_contentView
	                                                     attribute:NSLayoutAttributeTrailing
	                                                     relatedBy:NSLayoutRelationEqual
	                                                        toItem:self
	                                                     attribute:NSLayoutAttributeTrailing
	                                                    multiplier:1.0
	                                                      constant:0.0]];
	                                                      
	    ...
	    
	    
若真想用代码来实现自动布局，推荐一个框架[Masonry](https://github.com/SnapKit/Masonry)，它对**NSLayoutConstraint**做了一层封装，方便你用一种更直观更优雅的编码方式来实现布局约束。这个框架有对应的Swift版本[SnapKit](https://github.com/SnapKit/SnapKit)，也是同一个团队在进行维护。具体可查看官方文档。

### 第三方库

主要用到的第三方库：

+	高德地图SDK：用来做路径绘制和定位
+ 	ShareSDK：第三方分享
+  FMDB：本地数据库的操作
+  AFNetworking：网络请求
+  友盟SDK：做埋点统计和用户反馈

使用到的第三方库都放到了项目**Vendors**文件夹里，当时是直接将第三方的源码和资源拖到项目里，并手动给项目target添加需要的系统库。这并不是一种好的实践方式，引用管理第三方还是推荐使用[CocoaPods](https://github.com/CocoaPods/CocoaPods)。当然，这种做法也有一种好处，就是方便别人把项目download下来之后什么都不用管直接就能跑起来了。：）

### 蓝牙连接

应用可以通过蓝牙连接外设获取数据。当时产品我们会有自己配套的运动内衣，穿戴上之后可收集心率。应用通过扫描附近蓝牙设备并连接就可获取到心率的数据。

整个蓝牙相关的流程走的是标准的蓝牙协议，iOS自带相关框架`<CoreBluetooth/CoreBluetooth.h>`专门处理这个流程。

代码具体实现在 **Bluetooth** 文件夹下的 **YSBluetoothConnect** 类。

实现对应的代理来进行蓝牙事件回调处理，如设备的连接、断开，心率数据的获取。

	CBCentralManagerDelegate
	CBPeripheralDelegate
	
心率数据貌似在蓝牙设备中有固定的标准字段，代码里面这个字段的常量。

	static NSString *ServiceHeartRateUUIDStr = @"180D";
	static NSString *CharacteristicHeartRateUUIDStr = @"2A37";
	
所以应该只要是能发送心率的蓝牙设备都能给这个应用传输数据。代码里面对蓝牙设备名称的前缀做了判断处理，若想尝试自行连其他蓝牙设备可以把这个前缀判断注释掉。

有些界面需要有蓝牙数据才能显示，为了获取到模拟的蓝牙数据，直接将这个字段设为YES即可。

	// 设为YES时模拟生成心率
	static BOOL simulationMode = YES;

### 语音提示

**Voice** 文件夹下的 **YSVoicePrompt**类，运动时每公里提示，若连接心率设备时，会根据心率是否在特定的范围做相应提示。提示可设置男声女声，资源文件在Audio.bundle里，当时还是专门在淘宝找人录的。

### 数据库操作

主要实现在 **Database** 文件下。**FMDB**对系统自带的**SQLite**做了封装，使得可以直接用OC的方式来操作数据库。

数据库主要保存用户信息和每次跑步数据，，例如每次跑步记录的GPS路径坐标，运动过程中收集到的心率数据。界面展示相关信息时需要从数据库查询。

系统的**sqlite**并不是线程安全的，多线程同时对sqlite进行操作极有可能造成崩溃。数据库的读写比较耗时不能放到主线程里，多线程操作可以丢到**FMDB**的**FMDatabaseQueue**队列里进行。

### 网络请求

请求相关的处理实现在 **Network** 文件下。如用户的注册时的验证码，登录注销，云端数据同步和本地数据上传等。具体接口可看 **YSNetworkManager.h** 文件注释。

租用的服务器已过期，所以现在的请求基本都返回超时，只能看着接口自行脑补过程了。

### 地图功能

整个应用最核心的功能就是和地图相关的路径记录了。使用的是高德SDK。具体的功能实现在 **Manager/Map** 文件下。

主要的实现类为 **YSMapManager**

运动开始结束时调用的接口

	- (void)startLocation;
	- (void)endLocation;

**MAMapView** 为显示地图的视图，用来显示地图和绘制运动路径。

实现**MAMapView**的代理*MAMapViewDelegate*，定位实时更新时会收到对应的回调

	- (void)mapView:(MAMapView *)mapView didUpdateUserLocation:(MAUserLocation *)userLocation updatingLocation:(BOOL)updatingLocation

回调参数里会包含新的定位的信息，将新获取到的定位数据保存，并从新绘，即可看到实时的运动轨迹。

实时路径的绘制具体可看代码，也可看[这篇文章](http://moshuqi.github.io/2016/01/28/高德地图SDK实时绘制跑步路径-swift实现/)，高德地图实时路径绘制代码实现忽略其他的代码。

**YSMapPaintFunc** 类实现了将获取到的定位点依次连成一条路径显示在地图上。GPS定位有时候会存在一定的误差，绘制到地图上会显示毛刺，实现的时候用了个小算法将这些毛刺点做了平滑处理，具体自己看代码啦。

记录路径的过程中还需要计算一些其他的值，如总距离，平均速度，当前速度，配速等。这些东可以通过搜集到的定位数据和时间来进行计算。**YSMapPaintFunc**类用来做相应的计算。

### 计时器

**Manager/Time** 中实现。**YSTimeManager** 主要用了一个**Runloop**通过一定的时间间隔来不断刷新界面和时间相关的信息。需要注意的是**NSRunLoop**在主线程和子线程使用的区别。

**Manager/CaptchaTimer** 中的实现为倒计时，用户手机注册时会发送验证码，发送验证码按钮点击之后会有一定的时间将按钮置灰，以防止频繁的发送验证码。

### 应用配置

**Manager/Config** 的**YSConfigManager** 类用来记录应用的配置，通过系统的**NSUserDefaults**进行保存。需要配置的信息较少，也就语音提示选择男声或女声，界面上时候显示实时心率数据的面板，蓝牙默认连接。配置的选项不多。

