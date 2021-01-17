# 动态绘制寄宿图(Custom Drawing)

&nbsp;&nbsp;&nbsp;&nbsp;**给图层的`contents`属性赋一个CGImage类型的值不是设置图层寄宿图的唯一方式。我们也可以直接用Core Graphics直接绘制一个寄宿图**。我们能够通过继承UIView并实现`-drawRect:`方法来自定义绘制寄宿图。

（疑问：那*contentsScale*、*contentsRect*、*contentsCenter*属性会不会对动态生成的寄宿图产生影响？）

&nbsp;&nbsp;&nbsp;&nbsp;**`-drawRect:` 方法没有默认的实现，因为对UIView来说，寄宿图并不是必须的，它不在意那到底是单调的颜色还是有一个图片的实例**。如果UIView检测到子类重写其`-drawRect:` 方法，在调用该drawRect方法时，它就会为视图分配一个寄宿图，这个寄宿图的像素尺寸等于视图大小乘以 `contentsScale`的值。

(drawRect是没有默认实现的吗？有的，UIView有实现`drawRect`方法)

（探究contentsScale与寄宿图的关系）

```objective-c
    UIView * view = [[UIView alloc] init];
    NSLog(@"[view respondsToSelector:@selector(drawRect:)]=%d",[view respondsToSelector:@selector(drawRect:)]);
```

```
 [view respondsToSelector:@selector(drawRect:)]=1
```



&nbsp;&nbsp;&nbsp;&nbsp;**如果你不需要寄宿图，那就不要创建这个方法了，这会造成CPU资源和内存的浪费，这也是为什么苹果建议：如果没有自定义绘制的任务就不要在子类中写一个空的-drawRect:方法。**

（做一个实验：探究UIView中drawRect的默认实现，以及如果子类不重写drawRect:方法，UIView的drawRect方法会不会被调用）

&nbsp;&nbsp;&nbsp;&nbsp;当视图在屏幕第一次上出现的时候 `-drawRect:`方法就会被自动调用。`-drawRect:`方法里面的代码利用Core Graphics去绘制一个寄宿图，然后寄宿图就会被缓存起来直到它需要被更新（通常是因为开发者调用了`-setNeedsDisplay`方法，但是影响到表现效果的属性值被更改时，一些视图类型会被自动重绘，如`bounds`属性）。虽然`-drawRect:`方法是一个UIView方法，事实上都是底层的CALayer安排了重绘工作和保存了因此产生的图片。

&nbsp;&nbsp;&nbsp;&nbsp;CALayer有一个可选的`delegate`属性，遵循`CALayerDelegate`协议，当CALayer需要一个内容特定的信息时，就会从协议中请求。CALayerDelegate是一个非正式协议，其实就是说没有CALayerDelegate @protocol可以让你在类里面引用啦。你只需要添加相应的的方法，CALayer会自动进行调用。（`delegate`属性仅被声明为id类型，因此CALayerDelegate中的所有代理方法都可看作是可选的）。

&nbsp;&nbsp;&nbsp;&nbsp;当需要被重绘时，CALayer会请求它的代理给他一个寄宿图来显示。它通过调用下面这个方法做到的:

```objective-c
(void)displayLayer:(CALayerCALayer *)layer;
```

（结合流行框架源码，探究displayLayer的使用）

如果委托愿意，这是一个直接设置layer contents属性的机会，在这种情况下，不会再调用其他方法。如果委托没有实现-displayLayer:方法，CALayer会尝试调用下面的方法:

```objective-c
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```

&nbsp;&nbsp;&nbsp;&nbsp;**在调用这个方法之前，CALayer创建了一个合适尺寸的空寄宿图（尺寸由`bounds`和`contentsScale`决定）和一个Core Graphics的绘制上下文环境，为绘制寄宿图做准备，他作为ctx参数传入。**

&nbsp;&nbsp;&nbsp;&nbsp;让我们来继续第一章的项目让它实现CALayerDelegate并做一些绘图工作吧（见清单2.5）.图2.12是他的结果

清单2.5 实现CALayerDelegate

```objective-c
@implementation ViewController
- (void)viewDidLoad
{
  [super viewDidLoad];
  ￼
  //create sublayer
  CALayer *blueLayer = [CALayer layer];
  blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
  blueLayer.backgroundColor = [UIColor blueColor].CGColor;

  //set controller as layer delegate
  blueLayer.delegate = self;

  //ensure that layer backing image uses correct scale
  blueLayer.contentsScale = [UIScreen mainScreen].scale; //add layer to our view
  [self.layerView.layer addSublayer:blueLayer];

  //force layer to redraw
  [blueLayer display];
}

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx
{
  //draw a thick red circle
  CGContextSetLineWidth(ctx, 10.0f);
  CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
  CGContextStrokeEllipseInRect(ctx, layer.bounds);
}
@end
```

![图2.12](./2.12.png)

图2.12 实现CALayerDelegate来绘制图层

注意一下一些有趣的事情：

* 我们在blueLayer上显式地调用了`-display`。**不同于UIView，当图层显示在屏幕上时，CALayer不会自动重绘它的内容。它把重绘的决定权交给了开发者。**
* **尽管我们没有用`masksToBounds`属性，绘制的那个圆仍然沿边界被裁剪了。这是因为当你使用CALayerDelegate绘制寄宿图的时候，Core Graphics 并没有对超出边界外的内容提供绘制支持。**

&nbsp;&nbsp;&nbsp;&nbsp;现在你理解了CALayerDelegate，并知道怎么使用它。但是除非你创建了一个单独的图层，你几乎没有机会用到CALayerDelegate协议。因为当UIView创建了它的宿主图层时，它就会自动地把图层的delegate设置为它自己，并提供了一个`-displayLayer:`的实现，那所有的问题就都没了。

&nbsp;&nbsp;&nbsp;&nbsp;当使用寄宿了视图的图层的时候，你也不必实现`-displayLayer:`和`-drawLayer:inContext:`方法来绘制你的寄宿图。通常做法是实现UIView的`-drawRect:`方法，UIView就会帮你做完剩下的工作，包括在需要重绘的时候调用`-display`方法。

