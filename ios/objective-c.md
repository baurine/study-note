# Objective-C

简单了解一下，目的是能看懂 Objective-C 的代码。

以前对 Objective-C 有偏见，觉得它的语法其丑无比，但最近稍微深入了解了一下 Objective-C 的语法，觉得其实它的写法还是蛮有意思的，一次调用就像是在写一个人人都能看懂的英文句子一样，这不正是 Ruby 想要达到的目的吗?

## References

- [Objective-C 入门教程](http://www.runoob.com/w3cnote/objective-c-tutorial.html)
- [Objective-C 语法总结](http://mouxuejie.com/blog/2016-05-28/ios-learn-4/)
- [iOS 开发 60 分钟入门](https://github.com/qinjx/30min_guides/blob/master/ios.md)
- [Objective-C 语法快速参考](http://blog.jobbole.com/66202/)

## Note 1

Note for [Objective-C 入门教程](http://www.runoob.com/w3cnote/objective-c-tutorial.html)

### Objective-C - C 的超集

Objective-C 是 C 的超集，因此它兼容任何 C 的语法。它在 C 的基础上扩展了面向对象编程。

这里的内容只针对 Objective-C 面向对象这部分，如果不涉及类和对象的语法，基本和 C 一样。

#### 文件名

- .h - 头文件，声明接口
- .m - 源代码，可以包含 Objective-C 和 C 的实现代码
- .mm - 源代码，除 Objective-C 和 C 代码外，还可以包括 C++ 代码。

在源代码中包含头文件时，可以用 C 的 `#include` 编译选项，但更推荐用 Objective-C 的 `#import`，它作用和 `#include` 一样，但可以确保相同的文件只被包含一次。

#### 语法

Objective-C 的面向对象语法源于 Smalltalk 消息传递风格。所有其他非面向对象的语法，包括变量类型，预处理器，流程控制，函数声明与调用皆与 C 语言完全一致。

#### 消息传递

> Objective-C 最大的特色是承自 Smalltalk 的消息传递模型 (message passing)，此机制与今日 C++ 式之主流风格差异甚大。Objective-C 里，与其说对象互相调用方法，不如说对象之间互相传递消息更为精确...

C / C++ 的方法调用在编译时就决定了，而 Objective-C 的消息在运行时动态决定，典型的鸭子模型。(所以 Objective-C 综合了静态语言和动态语言的特点。)

#### 字符串

C 语言的字符串用起来不方便，Objective-C 一般使用 NSString 对象处理字符串，和其它高级语言处理字符串一样方便。只需要在一般的字符串前面加 `@` 符号。

    NSString* myString = @"My String\n";
    NSString* anotherString = [NSString stringWithFormat:@"%d %s %@", 1, "String", @"NSString"];

    // 从一个 C 语言字符串创建 Objective-C 字符串
    NSString*  fromCString = [NSString stringWithCString:"A C string"
                                       encoding:NSASCIIStringEncoding];

### 类

Objective-C 的类的说明和 C++ 一样，包含两部分：

- 类的声明 (interface)，写在 .h 中
- 类的实现 (implementation)，写在 .m 中

#### 类的声明

类的声明中只包括公开的变量和方法。

语法：

    @interface MyClass : NSObject
    {
      int       count;
      id        data;
      NSString* name;
    }
    - (id)initWithString:(NSString*)aName;
    + (MyClass*)createMyClassWithString:(NSString*)aName
                andString:(NSString*)bName;
    @end

- 类的声明用 @interface 编译选项开始，@end 结束
- 类名之后是继承自的父类，用冒号隔开
- 类的实例成员定义在大括号中
- 类的实例方法前用 `-` 声明
- 类的类方法前用 `+` 声明
- `()` 中声明的是方法的返回类型，以及参数的类型 (后面有更详细的说明)

**强类型与弱类型**

    MyClass *myObject1; // 强类型声明
    id myObject2;       // 弱类型声明

弱类型 id 一般用于集合中，且隐含了指针类型，类似 `void*`。

#### 类的实现

类的实现中，包括公开方法的实现，以及私有变量和方法。

语法：

    @implementation MyClass
    {
      int memberVar3; // 私有成员变量
    }
    + (MyClass*)createMyClassWithString:(NSString*)aName
                andString:(NSString*)bName {
      // ...
    }
    - (id)initWithString:(NSString*)aName {
      // ...
    }
    // 其它私有方法
    // ...
    @end

#### 创建对象

Objective-C 创建对象需通过 alloc 以及 init 两个消息。alloc 的作用是分配内存，init 则是初始化对象。 init 与 alloc 都是定义在 NSObject 里的方法。

    MyObject *my = [[MyObject alloc] init];

在 Objective-C 2.0 里，若创建对象不需要参数，则可直接使用 new。

    MyObject *my = [MyObject new];

若要自己定义初始化的过程，可以重写 init 方法。

### 方法

方法声明语法：

    - (void)insertObject:(id)anObject atIndex:(NSUInteger)index

- `-` 表示这是一个类的实例方法，`+` 表示类方法
- `(void)` 表示方法的返回类型
- `(id)` `(NSUInteger)` 表示参数类型
- `andObject` `index` 表示参数名
- `insertObject:` `atIndex:` 方法签名关键字，包括冒号，如果没有冒号，说明这个方法不需要参数。整个方法实际的名字是这些方法签名关键字的级联，比如这里，方法实际名字是 `insertObject:atIndex:`。

#### 发消息 (即方法调用)

Objective-C 用 `[]` 来表示方法调用 (不知道是不是仅此一家)。

    [myArray insertObject:anObj atIndex:0];

允许 `[]` 的嵌套。

    [[myAppObject getArray] insertObject:[myAppObject getObjectToInsert] atIndex:0];

### 属性

简单地说，属性就是帮你自动生成 setter 和 getter 方法用的。类似 ruby 的 `attr_writer` `attr_reader` `attr_accessor` 用法。

(这一小节写得不好)

    @property BOOL flag;
    @property (copy) NSString *nameObject; // 赋值时拷贝对象
    @property (readonly) UIView *rootView; // 只声明 getter 方法

每个可读属性将指定一个与属性同名的方法。上例中 rootView 将会得到一个 rootView 的方法。

    - (UIView*)rootView {
      // 类似实现
      return self._rootView;
    }

每个可写属性将指定一个形如 `setPropertyName:` 的方法，在其中属性名的第一个字母被转换为大写。上例中 nameObject 将会得到一个 `setNameObject:` 方法。

    - (void)setNameObject:(NSString*)nameObject {
      // 类似实现
      self._nameObject = [NSString stringWithFormat:@"%@", nameObject];
    }

如果要对一个属性同时产生 getter 和 setter 访问方法，可以用 `@synthesize` 声明。同时，如果用 `@dynamic` 声明，则表示 getter 和 setter 访问方法由开发者自己实现。

对属性的访问，具有多种形式，可以用传统的 `.` 号，还可以用 C++ 的 `->`，还可以用 `valueForKey:` `setValue:forKey:` 等形式。

    Person *aPerson = [[Person alloc] initWithAge: 53];

    aPerson.name = @"Steve";
    // 等于
    [aPerson setName: @"Steve"];

    NSLog(@"Access by message (%@), dot notation(%@), property name(%@) and direct instance variable access (%@)",
          [aPerson name], aPerson.name, [aPerson valueForKey:@"name"], aPerson->name);

类或协议的属性可以被动态的读取。略。

### 枚举

    // 使用NSEnumerator
    NSEnumerator *enumerator = [thePeople objectEnumerator];
    Person *p;

    while ( (p = [enumerator nextObject]) != nil ) {
      NSLog(@"%@ is %i years old.", [p name], [p age]);
    }

    // 使用依次枚举
    for ( int i = 0; i < [thePeople count]; i++ ) {
      Person *p = [thePeople objectAtIndex:i];
      NSLog(@"%@ is %i years old.", [p name], [p age]);
    }

    // 使用快速枚举
    for (Person *p in thePeople) {
      NSLog(@"%@ is %i years old.", [p name], [p age]);
    }

### 协议 - Protocol

Swift 中也有这个概念，是一样的意思，简单地理解成类似接口的概念。

声明语法：

    @protocol Locking
    - (void)lock;
    - (void)unlock;
    @end

用 @protocol 开始，用 @end 结束。

某个类表示遵循此协议：

    @interface SomeClass : SomeSuperClass <Locking>
    @end

实现此协议：

    @implementation SomeClass
    - (void)lock {
      // ...
    }
    - (void)unlock {
      // ...
    }
    @end

### Category

为已存在类扩展方法，对应应该就是 Swift 中的 extension。

其余略。

---

## Note - 2

Note for [Objective-C 语法总结](http://mouxuejie.com/blog/2016-05-28/ios-learn-4/)

### 对象

Objective-C 和其它语言最大的不同的一点在于，方法调用不用 `.` 号，而是用 `[]`，仅此一家了吧，而且几乎每个参数都有名字，参数之间也不用 `,` 分隔，而直接用空格分隔，参数不用 `()` 包围，而是跟在参数名字的 `:` 号。

当然，在 Objective-C 里，也不叫方法调用，而是叫给某个对象发消息，这个和 Ruby 是相同的思想。这决定了 Objective-C 的语言具有一种运行时的动态性。

来看看 Objective-C 如何调用对象的方法 (我们还是按传统地叫法称呼之)。

    Party *partyInstance = [Party alloc];
    [partyInstance init];

    [partyInstance addAttendee: somePerson
                   withDish: deviledEggs];

感受到了吗? `partyInstance addAttendee somePerson withDish deviledEggs`，是不是一句完整的句子。

上面同时演示了如何创建对象，一般需要 alloc 和 init 两个方法，alloc 为对象分配内存，但还没有初始化值，所以还不能正常工作，init 方法调用后，对象被初始化，才有了值，才可以正常工作。

对象的销毁：

    partyInstance = nil;

> 值得注意的是，Objective-C 中可以给空对象发消息，因此不需要做判空操作。(这就体现了 Objective-C 的动态性)

#### 常见对象

    NSObject  <---- NSArray   <---- NSMutableArray
              <---- NSString  <---- NSMutableString

NSObject 是所有类的父类。

**1 - NSString**

- NSString 表示不可变可符串
- NSMutableString 表示可变字符串

字符串表示：

    NSString *myString = @"Hello, world";

字符串内容用 `@` 符号开头。

格式化输出：

    int a = 1;
    float b = 2.5;
    char c = 'A';
    NSString *d = @"Hello";
    NSLog(@"Integer: %d, Float: %f, Char: %c, String: %@", a, b, c, d);

其余略。

**5.4 关于 autoreleasepool 和 ARC**

ARC 即 Automatic Reference Counting，自动引用计数。是相对于 MRC (Manual Reference Counting，手动引用计数) 而言的。

若使用 @autoreleasepool 关键字括起来，当括号里方法执行完时，实例对象会自动释放。

    @autoreleasepool {
      BNRItem *item = [BNRItem someItem];
    }

iOS 使用 ARC 和 autoreleasepool 机制实现内存管理。
