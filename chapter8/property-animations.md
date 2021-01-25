

## 属性动画

属性动画以一个图层的单个属性为目标，并为该属性指定一个目标值或置顶一个用来进行动画的值的范围。属性动画有两种：*基本动画*和*关键帧(Keyframe)动画*。

### 基本动画

动画是一种随时间发生的变化，最简单的变化形式是一个值变化到另一个值时，这正是设计CABasicAnimation所要建模的。

`CAAnimation`是Core Animation所支持的所有动画类型的抽象基类。`CAPropertyAnimation`是`CAAnimation`的一个子类，但同时也是一个抽象类。`CABasicAnimation`是一个非抽象类，其是`CAPropertyAnimation`的一个子类。作为一个抽象类，CAAnimation实际上自己并没有做很多事情。`CAAnimation`提供了一个*时间函数(timing function)*(在第十章中进行解释),一个*动画代理(Delegate)*(用来获取关于动画状态的反馈),和一个removedOnCompletion标识——用来标识动画在完成之后是否应该自动被释放(默认值为YES,防止应用程序的内存占用失控)。CAAnimation还实现了许多协议，包括CAAction协议(允许任何CAAnimation子类作为图层的*图层动作*)和CAMediaTiming协议(在第9章“图层时间”中详细解释)。

CAPropertyAnimation作用于由动画的keyPath值指定的单个属性。因为CAAnimation总是会应用到特定的CALayer，所以keyPath是相对于该层的。由于*keyPath*是一个点分割的键路径，因此可以指向一个层次结构中的任意嵌套对象，而不是简单地指向一个属性名。基于这一点，动画不仅可以应用到图层本身的属性，也可以应用到图层自身的成员对象的属性甚至*虚拟属性*。

`CABasicAnimation`通过如下三个属性扩展了CAPropertyAnimation：

```
id fromValue 
id toValue 
id byValue
```

这些属性的用途都是不言自明的:*fromValue*表示动画开始时，相关属性的值；*toValue*表示动画结束时，相关属性的值；*byValue*表示在动画过程中相关属性变化的相对值，即`byValue=toValue-fromValue`。

通过组合这三个属性，我们可以以各种不同的方式指定值变化过程。这三个属性的类型被定义为`id`而不是更具体的类型是因为属性动画可以与许多不同的属性类型一起使用，例如数值、向量、变换矩阵甚至颜色和图像。

id类型的属性可以存放任何NSObject的子类对象。但通常我们想要进行动画操作的属性的类型并不是NSObject的子类，因此我们需要将其包装成一个NSObject的子类的对象，或者对其进行*toll-free bridging*。如何将预期的数据类型转换为与id兼容的值并不总是很明显，但是表8.1列出了一些常见的情况。

| **Type**      | **Object Type** | 示例代码                                             |
| ------------- | --------------- | ---------------------------------------------------- |
| CGFloat       | NSNumber        | id obj = @(float);                                   |
| CGPoint       | NSValue         | id obj = [NSValue valueWithCGPoint:point);           |
| CGSize        | NSValue         | id obj = [NSValue valueWithCGSize:size);             |
| CGRect        | NSValue         | id obj = [NSValue valueWithCGRect:rect);             |
| CATransform3D | NSValue         | id obj = [NSValue valueWithCATransform3D:transform); |
| CGImageRef    | id              | id obj = (__bridge id)imageRef;                      |
| CGColorRef    | id              | id obj = (__bridge id)colorRef;                      |

`fromValue`、`toValue`和`byValue`这三个属性可以在各种组合中使用，但不应该同时指定这三个属性，因为这会导致矛盾。例如，如果你指定fromValue为2,toValue为4,byValue为3,Core Animation将不知道最终的值应该是4(由toValue指定)还是5 (fromValue + byValue)。关于如何使用这些属性的确切规则在CABasicAnimation头文件中有清晰的记录，所以我们在这里就不重复了。通常，你只需要指定toValue或byValue;其他值可以从上下文自动确定。

让我们尝试一个例子:我们将修改我们在第七章中编写的颜色淡出动画，先前我们用的是"隐式动画"，这一次我们将使用CABasicAnimation来创建一个“显式动画”。

清单8.1 使用CABasicAnimation设置图层背景颜色

