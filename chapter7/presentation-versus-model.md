# 呈现与模型

**`CALayer`的属性不同于一般的属性，因为改变一个图层的属性并不会立即在界面上产生效果（注：但其在内存中的值会立即发生变化），而是会随着时间逐渐更新**。这是怎么做到的呢?

**当你改变一个图层的属性时，属性值的确是立刻更新的（如果你读取它的数据，你会发现它的值在你设置它的那一刻就已经生效了），但这种改变不会立即反映在屏幕上。这是因为你设置的属性并没有直接调整图层的*外观(apperance)*，相反，它只是定义了在图层动画结束之后图层将会具有的*外观***。

**我们在设置`CALayer`的属性时，实际上是在定义一个控制图层在*当前动画事务*结束之后将会具有的外观的模型**。**然后，Core Animation扮演了一个*控制器*的角色，负责根据*图层动作*和*动画事务*的设置来不断更新屏幕上这些属性的视图状态**。

我们讨论的就是一个典型的*微型MVC模式*。**`CALayer`是一个与用户界面（就是MVC中的*view*）相关联的视觉类，但是在界面本身这个场景下，`CALayer`的行为更像是存储了视图在所有动画结束之后如何展示的模型(Model)。实际上，在苹果自己的文档中，图层树有时又被称为*模型图层树(Model Layer Tree)***。

**在iOS中，屏幕每秒钟重绘60次。如果动画时长比60分之一秒要长，Core Animation就需要在*动画属性*发生变更的时刻和*动画属性*的新值最终反映到屏幕上的时刻之间，对屏幕上的图层进行重新组织。这意味着`CALayer`除了需要知道相关动画属性的*真实值(Actual Value)*（就是你设置的值）之外，必须要知道相关*动画属性*的当前的*显示值(Display Value)***。

**每一个图层属性对应的*显示值*都存储在一个叫做*呈现图层/表示图层(presentation layer)*的独立图层当中**，我们可以通过其`-presentationLayer`方法来访问该图层对应的*表示图层*。***表示图层*实际上是模型图层的副本，但是它的属性值总是反映了了在任何指定时刻图层当前的对应外观效果**。**换句话说，你可以通过呈现图层的值来获取当前屏幕上真正“显示”出来的值**（图7.4）。

我们在第一章中提到除了图层树，另外还有*表示图层树*。***表示图层树*是由模型图层树中各个图层对应的*表示图层*所组成的树结构**。注意***表示图层*时在模型图层第一次被提交（即第一次显示在屏幕上时）时创建，所以在那之前调用`-presentationLayer`将会返回`nil`**。

你可能注意到有一个叫做`–modelLayer`的方法。**在*表示图层*上调用`–modelLayer`将会返回它底层对应的*模型图层***。但**在一个*模型图层*上调用`-modelLayer`会返回`–self`**。

<img src="./7.4.jpeg" alt="图7.4" title="图7.4" width="700" />

图7.4 一个移动的图层是如何通过数据模型呈现的

**大多数情况下，你不需要直接访问表示图层，你可以只与模型图层的属性交互，让Core Animation来更新显示**。但是在这两种情况下表示图层将会变得很有用，**动画同步(synchronizing animation)**和**处理用户交互**。

* 如果你在实现一个基于定时器的动画（见第11章“基于定时器的动画”），而不是基于动画事务的动画，此时知道图层某一时刻在屏幕上的准确位置对于摆放其它UI元素就会很有用。
* 如果你想让那些正在执行动画的图层响应用户输入，你可以使用`-hitTest:`方法（见第三章“图层几何学”）来判断指定图层是否被触摸，这个时候对*表示图层*而不是*模型图层*调用`-hitTest:`会显得更有意义，因为*表示图层*代表了用户当前看到的图层在屏幕上的外观和位置，而不是当前动画结束之后的位置。

（疑问：UIView做动画的时候是关闭了事件处理是吗？）

我们可以用一个简单的案例来演示第二种情况（见清单7.7）。在这个例子中，点击屏幕上的任意位置将会让图层平移到那里。点击图层本身可以随机改变它的颜色。我们通过对*表示图层*调用`-hitTest:`来判断是否被点击。

如果修改代码让`-hitTest:`直接作用于*colorLayer*而不是*表示图层*，你会发现当图层移动的时候它并不能正确显示。这时候你就需要点击图层将要移动到的位置而不是图层本身来响应点击（这就是为什么用呈现图层来响应交互的原因）。

清单7.7 使用`presentationLayer`图层来判断当前图层位置

```objective-c
@interface ViewController ()

@property (nonatomic, strong) CALayer *colorLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a red layer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(0, 0, 100, 100);
    self.colorLayer.position = CGPointMake(self.view.bounds.size.width / 2, self.view.bounds.size.height / 2);
    self.colorLayer.backgroundColor = [UIColor redColor].CGColor;
    [self.view.layer addSublayer:self.colorLayer];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get the touch point
    CGPoint point = [[touches anyObject] locationInView:self.view];
    //check if we've tapped the moving layer
    if ([self.colorLayer.presentationLayer hitTest:point]) {
        //randomize the layer background color
        CGFloat red = arc4random() / (CGFloat)INT_MAX;
        CGFloat green = arc4random() / (CGFloat)INT_MAX;
        CGFloat blue = arc4random() / (CGFloat)INT_MAX;
        self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    } else {
        //otherwise (slowly) move the layer to new position
        [CATransaction begin];
        [CATransaction setAnimationDuration:4.0];
        self.colorLayer.position = point;
        [CATransaction commit];
    }
}
```
@end
