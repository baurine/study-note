# VS Code 使用

该文档只针对 Mac 平台。

## 1. 常用操作

命令：(与 Sublime 的操作几乎相同)

- `cmd + shift + p` : 快捷的命令面板，可以实时搜索到相应的命令、选项及其快捷键，并执行相应的命令。也可用 `cmd + p` 再输入 `>` 进入此模式

- `cmd + p` : 文件切换，并配合 `:` `@` `@:` `#`，还可实现方便的文件内跳转
  - 直接输入文件名，快速打开相应文件
  - `:` : 定位到当前文件的某一行，可以用 `ctrl + g` 快速打开
  - `@` : 定位到当前文件的某个符号，可以用 `cmd + shift + o` 快速打开
  - `#` : 定位到全局符号，可以用 `cmd + t` 快速打开
  - `>` : 等同于 `cmd + shift + p`
  - `?` : 查看帮助，这个条目里的内容其实就是从这个帮助命令里得知的，更多帮助可以用此操作来获取

文件导航：

- `alt + cmd + ->/<-` : 快速地在打开的文档间切换，已修改成 `cmd + shift + [/]`，和终端及其它编辑器的操作保持一致
- `ctrl + cmd + ->/<-` : 将当前文档移到右边或左边分栏
- `cmd + 1/2/3` : 快速地在分栏之间切换
- `cmd + \` : 快速分栏
- `cmd + shift + v` : 预览 markdown 文件
- `alt + cmd + t` : 关闭其它 tab

如何实时预览 markdown 文件。  
因为使用 `cmd + shift + v` 快捷键后，产生的预览文件与当前文件是在同一个分栏中，同一个分栏中的文件只能显示一个文件，所以我们再用 `ctrl + cmd + ->/<-` 把预览文档移到另一个分栏中，这样就可以左右分栏同时显示，实时预览。

格式化代码：

- `alt + shift + f` : 全文格式化
- `cmd + k, cmd + f` : 格式化选中部分

## 2. 配置

`cmd + ,` 打开配置脚本。只做了些轻微调整。

    "editor.fontFamily": "Fira Mono, Menlo, Monaco, 'Courier New', monospace",
    "editor.tabSize": 2,
    "editor.wrappingColumn": 0,
    "editor.renderWhitespace": "all",
    "editor.cursorStyle": "block",
    "workbench.editor.enablePreviewFromQuickOpen": false

另外，调整了部分快捷键，把 `cmd + shift + [/]` 调整成切换 tab。

## 3. 插件

- vim 是必不可少的
