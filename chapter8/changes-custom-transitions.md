# 在动画过程中取消动画


之前提到过，你可以用`-addAnimation:forKey:`方法中的`key`参数来在添加动画之后检索一个动画，使用如下方法：

    - (CAAnimation *)animationForKey:(NSString *)key;

但这种方式并不支持在动画运行过程中修改动画，所以这个方法主要用来检测动画的属性，或者判断它是否被添加到当前图层中。

要终止一个特定的动画，我们可以用如下方法将它从图层移除掉：

    - (void)removeAnimationForKey:(NSString *)key;

或者我们可以使用这个方法删除所有的动画:

    - (void)removeAllAnimations;

**动画一旦被移除，图层的外观就会更新，以匹配当前的模型图层**。**动画在完成时自动删除，除非你通过设置动画的removedOnCompletion属性为NO来指定它们不应该删除。如果你将动画设置为不会自动删除，那么我们需要在该动画它不再需要的时候自己手动删除它;否则，它将留在内存中，直到图层本身最终被销毁。**

我们来扩展之前旋转飞船的示例，这里添加一个按钮来停止或者启动动画。这一次我们用一个非`nil`的值作为动画的键，以便之后可以移除它。`-animationDidStop:finished:`方法中的`flag`参数表明了动画是自然结束还是被打断，我们可以在控制台打印出来。如果你用停止按钮来终止动画，它会打印`NO`，如果允许它完成，它会打印`YES`。

清单8.15是更新后的示例代码，图8.6显示了结果。

清单8.15 开始和停止一个动画

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, strong) CALayer *shipLayer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //add the ship
    self.shipLayer = [CALayer layer];
    self.shipLayer.frame = CGRectMake(0, 0, 128, 128);
    self.shipLayer.position = CGPointMake(150, 150);
    self.shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    [self.containerView.layer addSublayer:self.shipLayer];
}

- (IBAction)start
{
    //animate the ship rotation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"transform.rotation";
    animation.duration = 2.0;
    animation.byValue = @(M_PI * 2);
    animation.delegate = self;
    [self.shipLayer addAnimation:animation forKey:@"rotateAnimation"];
}

- (IBAction)stop
{
    [self.shipLayer removeAnimationForKey:@"rotateAnimation"];
}

- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
{
    //log that the animation stopped
    NSLog(@"The animation stopped (finished: %@)", flag? @"YES": @"NO");
}

@end
```

<img src="./8.6.jpeg" alt="图8.6" title="图8.6" width="700"/>

图8.6 通过开始和停止按钮控制的旋转动画
