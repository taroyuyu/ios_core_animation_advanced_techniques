# 组透明


&nbsp;&nbsp;&nbsp;&nbsp;UIView有一个叫做`alpha`的属性来改变视图的透明度。CALayer有一个等价的属性叫做`opacity`（不透明度），这两个属性都**对层次结构有效**。也就是说，**如果你设置了一个图层的`opacity`，那它的子图层都会受此影响**。

&nbsp;&nbsp;&nbsp;&nbsp;iOS中，把一个控件呈现为不可用状态的常见做法是将其alpha属性设置为0.5（50%）。对于独立的视图来说还不错，但是当一个控件有子视图的时候就有点奇怪了，图4.20展示了一个内嵌了UILabel的自定义UIButton；左边是一个不透明的按钮，右边是50%透明度的相同按钮。我们可以注意到，里面的标签的轮廓跟按钮的背景很不搭调。

![图4.20](./4.20.png)

图4.20 右边的渐隐按钮中，里面的标签的边框清晰可见

&nbsp;&nbsp;&nbsp;&nbsp;这是由透明度的混合叠加造成的，**当你显示一个50%透明度的图层时，图层的每个像素都会一半显示自己的颜色，另一半显示图层下面的颜色**。这是正常的透明度的表现。但是如果图层包含一个同样显示50%透明的子图层时，你所看到的视图，50%来自子视图，25%来了图层本身的颜色，另外的25%则来自背景色。

&nbsp;&nbsp;&nbsp;&nbsp;在我们的示例中，按钮和标签都是白色背景。虽然他们都是50%的可见度，但是合起来的可见度（即不透明度）是75%，所以标签所在的区域看上去就没有周围的部分那么透明。所以看上去子视图就高亮了，使得这个显示效果都糟透了。

&nbsp;&nbsp;&nbsp;&nbsp;**理想状况下，当我们设置了一个图层的透明度，你希望它包含的整个图层树拥有像一个整体一样的透明效果。我们可以通过在Info.plist文件中将`UIViewGroupOpacity`为YES来达到这个效果，但是这个设置会影响到这个应用，整个app可能会受到不良影响**。如果`UIViewGroupOpacity`并未设置，iOS 6和以前的版本会默认为NO（也许以后的版本会有一些改变）。

&nbsp;&nbsp;&nbsp;&nbsp;另一个方法就是，你可以设置CALayer的一个叫做`shouldRasterize`属性（见清单4.7）来实现组透明的效果，**如果它被设置为YES，在应用透明度之前，图层及其子图层都会被整合成一个整体的图片，这样就没有透明度混合的问题了**（如图4.21）。

&nbsp;&nbsp;&nbsp;&nbsp;**将`shouldRasterize`属性设置为YES之后，我们还通常需要设置图层的`rasterizationScale`属性**。**默认情况下，所有图层都以1.0的比例光栅化。 所以如果你使用了`shouldRasterize`属性，所以如果你使用shouldRasterize属性，你应该总是确保你设置的光栅化比例与屏幕匹配，以防止在Retina屏幕上出现像素化的问题**。

&nbsp;&nbsp;&nbsp;&nbsp;和UIViewGroupOpacity一样，使用shouldRasterize属性会影响性能（我们在第12章『速度』和第15章『图层性能』将做出介绍），但性能影响是本地化的（译者注：这句话需要再翻译）。

清单4.7 使用`shouldRasterize`属性解决组透明问题

```objective-c
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *containerView;
@end

@implementation ViewController

- (UIButton *)customButton
{
  //create button
  CGRect frame = CGRectMake(0, 0, 150, 50);
  UIButton *button = [[UIButton alloc] initWithFrame:frame];
  button.backgroundColor = [UIColor whiteColor];
  button.layer.cornerRadius = 10;

  //add label
  frame = CGRectMake(20, 10, 110, 30);
  UILabel *label = [[UILabel alloc] initWithFrame:frame];
  label.text = @"Hello World";
  label.textAlignment = NSTextAlignmentCenter;
  [button addSubview:label];
  return button;
}

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create opaque button
  UIButton *button1 = [self customButton];
  button1.center = CGPointMake(50, 150);
  [self.containerView addSubview:button1];

  //create translucent button
  UIButton *button2 = [self customButton];
  ￼
  button2.center = CGPointMake(250, 150);
  button2.alpha = 0.5;
  [self.containerView addSubview:button2];

  //enable rasterization for the translucent button
  button2.layer.shouldRasterize = YES;
  button2.layer.rasterizationScale = [UIScreen mainScreen].scale;
}
@end
```

![图4.12](./4.21.png)

图4.21 修正后的图



https://www.appcoda.com.tw/calayer-introduction/

https://www.jianshu.com/p/b29c682679c4

https://www.jianshu.com/p/c214c30058ab

https://cloud.tencent.com/developer/ask/108961

https://stackoverflow.com/questions/7129902/any-way-to-disable-calayers-rasterizationscale-interpolation-anti-aliasing-in

