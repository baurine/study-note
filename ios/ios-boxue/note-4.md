# iOS Boxue - Note 4

### 第十课：Protocol & extension

Apple 在发布 Swift 2 的时候提出了 protocol oriented programming 的概念，改变了 protocol 只能用于定义接口的功能。通过 protocol extension，我们可以让 protocol 拥有默认实现，支持 type constraints 和 type traits。通过对这些特性的应用，我们的代码可以更具表现力和高度解藕，我们甚至会发现 Swift 自身，就是一个 protocol oritened language。

#### 第一小节：什么是 Protocol

作为 Swift 中的一种自定义类型，和 struct，class、enum 不同，我们使用 protocol 来定义某种约定，而不是某一个具体的类型，这种约定通常用于表示某些类型的共性。(初步理解为接口)

    protocol Engine {
      var cylinder: Int { get set }

      func start()
      func stop()
      func getName(label: String)
      func getName()
    }

#### 第二小节：让自定义类型 "遵从" protocols

(即实现接口)

#### 第三小节：使用标准库中的 protocol

(wow，很酷)

    struct Rational {
      var numerator: Int
      var denominator: Int
    }

    // Equatable，swift 提供的标准 protocol
    extension Rational: Equatable {}
    // 为什么实现是放在 {} 外面呢 ??
    func == (lhs: Rational, rhs: Rational) -> Bool {
      let equalNumerator = lhs.numerator == rhs.numerator
      let equalDenominator = lhs.denominator == rhs.denominator

      return equalNumerator && equalDenominator
    }

    // Comparable
    extension Rational: Comparable {}

    func < (lhs: Rational, rhs: Rational) -> Bool {
      let lQuotient =
        Double(lhs.numerator) / Double(lhs.denominator)
      let rQuotient =
        Double(rhs.numerator) / Double(rhs.denominator)

      return lQuotient < rQuotient
    }

    extension Rational : CustomStringConvertible {
      var description: String {
        return "\(self.numerator) / \(self.denominator) "
      }
    }

    // BooleanType, Hashable ... 略

#### 第四小节：认识 protocol extension

扩展一个 protocol 看似和扩展其它自定义类型没有太大区别，都是使用 extension 关键字，加上要扩展的类型的名字。但是，和定义 protocol 不同，我们可以在一个 protocol extension 中提供默认的实现。尽管此时我们还没有定义任何 "遵从" Flight 的类型，但是我们已经可以在 extension 中使用 Flight 的数据成员了。因为 Swift 编译器知道，任何一个 "遵从" Fligh 的自定义类型，一定会定义 Flight 约定的各种属性。

extension 即可以为 protocol 添加额外的功能，又可以为已有的方法提供默认实现。但是，这两种行为，却有着细微的差别，简单来说：通过 extension 添加到 protocol 中的内容，不算做 protocol 约定。

(明白，也就是说通过 extension 添加到 protocol 的属性或方法，实现类并不需要去实现它们)

在 extension 中使用 type constraints

除了在 extension 中提供默认实现之外，我们还可以为默认实现限定可用的条件。这个限定的方式，就叫做 type constraints。

    protocol OperationalLife {
      var maxFlyHours: Int { get }
    }

    extension Flight where Self: OperationalLife {
      func isInService() -> Bool {
        return self.flyHour < maxFlyHours
      }
    }

where 表示我们要进行一个 type contraints，Self 表示 "最终遵从 protocol 的类型" (在我们的例子里，也就是 A380) 。所以整个表达式的含义就是当某个类型同时遵从 OperationalLife 和 Flight 时，扩展 Flight，并提供 isInService 方法。

我们不能在一个 extension 中定义 stored property，因此，这里我们把 maxFlyHours 实现成了一个 computed property。

#### 第五小节：面向对象还是 protocol

这就是和 protocol oriented programming 相关的所有内容，简单来说，面对未来有可能出现的各种 "全功能型" 类型，我们应该学会把它们拆分为不同的 protocol，并重新把它们组合起来。

(是的，面向接口编程总体来说还是优于面向对象编程。但感觉这种方式不适合写 UI)

### 第十一课：Swift 中的异常和错误处理

#### 第一小节：异常处理——基础篇

got!

1. Enum, ErrorType
1. do... try ... catch
1. defer ... try!
1. try?
1. guard...else

#### 第二小节：声名狼籍的错误信息参数

除了抛出异常之外，很多时候，我们习惯于用某个字符串或数字表示一个错误... (c/c++ 中的传统做法)

#### 第三小节：函数式编程 + Either type

其实，面对有可能发生错误的情况，我们的处理思想是很简单的：成功时，可以访问结果；错误时，最好能得到错误的原因。

