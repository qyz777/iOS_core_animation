# 图层树
## 图层与视图

在iOS当中，所有的视图都是从一个叫做`UIView`的基类派生而来，`UIView`可以处理触摸实践，可以支持基于Core Graphics绘图，可以做仿射变换(例如旋转或者缩放)，或者简单的类似于滑动或者渐变的动画。

`CALayer`类在概念上和UIView类似，同样也是一些被层级关系管理的矩形块，同样也可以包含一些内容(像图片，文本或者背景颜色)，管理子图层的位置。它们有一些方法和属性用来做动画和变换。和`UIView`最大的不同是`CALayer`不处理用户的交互。

`CALayer`并不清楚具体的响应链(iOS通过视图层级管理用来传送触摸事件的机制)，由于它并不能够响应事件，即使它提供了一些方法来判断是否一个触点在图层范围之内。

## 平行层级关系
每一个`UIView`都有一个`CALayer`实例的图层属性，也就是所谓的backing layer，视图的职责就是管理这个图层，以确保当子视图在层级中添加或者被移除的时候，它们关联的图层也同样对应在层级关系树中有相同的操作。

*实际上这些背后关联的图层才是真正用来在屏幕上显示和做动画，`UIView`仅仅是对它的一个封装，提供了一些iOS类似于处理触摸的具体功能，以及 Core Animation底层方法的高级接口。*

***但是为什么iOS要基于`UIView`和`CALayer`提供的两个平行的层级关系呢？为什么不用一个简单的层级关系来处理所有事情呢？原因在于职责分离，这样也能避免很多重复代码。在iOS和Mac OS两个平台上，事件和用户交互有很多地方不同，基于多点触控的用户界面和基于鼠标键盘有本质的区别，这就是为什么iOS有UIKit和UIView，但是Mac OS有AppKit和NSView的原因。他们功能上很相似，但是在实现上有显著的区别。***

绘图，布局和动画，相比之下就是类似Mac笔记本和桌面系列一样应用于iPhone和iPad触屏概念。把这种功能的逻辑分开并应用到独立的Core Animation框架，苹果就能够在iOS和Mac OS之间共享代码，使得对苹果自己的OS开发团队和第三方开发者去开发两个平台的应用更加便捷。

实际上，这里并不是两个层级关系，而是四个，每一个都扮演不同的角色，除了视图层和图层树之外，还存在呈现树和渲染树。


___Layer也和View一样存在着一个层级梳妆结构，称之为图层树(Layer Tree)，直接创建的或者通过UIView获得的(View.layer)用于显示的图层树，称之为模型树(Model Tree)，模型树的背后还存在两份图层树的拷贝，一个是呈现树(Presentation Tree)，一个是渲染树(Render Tree)。呈现树可以通过普通layer(其实就是模型树)的layer.presentationLayer获得,而模型树则可以通过modelLayer属性获得(详情文档)。.模型树的属性在其被修改的时候就变成了新的值,这个是可以用代码直接操控的部分;呈现树的属性值和动画运行过程中界面上看到的是一致的。而渲染树是私有的你无法访问到。渲染树是对呈现树的数据进行渲染，为了不阻塞主线程，渲染的过程是在单独的进程或线程中进行的，所以你会发现Animation的动画并不会阻塞主线程。___

__模型层树__ 模型层树反映了一个layer静止不动画时的所有属性。比如说，当我们设置redBall.layer.cornerRadius到50来让它变成球时，我们就是在模型层上设置属性。模型层上的值是你的app交互的最多的。任何时候你改变一个layer的值时，都在更新它的模型层。模型层上的值不会在动画过程中改变，并会持续反应你添加动画前的值。

__表现层树__ 表现层树反映了动画时layer上的属性，并包含了运行动画时的变化值。你不应该在这个层树上设置任何值，通常都是在想要准确了解一个layer在哪或是其在动画过程中的行为时通过查看当前的动画值来与表现层树交互。

__渲染树__ 渲染树时苹果的私有值集合，用来执行渲染到屏幕上的实际绘制。你不需要与其交互或知道这些值。

当我们添加一个动画到layer的时候，动画会在layer 的表现树上操作这些值，当动画完成的时候，动画会自动从layer移除，并且表现树的值会变回模型树的值，因为这些值反映了真实、静止的layer属性。

## 图层的能力
如果说`CALayer`是`UIView`内部实现细节，那我们为什么要全面的了解它呢？苹果当然为我们提供了优美简洁的`UIView`接口，那么我们是否就没必要直接去理解Core Animation的细节了呢？

某周意义上说的确是这样的，对一些简单的需求来说，我们的确没必要处理`CALayer`，因为苹果已经通过`UIView`的高级API间接地使得动画变的简单。

但是这种简单不可避免地带来一些灵活上的缺陷。如果你略为想在底层做一些改变，或者使用一些苹果没有在`UIView`上实现的接口功能，这个除了介入Core Animation底层之外别无选择，例如

* 阴影，圆角，带颜色的边框
* 3D变换
* 非矩形范围
* 透明遮罩
* 多级非线性动画
