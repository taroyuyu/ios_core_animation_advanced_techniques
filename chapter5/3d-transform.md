

## 3D变换

CG前缀表示，`CGAffineTransform`类型属于Core Graphics框架，Core Graphics实际上是一个严格意义上的2D绘图API，并且`CGAffineTransform`仅用于2D变换(即仅应用于二维平面内的变换)。

在第三章中，我们提到了`zPosition`属性，可以用来让图层靠近或者远离相机（用户视角），`transform`属性（`CATransform3D`类型）可以真正做到这点，即**让图层在3D空间内移动或者旋转**。

和`CGAffineTransform`类似，`CATransform3D`也是一个矩阵，但是和2x3的矩阵不同，`CATransform3D`是一个可以在3维空间内做变换的4x4的矩阵（图5.6）。

<img src="./5.6.jpeg" alt="图5.6" title="图5.6" width="700"/>

图5.6 对一个三维空间中的点做`CATransform3D`矩阵变换

和`CGAffineTransform`矩阵类似，Core Animation提供了一系列的方法用来创建和组合`CATransform3D`类型的矩阵。这些函数和Core Graphics的类似，但是3D的平移和旋转提供了额外的z`参数，并且旋转函数除了旋转角度之外多出了`x`,`y`,`z`三个参数，这三个参数共同构成了一个定义旋转轴的向量：

    CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
    CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) 
    CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)

**你应该对X轴和Y轴比较熟悉了，分别以右和下为正方向（回忆第三章，这是iOS上的标准结构，在Mac OS，Y轴朝上为正方向），Z轴和这两个轴分别垂直，指向视角外为正方向**（图5.7）。

<img src="./5.7.jpeg" alt="图5.7" title="图5.7" width="700"/>

图5.7 X，Y，Z轴，以及围绕它们旋转的方向

由图所见，**绕Z轴的旋转等同于之前二维空间的仿射旋转变换，但是绕X轴和Y轴的旋转就突破了屏幕的二维空间，并且在用户视角看来发生了倾斜**。

举个例子：清单5.4的代码使用了`CATransform3DMakeRotation`对视图内的图层绕Y轴做了45度角的旋转，我们可以把视图向右倾斜，这样会看得更清晰。

结果见图5.8，但并不像我们期待的那样。

清单5.4 绕Y轴旋转图层

```objective-c
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //rotate the layer 45 degrees along the Y axis
    CATransform3D transform = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
    self.layerView.layer.transform = transform;
}

@end
```

<img src="./5.8.jpeg" alt="图5.8" title="图5.8" width="700"/>

图5.8 绕y轴旋转45度的视图

图层看起来并没有被旋转，而像是在水平方向上执行一个缩放变换得到的压缩效果，是哪里出了问题呢？

其实完全没错，视图看起来更窄实际上是因为我们是在对角线上看它.因此面向相机的视图更少。它看起来不正确的原因是没有透视。

### 透视投影

在真实世界中，当物体远离我们的时候，由于视角的原因物体看起来会变小。我们认为离我们远的那一面会比离我们近的那一面显得更短，但事实并非如此。我们现在看到的是视图的*等轴测投影*，这是一种保留平行线的3D绘图方法，很像我们之前使用的仿射变换。

在*等轴测投影*中，远处的物体和近处的物体保持同样的缩放比例，这种投影也有它自己的用处（例如建筑绘图，颠倒，和伪3D电子游戏），但当前我们并不需要。

[等轴测投影](https://zh.wikipedia.org/wiki/等轴测投影)

[【游戏】对等轴测投影 **Isometric Projection**]([游戏]对等轴测投影 Isometric Projection)

http://netedu.xauat.edu.cn/jpkc/netedu/jpkc/tmgc/wlzy/3/10.1.htm

[伪3D/2.5D游戏](https://www.wikiwand.com/zh-cn/偽三維)

https://cowlevel.net/question/1862577

https://www.ituring.com.cn/article/788?q=代码

https://indienova.com/indie-game-development/creating-isometric-worlds-a-primer-for-game-developers-gamedev/

https://jobyu.gitbooks.io/open-game-art-pixeltutorial/content/chapter3.html

为了解决这个问题，我们需要引入*投影变换*（又称作*z变换*）来对除了旋转之外的变换矩阵做一些修改，Core Animation并没有给我们提供进行透视变换的函数，因此我们需要手动修改矩阵值，幸运的是，很简单：

`CATransform3D`的**透视效果通过一个矩阵中的单个元素来控制：`m34`。`m34`（图5.9）用于按比例缩放X和Y的值来计算到底要离视角/摄像机有多远**。

<img src="./5.9.jpeg" alt="图5.9" title="图5.9" width="700"/>

图5.9 `CATransform3D`的`m34`元素，用来做透视

**`m34`的默认值是0，我们可以通过将`m34`设置为-1.0 / `d`来应用透视效果，`d`代表了想象中视角相机和屏幕之间的距离(以点为单位)**，那应该如何计算这个距离呢？实际上并不需要，大概估算一个就好了。

**因为视角相机实际上并不存在，所以可以根据屏幕上的显示效果自由决定它的放置的位置。通常500-1000就已经很好了，但对于特定的图层有时候更小后者更大的值会看起来更舒服，减少距离的值会增强透视效果，所以一个非常微小的值会让它看起来更加失真，然而一个非常大的值会让它基本失去透视效果**，对视图应用透视的代码见清单5.5，结果见图5.10。

清单5.5 对变换应用透视效果

```objective-c
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create a new transform
    CATransform3D transform = CATransform3DIdentity;
    //apply perspective
    transform.m34 = - 1.0 / 500.0;
    //rotate by 45 degrees along the Y axis
    transform = CATransform3DRotate(transform, M_PI_4, 0, 1, 0);
    //apply to layer
    self.layerView.layer.transform = transform;
}