```objective-c
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *layerView; 
@property (nonatomic, strong) IBOutlet CALayer *colorLayer;
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //create sublayer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);        
    self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer]; 
}
- (IBAction)changeColor {
    //create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX; 
    CGFloat green = arc4random() / (CGFloat)INT_MAX; 
    CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation]; animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil]; 
}
@end
```

当我们运行这个示例时，它并没有像预期的那样工作。点击按钮会使图层以动画的形式切到一个新的颜色，但它会立即恢复到它的原始值。

之所以会这样的原因是因为动画不会修改图层的模型(即模型图层)，只会修改它的表现形式(即表示图层)(见第七章)。一旦动画完成并从图层中移除，图层就会恢复到它的模型属性所定义的外观。由于我们从来没有改变过backgroundColor属性，所以图层回到了它原来的颜色。

**我们之前使用隐式动画的时候，*图层动作*也是使用CABasicAnimation来实现的**。(你可能还记得在第七章中，我们向控制台中打印了delegate的`-actionForLayer:forKey:`方法返回的结果，并看到返回的*图层动作*的类型是一个CABasicAnimation。)然而，之前，我们是通过设置属性来触发动画。现在我们直接执行相同的动画，但不再设置属性(因此出现了快速恢复的问题)。

将动画作为一个*图层动作*（然后只需要通过改变属性值即可触发动画）是目前为止保持属性值和动画状态同步的最简单方式。但是假设我们因为某些原因不能这样做，比如通常要执行动画的图层是一个视图的*寄宿图层*，我们在更新属性值时可以有两个选择：在动画开始之前更新属性值，或者在动画结束后立即更新属性值。

在动画开始之前更新属性是这些选项中比较简单的，但这意味着我们不能利用隐式的fromValue，所以我们需要手动设置动画中的fromValue以匹配层中的当前值。

考虑到这一点，如果我们在创建动画的地方和添加到图层的地方之间插入以下两行，它就应该消除反弹现象:

```
animation.fromValue = (__bridge id)self.colorLayer.backgroundColor; self.colorLayer.backgroundColor = color.CGColor;
```

这是可行的，但可能不可靠。**我们应该从*表示图层*(如果它存在的话)而不是*模型图层*中获得fromValue，以防已经有一个正在进行的动画**。同样，**因为这我们这个例子中这个图层不是一个*寄宿图曾*，我们应该在设置属性之前使用CATransaction禁用隐式动画，否则默认的图层动作可能会干扰我们的显式动画。(在实践中，显式动画似乎总是覆盖隐式动画，但这种行为没有文档记录，所以安全比遗憾好。)**

如果我们做了这些改变，我们会得到以下结果:

```
- (IBAction)changeColor {
    //create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    CALayer *layer = self.colorLayer.presentationLayer ?: self.colorLayer;
    animation.fromValue = (__bridge id)layer.backgroundColor;
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    self.colorLayer.backgroundColor = color.CGColor;
    [CATransaction commit];
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
}
```

如果每个动画都这样的话，那将要添加相当多的代码。幸运的是，我们可以自动地从CABasicAnimation对象本身获得这个信息，因此我们可以创建一个可重用的方法。清单8.2显示了第一个示例的修改版本，其中包括一个用于应用CABasicAnimation的方法，而不需要每次都重复这个样板代码。

```
- (void)applyBasicAnimation:(CABasicAnimation *)animation toLayer:(CALayer *)layer
{
    //set the from value (using presentation layer if available)
    animation.fromValue = [layer.presentationLayer ?: layer valueForKeyPath:animation.keyPath];
    //update the property in advance
    //note: this approach will only work if toValue != nil 
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    [layer setValue:animation.toValue forKeyPath:animation.keyPath]; 
    [CATransaction commit];
    //apply animation to layer
    [layer addAnimation:animation forKey:nil]; 
}
- (IBAction)changeColor {
    //create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX; CGFloat green = arc4random() / (CGFloat)INT_MAX; 
    CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation]; 
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor;
    //apply animation without snap-back
    [self applyBasicAnimation:animation toLayer:self.colorLayer]; 
}
```

这个简单的实现只处理带有toValue的动画，而不是byValue，但这是一个通用解决方案的良好开端。您可以将其打包为CALayer上的一个类别方法，使其更加方便和可重用。
要解决这样一个看似简单的问题，这看起来可能会有很多麻烦，但另一种方法要复杂得多。如果我们在开始动画之前没有更新目标属性，我们就不能在动画完全完成之前更新它，或者我们将取消正在进行的CABasicAnimation。这意味着我们需要在动画完成的时候更新属性，但在它从图层中移除之前，属性会快速恢复到它的原始值。我们怎么确定这个点?

