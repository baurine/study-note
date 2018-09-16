# 《码农翻身》读书笔记

2018/9/15

P8: 多线程最大的一个问题就是竞争，多线程访问相同变量要加锁。加锁要按照资源 "大小" 的顺序来申请锁。

P75: 自旋锁，spin_lock

实现：

    // OS 保证这是原子操作，返回 true 表示没有拿到锁，返回 false 表示拿到锁
    boolean test_and_set(*lock) {
      boolean rv = *lock;
      *lock = true;
      return rv;
    }

    // 1. try get the lock
    while(test_and_set(&lock)) {
      // do nothing, just wait to get the lock
    }

    // 2. get the lock, do something
    x = x + 1

    // 3. release the lock
    *lock = false;

自旋锁是死循环，所以不可以重入 (即不能自己调用自己)，否则造成死锁。

解决办法：每次成功申请锁后，要记录到底是谁申请的，还要用一个计数器记录重入的次数，持有锁的家伙再次申请锁只是给计数器加 1 而已。释放锁的时候则把计数器减 1，等于 0 时才表示真正地释放锁。

自旋锁因为是死循环，所以浪费 CPU 资源 (但内核里用的是自旋锁啊)，更好的办法是，申请锁时，如果失败，则休眠等待，然后等系统通知唤醒后再去重新申请。

多线程加锁是为了解决互斥访问的问题，多线程的另一个问题是线程间的同步：我必须等待另一个或多个家伙完成以后才能开始工作。

用信号量解决线程的同步问题，`wait(var_1)` 等待锁，`signal(var_1)` 通知其它等待此信号量的线程。

P89: 递归与尾递归

尾递归，把结果作为参数传入到下一次的自身调用，感觉像 reduce 的操作。

尾递归只用一个栈桢，不怕栈溢出。

当递归调用是函数体中最后执行的语句，且它的返回值不属于表达式的一部分时，这个递归就是尾递归。现代的编译器就会发现这个特点，生成优化的代码，复用栈桢。

P97: Java ClassLoader

Java 用 class loader 动态加载 class，class loader 分级别，像 String 这些自带的类必须由最顶层的 Bootstrap ClassLoader 来加载 (防止 String 类被 hack)，自己写的类由底层的 App ClassLoader 加载。

P98: Java 虚拟机

Java 虚拟机是基于堆栈的虚拟机，所有操作 (加减乘除) 都是在栈上实现的，不像底层 CPU，这些操作是通过寄存器和运算器实现的。

(但 Android 上的 Java 虚拟机 Dalvik 却是基于寄存器，和 CPU 类似)。

P120: ACID

数据库事务 4 个特性：原子性 (Automicity)，一致性 (Consistency)，隔离性 (Isolation)，持久性 (Durability)

P147: Java 动态代理

这个还没太懂。

P153: Java 注解

Java 注解相当于是方法的元数据，元数据，描述数据的数据。

元注解：注解的注解，@Target / @Retention。

自定义注解的定义：

    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface Test {
      boolean ignore() default false;
    }

使用 @Test 注解：

    public class CalculatorTest {
      @Test
      public void testAdd() {
        System.out.println("test add method");
      }

      @Test(ignore=true)
      public void testSubstract() {
        System.out.println("test substract method");
      }
    }

获取 @Test 注解：

    public class TestUtil {
      public static void main(String[] args) throws Exception {
        CalculatorTest obj = new CalculatorTest();
        run(obj);
      }

      public static void run(Object obj) throws Exception {
        for (Method m : obj.getClass().getMethods()) {
          Test t = m.getDeclaredAnnotatin(Test.class);
          if (t != null && !t.ignore()) {
            m.invoke(obj)
          }
        }
      }
    }

`m.invoke(obj)` 相当于 `obj.m()`

P170: Java Log 系统，正交性

正交性，x, y, z 轴，独立变化而不会相互影响，威力无比。因为变化被封装在一个维度上，你可以把这些概念任意组合，而不会变成意大利面条似的代码。

P176: 加锁还是不加锁

互斥锁，称为悲观锁。

另一种不加锁也能解决线程间竞争的问题，称为 CAS (Compare and Swap)，有点像 spin_lock，也是死循环 (不知道能不能重入呢...)

原理，举例说明：

1. 取内存 A 的值，假如为 10；
1. 对 A 进行操作，比如加 1，存入 B；
1. 准备把新值写回 A 中，但写回之前检测一下 A 在内存中的值变化了没，即用之前取到的值 10 和现在内存中 A 的值比较一下，如果还是 10，说明 A 没有被别人修改过，可以把 B 写入 A，否则说明 A 被其它线程修改了，放弃修改，回到第一步。

最后一步是原子操作，由操作系统和硬件保证，就是 Compare and Swap 一条指令。

伪代码：

    public inte next() {
      while(true) {
        int A = *a;
        int B = A + 1;
        if (compareAndSwap(*a, A, B)) {
          return B;
        }
      }
    }

