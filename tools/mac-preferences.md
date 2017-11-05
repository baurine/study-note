# Mac Preferences

## Mac 操作

- `cmd + ,` -- 快速调出各应用的设置界面

## 主题

- [Material theme for zsh/iTerm/terminal](https://github.com/carloscuesta/materialshell)
- [Material theme for Android Studio](https://github.com/ChrisRM/material-theme-jetbrains)

## 字体

- [Fira Mono](https://mozilla.github.io/Fira/)

## 工具

1. homebrew
1. zsh
1. iTerm
1. [tmux](./tmux.md)
1. ~~TotalSpace~~ (用了一段时间，觉得不是很需要)
1. ~~Alfred~~ (用了一段时间，觉得并不是很需要它的高级功能，而基本功能用 Spotlight 就够了)

**iTerm**

- `cmd + w`：关闭当前 pane
- `-------`
- `cmd + d`：左右分屏
- `cmd + shift + d`：上下分屏
- `cmd + [/]`：切换分屏
- `-------`
- `cmd + t`: 新建 tab
- `cmd + shift + [/]`：切换 tab
- `-------`
- `cmd + n`：新建窗口 (勾选上 Preference --> Profile --> General --> "Reuse previous session's directory")

**TotalSpace2 (Deprecated)**

简化 Mac 的多桌面切换。Serria 系统以上需要关闭系统完整性校验功能才能使用。

**Alfred (Deprecated)**

替代系统自带的 Spotlight。

## Chrome 插件

- Vimium
- Octotree / Sourcegraph
- JSONView
- Postman
- Grammarly
- Momentum

**Vimium**

使用 vim 的操作方式操作 chrome

- `shift + f`：显示所有可点击的链接，每一个链接都可以用一两个按钮打开，这一点相当地酷，超爱。很受启发，我觉得所有软件都应该加上类似这种 feature。对软件界面上的每一个可点击的按钮，理论上都应该有一个快捷键，这种才能完全摆脱鼠标的使用，但快捷键是有限的，多了容易冲突，因此可以使用 Vimium 这样的方式，点击 `sfift + f`，为每个按钮随机重新分配新的快捷键。

**Octotree**

Code tree for Github, super useful! Github 的完美搭档。

但是新出的 [Sourcegraph](https://about.sourcegraph.com/) 比 Octotree 更强大，可用来替代 Octotree。

## mac 开发配置

参考：

1. [Mac 开发配置手册](https://aaaaaashu.gitbooks.io/mac-dev-setup/content/index.html)
1. [Mac OS X 配置指南](https://mac-setup.wildflame.org/)

配置过程：

1. 安装搜狗五笔
1. 安装 chrome
1. 安装 homebrew
1. 安装 iTerm2
1. 安装 zsh 和 oh-my-zsh
1. 下载 material-theme 主题，分别为 zsh 和 iTerm2 配置此主题 (其实主要是配色)
1. 下载 Fira-Mono 字体，为 iTerm2 配置此字体
1. 安装 VSCode，修改字体为 Fira-Mono，修改字号，显示 whitespace, 设置 tabSize, 安装 vim 插件
1. 安装最新版的 Git，配置基本信息和私钥，安装 GitUp
1. 安装 rvm & ruby & bundler
1. 安装 node & npm
