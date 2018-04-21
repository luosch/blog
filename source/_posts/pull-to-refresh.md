title: iOS中的下拉刷新
date: 2016-07-18 15:18:28
tags: [技术]
---
### 1. 前言
目前大部分的iOS应用的列表都带有下拉刷新的功能，这个功能一般是用来获取更多的数据并填充到现有的列表中，当然下拉这个动作也可以带出其他的操作，例如旧版微信在消息列表下拉可以拍视频

要研究下拉刷新这个功能，首先要了解iOS中的列表视图，绝大部分的iOS app 的列表都是 UITableView，而 UITableView 继承于 UIScrollView，许多的滑动操作都是在 UIScrollView 中进行处理，包括我们的下拉动作，所以有必要仔细研究一下 UIScrollView 这个 UI控件

<!-- more -->

### 2. UIScrollView

UIScrollVie是UIView的一个子类，在要想弄懂 UIScrollView 是怎么工作之前，需要先了解一下 UIView，特别是视图渲染的两个步骤：

#### 2.1 光栅化（Rasterization）
在光栅化步骤中，视图不关心自己显示的位置，只专注于绘制自己的content。

![](/resource/Pull-To-Refresh-1.png)

在drawRect：方法被调用前，会为视图创建一个空白的图片来绘制content。该图片的坐标系统是视图的bounds。视图默认的bounds是{0，0，width，height}，超出该范围的部分会被舍弃。

### 2.2 组合（Composition）
在该步骤中，每个视图将自己光栅化的图片组合到自己父视图（superView）的光栅化图片上面。视图的frame决定了自己在父视图上的位置，frame的origin表明了视图光栅化图片左上角相对于父视图光栅化图片左上角的偏移量。一旦这两个视图被组合到一起，组合的结果图片将会和父视图的父视图（superView.superView）进行组合，这是一个雪球效应。

![](/resource/Pull-To-Refresh-2.png)

可以推导出公式：

```
CompositedPosition.x = View.frame.origin.x - SuperView.bounds.origin.x;

CompositedPosition.y = View.frame.origin.y - SuperView.bounds.origin.y;
```

### 2.3 UIScrollView的三个属性

接下来我们研究一下下拉刷新中会用到的三个属性

#### contentOffset

contentOffset描述了内容视图相对于scrollView窗口的位置(向上向左的偏移量)。默认值是CGPointZero，也就是(0,0)。当视图被拖动时，系统会不断修改该值。也可以通 setContentOffset:animated:方法让图片到达某个指定的位置。

![](/resource/Pull-To-Refresh-3.png)

以上图为例，Visible Area 的 contentOffset 为 {x: -80, y: -40}, 这是他与scrollView的origin的偏移量，当我们向下滑动10个单位像素，contentOffset的y值会变为-50，因为向下滑动这个动作使得ScrollView在往上走，偏移量的差值增大

同样地，当我们把列表往下拉的时候，contentOffset的y值会减小（因为内容视图相对于scrollView在往下走）

#### contentSize

描述了有多大范围的内容需要使用scrollView的窗口来显示，其默认值为CGSizeZero，也就是一个宽和高都是0的范围。

content size不会改变其bounds的大小，只是定义了可以滚动的区域。 我们在使用UIScrollView时，需要设置其content size的值大于bounds的值，否则就不能滚动。当contentSize设置的比bounds大的时候，用户就可以滚动视图了

#### contentInset

表示scrollView的内边距，也就是内容视图边缘和scrollView的边缘的留空距离，默认值是UIEdgeInsetsZero，也就是没间距。通常在需要刷新内容时才用得到

contentInset的一个使用场景是，键盘弹出时，会遮挡住很大一部分界面上的内容。但是，如果将contentInset的底部设为键盘的高度，这样就能显示被键盘遮挡住的部分。

![](/resource/Pull-To-Refresh-4.png)

### 3. 组件框架

组件框架分为两部分，是动画组件，是组件核心部分，处理下拉和加载动画的逻辑；另一部分是扩展接口，因为UITableView的继承自UIScrollView，所以可以用UIScrollView的扩展来实现下拉刷新的接口

```
.
├── AnimatedGifActivityIndicator.h => 动画组件
├── AnimatedGifActivityIndicator.m
├── UIScrollView+PullToRefresh.h   => 扩展接口
└── UIScrollView+PullToRefresh.m
```


### 3.1 状态机

在实现下拉刷新功能之前，先确定各个状态和他们之间的转换关系，状态图如下所示：

![](/resource/Pull-To-Refresh-5.jpg)

对于状态转换相关的控制，主要是由scrollViewDidScroll这个函数处理，具体函数如下：

![](/resource/Pull-To-Refresh-6.jpg)

其中contentOffset就是前文提过的偏移量，而self.progressThreshold是初始化时设置的下拉阀值，下拉幅度达到这个阀值，就认为下拉幅度达到100%

为了使状态转换能实时地进行，这里使用KVO（Key Value Observing）来监控scrollView的下拉幅度

![](/resource/Pull-To-Refresh-7.jpg)

### 3.2 接口部分

为了让下拉刷新组件的调用方式更为简单，这里使用了UIScrollView扩展的方式来编写接口部分，接口方法主要有以下四个，前三个用来添加动画，最后一个方法用于终止动画

![](/resource/Pull-To-Refresh-8.jpg)

#### 为扩展添加属性

我们实现接口的方式是扩展（extension），而在objective-c中扩展是不能添加属性的，为了能拥有属性，这里使用了objc的runtime机制来动态添加属性，也就是动画组件AnimatedGifActivityIndicator

![](/resource/Pull-To-Refresh-9.jpg)

### 3.3 调用方式

有了前面的铺垫，调用下拉刷新组件的方式变得十分简单

![](/resource/Pull-To-Refresh-10.jpg)

上图是1中动画效果的实现代码，只要引入了扩展接口，即UIScrollView+PullToRefresh.h，就可以在tableview中使用添加动画这个接口方法。而加载动画需要调用者去主动停止，也就是代码最后一行的 stopPullToRefreshAnimation 方法

### 4. 实现效果

![](/resource/ptr.gif)



