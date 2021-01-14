# 使用图层

&nbsp;&nbsp;&nbsp;&nbsp;首先我们来创建一个简单的项目，通过它来操纵一些`layer`的属性。打开Xcode，使用*Single View Application*模板创建一个工程。

&nbsp;&nbsp;&nbsp;&nbsp;在屏幕中央创建一个小视图（大约200 X 200的尺寸），当然你可以手工编码，或者使用Interface Builder。此外，需要确保你的视图控制器中添加了一个属性指向该视图。我们把该属性命名为`layerView`。

&nbsp;&nbsp;&nbsp;&nbsp;运行项目，应该能在浅灰色屏幕背景中看见一个白色方块（图1.3），如果没看见，可能需要调整一下背景window或者view的颜色

<img src="./1.3.jpeg" alt="图1.3" title="图1.3" width="700"/>

图1.3 灰色背景上的一个白色`UIView`

&nbsp;&nbsp;&nbsp;&nbsp;这并没有什么令人激动的地方，所以让我们来添加一些颜色，我们将在这个白色的方块中添加一个蓝色的小方块。

&nbsp;&nbsp;&nbsp;&nbsp;当然，我们可以简单地在这个已经存在的`UIView`上添加一个子视图来实现这个效果，但这不能真正让我们学到任何关于图层的东西。

&nbsp;&nbsp;&nbsp;&nbsp;于是让我们来创建一个`CALayer`，并且把它作为我们视图的相关图层的子图层。尽管`UIView`类的接口中暴露了图层属性，但是标准的Xcode项目模板并没有包含Core Animation相关头文件。所以如果我们不给项目添加合适的库，是不能够使用任何图层相关的方法或者访问它的属性。所以首先需要添加QuartzCore框架到Build Phases标签（图1.4），然后在vc的.m文件中引入<QuartzCore/QuartzCore.h>库。

<img src="./1.4.jpeg" alt="图1.4" title="图1.4" width="700"/>

图1.4 把QuartzCore库添加到项目

&nbsp;&nbsp;&nbsp;&nbsp;之后做了之后，就可以在代码中直接引用`CALayer`的属性和方法。在清单1.1中，我们用代码创建了一个`CALayer`，设置了它的`backgroundColor`属性，然后将它作为`layerView`的相关图层的子图层（这段代码的前提是通过IB创建了`layerView`并做好了连接），图1.5显示了结果。

CALayer的*backgroundColor*属性的类型是`CGColorRef`，而不是像UIView类的backgroundColor那样的UIColor，所以我们需要在设置颜色时使用UIColor对象的CGColor属性将UIColor转换成CGColorRef类型。如果你愿意，你可以使用Core Graphics提供的方法直接创建CGColor，但是使用UIColor可以避免手动内存管理。

清单1.1 给视图添加一个蓝色子图层
``` objective-c	
#import "ViewController.h"
#import <QuartzCore/QuartzCore.h>
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;
￼
@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //create sublayer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    blueLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
}
@end
```
<img src="./1.5.jpeg" alt="图1.5" title="图1.5" width="700"/>

图1.5 白色`UIView`内部嵌套的蓝色`CALayer`

&nbsp;&nbsp;&nbsp;&nbsp;**一个视图只有一个相关联的图层（自动创建），同时它也可以支持添加无数多个子图层**，从清单1.1可以看出，你可以显式地创建一个单独的图层，然后把它作为子图层添加到视图的关联图层中。尽管可以这样添加图层，但通常我们只是简单地使用视图和它的关联图层，而不需要手动地为其关联图层创建额外的子图层。

&nbsp;&nbsp;&nbsp;&nbsp;**在Mac OS 10.8之前，一个显著的性能缺陷就是使用了带有图层支持的*视图树*而不是使用位于单个视图背后的相对独立的图层树**。但是在iOS平台，使用轻量级的`UIView`类并没有显著的性能损失（当然在Mac OS 10.8之后，`NSView`的性能同样也得到很大程度的提高）。

&nbsp;&nbsp;&nbsp;&nbsp;使用带有图层支持的视图而不是*托管(hosted)*的`CALayer`的好处在于，你能在使用`CALayer`的所有底层特性的同时，使用`UIView`的高级API（比如*AutoResizing*、*AutoLayout*和事件处理）。

&nbsp;&nbsp;&nbsp;&nbsp;然而，出于以下几个原因，你可能更需要使用*托管(hosted)*的`CALayer`而不是带有图层支持的`UIView`:

（疑问：啥是*托管*的CALayer？应该就是包装的意思）

* 开发可以同时在Mac OS和iOS上运行的跨平台应用
* 使用多种不同的`CALayer`子类（见第六章，“特殊的图层“），但又不想创建一个新的`UIView`去包"托管"它们
* 做一些对性能特别挑剔的工作，比如对`UIView`一些可忽略不计的操作都会引起显著的不同（尽管如此，你可能会直接想使用OpenGL绘图）


&nbsp;&nbsp;&nbsp;&nbsp;但是这些例子都很少见，总的来说，处理视图会比单独处理图层更加方便。