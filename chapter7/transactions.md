# 事务

Core Animation基于这样一个假设：**屏幕上所做的一切操作都可以（或至少可能）以动画形式呈现**。**动画并不需要你在Core Animation中显式打开，相反需要明确地关闭，否则它会一直存在**。

**当你改变`CALayer`的一个*动画属性(animatable property)*时，这个改变并不能立刻反映在屏幕上。相反，它会从先前的值平滑过渡到新的值。这一切都是默认的行为，你不需要做额外的操作**。

这看起来这太棒了，以至于都不太敢相信，所以我们来用一个例子来演示一下：首先和第一章“图层树”一样创建一个蓝色的方块，然后添加一个按钮，随机改变它的颜色。代码见清单7.1。点击按钮，你会发现图层的颜色平滑过渡到一个新值，而不是跳变（图7.1）。

清单7.1 随机改变图层颜色

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;
@property (nonatomic, weak) IBOutlet CALayer *colorLayer;/*热心人发现这里应该改为@property (nonatomic, strong)  CALayer *colorLayer;否则运行结果不正确。
*/
@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create sublayer
    self.colorLayer = [CALayer layer];
    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer];
}

- (IBAction)changeColor
{
    //randomize the layer background color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;                                                                                       ￼
}

@end
```

<img src="./7.1.jpeg" alt="图7.1" title="图7.1" width="700" />

图7.1 添加一个按钮来控制图层颜色

这种类型的动画被称为*隐式动画*。之所以叫**"隐式"动画是因为我们并没有指定动画的类型，我们仅仅改变了一个属性，然后由Core Animation决定如何并且何时去做动画**。Core Animaiton同样支持*显式*动画，下章详细说明。

但当你改变一个属性，Core Animation是如何判断动画类型和持续时间的呢？**实际上动画执行的时间取决于当前*动画事务*的设置，动画类型取决于图层的*图层动作(actions)***。

***动画事务*实是Core Animation用来封装一组特定*属性动画*的机制**。**添加到*动画事务*中包含的任意*动画属性*都不会立即产生动画，而是在*动画事务*提交后开始产生动画**。

***动画事务*是通过`CATransaction`类来管理的，这个类的设计有些奇怪，不像你从它的名称所预期的那样去管理单个*动画事务*，而是管理一堆*动画事务*，同时不允许我们直接访问其所管理的*动画事务***。**`CATransaction`没有属性或实例方法，并且也不能用`+alloc`和`-init`方法创建它。`CATransaction`提供了一种基于栈的方式来管理*动画事务*，我们可以使用其类方法`+begin`来创建一个*动画事务*并将其压入到*动画事务栈*中，使用其类方法`commit`方法来从*动画事务栈*中弹出一个*动画事务***。

**对图层的*动画属性*所做的任何改变都会被添加到*动画事务栈*顶部的*动画事务*中**。**我们可以通过`+setAnimationDuration:`方法来设置*当前动画事务(位于动画事务栈栈顶的动画事务)*的动画持续时间，或者通过`+animationDuration`方法来获取*当前动画事务*的动画持续时间（默认0.25秒）**。

**Core Animation会在每个*run loop*周期中自动开始一次新的*动画事务***。（run loop是iOS负责收集用户输入，处理定时器或者网络事件并且重新绘制屏幕的东西）。**因此即使我们没有使用`[CATransaction begin]`显式地开始一次事务，我们在每个run loop循环中对*动画属性*所做的变更都会被放到同一个*动画事务*中，然后做一次0.25秒的动画**。

明白这些之后，我们就可以轻松地修改我们这个变色动画的持续时间了。我们当然可以使用`+setAnimationDuration:`方法来修改*当前动画事务*的动画持续时间，但在这里我们首先创建一个新的事务，这样修改动画持续时间就不会有别的副作用。因为修改*当前动画事务*的*动画持续时间*可能会导致同时发生的其它动画（如屏幕旋转），所以最好还是在调整动画之前压入一个新的*动画事务*。

修改后的代码见清单7.2。运行程序，你会发现色块颜色比之前变得更慢了。

清单7.2 使用`CATransaction`控制动画时间

```objective-c
- (IBAction)changeColor
{
    //begin a new transaction
    [CATransaction begin];
    //set the animation duration to 1 second
    [CATransaction setAnimationDuration:1.0];
    //randomize the layer background color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    ￼//commit the transaction
    [CATransaction commit];
}
```

如果你有用过`UIView`的动画方法来做一些动画效果，那么应该对这个模式很熟悉。**`UIView`有两个方法，`+beginAnimations:context:`和`+commitAnimations`，它们的工作方式和`CATransaction`的`+begin`和`+commit`方法类似**。**实际上在`+beginAnimations:context:`方法和`+commitAnimations`方法调用之间，对视图或者图层的*动画属性*所做的改变都会具有动画效果的原因是由于这两个方法实际上是在操纵`CATransaction`**。

**在iOS4中，Apple为UIView添加了一种基于block的动画方法：`+animateWithDuration:animations:`**。**执行一组属性动画时，使用这种方式在语法上会更加简单，但实质上它们都是在做同样的事情——新建一个*动画事务*，添加属性动画，提交*动画事务***。

**`animateWithDuration:animations:`方法会先调用`CATransaction`的`begin`方法然后，执行*block*，最后调用`CATransaction`的`commit`方法，这样通常在block中对*动画属性*所做的修改都会被添加到同一个*动画事务*中（除非你在block中又创建了新的*动画事务*）。这样也可以避免开发者由于对`+begin`和`+commit`匹配的失误造成的风险。**

