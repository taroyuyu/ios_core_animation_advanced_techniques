# 拉伸过滤


&nbsp;&nbsp;&nbsp;&nbsp;最后我们来谈谈`minificationFilter`和`magnificationFilter`这两个属性。**通常在iOS上，当你显示图像时，你应该尝试以正确的大小显示它们(也就是说，图像中的像素和屏幕上的像素之间有1:1的相关性)**。原因如下：

* 能够显示最好的画质，像素既没有缩小(即重新采样)也没有被拉伸。
* 能更好的使用内存，因为存储的像素不会超过所需要的数量。
* 最好的性能表现，GPU不需要为此额外的计算。

&nbsp;&nbsp;&nbsp;&nbsp;不过有时候，有必要以比实际大小大或比实际大小小的尺寸显示图像。比如说一个头像或是图片的缩略图(图像缩小)，再比如说一个可以被拖拽和伸缩的大图（图像拉伸）。这些情况下，为同一图片的不同大小存储不同的图片显得又不切实际。

&nbsp;&nbsp;&nbsp;&nbsp;**当图像以不同大小显示时，一种叫做*拉伸过滤(scaling filter)*的算法被应用到原始图像的像素上，以生成将显示在屏幕上的新像素**。

&nbsp;&nbsp;&nbsp;&nbsp;调整图片大小(Resize)没有一个统一的通用算法。这取决于需要拉伸的内容的特性，以及是放大还是是缩小。`CALayer`为此提供了三种拉伸过滤器方法，它们由以下字符串常量表示:：

* kCAFilterLinear
* kCAFilterNearest
* kCAFilterTrilinear

&nbsp;&nbsp;&nbsp;&nbsp;**minification（缩小图片）和magnification（放大图片）默认的过滤器都是`kCAFilterLinear`**，这个过滤器采用双线性滤波算法，它在大多数情况下都表现良好。双线性滤波算法通过**对多个像素取样最终生成新的值，这样的缩放效果很好，很平滑。但是如果放大一倍，图像就会变得模糊**。

&nbsp;&nbsp;&nbsp;&nbsp;`kCAFilterTrilinear`和`kCAFilterLinear`非常相似，大部分情况下二者都看不出来有什么差别。但`kCAFilterTrilinear`算法较`kCAFilterLinear`效果更好，其通过**将图像存储为多个不同大小（也叫多重贴图），并三维取样，同时结合大图和小图的存储进而得到最后的结果**。

&nbsp;&nbsp;&nbsp;&nbsp;这个方法的优点在于算法能够**从一系列已经接近于最终大小的图片中得到想要的结果，也就是说不要对很多像素同时取样。这不仅提高了性能，也避免了小概率因舍入错误引起的取样失灵的问题**。

![图4.14](./4.14.png)

图4.14 对于大图来说，双线性滤波和三线性滤波表现得更出色

&nbsp;&nbsp;&nbsp;&nbsp;`kCAFilterNearest`是一种比较武断的方法。从名字不难看出，这个算法（也叫最近邻滤过滤算法）就是**取样最近的单像素点而不管其他的颜色。这样做非常快，也不会使图片模糊。但是，最明显的效果就是，会使得压缩图片更糟，图片放大之后也显得块状或是马赛克严重**。

![图4.15](./4.15.png)

图4.15 对于没有斜线的小图来说，最近过滤算法要好很多

&nbsp;&nbsp;&nbsp;&nbsp;总的来说，对于比较小的图或者是差异特别明显，极少斜线的大图，最近过滤算法会保留这种差异明显的特质以呈现更好的结果。但是对于大多数的图尤其是有很多斜线或是曲线轮廓的图片来说，最近过滤算法会导致更差的结果。换句话说，线性过滤保留了形状，最近过滤则保留了像素的差异。

&nbsp;&nbsp;&nbsp;&nbsp;让我们来实验一下。我们对第三章的时钟项目改动一下，用LCD风格的数字方式显示。我们用简单的像素字体（一种用像素构成字符的字体，而非矢量图形）创造数字显示方式，用图片存储起来，而且用第二章介绍过的拼合技术来显示（如图4.16）。

![图4.16](./4.16.png)

图4.16 一个简单的运用拼合技术显示的LCD数字风格的像素字体

&nbsp;&nbsp;&nbsp;&nbsp;我们在Interface Builder中放置了六个视图，小时、分钟、秒钟各两个，图4.17显示了这六个视图是如何在Interface Builder中放置的。如果每个都用一个淡出的outlets对象就会显得太多了，所以我们就用了一个`IBOutletCollection`对象把他们和控制器联系起来，这样我们就可以以数组的方式访问视图了。清单4.6是代码实现。

清单4.6 显示一个LCD风格的时钟

```objective-c
@interface ViewController ()

@property (nonatomic, strong) IBOutletCollection(UIView) NSArray *digitViews;
@property (nonatomic, weak) NSTimer *timer;
￼￼
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad]; //get spritesheet image
  UIImage *digits = [UIImage imageNamed:@"Digits.png"];

  //set up digit views
  for (UIView *view in self.digitViews) {
    //set contents
    view.layer.contents = (__bridge id)digits.CGImage;
    view.layer.contentsRect = CGRectMake(0, 0, 0.1, 1.0);
    view.layer.contentsGravity = kCAGravityResizeAspect;
  }

  //start timer
  self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];

  //set initial clock time
  [self tick];
}

- (void)setDigit:(NSInteger)digit forView:(UIView *)view
{
  //adjust contentsRect to select correct digit
  view.layer.contentsRect = CGRectMake(digit * 0.1, 0, 0.1, 1.0);
}

- (void)tick
{
  //convert time to hours, minutes and seconds
  NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier: NSGregorianCalendar];
  NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
  ￼
  NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];

  //set hours
  [self setDigit:components.hour / 10 forView:self.digitViews[0]];
  [self setDigit:components.hour % 10 forView:self.digitViews[1]];

  //set minutes
  [self setDigit:components.minute / 10 forView:self.digitViews[2]];
  [self setDigit:components.minute % 10 forView:self.digitViews[3]];

  //set seconds
  [self setDigit:components.second / 10 forView:self.digitViews[4]];
  [self setDigit:components.second % 10 forView:self.digitViews[5]];
}
@end
```

如图4.18，这样做的确起了效果，但是图片看起来模糊了。看起来默认的`kCAFilterLinear`选项让我们失望了。

![图4.18](./4.18.png)

图4.18 一个模糊的时钟，由默认的`kCAFilterLinear`引起

&nbsp;&nbsp;&nbsp;&nbsp;为了能像图4.19中那样，我们需要在for循环中加入如下代码：

```objective-c
view.layer.magnificationFilter = kCAFilterNearest;
```

![图4.19](./4.19.png)

图4.19 设置了最近过滤之后的清晰显示



http://kmanong.top/kmn/qxw/form/article?id=76526&cate=63

https://www.jianshu.com/p/1d8967521cc4

