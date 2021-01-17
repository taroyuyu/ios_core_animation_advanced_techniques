# maskToBounds

&nbsp;&nbsp;&nbsp;&nbsp;现在我们的雪人总算是显示了正确的大小，不过你也许已经发现了另外一些事情：他超出了视图的边界。**默认情况下，UIView仍然会绘制超过其边界之外的内容或子视图。CALayer也是如此**。

（疑问：为什么会这样？即为什么一个layer的内容会超出其边界，这是为何？从其图形API(OpenGL/Metal)的角度来分析）

&nbsp;&nbsp;&nbsp;&nbsp;**UIView有一个叫做`clipsToBounds`的属性可以用来决定是否显示超出边界的内容，CALayer对应的属性叫做`masksToBounds`**，把它设置为YES，雪人就在边界里啦～（如图2.5）

（疑问：如何实现的？从图形API(OpenGL/Metal)的原理来理解）

![图2.5](./2.5.png)

图2.5 使用`masksToBounds`来裁剪图层内容