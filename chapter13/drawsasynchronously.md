## 异步绘制


&nbsp;&nbsp;&nbsp;&nbsp;UIKit的单线程特性意味着后台图像通常必须在主线程上更新，这意味着绘图中断用户交互，并可能使整个应用程序感觉无响应。我们无法控制绘图的缓慢，但如果我们能够避免让用户等待它完成，那就好了。

&nbsp;&nbsp;&nbsp;&nbsp;针对这个问题，有一些方法可以解决：在某些情况下，您可以**预先在单独的线程上以推测方式绘制视图内容，然后将结果图像直接设置为图层内容**。 然而，这实现起来可能不是很方便，但是在特定情况下是可行的。Core Animation提供了一些选择：**`CATiledLayer`和`drawsAsynchronously`属性**。

（也可以参考YYAsyncLayer）

https://juejin.cn/post/6844903902404411400

http://arthurchi.github.io/2016/10/10/YYKit异步绘制的简易探索/

http://arthurchi.github.io/2016/09/26/异步绘制探索的意外收获/

### CATiledLayer

&nbsp;&nbsp;&nbsp;&nbsp;我们在第六章简单探索了一下`CATiledLayer`。除了将图层再次分割成独立更新的小块（类似于脏矩形自动更新的概念），`CATiledLayer`还有一个有趣的特性：**在多个线程中为每个小块同时调用`-drawLayer:inContext:`方法。这就避免了阻塞用户交互而且能够利用多核心芯片来更快地绘制。一个只有单个贴图的CATiledLayer是实现异步更新图像视图的一种廉价方法**。

### drawsAsynchronously

&nbsp;&nbsp;&nbsp;&nbsp;iOS 6中，苹果为`CALayer`引入了这个令人好奇的属性，`drawsAsynchronously`属性对传入`-drawLayer:inContext:`的CGContext进行改动，允许CGContext延缓绘制命令的执行以至于不阻塞用户交互。

&nbsp;&nbsp;&nbsp;&nbsp;它与`CATiledLayer`使用的异步绘制并不相同。它自己的`-drawLayer:inContext:`方法只会在主线程调用，但是CGContext并不等待每个绘制命令的结束。相反地，它会将命令加入队列，当方法返回时，在后台线程逐个执行真正的绘制。

&nbsp;&nbsp;&nbsp;&nbsp;根据苹果的说法。这个特性在需要频繁重绘的视图上效果最好（比如我们的绘图应用，或者诸如`UITableViewCell`之类的），对那些只绘制一次或很少重绘的图层内容来说没什么太大的帮助。

https://www.jianshu.com/p/d50a3db71d30

https://segmentfault.com/a/1190000000390012

https://github.com/ShannonChenCHN/iOSAppOptimization/blob/master/UI%20渲染性能优化/屏幕图像显示原理和性能优化.md

https://www.raywenderlich.com/10317653-calayer-tutorial-for-ios-getting-started

https://my.oschina.net/u/2438875/blog/507545?fromerr=R4LnEaJ5

https://apisof.net/catalog/CoreAnimation.CALayer.DrawsAsynchronously

https://www.hackingwithswift.com/interviews/janina-kutyn-how-to-best-utilize-calayers

https://docs.huihoo.com/apple/wwdc/2012/session_506__optimizing_2d_graphics_and_animation_performance.pdf

https://www.objc.io/issues/3-views/moving-pixels-onto-the-screen/

https://www.kvraudio.com/forum/viewtopic.php?t=521827

http://just-works.blogspot.com/2014/09/improving-performance-when-dealing-with.html

https://news.ycombinator.com/item?id=5925693

https://www.xspdf.com/resolution/10151532.html