@end
```

<img src="./5.10.jpeg" alt="图5.10" title="图5.10" width="700"/>

图5.10 应用透视效果之后再次对图层做旋转

#### 消失点

当在透视角度绘图的时候，远离相机视角的物体将会变小变远，当远离到一个极限距离，它们可能就缩成了一个点，于是所有的物体最后都汇聚消失在一个点。

在现实中，这个点通常是视图的中心（图5.11），于是为了在应用中创建拟真效果的透视，这个点应该聚在屏幕中点，或者至少是包含所有3D对象的视图中点。

<img src="./5.11.jpeg" alt="图5.11" title="图5.11" width="700"/>

图5.11 灭点

**Core Animation定义了这个点位于变换图层的`anchorPoint`（通常位于图层中心，但也有例外，见第三章）。这就是说，当图层发生变换时，这个点永远位于图层变换之前`anchorPoint`的位置**。

**当改变一个图层的`position`，你也改变了它的灭点，做3D变换的时候要时刻记住这一点，当你视图通过调整`m34`来让它更加有3D效果，应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置（而不是直接改变它的`position`），这样所有的3D图层都共享一个灭点**。

#### `sublayerTransform`属性

如果有多个视图或者图层，每个视图或图层都执行3D变换，，那么有必要分别对每个视图或图层设置相同的m34值，并且确保在变换之前都在屏幕中央共享同一个`position`，如果用一个函数封装这些操作的确会更加方便，但仍然有限制（例如，你不能在Interface Builder中摆放视图），这里有一个更好的方法。

**`CALayer`有一个属性叫做`sublayerTransform`。它也是`CATransform3D`类型，但和对一个图层的变换不同，它影响到所有的子图层（不会影响图层自身）。这意味着你可以一次性对包含这些图层的容器做变换，于是所有的子图层都自动继承了这个变换方法**。

相较而言，通过在一个地方设置透视变换会很方便，同时它会带来另一个显著的优势：灭点被设置在*容器图层*的中点，从而不需要再对子图层分别设置了。这意味着你可以随意使用`position`和`frame`来放置子图层，而不需要把它们放置在屏幕中点，然后为了保证统一的灭点用变换来做平移。

我们来用一个demo举例说明。这里用Interface Builder并排放置两个视图（图5.12），然后通过设置它们容器视图的透视变换，我们可以保证它们有相同的透视和灭点，代码见清单5.6，结果见图5.13。

<img src="./5.12.jpeg" alt="图5.12" title="图5.12" width="700"/>

图5.12 在一个视图容器内并排放置两个视图

清单5.6 应用`sublayerTransform`

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //apply perspective transform to container
    CATransform3D perspective = CATransform3DIdentity;
    perspective.m34 = - 1.0 / 500.0;
    self.containerView.layer.sublayerTransform = perspective;
    //rotate layerView1 by 45 degrees along the Y axis
    CATransform3D transform1 = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
    self.layerView1.layer.transform = transform1;
    //rotate layerView2 by 45 degrees along the Y axis
    CATransform3D transform2 = CATransform3DMakeRotation(-M_PI_4, 0, 1, 0);
    self.layerView2.layer.transform = transform2;
}
```

<img src="./5.13.jpeg" alt="图5.13" title="图5.13" width="700"/>

图5.13 通过相同的透视效果分别对视图做变换

### 背面

**我们既然可以在3维空间下旋转图层，那么也可以从*背面*去观察它**。如果我们在清单5.4中把角度修改为`M_PI`（180度）而不是当前的` M_PI_4`（45度），那么将会把图层完全旋转一个半圈，于是完全背对了相机视角。

那么从背部看图层是什么样的呢，见图5.14

