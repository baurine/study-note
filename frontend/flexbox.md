# Flexbox

[Layout with Flexbox](https://facebook.github.io/react-native/docs/flexbox.html)

关于 Flexbox，一段时间不用，就容易忘记，对主轴、侧轴、justifyContent、alighItems 的作用开始感到混淆。所以就记录下自己的理解来增强印象。

先记录它们在 ReactNative 中的用法，因为可用的属性少一些，更容易理解。

在 ReactNative 中可用的 flex 属性及可能的取值：

- flexDirection: 'row', 'column'
- justifyContent: 'flex-start', 'center', 'flex-end', 'space-around', 'space-between'
- alignItems: 'flex-start', 'center', 'flex-end', 'stretch'
- alighSelf: 同 alignItems
- flex: 整数值，经常取值为 1

简单吧，比起 web 中还有一堆 flex-wrap，令人费解的 flex-bias ... ReactNative 中的 flex 的属性及值就这些了。

首先，来理解 flex 中最重要的两个概念，主轴和侧轴。轴嘛，我们常说 x 轴，y 轴，就是坐标系嘛。主轴和侧轴你可以理解成 flex 的坐标轴。因此它们互相垂直。

一般在编程的 UI 世界中，坐标系中的 x 轴，和物理世界一样，从左到右，y 轴就不太一样，一般是从上到下。

那么 flex 中的主轴和侧轴和 x / y 轴的对应关系是什么样的呢，是不是主轴就是 x 轴，侧轴就是 y 轴呢，并不是这样的。flex 中的主轴由 flexDirection 属性决定，可以说 flexDirection 就是用来定义主轴的方向的。如果它的值 'row'，那么主轴就是水平的 x 轴，侧轴自然是 y 轴；如果 flexDirecetion 的值是 'column'，那么主轴就是垂直的 y 轴，侧轴自然是 x 轴。所以，这是非常 make sense 的对吧。

所以，记住第一条定律，flexDirection 的值决定了主轴的方向。

在 web 中，由于考虑到显示它的屏幕，宽度大于高度，因此，flexDirection 默认的值为 'row'，主轴为从左到右的 x 轴。而在手机上，屏幕高度大于宽度，因此 ReactNative 把 flexDirection 的值默认设置为 'column'，主轴为从上到下的 y 轴，也是很好理解的。

一个物体在坐标系中的位置是由 x 轴和 y 轴同时决定的。因此，我们在 flex 中，用 justifyContent 来决定一个元素在主轴上的位置，用 alignItems 来决定一个元素在侧轴上的位置。

    flexDirection  --> 主轴
    justifyContent --> 元素在主轴上的位置
    alignItems     --> 元素在侧轴上的位置

但是有一点要特别注意，justifyContent 和 alignItems 属性是添加上容器元素上的，但作用却是在容器中的所有子元素上。因此，justifyContent 实际上是决定一个容器中的所有子元素在该容器的主轴上的位置，alignItems 是决定一个容器中的所有子元素在该容器的侧轴上的位置。

    flexDirection  --> 容器的主轴
    justifyContent --> 容器的所有子元素在该容器主轴上的位置
    alignItems     --> 容器的所有子元素在该容器侧轴上的位置

如果容器中的某个子元素，它不想随大流，它想获得特殊待遇，那么它就可以用 alignSelf 来改变自身在父容器侧轴上的位置 (嗯，只能改变在侧轴上的位置，不能改变在主轴上的位置)。它和 alighItems 最大的区别就是，alignSelf，正如其名 self，它是作用在元素自身上，而 alignItems 是作用在容器中的子元素上。

最后的 flex 属性，可以简单把它理解成 android 中的 weight 属性，它的值决定了该元素在父容器中主轴方向上所占的比例。如果一个子元素想撑满父容器的空间，就给此元素设置 `flex: 1`。如果一个容器中有两个子元素，每个元素的属性都是 `flex: 1`，那么它们在容器的主轴上将各占 50% 的空间。

在 ReactNative 中关于 flex 只需要了解这些就够了。有时间再把 web 中的 Flexbox 补充进来。
