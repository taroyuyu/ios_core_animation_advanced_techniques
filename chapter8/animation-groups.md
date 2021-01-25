# 动画组

## 动画组

**`CABasicAnimation`和`CAKeyframeAnimation`仅仅作用于单独的属性，而`CAAnimationGroup`可以把这些动画组合在一起。`CAAnimationGroup`是另一个继承于`CAAnimation`的子类，它添加了一个`animations`数组的属性，用来组合别的动画**。我们把清单8.6那种关键帧动画和调整图层背景色的基础动画组合起来（清单8.10），结果如图8.3所示。

添加一个动画组到一个图层和单独添加动画并没有本质上的区别，所以不能立即明确何时或为什么要使用这个类。它提供了一些便利，可以集体设置动画持续时间，或使用一个命令从一个层中添加和删除多个动画，但它的有用性只有在涉及分层计时时才真正变得明显。

清单8.10 组合关键帧动画和基础动画

```objective-c
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
    //add a colored layer
    CALayer *colorLayer = [CALayer layer];
    colorLayer.frame = CGRectMake(0, 0, 64, 64);
    colorLayer.position = CGPointMake(0, 150);
    colorLayer.backgroundColor = [UIColor greenColor].CGColor;
    [self.containerView.layer addSublayer:colorLayer];
    //create the position animation
    CAKeyframeAnimation *animation1 = [CAKeyframeAnimation animation];
    animation1.keyPath = @"position";
    animation1.path = bezierPath.CGPath;
    animation1.rotationMode = kCAAnimationRotateAuto;
    //create the color animation
    CABasicAnimation *animation2 = [CABasicAnimation animation];
    animation2.keyPath = @"backgroundColor";
    animation2.toValue = (__bridge id)[UIColor redColor].CGColor;
    //create group animation
    CAAnimationGroup *groupAnimation = [CAAnimationGroup animation];
    groupAnimation.animations = @[animation1, animation2]; 
    groupAnimation.duration = 4.0;
    //add the animation to the color layer
    [colorLayer addAnimation:groupAnimation forKey:nil];
}
```

<img src="./8.3.jpeg" alt="图8.3" title="图8.3" width="700"/>

图8.3 关键帧路径和基础动画的组合