#### CAAnimationDelegate

在第七章中使用隐式动画时，我们能够通过使用CATransaction*动画完成块*来检测动画何时结束。但是，当使用显式动画时，这种方法是不可用的，因为动画与事务没有关联。

为了知道*显式动画*何时完成，我们需要使用动画的delegate属性，它是一个符合CAAnimationDelegate协议的对象。

CAAnimationDelegate是一个特别的协议，所以你不会在任何头文件中找到定义的CAAnimationDelegate @protocol，但你可以在CAAnimation头文件或苹果的开发者文档中找到支持的方法。在这个例子中，我们使用-animationDidStop:finished:方法在动画完成后立即更新我们图层的背景色。

我们需要设置一个新的事务，并在更新属性时禁用层操作;否则，动画将发生两次-一次由于我们的显式的CABasicAnimation，然后再一次由于该属性的隐式动画动作。完整的实现参见清单8.3。

```
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //create sublayer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);   
    self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer]; }
- (IBAction)changeColor {
    //create a new random color
    CGFloat red = arc4random() / (CGFloat)INT_MAX; 
    CGFloat green = arc4random() / (CGFloat)INT_MAX; 
    CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
    UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation]; 
    animation.keyPath = @"backgroundColor";
    animation.toValue = (__bridge id)color.CGColor; 
    animation.delegate = self;
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil]; 
}
- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag {
    //set the backgroundColor property to match animation toValue
    [CATransaction begin];
    [CATransaction setDisableActions:YES]; 
    self.colorLayer.backgroundColor = (__bridge CGColorRef)anim.toValue; 
    [CATransaction commit];
}
@end
```

对`CAAnimation`而言，使用委托模式而不是一个完成块会带来一个问题，就是当你有多个动画的时候，无法在在回调方法中区分。在一个视图控制器中创建动画的时候，通常会用控制器本身作为一个委托（如清单8.3所示），但是所有的动画都会调用同一个回调方法，所以你就需要判断到底是那个图层的调用。

考虑一下第三章的闹钟，“图层几何学”，我们通过简单地每秒更新指针的角度来实现一个钟，但如果指针动态地转向新的位置会更加真实。

我们不能通过隐式动画来实现因为这些指针都是`UIView`的实例，所以图层的隐式动画都被禁用了。我们可以简单地通过`UIView`的动画方法来实现。但如果想更好地控制动画时间，使用显式动画会更好（更多内容见第十章）。使用`CABasicAnimation`来做动画可能会更加复杂，因为我们需要在`-animationDidStop:finished:`中检测指针状态（用于设置结束的位置）。

动画本身会作为一个参数传入委托的方法，也许你会认为可以控制器中把动画存储为一个属性，然后在回调用比较，但实际上并不起作用，**因为委托传入的动画参数是原始值的一个*深拷贝*，从而不是同一个值**。

**当使用`-addAnimation:forKey:`把动画添加到图层，这里有一个到目前为止我们都设置为`nil`的`key`参数。这里的键是`-animationForKey:`方法找到对应动画的唯一标识符，而当前动画的所有键都可以用`animationKeys`获取。如果我们对每个动画都关联一个唯一的键，就可以对每个图层循环所有键，然后调用`-animationForKey:`来比对结果。尽管这不是一个优雅的实现**。

幸运的是，还有一种更加简单的方法。像所有的`NSObject`子类一样，`CAAnimation`实现了KVC（键-值-编码）协议，于是你可以用`-setValue:forKey:`和`-valueForKey:`方法来存取属性。但是`CAAnimation`有一个不同的性能：它更像一个`NSDictionary`，可以让你随意设置键值对，即使和你使用的动画类所声明的属性并不匹配。

这意味着你可以对动画用任意类型打标签。在这里，我们给`UIView`类型的指针添加的动画，所以可以简单地判断动画到底属于哪个视图，然后在委托方法中用这个信息正确地更新钟的指针（清单8.4）。

