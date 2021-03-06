## 测量，而不是猜测

&nbsp;&nbsp;&nbsp;&nbsp;于是现在你知道有哪些点可能会影响动画性能，那该如何修复呢？好吧，其实不需要。有很多种诡计来优化动画，但如果盲目使用的话，可能会造成更多性能上的问题，而不是修复。

&nbsp;&nbsp;&nbsp;&nbsp;如何正确的测量而不是猜测这点很重要。根据性能相关的知识写出代码不同于仓促的优化。前者很好，后者实际上就是在浪费时间。

&nbsp;&nbsp;&nbsp;&nbsp;那该如何测量呢？第一步就是确保在真实环境下测试你的程序。

### 真机测试，而不是模拟器

&nbsp;&nbsp;&nbsp;&nbsp;当你开始做一些性能方面的工作时，一定要在真机上测试，而不是模拟器。模拟器虽然是加快开发效率的一把利器，但它不能提供准确的真机性能参数。

&nbsp;&nbsp;&nbsp;&nbsp;模拟器运行在你的Mac上，然而Mac上的CPU往往比iOS设备要快。相反，Mac上的GPU和iOS设备的完全不一样，模拟器不得已要在软件层面（CPU）模拟设备的GPU，这意味着**GPU相关的操作在模拟器上运行的更慢，尤其是使用`CAEAGLLayer`来写一些OpenGL的代码时候**。

&nbsp;&nbsp;&nbsp;&nbsp;这就是说在模拟器上的测试出的性能会高度失真。**动画可能在模拟器上运行流畅，但在真机上十分糟糕。也有可能在模拟器上运行的很卡，但可能在真机上很平滑**。你无法确定。

&nbsp;&nbsp;&nbsp;&nbsp;另一件重要的事情就是性能测试一定要用*发布*配置，而不是调试模式。因为当用发布环境打包的时候，编译器会引入一系列提高性能的优化，例如去掉调试符号或者移除并重新组织代码。你也可以自己做到这些，例如在发布环境禁用NSLog语句。你只关心发布性能，那才是你需要测试的点。

&nbsp;&nbsp;&nbsp;&nbsp;最后，最好在你支持的设备中性能最差的设备上测试：如果基于iOS6开发，这意味着最好在iPhone 3GS或者iPad2上测试。如果可能的话，测试不同的设备和iOS版本，因为苹果在不同的iOS版本和设备中做了一些改变，这也可能影响到一些性能。例如iPad3明显要在动画渲染上比iPad2慢很多，因为渲染4倍多的像素点（为了支持视网膜显示）。

### 保持一致的帧率

&nbsp;&nbsp;&nbsp;&nbsp;**为了做到动画的平滑，你需要以60FPS（帧每秒）的速度运行，以同步屏幕刷新速率。通过基于`NSTimer`或者`CADisplayLink`的动画你可以降低到30FPS，而且效果还不错，但没有办法为核心动画本身设置帧率。如果你不能持续地达到60帧，就会出现随机跳过的帧，这会让用户感觉很糟糕**。

&nbsp;&nbsp;&nbsp;&nbsp;你可以在使用的过程中明显感到有没有丢帧，但没办法通过肉眼来得到具体的数据，也没法知道你的做法有没有真的提高性能。你需要的是一系列精确的数据。

&nbsp;&nbsp;&nbsp;&nbsp;**你可以在程序中用`CADisplayLink`来测量帧率（就像11章“基于定时器的动画”中那样），然后在屏幕上显示出来，但应用内的FPS显示并不能够完全真实测量出Core Animation性能，因为它仅仅测出应用内的帧率。我们知道很多动画都在应用之外发生（在渲染服务器进程中处理）**所以，尽管应用内部FPS计数器对于某些类型的性能问题是一个有用的工具，但一旦你确定了一个问题区域，你就需要获得一些更准确和详细的指标来缩小原因。苹果提供了强大的工具来帮助我们做到这一点。