在 Java 中的实现：

    public class Sequence {
      private AtomicInteger count = new AtomicInteger(0)

      public int next() {
        while(true) {
          int current = count.get();
          int nextVal = current + 1;
          if (count.compareAndSet(current, nextVal)) {
            return nextVal
          }
        }
      }
    }

Atomic 类：

- AtomicBoolean
- AtomicInteger
- AtomicLong
- AtomicIntegerArray
- AtomicLongArray

对复杂的数据结构，可以使用 AtomicReference，但又会带来 ABA 的问题。

多个线程同时操作同一个引用，如果这个引用本身不发生变化，但修改的是引用所指向的对象的某个属性，那线程就会认为这个引用没有被修改。

解决办法就是给每个放入 AtomicReference 的对象都加入一个 version，叫 AtomicStampedReference。(所以每次修改都要同时修改 version 值，检测时也要检测 version 值。)

P182: Spring 的本质

AOP / IoC (DI)

问题来源：非功能性需求，是多个业务模块都需要的，是跨越模块的，比如日志，安全，事务，性能统计。

如何使这些非功能性需求和业务模块完美结合，又能够相互独立，不混杂在一起。

方法一：设计模式之模板方法

定义一个父类，在父类中把那些 "乱七八糟" 的非功能性代码写好，只留了一个口子 (抽象方法 doBusiness()) 让子类实现。子类变得清爽，只需要关注业务逻辑就可以了。

(我在 Android 中就是这么做的，在 BaseActivity 和 BaseFragment 中定义了大多数 Activity 和 Fragment 都要做的操作。)

缺点：不灵活，父类定义一切，要执行哪些非功能性代码，以什么顺序执行，等等，子类只能无条件接受。

方法二：设计模式之装饰器

装饰器就是包裹器，或者说 hook。

示例：

    public interface Command {
      public void execute();
    }

    // 用于记录日志的装饰器
    public class LoggerDecorator implements Command {
      Command cmd;
      public LoggerDecorator(Command command) {
        this.cmd = command;
      }
      public void execute() {
        Logger logger = Logger.getLogger(...);
        // 记录日志
        logger.debug("...");
        this.cmd.execute();
        logger.debug("...");
      }
    }

    // 用于性能统计的装饰器
    public class PerformanceDecorator implements Command {
      ...
    }

    class PlaceOrderCommand implements Command {
      public void execute() {
        // 执行下单操作
      }
    }

使用：

    Command cmd = new LoggerDecorator(
      new PerformanceDecorator(
        new PlaceOrderCommand()))
    cmd.execute()

装饰器比模板方法的优点之一就是灵活一点，可以使用任意的装饰器，还可以以任意次序执行。

仍然不够好，因为：

1. 一个处理日志/安全/事务/性能统计的类为什么要实现业务接口？
1. 如果其它业务模块没有实现 Command 接口，但是也想用这些非功能性模块呢？

最好的办法：把日志/安全/事务/性能统计这样的非功能性代码和业务代码完全隔离开来，因为它们的关注点和业务代码的关注点完全不同，它们之间应该是正交的。(又是**正交**)

如果把业务功能看成一层层面包，那么这些日志/安全/事务/性能统计像是一个个 "切面" (Aspect)。

如果我们能让这些 "切面" 和业务独立，并且能够非常灵活地 "织入" 业务方法中 ... 这就是面向切面编程 (AOP)。

AOP 的实现：动态代理 + XML 配置。

P210: JWT

从示图上看，JWT 整个过程中，只有服务器端有一个秘钥，不存在公钥私钥，只有加密，没有解密。

生成签名的过程，对 header 和 body 的内容进行 hash，用秘钥加密，生成签名。

收到 token 后的检验过程，从 token 中取出 header 和 body，进行 hash，用相同的秘钥加密，生成签名，和 token 中的签名进行比较，看是不是相同。相同则此 token 有效，没有被篡改。

P220: OAuth 的三种方式

- 直接让用户输用户名密码
- 隐式许可 (直接给可用的 token，容易泄露在 url 中)
- 授权码许可 (先给一个授权码，通过这个授权码再换直接可用的 token)

P227: 分布式

余数算法、一致性 Hash 算法、Hash 槽 (slot)

P256: select 与 epoll

Linux 上一个进程使用 select 最多只能监听 1024 个 socket，如果想支持更多连接，使用多进程。

epoll 没有这个限制。

P306: 命令式编程和声明式编程

命令式编程风格：指令清晰，面面俱到，在什么时间，做什么事情，怎么做，描述得非常清晰。

声明式编程风格：不说具体怎么做 (How)，只会描述要做什事 (What)。

示例：SQL，SQL 的最大特点是只声明我想要什么 (What)，就是不说怎么做 (How)。

    select name from students where score > 80;

另一个例子是函数式编程：

    int count = students.stream()
      .filter(s -> s.getAge() > 18)
      .count();

(但我觉得函数式编程的内部还是用命令式实现的，比如上面的 filter 高阶函数。)