清单8.4 使用KVC对动画打标签

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *hourHand;
@property (nonatomic, weak) IBOutlet UIImageView *minuteHand;
@property (nonatomic, weak) IBOutlet UIImageView *secondHand;
@property (nonatomic, weak) NSTimer *timer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //adjust anchor points
    self.secondHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    self.minuteHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    self.hourHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    //start timer
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];
    //set initial hand positions
    [self updateHandsAnimated:NO];
}

- (void)tick
{
    [self updateHandsAnimated:YES];
}

- (void)updateHandsAnimated:(BOOL)animated
{
    //convert time to hours, minutes and seconds
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
    NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];
    CGFloat hourAngle = (components.hour / 12.0) * M_PI * 2.0;
    //calculate hour hand angle //calculate minute hand angle
    CGFloat minuteAngle = (components.minute / 60.0) * M_PI * 2.0;
    //calculate second hand angle
    CGFloat secondAngle = (components.second / 60.0) * M_PI * 2.0;
    //rotate hands
    [self setAngle:hourAngle forHand:self.hourHand animated:animated];
    [self setAngle:minuteAngle forHand:self.minuteHand animated:animated];
    [self setAngle:secondAngle forHand:self.secondHand animated:animated];
}

- (void)setAngle:(CGFloat)angle forHand:(UIView *)handView animated:(BOOL)animated
{
    //generate transform
    CATransform3D transform = CATransform3DMakeRotation(angle, 0, 0, 1);
    if (animated) {
        //create transform animation
        CABasicAnimation *animation = [CABasicAnimation animation];
        [self updateHandsAnimated:NO];
        animation.keyPath = @"transform";
        animation.toValue = [NSValue valueWithCATransform3D:transform];
        animation.duration = 0.5;
        animation.delegate = self;
        [animation setValue:handView forKey:@"handView"];
        [handView.layer addAnimation:animation forKey:nil];
    } else {
        //set transform directly
        handView.layer.transform = transform;
    }
}

- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag
{
    //set final position for hand view
    UIView *handView = [anim valueForKey:@"handView"];
    handView.layer.transform = [anim.toValue CATransform3DValue];
}
```

我们成功的识别出每个图层停止动画的时间，然后更新它的变换到一个新值，很好。

不幸的是，即使做了这些，还是有个问题，清单8.4在模拟器上运行的很好，但当真正跑在iOS设备上时，我们发现在`-animationDidStop:finished:`委托方法调用之前，指针会迅速返回到原始值，这个清单8.3图层颜色发生的情况一样。

问题在于回调方法在动画完成之前已经被调用了，但不能保证这发生在属性动画返回初始状态之前。这同时也很好地说明了为什么要在真实的设备上测试动画代码，而不仅仅是模拟器。

我们可以用一个`fillMode`属性来解决这个问题，下一章会详细说明，这里知道在动画之前设置它比在动画结束之后更新属性更加方便。

### 关键帧动画

`CABasicAnimation`揭示了大多数隐式动画背后依赖的机制，这的确很有趣，但是显式地给图层添加`CABasicAnimation`相较于隐式动画而言，只能说费力不讨好。

`CAKeyframeAnimation`是另一种UIKit没有暴露出来但功能强大的类。和`CABasicAnimation`类似，`CAKeyframeAnimation`同样是`CAPropertyAnimation`的一个子类，它依然作用于单一的一个属性，但是和`CABasicAnimation`不一样的是，它不限制于设置一个起始和结束的值，而是可以根据一连串随意的值来做动画。

*关键帧*这一术语起源于传统动画，首席动画师只会画出发生重要事件的帧(关键帧)，然后技能不高的美术人员会画出介于这两者之间的帧(这很容易从关键帧中推断出来)。同样的原则也适用于CAKeyframeAnimation:你提供重要的帧，而Core Animation使用一种称为插值的过程来填补间隙。

[百度百科-关键帧动画](https://baike.baidu.com/item/关键帧动画/10223838)

我们可以用之前使用颜色图层的例子来演示，设置一个颜色的数组，然后通过关键帧动画播放出来（清单8.5）

清单8.5 使用`CAKeyframeAnimation`应用一系列颜色的变化

```objective-c
- (IBAction)changeColor
{
    //create a keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"backgroundColor";
    animation.duration = 2.0;
    animation.values = @[
                         (__bridge id)[UIColor blueColor].CGColor,
                         (__bridge id)[UIColor redColor].CGColor,
                         (__bridge id)[UIColor greenColor].CGColor,
                         (__bridge id)[UIColor blueColor].CGColor ];
    //apply animation to layer
    [self.colorLayer addAnimation:animation forKey:nil];
}