(wow， enum 的强大，帅呆了，有点像 promise 和 reactive 的链式编程感觉)

    enum Result<T> {
      case Success(T)
      case Failure(String)
    }

    func newDivide(dividend: Double, by: Double) -> Result<Double> {
      if by == 0 {
        return Result.Failure("Cannot divided by zero")
      } else {
        return Result.Success(dividend / by)
      }
    }

    let r = newDivide(4, by: 2)

    switch r {
      case let .Success(value) :
        print("\(value) ")
      case let .Failure(error) :
        print("\(error) ")
    }

抽象 Map 和 FlatMap

    enum Result<T> {
      case Success(T)
      case Failure(String)

      func map<P> (f: T -> P) -> Result<P> {
        switch self {
        case .Success(let value):
          return .Success(f(value) )
        case .Failure(let err):
          return .Failure(err)
        }
      }
    }

    // 有种 reactive 或 promise 的链式编程的感觉
    let r3 = r.map(numSqrt) .map(num2String)

#### 第四小节：处理 Either type 嵌套问题的 flatMap

    enum Result<T> {
      case Success(T)
      case Failure(String)

      func flatMap<P> (f: T -> Result<P>) -> Result<P> {
        switch self {
          case .Success(let value):
              return f(value)
          case .Failure(let err):
              return .Failure(err)
        }
      }
    }

    func numSqrt(num: Double) -> Result<Double> {
      if num < 0 {
        return Result.Failure("number cannot be negative")
      } else {
        return .Success(sqrt(num))
      }
    }

    let r3 = r.flatMap(numSqrt) .map(num2String)
    let r4 = r.map(sqrt) .map({ String(format: "%.10f", $0) })

这就是使用 Result<T> enumeration 处理错误的全部内容，结合函数式编程，它可以极大程度的简化库函数以及客户端调用代码的可读性。

(明白了，这代码很帅)

### 第十二课：Reactive Programming in Swift

Reactive Programming (简称 Rx) 是一种基于变化的新生编程范式，它最重要哲学就是把关注点从 "怎么做" 转移到 "做什么"。这意味着什么呢？我们可以它理解为："当某个对象发生某些变化时，依赖于它的其它对象应该如何得到通知？又该如何响应这个变化呢？" 为此，Rx 定义了一些概念和行为。我们基于 Swift，通过一系列视频，帮助大家掌握 Reactive Programming。

#### 第一小节：理解 Reactive 编程思想

    let strArray = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]
    let evenArray = strArray.map { Int($0)! }.filter { $0 % 2 == 0 }
    print(evenArray)

(说实话，这个例子在 RxJava 里没问题，但在 swift 里 ... strArray.map() 返回的仍然是一个 array，而不是一个 observable，filter() 返回的也还是 array，而不是一个 observable，所以这里还不是真正的响应式)

在下一段视频中，我们将以这个思想为基础，向大家进一步介绍 Reactive Programming 用到的基本概念和用法：

- 如何定义 "以时间为维度的数组"？
- 如何处理加工 "事件数组" 中的元素？
- 如何 "订阅" 事件？

Reactive 思想：用同步的方式写异步代码，以时间为维度的事件数组。

#### 第二小节：Hello world in RxSwift

    enum Event<Element> {
      case Next(Element)     // next element of a sequence
      case Error(ErrorType)  // sequence failed with error
      case Completed         // sequence terminated successfully
    }

    _ = rxUserInput.rx_text
        .map { (input: String) -> Int in
          if let lastChar = input.characters.last {
            if let n = Int(String(lastChar)) {
              return n
            }
          }
          return -1
        }
        .filter { $0 % 2 == 0 }
        .subscribeNext { print($0) }

#### 第三小节：理解 Observables and Observer

(这一节稍稍有点晕，但概念和 RxJava 中是一一对应的，只是写法有所不同)

在 RxJava 中，observer 有三个接口， onNext(T), onComplete(), onError(e)

但在 RxSwift 中，observer 只有一个接口，on(Event<T>)，参数也只有一种，但参数是 enum，有三种值。

- on(.Next(T))
- on(.Complemted)
- on(.Error(e))

所以，本质是上一样的。

视频有点快，看完视频再把文档看了一遍，结合之前对 RxJava 的学习和理解，基本上也明白了 RxSwfift 的工作原理。

OK，通过练习，完成理解了这一节的内容。

其中 defer (推迟) 的解释略有点不对，原文说：被 deferred 修饰后，事件序列只有在被订阅时才生成，并发送事件，并且，每订阅一次，就新生成一个事件序列对象。但是，一般的 observable 也是只有在被订阅后才产生并发送事件。

所以，准确的来说，被 deferred 修饰后，真正产生事件序列的那个 Observable，只有在外层的 Observable 被订阅后才去生成。相当于延迟去生成对象。就这是 defer 的意思。

