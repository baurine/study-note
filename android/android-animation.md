# Android 自定义 View 和动画总结

自定义 View 为何难，是因为它结合了 android 中 view 的尺寸计算 (measure)，布局 (layout)，绘图 (draw，canva/paint/shader/bitmap)，动画 (animator)，触摸事件(touch, onTouchEvent) 这些内容。把这些内容一一攻破，最后才能很好的写自定义 View。

但不是每个自定义 view 都需要实现上面的所有功能，不同种类的 view，需要关注不同的功能，比如自定义一种简单布局，可能只需要用到 measure 和 layout。而定义一些简单的 view，可以就只需要 measure 和 draw。

因为 android 支持 wrap_content 功能，所以无论定义哪种 view，measure 中缺少不了的。

* measure (onMeasure，难点)
* layout (onLayout，onSizeChanged，还可以)
* draw (canvas.draw, paint，不算难，shader/portduff)
* animation (容易)
* touch event (难，借助 ViewDragger，Scroller)

measure 和 layout，touch event 部分看《Android 开发艺述探索》，写得很详细。

## 动画

android 的动画主要分两大类，属性动画和之前的非属性动画，非属性动画又分逐帧动画和补间动画。

上面的动画是直接作用在某个 view 上的，此外，还有一些是作用在整个布局或 activity 上的动画，比如 LayoutAnimation，activity 的转场动画，共享元素动画，但这些动画的底层实现还是上面所说的两大类。

动画：

- 属性动画
- 非属性动画
  - 逐帧动画 (又叫帧动画)
  - 补间动画 (又叫 View 动画，难道是因为它只能作用在 view 上的原因?)

5.0 之后，实现动画又有了新的方法：AnimatedVectorDrawable，向量动画。(好吧，实际内部也是通过属性动画实现的，AnimatedVectorDrawable 其实是一个胶水，将 vector drawable 和 object animator 结合起来)，AnimatedVectorDrawable 很适合用来做两个矢量图标的变换动画。

### 《疯狂 Android 讲义》

#### 逐帧动画 (Frame)

需要自己定义每一帧，主要在 xml 中定义，使用 `<animation-list>` 和 `<item>` 来定义一个 AnimationDrawable。然后调用 AnimationDrawable 的 start() 和 stop() 方法来开始和停止动画。

从这一小节中获知，android 中的 view 也可以通过 view.setFrame(left, top, width, height) 来改变位置。这和 iOS 中能过 view.frame = CGRect(...) 一样。

#### 补间动画 (Tween)

只需要自己定义少数几个关键帧，然后由系统计算出中间缺失的帧，这就是补间动画。

补间动画只用于简单的变换，如 position, scale, rotation, alpha。如果只需要这些变换，可能会比使用属性动画简单一些。

android 使用 Animation 代表抽象的动画类，包括几个子类：AlphaAnimation, ScaleAnimation (需要指定缩放中心，pivotX, pivotY)，TranslateAnimation, RotateAnimation。

Interpolator，用来控制值如何变化，也就是控制如何生成缺失的帧。这和 iOS 中的 .CurveEaseIn, .CurveEaseOut, .CurveEaseInOut, .CurveLinear 是一个作用。

Interpolator 在任何动画中都是不可以缺少的。在逐帧动画中这个值是 LinearInterpolator。

一般用 xml 来定义补间动画，使用 `<set>` 和 `<scale>` `<alpha>` `<rotate>` `<translate>` 来表示一个 Animation。在代码中使用 AnimationUtils.loadAnimation(this, R.anim.anim) 来从 xml 中加载一个补间动画对象。然后如何作用在 view 上，与逐帧动画不一样的是，逐帧动画把自己设置到某个 view 的背景上，然后直接调用 AnimationDrawable.start()/stop()，而补间动画并不是需要把自己设置到某个 view 的背景或前景上，两者不必先关联起来，直接调用 view.startAnimation(Animation) 即可。

自定义补间动画...(略)

#### 属性动画

属性动画其实是补间动画的升级版，加强版。补间动画只能改变少数几种属性，而属性动画可以改变任意属性。

    Animator <-- ValueAnimator <-- ObjectAnimator

    ValueAnimator.ofInt/ofFloat(startVal, endVal)

    ObjectAnimator.ofInt/ofFloat(obj, "property", startVal, endVal)

ValueAnimator 只负责计算某个值的变换过程，需要自己手动监听其值的变化并作用到某个对象上，更灵活，但使用起来更麻烦。而 ObjectAnimator 会直接就变化的值作用到某个对象的某个属性上 (这个属性并一定要直实存在，只需要用 setter 函数就行)，使用上更简单。

其实属性动画里很重要的一个知识点还有 ViewPropertyAnimator，简化了对 view 的属性动画操作，但很多书都没有讲到。使用方法 `view.animate().x(100).y(200).alpha(0.5f).setDuration(300) ...`，郭霖的博客上讲到，稍候看一看。

另外，属性动画相比补间动画的一个很大的优越性在于，动画结束后，view 的属性是真正地进行了改变，如果位移后，是可以在新的位置接收各种点击事件的，而补间动画则不然，这是一种假象，动画结束后，表面上它的肉体已经移动新位置了，但灵魂还在旧的位置上，在新的位置上无法接收点击事件。(即 View 动画改变的只是显示，并不能响应事件)

从这点来看，iOS 上的 UIView.animateWithDuration() 应该也是属性动画。动画结束后，view 的属性得到了真正的改变。但属性值的改变是瞬间的，实现上和 android 的属性动画有很大区别。

### 《Android 开发艺术探索》

补充了 View 动画 (即补间动画) 的特殊使用场景：LayoutAnimation 和 Activity/Fragment 的转场动画。

就目前而言，虽然属性动画有着众多的优点，但并不是说补间动画就完全淘汰了，有些地方仍然使用补间动画没有任何问题，也更简单。

关于属性动画的补充，属性动画如果用 xml 定义的话，定义在 `res/animator` 路径下，也是用 `<set>` 标签定义，但包裹的不再是 `<scale>` `<translate>` 这些标签，则是 `<animator>` `<ojbectAnimator>` 这些标签。

不过呢！一般极少用 xml 来定义属性动画，绝大多数时候是直接用代码来实现的，因为 api 简单啊。而且还有一个原因是，在 xml 中定义时很难知道值的初始值是多少，只知道结束值。但初始值又必须指定，所以这就很为难了。

### 《Android 群英传》

嗯，这本书有简单的提及了 ViewPropertyAnimator。

这本书的新内容是比较详细地介绍了 svg 动画，及一些示例，还不错。原来那些矢理图标的变换动画通过 svg 动画来实现还是蛮简单的。

OK，关于 android 动画的复习就到此结束。