```

注意到序列中开始和结束的颜色都是蓝色，这是因为**`CAKeyframeAnimation`并不能自动把当前值作为第一帧（就像`CABasicAnimation`那样把`fromValue`设为`nil`）。动画会在开始的时候突然跳转到第一帧的值，然后在动画结束的时候突然恢复到原始的值。所以为了动画的平滑特性，我们需要开始和结束的关键帧来匹配当前属性的值**。

当然可以创建一个结束和开始值不同的动画，那样的话就需要在动画启动之前手动更新属性和最后一帧的值保持一致，就和之前讨论的一样。

我们用`duration`属性把动画时间从默认的0.25秒增加到2秒，以便于动画做的不那么快。运行它，你会发现动画通过颜色不断循环，但效果看起来有些*奇怪*。原因在于动画以一个*恒定的步调*在运行。当在每个动画之间过渡的时候并没有减速，这就产生了一个略微奇怪的效果，为了让动画看起来更自然，我们需要调整一下*缓冲*，第十章将会详细说明。

**提供一个数组的值就可以按照颜色变化做动画，但一般来说用数组来描述动画运动并不直观。`CAKeyframeAnimation`有另一种方式去指定动画，就是使用`CGPath`。`path`属性可以用一种直观的方式，使用Core Graphics函数定义运动序列来绘制动画**。

我们来用一个宇宙飞船沿着一个简单曲线的实例演示一下。为了创建路径，我们需要使用一个*三次贝塞尔曲线*，它是一种使用开始点，结束点和另外两个*控制点*来定义形状的曲线，可以通过使用一个基于C的Core Graphics绘图指令来创建，不过用UIKit提供的`UIBezierPath`类会更简单。

我们这次用`CAShapeLayer`来在屏幕上绘制曲线，尽管对动画来说并不是必须的，但这会让我们的动画更加形象。绘制完`CGPath`之后，我们用它来创建一个`CAKeyframeAnimation`，然后用它来应用到我们的宇宙飞船。代码见清单8.6，结果见图8.1。

清单8.6 沿着一个贝塞尔曲线对图层做动画

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a path
    UIBezierPath *bezierPath = [[UIBezierPath alloc] init];
    [bezierPath moveToPoint:CGPointMake(0, 150)];
    [bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
    //draw the path using a CAShapeLayer
    CAShapeLayer *pathLayer = [CAShapeLayer layer];
    pathLayer.path = bezierPath.CGPath;
    pathLayer.fillColor = [UIColor clearColor].CGColor;
    pathLayer.strokeColor = [UIColor redColor].CGColor;
    pathLayer.lineWidth = 3.0f;
    [self.containerView.layer addSublayer:pathLayer];
    //add the ship
    CALayer *shipLayer = [CALayer layer];
    shipLayer.frame = CGRectMake(0, 0, 64, 64);
    shipLayer.position = CGPointMake(0, 150);
    shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    [self.containerView.layer addSublayer:shipLayer];
    //create the keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position";
    animation.duration = 4.0;
    animation.path = bezierPath.CGPath;
    [shipLayer addAnimation:animation forKey:nil];
}

@end
```

<img src="./8.1.jpeg" alt="图8.1" title="图8.1" width="700"/>

图8.1 沿着一个贝塞尔曲线移动的宇宙飞船图片

运行示例，你会发现飞船的动画有些不太真实，这是因为当它运动的时候永远指向右边，而不是指向曲线切线的方向。你可以调整它的`affineTransform`来对运动方向做动画，但很可能和其它的动画冲突。

幸运的是，Apple预见到了这点，并且给`CAKeyFrameAnimation`添加了一个`rotationMode`的属性。设置它为常量`kCAAnimationRotateAuto`（清单8.7），图层将会根据曲线的切线自动旋转（图8.2）。

