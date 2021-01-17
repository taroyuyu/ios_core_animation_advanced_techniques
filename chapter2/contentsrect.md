# contentsRect


&nbsp;&nbsp;&nbsp;&nbsp;**CALayer的`contentsRect`属性允许我们在图层中显示寄宿图的一小块区域(即不显示整个寄宿图)**。这图像的裁剪和拉伸方面，*contentsRect*提供的功能比使用*contentsGravity*更加灵活。

&nbsp;&nbsp;&nbsp;&nbsp;**和`bounds`‘、`frame`不同，`contentsRect`不是以点作为单位，而是使用了*单位坐标*，单位坐标的取值在0到1之间，是一个相对值（相对于像素和点这样的绝对值）。所以他们是相对与寄宿图的尺寸的。**iOS使用了以下的*坐标类型*：

* 点 —— 在iOS和Mac OS中最常见的坐标类型。**点就像是虚拟的像素，也被称作逻辑像素**。**在标准清晰度的设备上，一个点就是一个像素，但是在Retina设备上，一个点等于2×2个像素。**iOS使用点来测量所有屏幕坐标，就是为了使布局在Retina设备和普通设备上能有一致的视觉效果。
* 像素 —— **物理像素坐标并不会用来进行布局操作，但是与图像处理仍然有关系**。**UIImage是一个屏幕分辨率解决方案，以*点*来表示其大小**。但是一些底层的图片表示，如CGImage，就会使用像素坐标，所以你要清楚在Retina设备和普通设备上，他们表现出来了不同的大小。
* 单位 —— 单位坐标是一种方便的方式，用于指定相对于图像大小或图层边界的度量值。因此，如果尺寸发生变化，不需要进行调整。单位坐标在OpenGL中用于纹理坐标等内容，它们也经常用于核心动画中。

&nbsp;&nbsp;&nbsp;&nbsp;**`contentsRect`的默认值是{0, 0, 1, 1}，这意味着整个*寄宿图*默认都是可见的**，如果我们指定一个小一点的矩形，图片就会被裁剪（如图2.6）

![图2.6](./2.6.png)

图2.6 一个自定义的`contentsRect`（左）和之前显示的内容（右）

&nbsp;&nbsp;&nbsp;&nbsp;**事实上将`contentsRect`的原点设置成负数或者将其尺寸设置成大于{1, 1}也是可以的。这种情况下，最外面的像素会被拉伸以填充剩下的区域。**

（什么意思呢？）

&nbsp;&nbsp;&nbsp;&nbsp;`contentsRect`在app中最有趣的地方在于一个叫做*image sprites*（图片合并/图片精灵）的用法。如果你有游戏编程的经验，那么你一定对图片合并的概念很熟悉，图片能够在屏幕上独立地变更位置。抛开游戏编程不谈，这个技术常用来指代载入拼合的图片，跟移动图片一点关系也没有。

注：https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/CSS_Image_Sprites

https://www.jianshu.com/p/f0453e4a85eb

https://www.w3school.com.cn/css/css_image_sprites.asp

https://zhuanlan.zhihu.com/p/32119908

https://blog.csdn.net/qq_29837295/article/details/99304043

https://juejin.cn/post/6844904040887910414

https://www.shejidaren.com/css-sprites-optimize-retina-2x-image.html

https://www.yuque.com/freedom93/ykx/eklqfh

https://www.dazhuanlan.com/2019/12/16/5df6e4880f84e/?__cf_chl_jschl_tk__=c6d71539032371641041cfc376ea18f6711010ac-1610866222-0-AcWRMO_RtvF_L2LsY1CurHaVs8jzBxEpehg-C2R6ksC2pIh7wnFo97eBD8OtiHoQ3veDZEOGH1Oi1SqIuxdT56ChIJlU9u30kJnCv9a--tKp4O-vh4R0GV-qLRy_xOmVlHU0ErsayDanYi36s82IAX78mlGgTVSJIAVcU-aCG0laRpbMM4i3x3BPbEa5Aoi1O2fWeekeCXamD6hkerW6_wRbClG6FbuOHCjwKL0ewYgwKi6GyiVPJO4SpNTCCCevnOABfquZzktsqJwvB0TDLew_-S2srm4o7gTKRurQTkyWt1A17xCmRNUPY4loOCMZkr2UdkQjNnzBFwZFojzEXKw

