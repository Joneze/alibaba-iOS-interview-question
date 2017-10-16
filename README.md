#阿里三面面试题：
* 1.dSYM你是如何分析的？
* 2.多线程有哪几种？你更倾向于哪一种？
* 3.单例弊端？
* 4.如何把异步线程转换成同步任务进行单元测试？
* 5.介绍下App启动的完成过程？
* 6.比如App启动过慢，你可能想到的因素有哪些？
* 7.0x8badf00d表示是什么？
* 8.怎么防止反编译？
* 9.说说你遇到到的技术难点？
* 10.说说你了解的第三方原理或底层知识？

### 1、dSYM你是如何分析的
#### 方法1 使用XCode
这种方法可能是最容易的方法了。
1. 要使用Xcode符号化 crash log，你需要下面所列的3个文件：
2. crash报告（.crash文件）
3. 符号文件 (.dsymb文件)
4. 应用程序文件 (appName.app文件，把IPA文件后缀改为zip，然后解压，Payload目录下的appName.app文件), 这里的appName是你的应用程序的名称。
5. 把这3个文件放到同一个目录下，打开Xcode的Window菜单下的organizer，然后点击Devices tab，然后选中左边的Device Logs。
6. 然后把.crash文件拖到Device Logs或者选择下面的import导入.crash文件。
7. 这样你就可以看到crash的详细log了。

#### 方法2 使用命令行工具symbolicatecrash
1. 有时候Xcode不能够很好的符号化crash文件。我们这里介绍如何通过symbolicatecrash来手动符号化crash log。
2. 在处理之前，请依然将“.app“, “.dSYM”和 ".crash"文件放到同一个目录下。现在打开终端(Terminal)然后输入如下的命令：
3. export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
4. 然后输入命令：
5. /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash appName.crash appName.app > appName.log
6. 现在，符号化的crash log就保存在appName.log中了。

#### 方法3 使用命令行工具atos
1. 如果你有多个“.ipa”文件，多个".dSYMB"文件，你并不太确定到底“dSYMB”文件对应哪个".ipa"文件，那么，这个方法就非常适合你。
2. 特别当你的应用发布到多个渠道的时候，你需要对不同渠道的crash文件，写一个自动化的分析脚本的时候，这个方法就极其有用。
3. 具体方法 请百度

> 本文分析了拿到用户的.crash文件之后，如何符合化crash文件的3种方法，分别有其适用场景，方法3适用于自动化crash文件的分析。

### 2.多线程有哪几种？你更倾向于哪一种？
1. NSThread 
2. Cocoa NSOperation （使用NSOperation和NSOperationQueue）
3. GCD  （Grand Central Dispatch）

**1.NSThread:(两种创建方式)**

```
[NSThread detachNewThreadSelector:@selector(doSomething:) toTarget:self withObject:nil];

NSThread *myThread = [[NSThread alloc] initWithTarget:self selector:@selector(doSomething:) object:nil];

[myThread start];

```
优点：NSThread 比其他两个轻量级。
缺点：需要自己管理线程的生命周期，线程同步，线程同步时对数据的加锁会有一定的系统开销。

**2.Cocoa Operation **

```
NSOperationQueue*oprationQueue= [[NSOperationQueuealloc] init];

oprationQueueaddOperationWithBlock:^{

//这个block语句块在子线程中执行

}
```
优点：不需要关心线程管理，数据同步的事情。
Cocoa Operation 相关的类是 NSOperation ，NSOperationQueue。NSOperation是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的两个子类：NSInvocationOperation 和 NSBlockOperation。创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行，我们会把我们的执行操作放在NSOperation中main函数中。

**3.GCD**
Grand Central Dispatch (GCD)是Apple开发的一个多核编程的解决方法，GCD是一个替代诸如NSThread, NSOperationQueue, NSInvocationOperation等技术的很高效和强大的技术。它让程序平行排队的特定任务，根据可用的处理资源，安排他们在任何可用的处理器核心上执行任务，一个任务可以是一个函数(function)或者是一个block。
dispatch queue分为下面三种：
private dispatch queues，同时只执行一个任务，通常用于同步访问特定的资源或数据。
global dispatch queue，可以并发地执行多个任务，但是执行完成的顺序是随机的。
Main dispatch queue 它是在应用程序主线程上执行任务的。


