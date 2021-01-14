#图层的树状结构

>巨妖有图层，洋葱也有图层，你有吗？我们都有图层 -- 史莱克

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation其实是一个令人误解的命名。你可能认为它只是用来做动画的，但实际上它是从一个叫做*Layer Kit*这么一个不怎么和动画有关的名字演变而来，所以做动画这只是Core Animation特性的冰山一角。

&nbsp;&nbsp;&nbsp;&nbsp;Core Animation是一个*合成(compose)引擎*，它的职责就是尽可能快地*合成(compose)*屏幕上不同的*可视内容(Visual Content)*，这些*可视内容*被分解成独立的*图层*，存储在一个叫做*图层树(Layer Tree)*的层次结构之中。于是这个树形成了**UIKit**以及在iOS应用程序当中你所能在屏幕上看见的一切的基础。

&nbsp;&nbsp;&nbsp;&nbsp;在我们讨论动画之前，我们将从*图层树*开始，介绍一下Core Animation的*静态组合*以及布局特性。


