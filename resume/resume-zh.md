# [姓名]

[性别] | [出生年份] | [手机号] | [邮箱]

您还可以通过以下链接对我进行深入的了解：

1. 个人博客 - <http://baurine.github.io/>
1. GitHub - <https://github.com/baurine>
1. StackOverflow (当前得分 1396) - <http://stackoverflow.com/users/2998877/spark-bao>
1. Study Notes Overview - <https://baurine.gitbooks.io/study-note/>

## 教育背景

- 2004 - 2008，北京化工大学 (211 院校) 电子信息工程，本科。
- 具备英语口语交流，阅读及写作能力，具有外企工作经历。

## 个人概述及技能

1. 扎实的计算机基础。
1. 具有不错的代码抽象能力，并且我认为代码抽象能力是一个程序员最重要的能力之一。
1. 生活中是狮子座，写代码时是处女座。具有良好的编码风格，不能忍受糟糕的代码。良好的文档意识，包括注释，README，Wiki，Git Commit Message，Issue。
1. 持续学习，并乐于与同事分享。在公司累计做过十余场技术分享，主题涉及 Android，iOS，前端，后端。

关于技能，出于好奇心与兴趣，涉猎广泛，曾从事芯片电路设计以及嵌入式开发，后来逐渐转到纯软件的应用层开发。目前比较擅长 Android 和 Web 开发。

### 基础

1. 仔细阅读过《C 程序设计语言》，《C++ Primer》和《Effective C++》，对指针的使用很有自信，理解面向对象、模板编程的思想。
1. 仔细阅读过《大话数据结构》，用 C/C++ 实现过各种数据结构，算法尚可。
1. 仔细阅读过《大话设计模式》，能理解和使用常用的设计模式。
1. 理解网络协议及 RESTful API 的使用和设计。

### Android

1. 掌握自定义 View 的开发。
1. 掌握 MVP，DataBinding，RxJava，Dagger2 思想。
1. 熟练使用 Retrofit。
1. 理解并尽量遵循 Material Design 设计规范。
1. 多个实际项目经验及开源项目。

### iOS

1. 理解 iOS 开发中的 MVC 思想。
1. 掌握 Swift 语言，理解 Protocol，闭包，Enum。
1. 会使用 RxSwift。

### 前端

1. 理解原型链，掌握 ES6 语法，箭头函数等特性。
1. 掌握 Promise / Generator / Async 的使用。
1. 理解 React 思想。
1. 理解 Redux 思想，Middleware，Thunk，Saga，参与作者教学视频的中文翻译 ([链接](https://github.com/Mr-Wiredancer/getting-started-with-redux))。
1. 理解 Vue 思想，参与 Vue 2.0 的文档翻译 ([链接](https://github.com/hayeah/vuejs.org))。
1. 使用 React Native 开发跨平台应用。

### 后端

1. 使用 Rails 开发网站和 API，完整阅读过《Ruby on Rails Tutorial》、《Agile Web Developement》、《Programing Ruby》、《Metaprogramming Ruby》，理解 Ruby 对象模型。
1. 理解 GraphQL 思想，并在 Rails 中实现 GraphQL API。
1. 会使用 PostgreSQL 及其全文搜索功能。
1. 会使用 Node & Express。

### 设计

1. 使用 Sketch 进行简单的设计，并自行切图，无须设计师切图。

### 其它

1. 熟练 Git 操作，会使用 `git reflog` 拯救误操作，`git rebase -i` 修改历史，熟悉 Gitflow, Git hooks。
1. 使用 Docker & GitLab 进行持续集成。
1. 日常 Vim。

## 项目经验

1. Pyro Music (Android)
   - DJ 音乐社交 APP，独立承担 Android 版的开发。
   - 具备一般网络音乐播放器的功能，比如网易云音乐，在线播放音乐，通知栏控制面板，播放列表管理。
   - 具备社交功能，登录/注册，Feed 流，关注，评论，赞，分享，私信，消息通知。
1. 试验助手 (iOS/Android)
   - 帮助医生管理病人，病人联系医生的 APP，具备日程管理，联系人管理，文件管理，日程通知等功能。
   - 使用 React Native 同时开发 iOS 和 Android 版本。
   - 使用 Redux 管理数据。
1. Airpocalypse (Android)
   - 一款简洁的天气应用。
   - MVP + DataBinding。
   - 抽取出 MultiTypeAdapter，SimplePullRefreshLayout，SwipeBackView，PermissionUtil 4 个单独的库。
1. PodKnife (Web)
   - 一个 Podcast 的搜索引擎，主要负责后端和用 React 实现的前端部分。
   - 采用 Rails + React-Rails 开发。
   - 借助 React-Rails gem，在 Rails 中使用 React Component 作为 View，并使用服务端渲染来优化 SEO。
   - 使用 PostgreSQL 的全文搜索功能实现搜索功能。
1. Cohort (Web)
   - 社交聚合应用。
   - 前端用 React 实现，后端 Python 提供 API，用 Node Express 作为中间层实现服务端渲染。
1. 公司新版网站 (Web)
   - 部分参与，采用 Meteor + React 开发。
   - Meteor 提供数据，React 显示数据。
   - 组件化思想，使用 React 实现每一个单独的 Component，然后在后台定义每一个页面由哪些 Compoent 组成。
1. MeShare (APP)
   - 一个智能摄像头的 APP 端，查看监控状况及对摄像头进行远程控制。
   - 作为核心开发人员，使用 C++ 为上层的 Android/iOS/Windows 应用编写底层核心功能库，用于和智能设备进行网络通信并进行各种控制。贡献了超过 70% 的 代码。
   - 使用了 Google 的 libjingle 线程 (类似 chrome base 库的线程模型) 和异步网络库作为框架，8 个以上的线程同时工作，使用了一些技巧，使除了框架本身消息队列的锁外，其它地方完全规避了锁的使用。进行压力测试，没有崩溃和内存泄漏。为了用好 libjingle，深入阅读并深刻理解了 libjingle 源码，并理解了 p2p 原理，stun 等网络协议。
1. 豌豆荚手机助手 (Windows)
   - 部分参与，使用 C++ 开发，使用 chrome base 库的线程模型。

## 工作经历

1. 2015/8 - 至今，Ekohe (易空海)，Mobile Developer Leader
   - 担任 Android 主程，承担各个项目 APP 端的主力开发，并协助其它同事进行前端和后端的开发。
   - 作为 Mobile Developer Leader，培养和管理新人，制定规范，技术分享。

1. 2013/9 - 2015/7，MeShare (微享)，C++ 主程
   - 作为核心开发人员，使用 C++ 为上层的 Android/iOS/Windows 应用编写底层核心功能库。

1. 2010/9 - 2013/7，豌豆荚，C++ 开发
   - 作为主力工程师参与豌豆荚 Windows 客户端 1.x 与 2.0 的开发，经历了用户数从几十万到数千万的增长。

## 业余爱好

跑步，游泳，骑行，健身。羽毛球和乒乓球打得也还可以。参加过奥运半程铁人三项。

从游泳中收获了一个重要的道理：只要你去练习，总是会有进步的，即使是微小的，终有一天，你能感受到量变带来的质变。知识，只要你一点一点地去啃，终有一天是能够理解的，你能感受到那种醍醐灌顶的感觉。