###3、单利的弊端
**优点**：
1：一个类只被实例化一次，提供了对唯一实例的受控访问。
2：节省系统资源
3：允许可变数目的实例。

**缺点：**
1：一个类只有一个对象，可能造成责任过重，在一定程度上违背了“单一职责原则”。
2：由于单利模式中没有抽象层，因此单例类的扩展有很大的困难。
3：滥用单例将带来一些负面问题，如为了节省资源将数据库连接池对象设计为的单例类，可能会导致共享连接池对象的程序过多而出现连接池溢出；如果实例化的对象长时间不被利用，系统会认为是垃圾而被回收，这将导致对象状态的丢失。

###4、如何把异步线程转换成同步任务进行单元测试？
[参考此篇博客](http://blog.csdn.net/Jason_chen13/article/details/50549387) 里面的信号量的解释，Dispatch Semaphore 信号量 [在项目中的应用：强制把异步任务转换为同步任务来方便进行单元测试](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse%E7%9A%84%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86%E6%80%9D%E8%B7%AF/Parse%E7%9A%84%E5%BA%95%E5%B1%82%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%A4%84%E7%90%86%E6%80%9D%E8%B7%AF.md#%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8%E5%BC%BA%E5%88%B6%E6%8A%8A%E5%BC%82%E6%AD%A5%E4%BB%BB%E5%8A%A1%E8%BD%AC%E6%8D%A2%E4%B8%BA%E5%90%8C%E6%AD%A5%E4%BB%BB%E5%8A%A1%E6%9D%A5%E6%96%B9%E4%BE%BF%E8%BF%9B%E8%A1%8C%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
###5、介绍下App启动的完成过程？

* ①.先加载Main函数

* ②.在Main函数里的 UIApplicationMain方法中，创建Application对象 创建Application的Delegate对象

* ③.创建主循环，代理对象开始监听事件

* ④.启动完毕会调用 didFinishLaunching方法，并在这个方法中创建UIWindow

* ⑤.设置UIWindow的根控制器是谁

* ⑥.如果有storyboard，会根据info.plist中找到应用程序的入口storyboard并加载箭头所指的控制器

* ⑦.显示窗口



> 本文考虑的时步骤③之后到步骤⑦结束时将要调用的方法

> 其中有AppDelegate,ViewController,MainView（控制器的View）,ChildView（子控件的View）的18个方法

****
**AppDelegate中的：**

* 1.application:didFinishLaunchingWithOptions:

* 2.applicationDidBecomeActive:


**ViewController中的：**

* 3.loadView

* 4.viewDidLoad

* 5.load

* 6.initialize

* 7.viewWillAppear

* 8.viewWillLayoutSubviews

* 9.viewDidLayoutSubviews

* 10.viewDidAppear


**MainView（控制器的View）中的**：

* 11.initWithCoder（如果没有storyboard就会调用initWithFrame，这里两种方法视为一种）

* 12.awakeFromNib

* 13.layoutSubviews

* 14.drawRect

**ChildView（子控件View）中的：**

* 15.initWithCoder（如果没有storyboard就会调用initWithFrame，这里两种方法视为一种）

* 16.awakeFromNib

* 17.layoutSubviews

* 18.drawRect


> 那么问题来了，不往下看你可以把上面的十八个方法排个顺序么？

![](media/15078580598578/15078599629062.jpg)


```
+ (void)load; //这是应用程序启动就会调用的方法，在这个方法里写的代码最先调用
```


```
+ (void)initialize; //这个是需要用到本类时才调用，这个方法里一般写设置导航控制器的主题啊之类的，
//如果在后面的方法设置导航栏主题就晚了！（当然在上面的方法里也能写）
```


```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions;
//这个方法里面会创建UIWindow，设置根控制器并展现，
//比如某些应用程序要加载授权页面也是在这加，也可以设置观察者，监听到通知切换根控制器
```


```
ChildView - (instancetype)initWithCoder:(NSCoder *)aDecoder;
//这里反正我是万万没想到，childView的initwithcoder会在MainView的方法之前调用，
//父的都还没出来，就先整子控件？ 有了解比较透彻的博友恳请告诉我谢谢。
```


```
MainView - (instancetype)initWithCoder:(NSCoder *)aDecoder;
// 就是关于应用程序的数据存储后的解档操作。
```


```
MainView - (void)awakeFromNib;
//在这个方法里设置view的背景等一系列普通操作，不要写关于frame的还不准，
//在使用IB的时候才会涉及到此方法的使用，当.nib文件被加载的时候，
//会发送一个awakeFromNib的消息到.nib文件中的每个对象，
//每个对象都可以定义自己的awakeFromNib函数来响应这个消息，执行一些必要的操作。 
```

```
ChildView - (void)awakeFromNib
//子控件也有本方法，重写父类的方法。基本用法同上 
```


```
- (void)loadView; 
//创建视图的层次结构，这里需要注意，
//在没有创建控制器的view的情况下不能直接写 self.view 因为self.view的底层是：
    if（_view == nil）{
　   　_view = [self loadView]
    }
//所以这么写会直接造成死循环。
//如果重写这个loadView方法里面什么都不写，会显示黑屏。
```

```
- (void)viewDidLoad;
//卧槽，这个方法是用的最多的方法，但是在之后的开发中就会发现越来越不靠谱，
//很多东西都还没加载完毕，各种取值都不准确，很少在这里面写东西了。 
//这里只是把视图元件加载完成
```


```
- (void)viewWillAppear:(BOOL)animated;
//视图将要出现，这个方法用的非常多，比如如果要设置导航栏的setNavigationBarHiden:animate: 
//就必须要在这里写，才能完美契合，不卡跳。 还有很多比如监听屏幕旋转啦，

//viewWillTransitionToSize:可能要在本方法里再调一次，
//或者就是新到这个界面要reloadData或是自动下拉刷新等 都是写在本方法里
```


```
- (void)viewWillLayoutSubviews;
//视图将要布局子视图，苹果建议的设置界面布局属性的方法，
//这个方法和viewWillAppear里，系统的底层都是没有写任何代码的，也就是说这里面不写super 也是可以的
```


```
MainView  - (void)layoutSubviews;
//在这个方法里一般设置子控件的frame，因为这里相当于是布局基本完成了，
//设置时取到的frame或者是self.bounds才最准，如果在awakeFromeNib里写会不准确 。
//还有这里要切记千万不能把super layoutSubviews忘了，可能最后都很难找到这个bug
```

```
- (void)viewDidLayoutSubviews;
//这个方法我也是玩玩没想到，控制器的view的子控件还没有布局好呢，怎么这个控制器就已经说布局全部完成了？
//那后边的布局就不等了？ 有独到见解的也恳请你告诉我，这其中苹果的意思到底是什么。 
```
    
```
ChildView - (void)layoutSubviews;
//控制器的子控件里的子控件的布局就在这里写了。 
```

```
MainView - (void)drawRect:(CGRect)rect;
//因为默认所有额UI控件都是画上去的，在这一步就是把所有的东西画上去，
//有时候需要用到Quartz2D的知识的时候都是在这个方法里话，但也是要注意别忘了写super，
//不然系统原本的东西就都画不上来了，这里要建议尽可能使用贝塞尔路径画图形，
//因为系统默认的那个上下文画法有时可能会内存泄露。drawRect方法只能在加载时调用一次，
//如果后面还需要调用，比如下载进度的圆弧，需要一直刷帧，
//就要使用setNeedsDisplay来定时多次调用本方法
```

```
ChildView - (void)drawRect:(CGRect)rect;
//view的子控件内部的画图方法，有时可以自己自定义label 中间带个删除线的（用来写打折前的原价） 就是在这里画根线 。
```


```
- (void)viewDidAppear:(BOOL)animated;
//把上面的画图都画完了，这里就会显示，视图完全加载完成。
//在这里的操作可能就是设置页面的一些动画,或者是设置tableView，collectionView，
//QQ聊天页面啥的滚动到底部scrollToIndexPath之类的代码操作。
```

```
- (void)applicationDidBecomeActive:(UIApplication *)application;
//最后这是AppDelegate的应用程序获取焦点方法，真正到了这里，才是所有东西全部加载完毕，应用程序整装待发保持最佳状态等待用户操作。
//这个方法中一般会写关于弹出键盘的方法，比如有的用户登录界面为了更好的用户体验，
//就让你在刚打开程序来到登录界面的时候，光标的焦点就自动在账号的文本框里闪烁，
//也就是设置账号文本框为第一响应者。键盘在页面加载完毕后从下方弹出，这种代码一般就在本方法写。
```


###6、比如App启动过慢，你可能想到的因素有哪些？
###### 1. App启动过程

1. 解析Info.plist 
    * 加载相关信息，例如如闪屏
    * 沙箱建立、权限检查
2. Mach-O加载 
    * 如果是胖二进制文件，寻找合适当前CPU类别的部分
    * 加载所有依赖的Mach-O文件（递归调用Mach-O加载的方法）
    * 定位内部、外部指针引用，例如字符串、函数等
    * 执行声明为__attribute__((constructor))的C函数
    * 加载类扩展（Category）中的方法
    * C++静态对象加载、调用ObjC的 +load 函数  
3. 程序执行 
    * 调用main()
    * 调用UIApplicationMain()
    * 调用applicationWillFinishLaunching

###### 2、影响启动性能的因素
1. main()函数之前耗时的影响因素
    * 动态库加载越多，启动越慢。
    * ObjC类越多，启动越慢
    * C的constructor函数越多，启动越慢
    * C++静态对象越多，启动越慢
    * ObjC的+load越多，启动越慢

2. main()函数之后耗时的影响因素
    * 执行main()函数的耗时
    * 执行applicationWillFinishLaunching的耗时
    * rootViewController及其childViewController的加载、view及其subviews的加载

> 另外参考一下今日头条的启动优化方案

，针对于今日头条这个App我们可以优化的点如下：

* 纯代码方式而不是storyboard加载首页UI。
* 对didFinishLaunching里的函数考虑能否挖掘可以延迟加载或者懒加载，需要与各个业务方pm和rd共同check 对于一些已经下线的业务，删减冗余代码。
* 对于一些与UI展示无关的业务，如微博认证过期检查、图片最大缓存空间设置等做延迟加载。
* 对实现了+load()方法的类进行分析，尽量将load里的代码延后调用。
* 上面统计数据显示展示feed的导航控制器页面(NewsListViewController)比较耗时，对于viewDidLoad以及viewWillAppear方法中尽量去尝试少做，晚做，不做。

### 7、0x8badf00d表示是什么？
看门狗超时，在iOS上，它经常出现在执行一个同步网络调用而阻塞主线程的情况。因此，永远不要进行同步网络调用。

### 8、怎么防止反编译？
1. 本地数据加密。
    * iOS应用防反编译加密技术之一：对NSUserDefaults，sqlite存储文件数据加密，保护帐号和关键信息
2. URL编码加密。
    * iOS应用防反编译加密技术之二：对程序中出现的URL进行编码加密，防止URL被静态分析
3. 网络传输数据加密。
    * iOS应用防反编译加密技术之三：对客户端传输数据提供加密方案，有效防止通过网络接口的拦截获取数据
4. 方法体，方法名高级混淆。
    * iOS应用防反编译加密技术之四：对应用程序的方法名和方法体进行混淆，保证源码被逆向后无法解析代码
5. 程序结构混排加密。
    * iOS应用防反编译加密技术之五：对应用程序逻辑结构进行打乱混排，保证源码可读性降到最低

> 其实我觉得大可不必，本身反编译成本就很大，代码这么多，一个个反编译过来是在蛋疼，就算有伪代码也需要理解，而且有些代码就算有伪代码也很难理解。

> 只要做好核心代码，做好混淆就行了，比如涉及到密码，核心算法。

### 9、说说你遇到到的技术难点？
自行发挥吧！
### 10、说说你了解的第三方原理或底层知识？
自行发挥吧！






