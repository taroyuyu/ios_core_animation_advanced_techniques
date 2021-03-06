## 软件绘图


&nbsp;&nbsp;&nbsp;&nbsp;**在Core Animation的上下文中，术语*绘图*通常中指代软件绘图（意即：不由GPU协助的绘图操作）**。**在iOS中，软件绘图主要是使用Core Graphics框架完成的**。但**与硬件加速渲染和合成（由Core Animation和OpenGL触发）相比，Core Graphics 的速度确实很慢，尽管使用它有时候是必要的**。

&nbsp;&nbsp;&nbsp;&nbsp;**除了速度慢之外，软件绘图还需要大量内存**。**`CALayer`本身需要的内存相对较少; 但其*寄宿图*会占用RAM中的大量空间.只有它的寄宿图会消耗一定的内存空间**。**直接给图层的`contents`属性赋值一张图片，并不需要增加额外的内存。如果同一张图片被多个图层作为`contents`属性，那么他们将会共用同一块内存，而不是复制内存块**。

&nbsp;&nbsp;&nbsp;&nbsp;但是**一旦你实现了`CALayerDelegate`协议中的`-drawLayer:inContext:`方法或者`UIView`中的`-drawRect:`方法（其实就是前者的包装方法），图层就创建了一个*离屏绘图上下文(offscreen drawing context)*，这个上下文需要的大小的内存可从这个算式得出：图层宽\*图层高\*4字节，宽高的单位均为像素**。**对于一个在Retina iPad上的全屏图层来说，这个内存量就是 2048\*1526\*4字节，相当于12MB内存，图层每次重绘的时候都需要重新抹掉内存然后重新填充**。

（疑问：如果图层大小不变，每次重绘的时候使用的是同一个图形上下文吗？即不会重新分配内存）

&nbsp;&nbsp;&nbsp;&nbsp;软件绘图的代价昂贵，除非绝对必要，你应该避免重绘你的视图。**提高绘制性能的秘诀通常是尽量少绘图**。
