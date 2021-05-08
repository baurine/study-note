# JakeLin iOS Develop

## Note 1 - 使用 Swift 开发 iOS 8 App 实战

Note for [使用 Swift 开发 iOS 8 App 实战](http://www.imooc.com/learn/173) by JakeLin.

### 第一章 十二生肖

基本知识，没有特别的要点

1. 遇到问题时优先使用 xcode 自带的各种帮助文档，而不是 google
2. 右下角的 snippet 和 resource 工具有点帮助 (就是和控件库并列的那几个选项)
3. 原来可以 disable 掉 auto layout 和 size class (建议不要这么做)
4. 在项目中新建一个 playgroud 用来快速测试某些算法，而不是直接在原来的代码中写测试
5. 点击按钮或空白处后让键盘消失，在 action 函数和 viewController 的 touchesEnded 函数中调用 textFiled.resignFirstResponder()
6. 要取消原来建立的 outlet 时，不要只是简单地删除 .swfit 中的代码，还要在 storyboard 中删除，否则 app 会崩溃

### 第二章 相亲神器

略，没什么新内容，主要是讲解了各种常见控件的使用。

UITextFiled 点击键盘的 "确定" 或 "下一项" 后关闭键盘的操作，需要实现 UITextFiledDelegate，实现 textFieldShouldReturn 方法。

(在 2.0 上已经失效... 而且 protocol 的实现不用加 override 关键字，奇怪，我怎么没印象了，复习了 boxue.io 的内容，确实是不用加 override)

### 第三章 女神画像

Storyboards 和 Segues

1. Storyboard
2. Segue / Unwind Segue (在 viewController 之间传递数据)
3. UIPickerView (也有 dataSource 和 delegate)

实现 UI 的三种方式：

1. nib (xib)，本质上是一样的，只是 nib 是以前的格式，里面的内容是二进制，不方便版本管理，xib 是 xml 格式
2. 纯代码手写
3. Storyboard

本课再次强化了 segue 的使用，在 prepareSegue() 方法中，根据 segue.identifier 和 segue.destinationViewController，为目标 ViewController 传值。

当返回前一个 ViewController 时，可以通过 unwind segue 进行传值。

练习 DONE!

手动实现 ViewController 的跳转则是要三步：(可以复习 kevinzhou 的视频)

1. 为 btn 绑定 action 函数
2. 在 storyboard 中创造从 ViewController 到 ViewController 的 segue
3. 在 action 函数中调用 performSegueWithIdentifier(segueIndentifier, nil)

unwind segue 的使用，要先在 view controller 中写好 @IBAction 的响应函数，才能在 Storyboard 中创建 unwind segue。

### 第四章 女神画像之 Navigation Controller

将原来的 view controller 嵌入到 navigation controller 中。

navigation controller 只是一个容器，它与第一个要显示的 view controller 之间的关系不是一般的 segue，而是 relationship segue - root view controler。

首个要显示的 view controller 与后续的 view controller 通过 segue 跳转时，如果 segue 是 push 操作，后续的 view controller 将继续有 navigation bar，如果是 modal，则不再有 navigation bar。

分享：再次演示 presentViewController 的使用。

    presentViewController(viewController, animated: true, completion: nil)

### 第五章 Tabbar Controller

明白了 tabbar view controller 与 navigation controller 还有具体的显示内容的 view controller 之间的关系，可以一层套一层。

Tabbar view controller 与具体的显示内容的 view controller 之间的 segue 关系也是 relationship segue。

所以，综上，容器 controller 与内容 controller 之间的 segue 为 relationship segue，内容与内容之前的 segue 为 manul segue。这两种是不一样的。

### 第六章 AutoLayout

看完了，没有新内容。

可以利用 `add missing constraints` 为布局自动添加上约束。

`equal width` 的约束，并不是说两者完全相等，而是两者的值可以有差值，当差值为 0 时是相等，当差值不为 0 时是大于或小于。

### 第七章 UITableViewController - Todo App

再次复习了 table view 的使用，学到了一些新内容。

1. 给 table view 绑定 dataSource/delegate 也可以在 storyboard 的 document outline 中进行，所以就跟 android 一样，可以在 xml 设定，也可以用 java 代码实现，iOS 中很多操作即可以在 storyboard 中实现，也可以通过代码实现。
2. custom table view cell，不一定要定义一个 CustomTableViewCell 的类，可以通过为控件设定 tag，然后调用 cell.viewWithTag() 来得到这个控件。
3. 复习 prepareSegue 和 unwind segue 的使用
4. 删除 item 功能

        func tableView(tableView: UITableView, commitEditingStyle editingStyle: UITableViewCellEditingStyle, forRowAtIndexPath indexPath: NSIndexPath) {
          if (editingStyle == UITableViewCellEditingStyle.Delete) {
            todoManager.todoItems.removeAtIndex(indexPath.row)
            tableView.deleteRowsAtIndexPaths([indexPath], withRowAnimation: UITableViewRowAnimation.Left)
          }
        }

5. 编辑模式

        override func setEditing(editing: Bool, animated: Bool) {
          super.setEditing(editing, animated: animated)
          tableView.setEditing(editing, animated: animated)
        }

6. 拖拽移动模式

        func tableView(tableView: UITableView, canMoveRowAtIndexPath indexPath: NSIndexPath) -> Bool {
          return editing
        }

        func tableView(tableView: UITableView, moveRowAtIndexPath sourceIndexPath: NSIndexPath, toIndexPath destinationIndexPath: NSIndexPath) {
          let moveTodo = todoManager.todoItems.removeAtIndex(sourceIndexPath.row)
          todoManager.todoItems.insert(moveTodo, atIndex: destinationIndexPath.row)
        }

7. search bar

## Note 2 - iOS 动画入门

[iOS 动画入门](http://www.imooc.com/learn/392)

### 第一章 动画概述

略。

### 第二章 创建动画的灵感

动画的十二原则：<http://www.jianshu.com/p/1858a8733ba3>

灵感：dribble, capptivate, google material design

### 第三章 UIView 动画详解

    UIView.animatieWithDuration(duration, animations)

- postion
- opactity
- scale
- color
- rotation

**postion**

    UIView.animateWithDuration(1, animations: {
      self.blueSquare.center.x = self.view.bounds.width - self.blueSquare.center.x
    })

注意，如果使用了 AutoLayout 布局，且在 viewDidAppear() 中运行动画，此时的动画会和 view 的 constraint 冲突，导致效果与预期不等，此时，需要 dispatch_asyn 一个任务来完成动画，像这样：

    override func viewDidAppear(animated: Bool) {
      super.viewDidAppear(animated)

      dispatch_async(dispatch_get_main_queue()) {
        UIView.animateWithDuration(1) {
          self.blueSquare.center.x = self.view.bounds.width - self.blueSquare.center.x
          self.redSquare.center.y = self.view.bounds.height - self.redSquare.center.y
        }
      }
    }

**opacity**

    UIView.animateWithDuration(1, animations: {
      self.blueSquare.alpha = 0.2
    })

**scale**

要用 transform, 没有 scale 的属性

    UIView.animateWithDuration(1, animations: {
      self.blueSquare.transform = CGAffineTransformMakeScale(2.0, 2.0)
    })

**color**

    UIView.animateWithDuration(1, animations: {
      self.blueSquare.backgroundColor = UIColor.redColor()
      self.nameLabel.textColor = UIColor.whiteColor()
    })

**rotation**

也要用 transform 属性

    // 哎，这里其实可以直接用 options: .Repeat 替代
    func spin () {
      UIView.animateWithDuration(1, delay: 0, options: UIViewAnimationOptions.CurveLinear, animations: {
        self.wheel.transform = CGAffineTransformRotate(self.wheel.transform, CGFloat(M_PI))
      }) { (finished) -> Void in
        self.spin()
      }
    }

## Note 3 - iOS 动画进阶

[iOS 动画进阶](http://www.imooc.com/learn/395)

还是讲 UIView 动画，我还以为会讲 CALayer 呢。

讲的是 UIView 的 delay 和 options 参数。略看吧。

Timing / Repeat / Easing / Spring

动画的过程：

1. 初始状态
2. 结束状态 (animations)
3. 执行时长 (delay)
4. 执行过程 (options: repeat, easing, spring)
5. 执行完毕的处理 (completion)

**Timing**

动画的时机

**Repeat**

    options: [.Repeated, .AutoReverse]

**Easing**

    opitons: [.CurveEaseInOut, .CurveEaseIn, .CurveEaseOut, .CurveLinear]

- EaseIn (慢进)
- EaseOut(慢出)
- EaseInOut(慢进慢出),

来自贝塞尔曲线

- <http://cubic-bezier.com/> (这个网站不错)
- <http://easings.net/>

模拟器的 Debug --> Show Animation 可以把模拟器 CPU 速度放慢

**Spring**

前面的动画效果是基本数学计算，Spring 是基本物理碰撞检测。(斯坦福的课程里也有讲到)

效果取决于弹簧的弹力系数，初始速度，初始重力。

    UIView.animateWithDuration(3, delay: 0, usingSpringWithDamping: 0.2, initialSpringVelocity: 0.2, options: [],
                               animations: {
                                 self.springSquare.center.x = self.view.bounds.width - self.springSquare.center.x
                              }, completion: nil)

### iOS 动画案例之会跳舞的界面

资料：

- <http://www.imooc.com/learn/441>
- <http://www.imooc.com/learn/442>

上篇讲了使用 sketch 设计一个登录界面，没有新内容。

在这个设计例子中，也使用了渐变色作为页面背景，让我想起了 kevinzhou 在设计 Lolita 中也使用渐变作为背景，是种不错的设计。

下篇在 iOS 中实现动画，关于动画，其实没有新的知识点，主要是 position，scale 的变化，结合 spring 效果，最后的警告框使用了新的 API - UIView.transitionWithView，可以快速的实现一些常见动画，比如 flip, fade 等效果。

动画之外倒是有一些新的知识点，比如：

1. TextFiled 修改默认效果，增加 left padding (通过增加 leftView 实现)
2. iOS 的 .9 图的实现，slicing
3. 为按钮增加圆角
4. 进一步理解了 frame 的作用

了解了一下，android 中的 spring 效果要通过 Facebook 的 rebound 库来实现。

DONE！

通过实践发现的问题和总结：

1. 使用 AutoLayout 后，在 viewDidLoad() 中，view 的尺寸也是不一定可靠的。我在 viewDidLoad() 中打印某个 view 的 center 值，只有 y 值是正确的，x 值不正确。

        // login btn
        // the lgoin.center is not correct here
        // loginOriPos = login.center
        print(login.center)
        login.hidden = true
        login.center.x -= view.bounds.width

2. .TransitionFlipFromTop, flip / fade 这些效果只能和 transitionWithView 方法配合使用，是作用在整个 view 上的，不能和 animateWithDuration 配合使用。

## Note 4 - iOS Core Animation

[iOS Core Animation](https://zsisme.gitbooks.io/ios-/content/index.html)

(有空要把这本书看完)

粗略地看了一下，明白了像 borderWidth 和 borderColor 为什么要作用在 view.layer，而不是作用在 view 上。

> 如果说 CALayer 是 UIView 内部实现细节，那我们为什么要全面地了解它呢？苹果当然为我们提供了优美简洁的 UIView 接口，那么我们是否就没必要直接去处理 Core Animation 的细节了呢？

> 某种意义上说的确是这样，对一些简单的需求来说，我们确实没必要处理 CALayer，因为苹果已经通过 UIView 的高级 API 间接地使得动画变得很简单。

> 但是这种简单会不可避免地带来一些灵活上的缺陷。如果你略微想在底层做一些改变，或者使用一些苹果没有在 UIView 上实现的接口功能，这时除了介入 Core Animation 底层之外别无选择。

> 我们已经证实了图层不能像视图那样处理触摸事件，那么他能做哪些视图不能做的呢？这里有一些 UIView 没有暴露出来的 CALayer 的功能：

> - 阴影，圆角，带颜色的边框
> - 3D 变换
> - 非矩形范围
> - 透明遮罩
> - 多级非线性动画

在 android 上，CALayer 对应的可能应该是 Canvas 了吧。
