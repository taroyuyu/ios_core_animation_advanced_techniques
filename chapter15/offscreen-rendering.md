## 离屏渲染

&nbsp;&nbsp;&nbsp;&nbsp;**只要指定的图层属性组合意味着无法进行预合成就无法直接将图层绘制到屏幕上，就会进行*屏幕外渲染/离屏渲染***。**屏幕外渲染并不一定意味着软件绘图，但它意味着在图层显示之前，必须首先将图层（由CPU或GPU）渲染到屏幕外的上下文中**。触发屏幕外渲染的层属性如下：

* **圆角（当和`maskToBounds`一起使用时）**
* **图层蒙板**
* **阴影**

&nbsp;&nbsp;&nbsp;&nbsp;***离屏渲染*和我们启用光栅化所发生的事情很相似，但[1]它并不会像光栅化图层那样消耗额外的内存，[2]其子图层并不会受到影响，而且[3]*离屏渲染*的结果不会被缓存（即子图层不一定会进行离屏渲染。但光栅化，会将整个图层的内容包括子图层的内容绘制到一个单独的上下文中），因此没有长期内存的影响。但是，如果有太多图层进行*离屏渲染*则依然会对性能产生显著影响**。

**对于需要进行*离屏渲染*的图层，如果它和它的子图层都不需要频繁的重绘，那么启用栅格化（即将`shouldRasterize`设置为true）是有益的**。

**对于需要进行*离屏渲染*并需要动画（或其子图层需要动画，即需要频繁地重绘）的图层，我们可以使用CAShapeLayer，contentsCenter或shadowPath来实现相似的外观，而对性能的影响较小**。 

&nbsp;&nbsp;&nbsp;&nbsp;**对于那些需要动画而且要在屏幕外渲染的图层来说，你可以用`CAShapeLayer`，`contentsCenter`或者`shadowPath`来获得同样的表现而且较少地影响到性能**。

### CAShapeLayer

&nbsp;&nbsp;&nbsp;&nbsp;**`cornerRadius`和`maskToBounds`独立作用的时候都不会有太大的性能问题，但是当他俩结合在一起，就触发了屏幕外渲染**。**有时候你想显示圆角并沿着图层裁切子图层的时候，你可能会发现你并不需要沿着圆角裁切，这个情况下用`CAShapeLayer`就可以避免这个问题了**。

&nbsp;&nbsp;&nbsp;&nbsp;通过使用方便的+bezierPathWithRoundedRect:cornerRadius: UIBezierPath的构造函数(参见清单15.1)绘制圆角矩形，您可以获得圆角的效果，并且仍然可以剪切到层的(矩形)边界，而不会带来性能开销。这并不比使用cornerRadius快，但意味着masksToBounds属性不再带来性能损失。

清单15.1 用`CAShapeLayer`画一个圆角矩形

```objective-c
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>

@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //create shape layer
    CAShapeLayer *blueLayer = [CAShapeLayer layer];
    blueLayer.frame = CGRectMake(50, 50, 100, 100);
    blueLayer.fillColor = [UIColor blueColor].CGColor;
    blueLayer.path = [UIBezierPath bezierPathWithRoundedRect:
    CGRectMake(0, 0, 100, 100) cornerRadius:20].CGPath;
    ￼
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```

### 可伸缩图片

&nbsp;&nbsp;&nbsp;&nbsp;另一个创建圆角矩形的方法就是用一个圆形内容图片并结合第二章『寄宿图』提到的**`contensCenter`属性去创建一个可伸缩图片（见清单15.2）.理论上来说，这个应该比用`CAShapeLayer`要快，因为一个可拉伸图片只需要18个三角形（一个图片是由一个3*3网格渲染而成），然而，许多都需要渲染成一个顺滑的曲线。在实际应用上，二者并没有太大的区别**。

清单15.2 用可伸缩图片绘制圆角矩形

```objective-c
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    //create layer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50, 50, 100, 100);
    blueLayer.contentsCenter = CGRectMake(0.5, 0.5, 0.0, 0.0);
    blueLayer.contentsScale = [UIScreen mainScreen].scale;
    blueLayer.contents = (__bridge id)[UIImage imageNamed:@"Circle.png"].CGImage;
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```

&nbsp;&nbsp;&nbsp;&nbsp;**使用可伸缩图片的优势在于它可以绘制成任意边框效果而不需要额外的性能消耗。举个例子，可伸缩图片甚至还可以显示出矩形阴影的效果**。

### shadowPath

&nbsp;&nbsp;&nbsp;&nbsp;在第2章我们有提到`shadowPath`属性。如果图层是一个简单几何图形如矩形或者圆角矩形（假设不包含任何透明部分或者子图层），创建出一个对应形状的阴影路径就比较容易，而且Core Animation绘制这个阴影也相当简单，避免了屏幕外的图层部分的预排版需求。这对性能来说很有帮助。

&nbsp;&nbsp;&nbsp;&nbsp;如果你的图层是一个更复杂的图形，生成正确的阴影路径可能就比较难了，这样子的话你可以考虑用绘图软件预先生成一个阴影背景图。

