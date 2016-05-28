---
layout: post
title:  iOS 自动化测试 UI Automation
category: "iOS"
---

iOS提供了一个框架UIAutomation，可用来实现自动化测试，可以通过这个框架自定义一些列操作，自动的运行在真机设备或者xcode的模拟器上。

UIAutomation框架的测试方式属于非注入式的，测试内容和最终上线内容一致，不需要源码，但是功能会有所受限。注入式的自动化测试需要用到第三方的工具。本文主要用到的是前者的方式。

测试环境需要用到instruments，xcode自带的工具。本文的例子为直接在模拟器上进行测试。

先用个现成的项目来展示下大概的用法。打开个博主自己写的跑步软件，在Xcode栏->选则Produce->Profile，之后选择Automation

![image1]({{ site.url }}/assets/AutoTest/2.jpg)

进入Automation的界面

![image1]({{ site.url }}/assets/AutoTest/8.jpg)

这个界面的更多细节在下面的Demo中解释，先看到界面的黑色区域，为编写脚本代码的地方，在此处添加脚本后点击左上角的红色圆形按钮，运行起App后便会自动运行脚本的内容。

在App运行的过程中可以实时修改脚本代码，并点击代码编辑区域下方的“播放”按钮，能实时运行脚本。

UIAutomation是通过层级访问的方式来定位到某一个具体元素的，苹果提供了一个logElementTree()的方法，打印控件树。通过运行这个方法，能看到App当前界面UI的层级。打印层级的脚本代码如下：

	var target = UIATarget.localTarget();
	target.logElementTree();

App的界面如图

![image1]({{ site.url }}/assets/AutoTest/10.jpg)

选中**Trace Log**展开打印的结果，可以看到界面上的各种控件信息，如标签、按钮的文本位置等。

![image1]({{ site.url }}/assets/AutoTest/9.jpg)

接下来通过脚本实现一个最基本的操作，自动点击“开始跑步”按钮开始跑步的功能。

“开始跑步”按钮在展开层级上的信息，可以看到为一个**UIAButton**，**name**属性为“开始跑步”

![image1]({{ site.url }}/assets/AutoTest/11.jpg)

脚本的代码实现如下：

	var target = UIATarget.localTarget();
	var app = target.frontMostApp();
	var window = app.mainWindow();
	
	var runBtn = window.buttons()["开始跑步"];
	runBtn.tap();
	
**UIATarget** 对象代表待测应用所在环境的最高层级UI,在这里*localTarget()*表示运行app的这台iPhone设备。

**UIAApplication** 对象代表app层级的UI，这里通过*frontMostApp()*方法得到的对象，就是指正在运行的app。

**UIAWindow**对象代表app中window层级的UI,这里通过mainWindow()方法得到的对象，指当前app中的主窗体，一个app的当前界面通常只会有一个主窗体。

实际项目中，不同元素的差异都是从window层级开始的，在window层级往上，都是一样的。

自己编写测试代码，具体参考[官方文档 UI Automation JavaScript Reference for iOS](https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/)，对App的各个操作最终都是通过这些api来实现的。

*buttons()*方法用来获取界面上的所有按钮元素，然后通过**name**属性获取对应的按钮。**name**属性默认值为按钮的文本，也可以通过设置**UIButton**的**accessibilityLabel**属性来改变**name**的值。也可通过IB修改

![image1]({{ site.url }}/assets/AutoTest/12.jpg)