清单8.7 通过`rotationMode`自动对齐图层到曲线

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a path
    ...
    //create the keyframe animation
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    animation.keyPath = @"position";
    animation.duration = 4.0;
    animation.path = bezierPath.CGPath;
    animation.rotationMode = kCAAnimationRotateAuto;
    [shipLayer addAnimation:animation forKey:nil];
}
```

<img src="./8.2.jpeg" alt="图8.2" title="图8.2" width="700"/>

图8.2 匹配曲线切线方向的飞船图层

### 虚拟属性

之前提到过属性动画实际上是针对于*key path*而不是一个键，这就意味着可以对子属性甚至是*虚拟属性*做动画。但是*虚拟*属性到底是什么呢？

考虑一个旋转的动画：如果想要对一个物体做旋转的动画，那就需要作用于`transform`属性，因为`CALayer`没有显式提供角度或者方向之类的属性，代码如清单8.8所示

清单8.8 用`transform`属性对图层做动画

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //add the ship
    CALayer *shipLayer = [CALayer layer];
    shipLayer.frame = CGRectMake(0, 0, 128, 128);
    shipLayer.position = CGPointMake(150, 150);
    shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    [self.containerView.layer addSublayer:shipLayer];
    //animate the ship rotation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"transform";
    animation.duration = 2.0;
    animation.toValue = [NSValue valueWithCATransform3D: CATransform3DMakeRotation(M_PI, 0, 0, 1)];
    [shipLayer addAnimation:animation forKey:nil];
}

@end
```

这么做是可行的，但看起来更因为是运气而不是设计的原因，如果我们把旋转的值从`M_PI`（180度）调整到`2 * M_PI`（360度），然后运行程序，会发现这时候飞船完全不动了。这是因为这里的矩阵做了一次360度的旋转，和做了0度是一样的，所以最后的值根本没变。

现在继续使用`M_PI`，但这次用`byValue`而不是`toValue`。也许你会认为这和设置`toValue`结果一样，因为0 + 90度 == 90度，但实际上飞船的图片变大了，并没有做任何旋转，这是因为变换矩阵不能像角度值那样叠加。

那么如果需要独立于角度之外单独对平移或者缩放做动画呢？由于都需要我们来修改`transform`属性，实时地重新计算每个时间点的每个变换效果，然后根据这些创建一个复杂的关键帧动画，这一切都是为了对图层的一个独立做一个简单的动画。

幸运的是，有一个更好的解决方案：为了旋转图层，我们可以对`transform.rotation`关键路径应用动画，而不是`transform`本身（清单8.9）。

清单8.9 对虚拟的`transform.rotation`属性做动画

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //add the ship
    CALayer *shipLayer = [CALayer layer];
    shipLayer.frame = CGRectMake(0, 0, 128, 128);
    shipLayer.position = CGPointMake(150, 150);
    shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    [self.containerView.layer addSublayer:shipLayer];
    //animate the ship rotation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"transform.rotation";
    animation.duration = 2.0;
    animation.byValue = @(M_PI * 2);
    [shipLayer addAnimation:animation forKey:nil];
}

@end
```

结果运行的特别好，用`transform.rotation`而不是`transform`做动画的好处如下：

* 我们可以不通过关键帧一步旋转多于180度的动画。
* 可以用相对值而不是绝对值旋转（设置`byValue`而不是`toValue`）。
* 可以不用创建`CATransform3D`，而是使用一个简单的数值来指定角度。
* 不会和`transform.position`或者`transform.scale`冲突（同样是使用关键路径来做独立的动画属性）。

`transform.rotation`属性有一个奇怪的问题是它其实*并不存在*。这是因为`CATransform3D`并不是一个对象，它实际上是一个结构体，也没有符合KVC相关属性，`transform.rotation`实际上是一个`CALayer`用于处理动画变换的*虚拟*属性。

你不可以直接设置`transform.rotation`或者`transform.scale`，他们不能被直接使用。当你对他们做动画时，Core Animation自动地根据通过`CAValueFunction`来计算的值来更新`transform`属性。

`CAValueFunction`用于把我们赋给虚拟的`transform.rotation`简单浮点值转换成真正的用于摆放图层的`CATransform3D`矩阵值。你可以通过设置`CAPropertyAnimation`的`valueFunction`属性来改变，于是你设置的函数将会覆盖默认的函数。

`CAValueFunction`看起来似乎是对那些不能简单相加的属性（例如变换矩阵）做动画的非常有用的机制，但由于`CAValueFunction`的实现细节是私有的，所以目前不能通过继承它来自定义。你可以通过使用苹果目前已经提供的常量（目前都是和变换矩阵的虚拟属性相关，所以没太多使用场景了，因为这些属性都有了默认的实现方式）。