https://designingforperformance.com/optimizing-images/

&nbsp;&nbsp;&nbsp;&nbsp;通过图像合并技术，许多精灵会被打包成一个大图像从而只需加载一次。在内存使用、加载时间和呈现性能方面，与使用多个单独的图像文件相比，这带来了各种好处。

&nbsp;&nbsp;&nbsp;&nbsp;2D游戏引擎，例如Cocos2D，使用了拼合技术，它使用OpenGL来显示图片。不过我们可以利用contentsRect的强大功能在普通UIKit应用程序中使用精灵。

&nbsp;&nbsp;&nbsp;&nbsp;首先，我们需要一个拼合后的图表（称为*精灵表(sprite sheet)*） —— 一个包含一些小图片(精灵图)的大图片。如图2.7所示：

![图2.7](./2.7.png)

&nbsp;&nbsp;&nbsp;&nbsp;接下来，我们要在app中载入并显示这些*精灵*。规则很简单：像平常一样载入我们的大图，然后把它赋值给四个独立的图层的`contents`属性，然后设置每个图层的`contentsRect`属性来指定对应精灵的区域。

&nbsp;&nbsp;&nbsp;&nbsp;我们的工程中需要一些额外的视图。（为了避免太多代码。我们将使用Interface Builder来定位他们的位置，如果你愿意还是可以用代码的方式来实现的）。清单2.3有需要的代码，图2.8展示了结果

```objective-c

@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *coneView;
@property (nonatomic, weak) IBOutlet UIView *shipView;
@property (nonatomic, weak) IBOutlet UIView *iglooView;
@property (nonatomic, weak) IBOutlet UIView *anchorView;
@end

@implementation ViewController

- (void)addSpriteImage:(UIImage *)image withContentRect:(CGRect)rect ￼toLayer:(CALayer *)layer //set image
{
  layer.contents = (__bridge id)image.CGImage;

  //scale contents to fit
  layer.contentsGravity = kCAGravityResizeAspect;

  //set contentsRect
  layer.contentsRect = rect;
}

- (void)viewDidLoad
{
  [super viewDidLoad]; //load sprite sheet
  UIImage *image = [UIImage imageNamed:@"Sprites.png"];
  //set igloo sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0, 0, 0.5, 0.5) toLayer:self.iglooView.layer];
  //set cone sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0, 0.5, 0.5) toLayer:self.coneView.layer];
  //set anchor sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0, 0.5, 0.5, 0.5) toLayer:self.anchorView.layer];
  //set spaceship sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0.5, 0.5, 0.5) toLayer:self.shipView.layer];
}
@end
```
![图2.8](./2.8.png)

&nbsp;&nbsp;&nbsp;&nbsp;*图像合并*技术是一种简洁的方式减少应用程序的大小和提高图像加载性能（单张大图比多张小图载入地更快,压缩性能更好），但是如果仅通过手动设置精灵图的话，他们还是有一些不方便的，如果你需要在一个已经创建好的品和图上做一些尺寸上的修改或者其他变动，无疑是比较麻烦的。

&nbsp;&nbsp;&nbsp;&nbsp;Mac上有一些商业软件可以为你自动拼合图片，这些工具自动生成一个包含拼合后的坐标的XML或者plist文件，拼合图片的使用大大简化。这个文件可以和图片一同载入，并给每个拼合的图层设置`contentsRect`，这样开发者就不用手动写代码来摆放位置了。

&nbsp;&nbsp;&nbsp;&nbsp;这些文件通常在OpenGL游戏中使用，不过呢，你要是有兴趣在一些常见的app中使用拼合技术，那么一个叫做LayerSprites的开源库（[https://github.com/nicklockwood/LayerSprites](https://github.com/nicklockwood/LayerSprites))，它能够读取Cocos2D格式中的拼合图并在普通的Core Animation层中显示出来。



应用二：http://www.cocoachina.com/articles/12622