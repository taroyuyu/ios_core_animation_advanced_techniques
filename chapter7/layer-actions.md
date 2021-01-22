# 图层动作

现在让我们来做个实验，试着直接对UIView的*寄宿图层*做动画而不是对一个*独立的图层(即*非寄宿图层*)*做动画。清单7.4是对清单7.2代码的一点修改，移除了`colorLayer`，并且直接设置`layerView`关联图层的背景色。

清单7.4 直接设置图层的属性

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //set the color of our layerView backing layer directly
    self.layerView.layer.backgroundColor = [UIColor blueColor].CGColor;
}

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
    self.layerView.layer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    //commit the transaction
    [CATransaction commit];
}
```

运行程序，你会发现当按下按钮，图层颜色会瞬间切换到新的值，而不是像之前一样平滑地过渡到新的值。发生了什么呢？隐式动画好像对`UIView`地*寄宿图层*不生效，就像禁用了一样。

试想一下，如果无论什么时候修改UIView的*动画属性*，其*动画属性*都有动画效果的话，我们应该都能够观察到。所以，如果说UIKit建立在Core Animation（默认对所有东西都做动画）之上，那么隐式动画是如何被UIKit禁用掉呢？

我们知道Core Animation通常会对`CALayer`*动画属性*的改变做动画，但是**`UIView`把它*寄宿图层*的这个行为关闭了**。为了更好说明这一点，我们需要知道隐式动画是如何实现的。

我们把*动画属性*更改时`CALayer`自动添加的动画称作*图层动作(action)*，当`CALayer`的*动画属性*被修改时候，它会调用`CALayer`的`-actionForKey:`方法，并传递*动画属性*的名称。剩下的操作都在`CALayer`的头文件中有详细的说明，实质上是如下几步：

* 图层首先检查它是否有委托，以及这个委托是否实现了`CALayerDelegate`协议中指定的`-actionForLayer:forKey`方法。如果有，直接调用并返回结果。
* 如果没有委托，或者委托没有实现`-actionForLayer:forKey`方法，图层接着检查包含属性名称对应行为映射的`actions`字典。
* 如果`actions字典`没有包含对应的属性，那么图层接着在它的`style`字典接着搜索属性名匹配的图层动作。
* 最后，如果在`style`里面也找不到对应的*图层动作*，那么图层将会直接调用定义了每个属性的*标准图层动作*的`-defaultActionForKey:`方法。

**查找结束后，`-actionForKey:`要么返回nil（这种情况下将不会有动画发生，*动画属性*会立即发生改变），要么返回一个遵循`CAAction`协议的对象，最后`CALayer`会使用之前的和现在的属性值的做动画**。

于是这就解释了UIKit是如何禁用隐式动画的：**每个`UIView`作为其*寄宿图层*的委托，并且提供了`-actionForLayer:forKey`方法的实现。当不在一个*UIView动画块(由IView的`beginAnimations:context:`和`commitAnimations`调用之间执行的代码块或`+animateWithDuration:animations:`的block中的代码)*中时，`UIView`的`actionForLayer:forKey`方法对所有图层都返回`nil`，但是在动画block范围之内，它就返回了一个非空值**。我们可以用一个demo做个简单的实验（清单7.5）

清单7.5 测试UIView的`actionForLayer:forKey:`实现

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //test layer action when outside of animation block
    NSLog(@"Outside: %@", [self.layerView actionForLayer:self.layerView.layer forKey:@"backgroundColor"]);
    //begin animation block
    [UIView beginAnimations:nil context:nil];
    //test layer action when inside of animation block
    NSLog(@"Inside: %@", [self.layerView actionForLayer:self.layerView.layer forKey:@"backgroundColor"]);
    //end animation block
    [UIView commitAnimations];
}

@end
```

运行程序，控制台显示结果如下：

    $ LayerTest[21215:c07] Outside: <null>
    $ LayerTest[21215:c07] Inside: <CABasicAnimation: 0x757f090>

结果和我们预想的一样，**当*动画属性*在*UIView的动画块*之外发生改变时，`UIView`直接通过返回`nil`来禁用隐式动画**。**如果在动画块范围之内更改*动画属性*，则根据*动画属性*的类型返回相应的*图层动作***，在这个例子就是`CABasicAnimation`（第八章“显式动画”将会提到）。

当然**令`-actionForLayer:forKey`返回`nil`并不是禁用隐式动画唯一的办法**，**`CATransacition`有个方法叫做`+setDisableActions:`，可以用来对所有属性同时启用或者关闭隐式动画**。如果在清单7.2的`[CATransaction begin]`之后添加下面的代码，同样也会阻止动画的发生：

    [CATransaction setDisableActions:YES];

总结一下，我们知道了如下几点

* `UIView`为其*寄宿图层*禁用了隐式动画。针对这种图层做动画的唯一办法就是使用`UIView`的动画函数（而不是依赖`CATransaction`），或者继承`UIView`，并覆盖`-actionForLayer:forKey:`方法，或者直接创建一个显式动画（具体细节见第八章）。
* 对于托管图层（hosted layer，即*非寄宿图层*），我们可以通过实现图层的`-actionForLayer:forKey:`委托方法，或者提供一个`actions`字典来控制隐式动画。

我们来对颜色渐变的例子使用一个不同的*图层动作*，通过给`colorLayer`设置一个自定义的`actions`字典。我们也可以使用委托来实现，但是`actions`字典可以写更少的代码。那么到底改如何创建一个合适的行为对象呢？

*图层动作*通常是一个被Core Animation*隐式*调用的*显式*动画对象。这里我们使用的是一个实现了`CATransaction`的实例，叫做*push transition*。

第八章中将会详细解释过渡，不过对于现在，知道`CATransition`遵循`CAAction`协议，并且可以当做一个图层行为就足够了。结果很赞，不论在什么时候改变背景颜色，新的色块都是从左侧滑入，而不是默认的渐变效果。

清单7.6 实现自定义行为

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
    //add a custom action
    CATransition *transition = [CATransition animation];
    transition.type = kCATransitionPush;
    transition.subtype = kCATransitionFromLeft;
    self.colorLayer.actions = @{@"backgroundColor": transition};
    //add it to our view
    [self.layerView.layer addSublayer:self.colorLayer];
}

- (IBAction)changeColor
{
    //randomize the layer background color
    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    CGFloat green = arc4random() / (CGFloat)INT_MAX;
    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
}

@end
```

<img src="./7.3.jpeg" alt="图7.3" title="图7.3" width="700" />

图7.3 使用推进过渡的色值动画
