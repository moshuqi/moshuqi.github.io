---
layout: post
title:  iOS 自动化测试 UI Automation
category: "iOS"
---

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

![image1]({{ site.url }}/assets/AutoTest/8.jpg)









	
