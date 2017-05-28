# YAML Note

## References

1. [YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)
1. [YAML in Wikipedia](https://zh.wikipedia.org/wiki/YAML)

## Note

不复杂的东西，看上面 2 篇链接就行了。记住前面有 `-` 的表示 list，没有 `-` 表示 hash 基本就行了。高级一点的用法就是引用，有点继承的意思，用 `&` 表示锚点，`<<` 表示合并到当前数据，`*` 用来引用锚点。这个用法在 rails 的工程配置文件中经常见到。

    defaults: &defaults
      adapter:  postgres
      host:     localhost

    development:
      database: myapp_development
      <<: *defaults

    test:
      database: myapp_test
      <<: *defaults

相比 JSON 的优势：允许注释，支持引用。

相比 JSON 的劣势：强制缩进。一方面是有人不喜欢强制缩进，另一方面是在网络上进行传输时，无法消除空格，体积会比 JSON 大，不过貌似现在也没有人使用它来在网络上传输数据，毕竟 JSON 已经一统天下了，所以 YAML 作为本地配置文件还是有优势的。
