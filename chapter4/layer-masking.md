# 图层蒙板

&nbsp;&nbsp;&nbsp;&nbsp;通过`masksToBounds`属性，我们可以沿边界裁剪图形；通过`cornerRadius`属性，我们还可以设定一个圆角。但是**有时候你希望展现的内容不是一个矩形或圆角矩形，而是其它形状。比如，你想展示一个带有星形框架的图片，又或者想让一些滚动文本的边缘慢慢渐变成背景色，而不是一个突兀的边界**。

&nbsp;&nbsp;&nbsp;&nbsp;**通过使用一个带有alpha通道的32位png图片，你可以指定一个包含任意*alpha蒙板(alpha mask)*的*寄宿图*，这通常是创建非矩形视图的最简单方式。但是这个方式不能让你以编码的方式动态地生成蒙板来裁剪图像，也不能让子图层或子视图裁剪成同样的形状**。

&nbsp;&nbsp;&nbsp;&nbsp;CALayer有一个叫做`mask`的属性可以解决这个问题。这个属性本身就是个CALayer类型，**有和其他图层一样的绘制和布局属性。它的使用方式与子图层类似，相对于"父图层"（即拥有该属性的图层）进行布局，但是它不是以子图层的形式出现**。**不同于那些绘制在父图层中的子图层，`mask`图层定义的是父图层的部分可见区域**。

&nbsp;&nbsp;&nbsp;&nbsp;**`mask`图层的颜色是无关紧要的，真正重要的是图层的轮廓。`mask`图层就像是一个饼干切割机，`mask`图层实心的部分会被保留下来，其它部分的则会被抛弃**。（如图4.12）

&nbsp;&nbsp;&nbsp;&nbsp;**如果`mask`图层比父图层要小，那么只有父图层(或父图层的子图层)与蒙版相交的部分是可见的，除此以外的一切都内容会被隐藏起来**。

![图4.12](./4.12.png)

图4.12 把图片和蒙板图层作用在一起的效果

&nbsp;&nbsp;&nbsp;&nbsp;为了演示这一点，让我们创建一个简单的项目，使用图层蒙版属性为一张图片蒙上另一张图片。为了简化，我们将使用UIImageView在Interface Builder中创建我们的图像层，这样只需要通过编程创建和应用蒙版层。清单4.5显示了完成此操作的代码，图4.13显示了结果。

清单4.5 应用蒙板图层

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *imageView;
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create mask layer
  CALayer *maskLayer = [CALayer layer];
  maskLayer.frame = self.layerView.bounds;
  UIImage *maskImage = [UIImage imageNamed:@"Cone.png"];
  maskLayer.contents = (__bridge id)maskImage.CGImage;

  //apply mask to image layer￼
  self.imageView.layer.mask = maskLayer;
}
@end
```

![图4.13](./4.13.png)

图4.13 使用了`mask`之后的UIImageView

&nbsp;&nbsp;&nbsp;&nbsp;**CALayer蒙板图层真正厉害的地方在于蒙板图不局限于静态图。任何由图层构成的都可以作为`mask`属性，这意味着你的蒙板可以通过代码甚至是动画实时生成**。