代码：

    print("---------- emtpy ----------")
    _ = Observable<Int>.empty()
        .subscribe {
          (event: RxSwift.Event<Int>) -> Void in
          print(event)
        }

    print("---------- just ----------")
    _ = Observable.just("Boxue")
        .subscribe {
          (event: RxSwift.Event<String>) -> Void in
          print(event)
        }

    print("---------- of ----------")
    _ = Observable.of(1,2,3,4,5)
        .subscribe {
          (event: RxSwift.Event<Int>) -> Void in
          print(event)
        }

    print("---------- error ----------")
    let err = NSError(domain: "Test", code: -1, userInfo: nil)
    _ = Observable<Int>.error(err)
        .subscribe {
          (event: RxSwift.Event<Int>) -> Void in
          print(event)
        }

    print("---------- create ----------")
    func myJust(event: Int) -> Observable<Int> {
      return Observable.create { observer in
        if event % 2 == 0 {
          observer.on(.Next(event))
          observer.on(.Completed)
        } else {
          let err = NSError(domain: "Not an even number",
                            code: 401,
                            userInfo: nil)
          observer.on(.Error(err))
        }
        return NopDisposable.instance
      }
    }

    _ = myJust(10).subscribe { print($0) }
    _ = myJust(5).subscribe { print($0) }

    print("---------- generate ----------")
    _ = Observable.generate(initialState: 0, condition: {$0<10}, iterate: {$0+1})
        .subscribe { print($0) }

    _ = Observable.generate(initialState: 0, condition: {$0<10}, iterate: {$0+1})
        .map({ (val: Int) -> Int in
          print(val)
          return val*2
        })
        .subscribe { print($0) }

    print("---------- defered ----------")
    let deferredSeq = Observable<Int>.deferred {
      print("generating")

      return Observable.generate(initialState: 0,
                                 condition: {$0<10},
                                 iterate: {$0+1})
    }

    _ = deferredSeq.subscribe { print($0) }

#### 第四小节：理解 Disposable & DisposeBag

完全明白！

    self.interval = Observable.interval(0.5, scheduler: MainScheduler.instance)
    self.subscription = self.interval.map { String($0) }.subscribeNext { self.counterLabel.text = $0 }
    self.subscription.addDisposableTo(bag)

    self.stopBtn.rx_tap.subscribeNext {
      print("Dispose interval")
      // self.subscription.dispose()
      self.bag = nil
    }
    .addDisposableTo(bag)

有限序列 和 无限序列，令我恍然大悟。原来 RxBus 中的事件是无限序列，所以它可以持续地发出事件。我之前狭义地把 Rx 理解为有限序列了。

RxBus，`stopBtn.rx_tap`, `rxUserInput.rx_text` 这些都是无限序列。

#### 第五小节：RxSwift UI 交互 1

(我叉我叉我叉叉叉，这代码，酷毙了)

    // email input
    self.emailInput.layer.borderWidth = 1

    let emailObservable = self.emailInput.rx_text.map { InputValidator.isValidEmail($0) }

    emailObservable.map { (valid: Bool) -> UIColor in
      return valid ? UIColor.greenColor() : UIColor.clearColor()
    }
    .subscribeNext {
      self.emailInput.layer.borderColor = $0.CGColor
    }
    .addDisposableTo(bag)

    // password input
    self.pwdInput.layer.borderWidth = 1

    let pwdObservable = self.pwdInput.rx_text.map { InputValidator.isValidPassword($0) }

    pwdObservable.map { (valid: Bool) -> UIColor in
      return valid ? UIColor.greenColor() : UIColor.clearColor()
    }
    .subscribeNext {
      self.pwdInput.layer.borderColor = $0.CGColor
    }
    .addDisposableTo(bag)

    // signup
    Observable.combineLatest(emailObservable, pwdObservable) {
      (emailValid:Bool, pwdValid:Bool)->Bool in
      return emailValid && pwdValid
    }
    .subscribeNext {
      self.signupBtn.enabled = $0
    }
    .addDisposableTo(bag)

#### 第六小节：RxSwift UI 交互 2

(哇，牛叉的代码，佩服啊)

本节要点：

1. subject，既可以作为 Observer 订阅到另的 Observable 上，又可以作为 Observable 供其它的 Observer 订阅。这里使用了一个实体 subject，Variable<Type>
2. bindTo，相类于 subscribeNext

