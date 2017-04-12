# Ruby Note

## `attr_reader / attr_writer / attr_accessor`

Ruby 的类的成员变量的定义方式是 `@variable`，但是在类外部，我们无法通过 `instance.@variable` 的方式访问它。这种机制决定了 ruby 的类成员变量只能是私有的，如果想从外部访问它，就必须为它定义公开的方法，或者使用属性访问器。

属性访问器的原理是自动为这些成员变量生成公开方法。所以可见，对于 ruby 来说，对外公开的只有方法，没有属性。这个特性好啊，相比 Java 这种语言，在实现 Model 时，就会很犯难，到底是直接把属性都 public 了，还是 private + getter / setter，后者实在是啰嗦。

### 1. 手动定义方法

    class Foo
      def initialize
          @valid = false
      end

      def valid?
          @valid
      end
    end

### 2. 使用属性访问器

    class Foo
      attr_reader :name
      attr_writer :age
      attr_accessor :gender

      attr_reader :valid
      alias valid? valid

      def initialize(name)
        @name = name
        @valid = false
      end
    end

`attr_reader` 定义的成员变量，可以在外部被访问，但不能在外部被修改，只能在内部被修改。`attr_reader :name` 等价于：

    def name
      @name
    end
 
`attr_reader` 定义的成员变量在内部的访问方法：

    @name = 'bar'

如果在内部访问时不加 `@`，那么实际访问的是局部变量

    # 局部变量
    name = 'bar'

在外部访问的方式：

    f = Foo.new
    puts f.name

    f.name = 'foo' # ERROR! 不能被修改，不存在 f.name= 方法

`attr_writer` 定义的成员变量，可以在外部被修改，但不能在外部被访问。`attr_writer :age` 等价于：

    def age=(val)
      @age = val
    end

在外部被修改：

    f = Foo.new
    f.age = 20

    puts f.age # ERROR! 不能被访问，不在存 f.age 方法

使用 `attr_writer` 定义的成员变量，在内部被修改时，有 2 种使用方式：

    @age = 40
    # or
    self.age = 40

    # 局部变量
    age = 40

`attr_accessor` 定义的成员变量，既可以在外部被访问，也可以被修改。`attr_accessor :gender` 等价于：

    def gender
      @gender
    end
    
    def gender=(val)
      @gender = val
    end

对于 boolean 型的成员变量，我们习惯于在它的访问方法名中加上 `?`，所以我们可以用 alias 关键字来定义一个别名。

    attr_reader :valid
    alias valid? valid

使用：

    f = Foo.new
    puts f.valid?
