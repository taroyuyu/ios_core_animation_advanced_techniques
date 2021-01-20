# 剪切变换


Core Graphics为你提供了计算变换矩阵的一些函数，所以很少需要直接设置`CGAffineTransform`的值。但其中一种情况是，**当您想要创建*剪切变换*时，Core Graphics没有提供内置函数**。

斜切变换是放射变换的第四种类型，较于平移，旋转和缩放并不常用（这也是Core Graphics没有提供相应函数的原因），但有些时候也会很有用。我们用一张图片可以很直接的说明效果（图5.5）。也许用“倾斜”描述更加恰当，具体做变换的代码见清单5.3。

<img src="./5.5.jpeg" alt="图5.5" title="图5.5" width="700"/>

图5.5 水平方向的斜切变换

清单5.3 实现一个斜切变换

```objective-c
@implementation ViewController

CGAffineTransform CGAffineTransformMakeShear(CGFloat x, CGFloat y)
{
    CGAffineTransform transform = CGAffineTransformIdentity;
    transform.c = -x;
    transform.b = y;
    return transform;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    //shear the layer at a 45-degree angle
    self.layerView.layer.affineTransform = CGAffineTransformMakeShear(1, 0);
}

@end
```

https://en.wikipedia.org/wiki/Shear_mapping

https://www.cut-the-knot.org/WhatIs/WhatIsShearTransform.shtml

https://baike.baidu.com/item/剪切变换

https://www.cnblogs.com/shine-lee/p/10950963.html

https://www.zhihu.com/question/20666664/answer/197732108

http://staff.ustc.edu.cn/~tongwh/CG_2019/slides/cg_ch03_03.pdf