*tap()*方法既向按钮做一个点击操作，若要模拟其他操作可参考[UIAButton 提供的接口](https://developer.apple.com/library/ios/documentation/ToolsLanguages/Reference/UIAButtonClassReference/index.html#//apple_ref/doc/uid/TP40009900)，不同的控件系统都提供了对应的操作接口。

当前App正在模拟器中运行，加入脚本的代码后直接点击下方的“播放”按钮，App进入跑步功能。

![image1]({{ site.url }}/assets/AutoTest/13.jpg)

自动操作的实现大概就是这样。当然，现实的自动化测试中肯定会涉及到更复杂的操作流程，可以通过脚本调用各种控件对应的事件操作接口，自定义操作的顺序，来实现自动化测试流程。


<h3>一个简单的Demo</h3>

上边展示用到的是项目的代码，接下来实现一个简单的Demo，实现一个最基本的脚本自动测试。

具体实现内容如下图所示，界面上有若干按钮，按钮初始化标签值为0，编写脚本随机在界面上进行点击事件，当点中按钮时，对应的按钮标签值加1。这种模拟界面上随机点击操作事件的方式可以用来测试App功能的稳定性和健壮性。最后附demo的完整源码。

![image1]({{ site.url }}/assets/AutoTest/1.jpg)

首先新建一个工程，命名为**AutoTest**。

创建一个**UIButton**的子类**MyButton**，按钮被点击时标签的值自动递增1，这个功能放在**MyButton**类中实现。

***MyButton*** .h文件

	#import <UIKit/UIKit.h>
	
	@interface MyButton : UIButton
	
	- (void)click;
	
	@end
	
按钮被点击时调用*click*方法，标签值递增。

***MyButton*** .m文件

	#import "MyButton.h"
	
	@interface MyButton ()
	
	@property (nonatomic, assign) NSInteger count;
	
	@end
	
	@implementation MyButton
	
	- (id)init
	{
	    self = [super init];
	    if (self)
	    {
	        self.count = 0;
	        [self setTitle:[NSString stringWithFormat:@"%@", @(self.count)] forState:UIControlStateNormal];
	        [self setTitleColor:[UIColor grayColor] forState:UIControlStateNormal];
	    }
	    
	    return self;
	}
	
	- (void)click
	{
	    self.count ++;
	    [self setTitle:[NSString stringWithFormat:@"%@", @(self.count)] forState:UIControlStateNormal];
	}
	
	@end
	
重载*init*方法，初始化时给**count**属性附初始值1。

在创建工程自动生成的试图控制器类**ViewController**中添加按钮。

引入自定义类的头文件

	#import "MyButton.h"
	
定义***clickBtn:***方法，作为按钮点击时调用的方法。

	- (void)clickBtn:(id)sender
	{
	    MyButton *btn = (MyButton *)sender;
	    [btn click];
	}
	
定义***addButtons***方法，向界面添加按钮。

	- (void)addButtons
	{
	    CGFloat w = CGRectGetWidth(self.view.frame);
	    CGFloat h = CGRectGetHeight(self.view.frame);
	    
	    CGFloat distance = 20;      // 按钮之间、按钮和屏幕边缘之间的间距
	    NSInteger rowCount = 9;    // 行数
	    NSInteger columnCount = 5;  // 列数
	    
	    CGFloat btnW = (w - distance * (columnCount + 1)) / columnCount;    // 按钮宽度
	    CGFloat btnH = (h - distance * (rowCount + 1)) / rowCount;          // 按钮高度
	    
	    // 将按钮添加到界面上
	    for (NSInteger i = 0; i < (rowCount * columnCount); i++)
	    {
	        CGFloat x = (i % columnCount) * (btnW + distance) + distance;
	        CGFloat y = (i / columnCount) * (btnH + distance) + distance;
	        CGRect frame = CGRectMake(x, y, btnW, btnH);
	        
	        MyButton *btn = [[MyButton alloc] init];
	        btn.accessibilityLabel = [NSString stringWithFormat:@"%@", @(i)];
	        btn.frame = frame;
	        
	        [btn addTarget:self action:@selector(clickBtn:) forControlEvents:UIControlEventTouchUpInside];
	        [self.view addSubview:btn];
	    }
	}

一共有9行，每行5个按钮，每个按钮尺寸大小相等，按钮与按钮之间、按钮与屏幕边缘之间的具体固定。

在视图控制器的生命周期***viewDidAppear***函数里将按钮添加上。

	- (void)viewDidAppear:(BOOL)animated
	{
	    [super viewDidAppear:animated];
	    
	    [self addButtons];
	}

按钮的布局需要用到视图控制器**view**的大小，而**view**的大小在*viewDidAppear:*时才确定下来，所以在这个时候才将按钮添加到界面上。

至此Demo App方面的代码实现完成，跑起来界面效果如上图所示。

<h3>编写自动化脚本</h3>

运行profile：xcode工具栏->Product->Profile，选择Automation选项

![image1]({{ site.url }}/assets/AutoTest/2.jpg)

之后进入界面，如下图所示，图中标记几处需要留意的地方。

![image1]({{ site.url }}/assets/AutoTest/3.jpg)

中间的黑色区域为编写JavaScript脚本的地方，系统会自己默认自动添加了行代码

	var target = UIATarget.localTarget();
	
编码区域上方的**Script**按钮点击展开还会有另外两个选项：**Trace Log**和**Editor Log**，分别用来打印对应的记录。

左上角的红色圈按钮，点击会运行程序。

图下方的三个按钮，第一个三角形图标的按钮，可以在程序已经跑起来的时候，实时更改脚本并运行；第二个红色圆形按钮用来录制操作，能将对App的每一次操作生成对应的脚本语句，下次直接跑脚本就能将之前的操作自动重复一次了。

右边还有按钮能导入导出脚本，给脚本文件改名，暂停继续脚本的运行等。

<h3>脚本的录制</h3>

点击下方的录制按钮，这时候程序运行起来，进行一系列操作，例如，从左上角开始，将外围的一圈按钮按顺时针顺序依次点击。结果如图：

![image1]({{ site.url }}/assets/AutoTest/4.jpg)

在点击的过程中会发现，每次点击操作在脚本编辑区域都会生成一行代码

![image1]({{ site.url }}/assets/AutoTest/5.jpg)

代码生成的速度会比实际的操作稍微延时。

开始以为生成的代码都是一样的都是

	target.frontMostApp()
	
所以就很好奇它是怎样具体去区分每一个按钮的呢？直到无意中把代码区域的拖动全选了下才发现。。。

![image1]({{ site.url }}/assets/AutoTest/7.jpg)

突然感受到深深的恶意...-_-

点击按钮之间的空白区域也会生成对应的脚本代码，包含了具体的点击位置数据

![image1]({{ site.url }}/assets/AutoTest/6.jpg)

录制完成之后点击旁边的正方形图标按钮。之后可以点击左上角红色圆形按钮，这时候程序运行起来，会自动运行刚刚录制所生成的脚本代码，这时候能看到看看点击了一轮按钮的操作又自己跑了一遍。

脚本的录制方式大概情况就是这样。

<h3>随机点击操作</h3>

接下来用非录制的方式创建个脚本，用来在界面上进行随机位置的点击。

随机点击屏幕脚本

	var target = UIATarget.localTarget();
	var app = target.frontMostApp();
	var window = app.mainWindow();
	
	var rect = target.rect();
	var w = rect.size.width;
	var h = rect.size.height;
	
	for (;;)
	{
	    var x = Math.random() * w;
	    var y = Math.random() * h;
	    target.tap({x, y});
	}

*target.rect()*获取的是屏幕界面的rect，包含了屏幕的尺寸大小。

通过*for*无限循环来产生随机的点击事件，点击的位置为屏幕范围内的任意点。**target**的*tap()*方法传递位置参数，并在界面上进行点击操作。

运行节本的过程中能够看到界面上按钮的值不停递增

![image1]({{ site.url }}/assets/AutoTest/14.jpg)

instrument上也不断有打印输出

![image1]({{ site.url }}/assets/AutoTest/15.jpg)

若想控制脚本点击事件的间隔，可以再每次点击之后加上**target**的*delay()*方法，参数为秒，延迟一定的时间后再继续运行脚本。

延迟2秒再继续

	target.delay(2);
	
可以输出log内容

	UIALogger.logMessage(msg);

<h3>每一次点击操作都针对按钮</h3>

有时候App上的按钮可能只有几个，全屏的随机点击事件点击到按钮的概率比较低，此时需要每次点击事件都针对按钮进行操作，脚本可以这样处理

	var target = UIATarget.localTarget();
	var app = target.frontMostApp();
	var window = app.mainWindow();
	
	var count = window.buttons().toArray().length;
	var msg = "total buttons: " + count.toString();
	UIALogger.logMessage(msg);
	
	for (;;)
	{
	    var index = Math.floor(Math.random() * count);
	    var btn = window.buttons()[index];
	    btn.tap();
	    target.delay(1);
	}

**window**的*buttons()*返回的是它所包含的所有按钮，为**UIAElementArray**类型，转换成Array后计算元素的数量。

***window.buttons()[index]*** **index**为按钮的索引值，可以通过下标索引的方式来获取其中的某一个按钮。

获取到按钮后通过*tap()*产生点击事件。每次操作间有个1秒的延迟。


<h3>结</h3>

自动化测试的方式基本用法如上所示。Demo展示的内容比较基本，还有各种更复杂的玩法和细节改天有空再研究了。还有一些第三方提供的自动化工具之类的，以后专门再写一篇文章。

脚本的代码直接复制粘贴上即可跑起来。

[工程源码地址](https://github.com/moshuqi/DemoCodes/tree/master/AutoTest)

转载请保留原文地址：https://moshuqi.github.io/ios/2016/03/08/iOS自动化测试-Automation.html

<h4>完。</h4>





	