<img src="./5.14.jpeg" alt="图5.14" title="图5.14" width="700"/>

图5.14 视图的背面，一个镜像对称的图片

如你所见，**图层默认是双面的（通过doubleSided属性控制），反面显示的是正面的一个镜像**。

但这并不是一个很好的特性，因为如果图层包含文本或者其他控件，那用户看到这些内容的镜像图片当然会感到困惑。另外也有可能造成资源的浪费：想象用这些图层形成一个不透明的固态立方体，既然永远都看不见这些图层的背面，那为什么浪费GPU来绘制它们呢？

**`CALayer`有一个叫做`doubleSided`的属性来控制图层的背面是否要被绘制。这是一个`BOOL`类型，默认为`YES`，如果设置为`NO`，那么当图层正面从相机视角消失的时候，它将不会被绘制**。

###  扁平化图层

如果对包含已经做过变换的图层的图层做反方向的变换将会发什么什么呢？是不是有点困惑？见图5.15

<img src="./5.15.jpeg" alt="图5.15" title="图5.15" width="700"/>

图5.15 反方向变换的嵌套图层

注意做了-45度旋转的内部图层是怎样抵消旋转45度的图层，从而恢复正常状态的。

**如果内部图层相对外部图层做了相反的变换（这里是绕Z轴的旋转），那么按照逻辑这两个变换在内部图层上将被相互抵消**。

验证一下，相应代码见清单5.7，结果见5.16

清单5.7 绕Z轴做相反的旋转变换

```objective-c
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *outerView;
@property (nonatomic, weak) IBOutlet UIView *innerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //rotate the outer layer 45 degrees
    CATransform3D outer = CATransform3DMakeRotation(M_PI_4, 0, 0, 1);
    self.outerView.layer.transform = outer;
    //rotate the inner layer -45 degrees
    CATransform3D inner = CATransform3DMakeRotation(-M_PI_4, 0, 0, 1);
    self.innerView.layer.transform = inner;
}

@end
```

<img src="./5.16.jpeg" alt="图5.16" title="图5.16" width="700"/>

图5.16 旋转后的视图

运行结果和我们预期的一致。现在在3D情况下再试一次。修改代码，让内外两个视图绕Y轴旋转而不是Z轴，再加上透视效果，以便我们观察。注意不能用`sublayerTransform`属性，因为内部的图层并不直接是容器图层的子图层，所以这里分别对图层设置透视变换（清单5.8）。

清单5.8 绕Y轴相反的旋转变换

```objective-c
- (void)viewDidLoad
{
    [super viewDidLoad];
    //rotate the outer layer 45 degrees
    CATransform3D outer = CATransform3DIdentity;
    outer.m34 = -1.0 / 500.0;
    outer = CATransform3DRotate(outer, M_PI_4, 0, 1, 0);
    self.outerView.layer.transform = outer;
    //rotate the inner layer -45 degrees
    CATransform3D inner = CATransform3DIdentity;
    inner.m34 = -1.0 / 500.0;
    inner = CATransform3DRotate(inner, -M_PI_4, 0, 1, 0);
    self.innerView.layer.transform = inner;
}
```

预期的效果应该如图5.17所示。

<img src="./5.17.jpeg" alt="图5.17" title="图5.17" width="700"/>

图5.17 绕Y轴做相反旋转的预期结果。

但其实这并不是我们所看到的，相反，我们看到的结果如图5.18所示。发什么了什么呢？内部的图层仍然向左侧旋转，并且发生了扭曲，但按道理说它应该保持正面朝上，并且显示正常的方块。

这是由于**尽管Core Animation的图层存在于3D空间之内，但它们并不都存在*同一个*3D空间。每个图层的3D场景其实是扁平化的，当你从正面观察一个图层，看到的实际上由子图层创建的想象出来的3D场景，但当你倾斜这个图层，你会发现实际上这个3D场景仅仅是被绘制在图层的表面**。

<img src="./5.18.jpeg" alt="图5.18" title="图5.18" width="700"/>

图5.18 绕Y轴做相反旋转的真实结果

打个比方，这就像你在玩3D游戏时倾斜屏幕一样。在游戏中，你可能会看到前方有一堵墙，但倾斜屏幕无法让你看到墙的四周。屏幕上显示的场景不会因为你看它的角度而改变;；图层也是同样的道理。

**这使得用Core Animation创建非常复杂的3D场景变得十分困难。你不能使用图层树来构建一个分层的3D结构**--在相同场景下的任何3D表面必须和同样的图层保持一致，这是因为每个的父视图都把它的子视图扁平化了。

至少当你用正常的`CALayer`的时候是这样，`CALayer`有一个叫做`CATransformLayer`的子类来解决这个问题。具体在第六章“特殊的图层”中将会具体讨论。