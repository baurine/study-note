# Mac Preferences

## Mac 操作

- `cmd + ,` -- 快速调出各应用的设置界面

## 主题

- [Material theme for zsh / iTerm / terminal](https://github.com/carloscuesta/materialshell)
- [Material theme for Android Studio](https://github.com/ChrisRM/material-theme-jetbrains)

注意，主题只包括配色，不包括字体和字号，这个需要额外自己再配置。

## 字体

- [Fira Mono](https://mozilla.github.io/Fira/)

安装字体时，先打开 Font Book 应用，点击工具栏的 "+" 按钮，添加新字体，在弹出的窗口中，选择下载后解压生成的字体文件夹，这样可以将文件夹中的所有字体一次性全部安装上。不要在文件夹里一个字体一个字体双击安装。

## 常用工具

1. [LICEcap](https://www.cockos.com/licecap/) - GIF 格式录屏应用
1. [Typora](https://typora.io/) - Markdown Editor

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

另一个选择是 [Sourcegraph](https://about.sourcegraph.com/)。

## mac 开发环境配置

### 通用开发环境

参考：

1. [Mac 开发配置手册](https://aaaaaashu.gitbooks.io/mac-dev-setup/content/index.html)
1. [Mac OS X 配置指南](https://mac-setup.wildflame.org/)

配置过程：

1. 安装搜狗五笔 (官网下载安装)
1. 安装 chrome (官网下载安装)
1. 安装 [homebrew](https://brew.sh/) (官网有教程，一条指令)

        /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

1. 安装 iTerm2 (官网下载安装)
1. 安装 zsh 和 oh-my-zsh

   zsh 是 shell，oh-my-zsh 是 zsh 扩展功能和主题集合，通过 homebrew 来安装。

        # install zsh
        $ brew install zsh zsh-completions

        # install oh-my-zsh
        $ curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

1. 安装 [tmux](./tmux.md) (可选)
1. 下载 material-theme 主题 (只是配色)，分别为 zsh 和 iTerm2 配置此主题，记得 2 个都要配置才行。

   配置步骤在 material-theme 的 README 中有。

   把 mac 自带的 terminal 也配置上此主题。

   这个主题里有两种配色，一种是 dark，一种是 oceanic，选择 oceanic。

1. 下载 Fira 家族字体，为 iTerm2 配置 Fira Mono 字体
1. 安装 VSCode，修改字体为 Fira Mono，修改字号，显示 whitespace，设置 tabSize，安装 vim 插件

        {
          "editor.tabSize": 2,
          "editor.renderWhitespace": "all"
        }

1. 安装最新版的 Git，配置基本信息和私钥 (看 Git Note config 部分)

   使用 `git config --global` 配置 user.name 和 user.email，core.excludesfile

        $ git config --global user.name xxxx
        $ git config --global user.email xxxx
        $ git config --global core.excludesfile ~/.gitignore_global

        # ~/.gitignore_global
        *~
        .DS_Store
        .githook
        _book
        pods
        *.xcbkptlist

1. 安装 [GitUp](http://gitup.co/) (官网下载安装)

**brew**

查看某个应用 (比如 mysql) 的信息，比如如何启动：

    $ brew info mysql
    ...
    To have launchd start mysql now and restart at login:
      brew services start mysql

启动，重启，停止 mysql 服务：

    $ brew services start|restart|stop mysql

查看服务列表：

    $ brew services list
    Name  Status  User       Plist
    redis started ...
    php71 stopped
    nginx stopped
    mysql stopped

**iTerm2**

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

### Rails 开发环境

安装 rvm & ruby & bundler (看 Rails Study Note 配置)

### Node 开发环境

官网下载 node 安装包安装，安装包中自带 npm

### Android 开发环境

经过测试，只要安装一个 Android Studio 3.0 就行了，安装过程中会自动把需要的依赖都下载安装，连 Java 环境都不用装了，好神奇。

但是如果要在命令行下使用 gradle 命令手动进行编译，还是需要额外安装 Java SDK 的。直接到 [Oracle 官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载最新的安装包进行安装就行了，比如 jdk-8u161-macosx-x64.dmg。

安装了 JDK 后在命令行中就能直接使用 java 命令了，用 `java -version` 查看版本号。我并没有设置 `JAVA_HOME` 和 CLASSPATH 环境变量，并将它们写到 PATH 中，也没出什么问题，在命令行下用 gradle 命令编译 apk 成功了。如果后面遇到问题再配置 `JAVA_HOME` 和 CLASSPATH。

另外在命令行下使用时，还需要将 Android SDK 的路径写到环境变量 `ANDROID_HOME` 中。

通过 Android Studio 安装的 Android SDK 默认放在 ~/Library/Android/sdk 目录下，所以在 ~/.zshrc 最后加一行：

    export ANDROID_HOME=~/Library/Android/sdk

### iOS 开发环境

1. 安装 XCode
1. 安装 [CocoaPods](https://cocoapods.org/)

        $ sudo gem install cocoapods

### Web 开发环境

至少还需要安装的有：

1. PostgreSQL (看 PostreSQL Note)
1. MySQL (看 MySQL Note)
1. MongoDB (看 MongoDB Note)
1. Redis (看 Redis Note)
1. Nginx (看 Nginx Note)
1. Python3 (看 Python3 Note)

### 设计

- 设计工具：Sketch
- 原型工具：[Figma](https://www.figma.com/)
- 流程图：[draw.io](https://www.draw.io/)
