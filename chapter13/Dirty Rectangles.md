## 脏矩形

有时候用`CAShapeLayer`或者其他矢量图形图层替代Core Graphics并不是那么切实可行。比如我们的绘图应用：我们用线条完美地完成了矢量绘制。但是设想一下如果我们能进一步提高应用的性能，让它就像一个黑板一样工作，然后用『粉笔』来绘制线条。模拟粉笔最简单的方法就是用一个『线刷』图片然后将它粘贴到用户手指碰触的地方，但是这个方法用`CAShapeLayer`没办法实现。

我们可以给每个『线刷』创建一个独立的图层，但是实现起来有很大的问题。屏幕上允许同时出现图层上限数量大约是几百，那样我们很快就会超出的。这种情况下我们没什么办法，就用Core Graphics吧（除非你想用OpenGL做一些更复杂的事情）。

我们的『黑板』应用的最初实现见清单13.3，我们更改了之前版本的`DrawingView`，用一个画刷位置的数组代替`UIBezierPath`。图13.2是运行结果

```
#import "DrawingView.h"
#import 
#define BRUSH_SIZE 32

@interface DrawingView ()

@property (nonatomic, strong) NSMutableArray *strokes;

@end

@implementation DrawingView

- (void)awakeFromNib
{
    //create array
    self.strokes = [NSMutableArray array];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get the starting point
    CGPoint point = [[touches anyObject] locationInView:self];

    //add brush stroke
    [self addBrushStrokeAtPoint:point];
}

- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get the touch point
    CGPoint point = [[touches anyObject] locationInView:self];

    //add brush stroke
    [self addBrushStrokeAtPoint:point];
}

- (void)addBrushStrokeAtPoint:(CGPoint)point
{
    //add brush stroke to array
    [self.strokes addObject:[NSValue valueWithCGPoint:point]];

    //needs redraw
    [self setNeedsDisplay];
}

- (void)drawRect:(CGRect)rect
{
    //redraw strokes
    for (NSValue *value in self.strokes) {
        //get point
        CGPoint point = [value CGPointValue];

        //get brush rect
        CGRect brushRect = CGRectMake(point.x - BRUSH_SIZE/2, point.y - BRUSH_SIZE/2, BRUSH_SIZE, BRUSH_SIZE);

        //draw brush stroke    ￼
        [[UIImage imageNamed:@"Chalk.png"] drawInRect:brushRect];
    }
}
@end
```

这个实现在模拟器上表现还不错，但是在真实设备上就没那么好了。问题在于每次手指移动的时候我们就会重绘之前的线刷，即使场景的大部分并没有改变。我们绘制地越多，就会越慢。随着时间的增加每次重绘需要更多的时间，帧数也会下降（见图13.3），如何提高性能呢？

为了减少不必要的绘图，Mac OS和iOS都将屏幕划分为需要重绘和不需要重绘的区域。需要重绘的部分称为“脏”区域。由于裁剪和混合边缘的复杂性，重新绘制非矩形区域是不切实际的，因此将重绘围绕脏区域与给定视图重叠的部分的整个包含矩形，这就是脏矩形。 

当一个视图被改动过了，TA可能需要重绘。但是很多情况下，只是这个视图的一部分被改变了，所以重绘整个寄宿图就太浪费了。但是Core Animation通常并不了解你的自定义绘图代码，它也不能自己计算出脏区域的位置。不过，你的确可以提供这些信息。

当你检测到指定视图或图层的指定部分需要被重绘，你直接调用`-setNeedsDisplayInRect:`来标记它，然后将影响到的矩形作为参数传入。这这将导致视图的`-drawRect:`方法(或层委托的`-drawLayer:inContext:`方法)在下次显示更新之前被自动调用。

传递给`-drawLayer:inContext:`的CGContext将自动被剪切以匹配脏矩形。要找出这个矩形的尺寸，可以使用`CGContextGetClipBoundingBox()`函数从上下文本身获取它。这在使用`-drawRect`时更简单:因为CGRect直接作为参数传递。
您应该**尽量将绘图限制在图像中与此矩形重叠的部分。您在dirty CGRect之外绘制的任何内容都将被自动剪切**，但是这样CPU花在计算和抛弃上的时间就浪费了，实在是太不值得了。

相比依赖于Core Graphics为你重绘，裁剪出自己的绘制区域可能会让你避免不必要的操作。那就是说，如果你的裁剪逻辑相当复杂，那还是让Core Graphics来代劳吧，记住：当你能高效完成的时候才这样做。

清单13.4 展示了一个`-addBrushStrokeAtPoint:`方法的升级版，它只重绘当前线刷的附近区域。另外也会刷新之前线刷的附近区域，**我们也可以用`CGRectIntersectsRect()`来避免重绘任何旧的线刷以不至于覆盖已更新过的区域**。这样做会显著地提高绘制效率（见图13.4）

  清单13.4 用`-setNeedsDisplayInRect:`来减少不必要的绘制

```
- (void)addBrushStrokeAtPoint:(CGPoint)point
{
    //add brush stroke to array
    [self.strokes addObject:[NSValue valueWithCGPoint:point]];

    //set dirty rect
    [self setNeedsDisplayInRect:[self brushRectForPoint:point]];
}

- (CGRect)brushRectForPoint:(CGPoint)point
{
    return CGRectMake(point.x - BRUSH_SIZE/2, point.y - BRUSH_SIZE/2, BRUSH_SIZE, BRUSH_SIZE);
}

- (void)drawRect:(CGRect)rect
{
    //redraw strokes
    for (NSValue *value in self.strokes) {
        //get point
        CGPoint point = [value CGPointValue];

        //get brush rect
        CGRect brushRect = [self brushRectForPoint:point];
        ￼
        //only draw brush stroke if it intersects dirty rect
        if (CGRectIntersectsRect(rect, brushRect)) {
            //draw brush stroke
            [[UIImage imageNamed:@"Chalk.png"] drawInRect:brushRect];
        }
    }
}
```



https://blog.csdn.net/yhhwatl/article/details/84593276

https://blog.csdn.net/weiwangchao_/article/details/6970590

http://orientye.com/2d-dirty-rect/

https://juejin.cn/post/6844903918623784974

https://zh.wikipedia.org/wiki/二维图像引擎

http://blog.sina.com.cn/s/blog_718f290a0100zl6c.html

https://www.jianshu.com/p/e44a366b5385

https://www.cnblogs.com/WentingC/p/9274701.html

https://zsisme.gitbooks.io/ios-/content/chapter15/dirty-rectangles.html

http://www.lankx.com/zh/advanced-topics/dirty-region.html

http://huanzizone.blog.sohu.com/73363534.html

https://yq.aliyun.com/articles/301691

http://www.alloyteam.com/2013/12/canvas-animation-optimized-discussion/

https://blog.codingnow.com/2006/05/dirtyrect_demo.html

https://mwping.github.io/android/drawSoftware.html

https://answer-id.com/zh/50822757