示例：

    // birthday
    self.birthday.layer.borderWidth = 1
    let birthdayObservable = self.birthday.rx_date.map {
      InputValidator.isValidDate($0)
    }
    birthdayObservable.map {
      $0 ? UIColor.greenColor() : UIColor.clearColor()
    }
    .subscribeNext {
      self.birthday.layer.borderColor = $0.CGColor
    }
    .addDisposableTo(bag)

    // gender
    let genderSelection = Variable<Gender>(.NotSelect)

    self.male.rx_tap.map { Gender.Male }
    .bindTo(genderSelection)
    .addDisposableTo(bag)

    self.female.rx_tap.map { Gender.Female }
    .bindTo(genderSelection)
    .addDisposableTo(bag)

    genderSelection.asObservable().subscribeNext {
      switch $0 {
        case .Male:
          self.male.setImage(UIImage(named: "check"), forState: .Normal)
          self.female.setImage(UIImage(named: "uncheck"), forState: .Normal)
        case .Female:
          self.male.setImage(UIImage(named: "uncheck"), forState: .Normal)
          self.female.setImage(UIImage(named: "check"), forState: .Normal)
        default:
          break
      }
    }
    .addDisposableTo(bag)

    // update btn
    let genderEnableObservable = genderSelection.asObservable().map {
      $0 != .NotSelect ? true : false
    }
    Observable.combineLatest(birthdayObservable, genderEnableObservable) {
      $0 && $1
    }
    .bindTo(self.update.rx_enabled)
    .addDisposableTo(bag)

#### 第七小节：RxSwift UI 交互 3

要点：

1. `rx_enable`，`rx_value`，这些 UI 的 rx 扩展很多都是 subject，既作 Observable，又做 Observer
2. skip(1) 的作用

RxSwift 对 UISwitch 和 UISlider 也进行了扩展，分别给它们添加了一个叫做 rx_value 的属性。在 UISwitch 里，它的类型是 ControlProperty<Bool>；在 UISlider 里，它的类型是 ControlProperty<Float>。和 Variable 类似，ControlProperty 也可以既是 Observer，又是 Observable。

    // switch and progress
    self.knowSwift.rx_value.map {
      $0 ? 0.25 : 0
    }
    .bindTo(self.swiftLevel.rx_value)
    .addDisposableTo(bag)

    self.swiftLevel.rx_value.map {
      $0 > 0
    }
    .bindTo(self.knowSwift.rx_value)
    .addDisposableTo(bag)

    // stepper
    self.passionToLearn.rx_value.skip(1).subscribeNext {
      self.heartHeight.constant = CGFloat($0 - 10)
    }
    .addDisposableTo(bag)


(疑问：双向绑定后，switch 和 progress 两个组件为什么不会死循环呢？)

#### 第八小节：基于 RxSwift 的网络编程 - I

这一小节将结果用 Dictonary<> 返回我觉得不合理，应该另外定义 Model，将 json 转成 model 返回

要点：

1. 利用 filter 过滤过短文字
2. 利用 throttle 过滤一段时间的事件
3. 利用 Observable.create 将 Alamofire 包装成 Observable
4. 使用 SwiftyJSON 处理返回的 response
5. 处理 Observable.create 参数的返回值，

        return AnonymousDisposable {
          request.cancel()
        }

6. 使用 flatMap 处理直接返回 Observable 的变换。

        private func searchForGithub(repositoryName: String) -> Observable<Repositories> {
          return Observable.create { (observer: AnyObserver<Repositories>) -> Disposable in
            let url = "https://api.github.com/search/repositories"
            let parameters = ["q": repositoryName + " stars:>=2000"]

            let request = Alamofire.request(.GET, url,
              parameters: parameters,
              encoding: .URLEncodedInURL)
              .responseJSON { response in
                switch response.result {
                case .Success(let json):
                  let repositories = self.parseGithubResponse(json)
                  observer.on(.Next(repositories))
                  observer.on(.Completed)
                case .Failure(let error):
                  observer.on(.Error(error))
                }
            }

            return AnonymousDisposable {
              request.cancel()
            }
          }
        }

        self.repositoryName.rx_text
          .filter {
            $0.characters.count > 2
          }
          .throttle(0.5, scheduler: MainScheduler.instance)
          .flatMap {
            self.searchForGithub($0)
          }
          .subscribeNext {
            print("Search result: \($0)")
          }
          .addDisposableTo(bag)

#### 第九小节：RxDataSource 创建 UITableView - I

嗯，这一节，有点绕，用了 RxDataSource，感觉不如使用原生的 datasource 方便和好理解。

暂时先略过吧。

#### 第十小节：RxDataSource 创建 UITableView - II

创建包含多个 setion 的 datasource，略过。

我会选择使用传统的方式实现。

#### 第十一小节：RxDelegate 代理 UITableView 事件

RxSwift 为我们提供了一种机制，叫做 DelegateProxy，可以帮助我们实现上面例子中的 rxDidSelectRowAtIndexPath。

类似中间件的概念吧，先略过。

2016/7/1 DONE!
