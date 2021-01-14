# 图层的能力
&nbsp;&nbsp;&nbsp;&nbsp;如果说`CALayer`只是封装了`UIView`内部的实现细节，那我们为什么要全面地了解它呢？既然苹果为我们提供了优美简洁的`UIView`接口，那么我们是否就没必要直接去处理Core Animation的细节了呢？

&nbsp;&nbsp;&nbsp;&nbsp;从某种意义上说，的确是这样，对一些简单的需求，我们确实没有必要处理`CALayer`，因为苹果已经通过`UIView`提供的高级API提供了强大的功能，例如动画。

&nbsp;&nbsp;&nbsp;&nbsp;但是这种简单性会不可避免地带来一些灵活上的缺陷。如果你想做一些稍微不同寻常的事情，或者想使用一些苹果没有在`UIView`中暴露的功能，这时除了介入Core Animation之外别无选择。

&nbsp;&nbsp;&nbsp;&nbsp;我们已经证实了图层不能像视图那样处理触摸事件，那么它能做哪些视图不能做的事情呢？这里有一些`UIView`没有暴露出来的但CALayer提供的功能：

* 阴影(Drop shadows)，圆角(rounded corners)，带颜色的边框(colored borders)
* 3D变换(3D transforms and positioning)
* 非矩形范围(Nonrectangular bounds)
* 透明遮罩(Alpha masking of content)
* 多级非线性动画(Multistep, nonlinear animations)

&nbsp;&nbsp;&nbsp;&nbsp;我们将会在后续章节中探索这些功能，首先我们要关注一下在应用程序当中`CALayer`是怎样被利用起来的。

