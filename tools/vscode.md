# VS Code 使用

该文档只针对 Mac 平台。

## 1. 常用操作

命令：

- `cmd + p` : 打开快捷操作输入框，默认操作是快速打开某个文件，如果首字符为特殊字符，比如 `>`, `?`, `:`, `#`, `@`, `@:`，则变为其它各种快捷操作
  - `?` : 查看帮助，这个条目里的内容其实就是从这个帮助命令里得知的，更多帮助可以用此操作来获取
  - `>` : 快捷命令，可以用 `cmd + shift + p` 快速打开
  - `:` : 定位到当前文件的某一行，可以用 `ctrl + g` 快速打开
  - `@` : 定位到当前文件的某个符号，可以用 `cmd + shift + o` 快速打开
  - `#` : 定位到全局符号，可以用 `cmd + t` 快速打开

- `cmd + shift + p` : 等同于 `cmd + p` 再输入 `>`

文件导航：

- `alt + cmd + ->/<-` : 快速地在打开的文档间切换
- `ctrl + cmd + ->/<-` : 将当前文档移到右边或左边分栏
- `cmd + 1/2/3` : 快速地在分栏之间切换
- `cmd + \` : 快速分栏
- `cmd + shift + v` : 预览 markdown 文件

如何实时预览 markdown 文件。  
因为使用 `cmd + shift + v` 快捷键后，产生的预览文件与当前文件是在同一个分栏中，同一个分栏中的文件只能显示一个文件，所以我们再用 `ctrl + cmd + ->/<-` 把预览文档移到另一个分栏中，这样就可以左右分栏同时显示，实时预览。

## 2. 配置

`cmd + ,` 打开配置脚本。只做了些轻微调整。

    "editor.fontFamily": "Fira Mono, Menlo, Monaco, 'Courier New', monospace",
    "editor.tabSize": 2,
    "editor.wrappingColumn": 0,
    "editor.renderWhitespace": "all",
    "editor.cursorStyle": "block"

## 3. 插件

- vim 是必不可少的
