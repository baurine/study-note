# Prometheus and Grafana

资源：

- [prometheus-book](https://github.com/yunlzheng/prometheus-book)
- [prometheus docs](https://prometheus.io/docs/introduction/overview/)
- [grafana docs](https://grafana.com/docs/)
- [InfluxDB docs](https://github.com/jasper-zhang/influxdb-document-cn)
- [时序数据库 InfluxDB 使用详解](https://juejin.im/entry/5b7394d5e51d4566633b2400)

顺便把 InfluxDB 的文档也看了一下，时序数据库，重点在于添加和查询数据，一般不允许修改和手动删除数据。和通用的数据库使用场景不一样。InfluxDB 的查询语句和 SQL 差不多。时序数据库有标签 (tag) 的概念，类似通用数据库中有索引的列，没有索引的列则为值。标签用于查询时的条件，而值为查询结果。

prometheus-book 讲得很详细了。

整体架构：

![](../art/prometheus-arch.png)

监控系统及可视化。prometheus 和 grafana 是两套独立的系统，但人们一般将它俩配套使用，前者负责抓取管理监控数据，后者专注展示监控数据，前者通过提供 HTTP API 供后者使用。(前后端分离的好处)

prometheus 内部使用自己专门的时序数据库，没有用 InfluxDB。其查询语法是 PromQL，和普通的 SQL 不一样，但很好理解。

对于 prometheus，之前我有个误解，以为像 exporter 这些产生监控数据的客户端，在获得监控数据的那一刻，就会把当时的时间戳及数据保持在本地，然后等 prometheus 来抓取后直接放进数据库中。实际并不是这样：

1. 时间戳在由 prometheus 存到时序数据库中那一刻产生，在其它环节并不会保存时间。所以，数据真正产生的那一刻的时间和它存到数据库中的时间戳并不能完美的对应。因此监控图展示的情况只能是一个大概，并不完全精确。
1. exporter 提供类似 `/metric` 这样的 http 接口供 prometheus 抓取，exporter 的抓取的数据并不一定需要持久化，它的数据有可能一直在内存中。
1. exporter 可能并不累积数据。比如 prometheus 设置了每 15s 从 exporter 抓取一次数据，而 exporter 每 5s 获取一次数据。当 prometheus 抓取数据时，exporter 已经获取了 3 次数据了，但 exporter 并不会将这三次的数据全部累积在内存中，而只是留下最新一次获取的数据并返给 prometheus，它总是会用最新的数据覆盖历史的数据。因此，它并不会产生数据堆积 (我之前有这个疑惑)。

grafana 专注展示，它支持很多数据源，比如 InfluxDB，而 prometheus 只是它支持的对象之一。

根据 prometheus-book 在 macOS 上搭建一个最小演示环境：

1. 官网下载 prometheus 的安装包，解压后运行 `./prometheus`，可以在浏览器通过 http://localhost:9090 访问 prometheus 自己的简陋的 web ui
1. 官网下载 `node_exporter` 安装包，解压后运行 `./node_exporter`，它向 prometheus 暴露了 `localhost:9100/metrics` 接口
1. 在 prometheus 的解压目录下修改 prometheus.yml，声明增加一个抓取监控数据的 job，它的抓取目标是 `localhost:9100`，即 `node_exporter`，然后重启 prometheus

   ```yml
   scrape_configs:
     # default
     - job_name: "prometheus"
       static_configs:
         - targets: ["localhost:9090"]

     # new
     - job_name: "node"
       static_configs:
         - targets: ["localhost:9100"]
   ```

1. 通过 docker 安装并启动 grafana

   ```shell
   $ docker run -d -p 3000:3000 grafana/grafana
   ```

1. 访问 `localhost:3000` 进入 grafana，添加 data source，选择 prometheus，在 Settings 的 URL 中默认值是 "http://localhost:9090"，一直添加不上，后来请教了同事，因为 grafana 是通过 docker 启动的，而 prometheus 是在本机启动的，所有前面的 localhost 并不是指本机，而是 docker 中的本机网络，然后把 localhost 改成本机 IP 后就添加上了。(?? 感觉哪里不对)

如上面配置文件 prometheus.yml 中所示，每一种 job 就是抓取同类数据的任务，相同的 job 可能存在多个实例提供数据，比如：

```yaml
- job_name: "node"
  static_configs:
    - targets: ["localhost:9100", "192.168.1.100:9100"]
```

其它理解起来也没有多大难度。

接下来重点看看 grafana，目标是能为它写一些 plugin。

正如上面所言，grafana 支持多种数据源，prometheus 只是它支持的对象之一，因此它可以把多种数据源汇总到一起显示。

可以在 grafana 上创建多个 dashboard，然后在每个 dashboard 中放置多个 panel。不同的 panel 可以选择不同的数据源，但同一个 panel 只能使用一种数据源，然后对每一个 panel 可以设置它的图表类型 (graph/singlestat/gauge...)，添加查询语句，同一个 panel 可以添加 n 条同类的查询语句，然后将数据汇总显示在同一个 panel 上进行对比显示。

(把官方文档仔细看一遍...把整体目录瞄了一眼，没有很复杂的东西，很多细节的东西先跳过，需要时再回来查，先重点看看 plugins 一节。)

## PromQL

- [PromQL tutorial for beginners and humans](https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085)
- [PROMQL FOR HUMANS](https://timber.io/blog/promql-for-humans/)

## Grafana Plugin

- [Grafana Panel 插件开发实践](https://juejin.im/post/5acb16db6fb9a028bc2e0ab9)
- [Grafana 的 Datasource 插件开发实践一](https://juejin.im/post/5addbcbd5188256715474452)
- [Grafana 的 Datasource 插件开发实践二](https://juejin.im/post/5ae188f0f265da0b8c24aab0)
- [Grafana 插件开发入门](https://my.oschina.net/u/2699666/blog/1615957)
- [官方文档](https://grafana.com/docs/plugins/developing/development/) - 不是很详细
- [官文示例 - clock panel](https://grafana.com/blog/2016/04/08/timing-is-everything.-writing-the-clock-panel-plugin-for-grafana-3.0/) - 很有用
- [Developing Your Visualization Plugin for Grafana](https://dzone.com/articles/development-of-visualization-plugin-for-grafana)

另外，官方 plugins 的源码部分放在官方 repo 的 public/app/plugins 目录中，比如：https://github.com/grafana/grafana/blob/master/public/app/plugins/panel/piechart/PieChartPanel.tsx

Grafana 插件有三种：panel, datasource, app，panel 是指图表，datasource 是指数据源，app 应该是指 panel + datasource 吧，重点关注 panel。panel 主要又分两种：PanelCtrl，MetricPanelCtrl，前者比较独立，不需要与外部数据打交道，比如计时器这种，而后者需要外部从数据源获取的数据并渲染。

数据源是数据的生产者，panel 是数据的消费者。数据源从任意处获取数据，加工成统一的格式，然后传递给 panel 进行绘制。目前统一的数据格式有两种，一种是 time series，一种是 table。前者所有 datasource 都能产生，后者目前只有 InfluxDB datasource 产生。

time series 格式数据长这样：

```json
[
  {
    "target": "upper_75",
    "datapoints": [
      [622, 1450754160000],
      [365, 1450754220000]
    ]
  },
  {
    "target": "upper_90",
    "datapoints": [
      [861, 1450754160000],
      [767, 1450754220000]
    ]
  }
]
```

table 格式数据长这样：

```json
[
  {
    "columns": [
      {
        "text": "Time",
        "type": "time",
        "sort": true,
        "desc": true
      },
      {
        "text": "mean"
      },
      {
        "text": "sum"
      }
    ],
    "rows": [
      [1457425380000, null, null],
      [1457425370000, 1002.76215352, 1002.76215352]
    ],
    "type": "table"
  }
]
```

为了开发插件，我们取消用 docker 运行 grafana，而是直接在本地安装运行。根据官方文档，在 macOS 上可以用 brew 安装，用 brew services 启动：

```shell
$ brew install grafana
$ brew services start|stop|restart grafana
$ brew services list
```

grafana 的配置文件放在 `/usr/local/etc/grafana/grafana.ini` 中，打开这个配置文件，可以看到 plugins 的默认存放路径是 `/usr/lib/grafana/plugins`，为了方便开发，我们把它改成我们自己的一个工作目录。

```ini
# Directory where grafana will automatically scan and look for plugins
;plugins = /var/lib/grafana/plugins
plugins = /Users/myname/my_workspace/grafana_plugins
```

然后在 `~/my_workspace/grafana_plugins` 目录下 git clone 各种 example plugins，然后在各 plugin 目录中 yarn install 和 yarn build，build 后生成的文件都会放在 dist 目录中。grafana 启动时会扫描 plugins 中的所有文件夹，如果当前目录下有 dist 目录则加载 dist 目录，否则如果此目录有 plugin.json 文件存在，则加载此目录。

> The plugin.json file is the same concept as the package.json file for an npm package. When Grafana starts it will scan the plugin folders and mount every folder that contains a plugin.json file unless the folder contains a subfolder named dist. In that case grafana will mount the dist folder instead. (https://grafana.com/docs/plugins/developing/code-styleguide/#plugin-json-mandatory)

开发阶段我们用 `yarn run watch` 命令，当修改插件代码时，只要刷新 grafana web 页面就能重新加载新的插件，而不需要重启 grafana server。

在 panel 的开发中，主要是监听 5 种事件回调：

- init-edit-mode: can be used to add tabs when editing a panel
- panel-teardown: can be used for clean up
- data-received: is an event in that is triggered on data refresh and can be hooked into
- data-snapshot-load: is an event triggered to load data when in snapshot mode.
- data-error: is used to handle errors on dashboard refresh.

data-received 中能够拿到从 data source 中获取的数据，然后进行一些加工 (?? 为什么还要加工，我觉得可以直接拿来用啊)

init-edit-mode 用来加载配置 panel 的 html 模板，可以加载多个配置模板。

panel-teardown 一般不 care，除非你用了定时器，就要在这个回调中把定时器 cancel 掉。

data-snapshot-load 使用和 data-received 一样的回调就行了，data-error 回调中将数据清空就行。

具体在哪个函数中进行绘制呢，通过查看一些示例，应该是在一个叫 link() 的函数，还需要在 link() 函数中定义一个 render() 的函数。(link 函数到底是啥作用，grafana 文档上完全没有提及...难道是 angular 自己的内容，好像确实是...整个 PanelCtrl 实际是一个自定义的 angular directive)

- [让 Directive 动起来 link()](https://hairui219.gitbooks.io/learning_angular/content/zh/chapter05_5.html)

link() 函数理解成类似 DOMLoad 或 attach，或 react 中的 onComponetDidMount 生命周期，操作 DOM 的逻辑只能在 link() 中进行，嗯，恍然大悟。

我理解 angular 中定义 directive 类似 react 中定义 component，link() 函数就相当于 onComponentDidMount()。

```js
// 示例 1
link(scope, elem, attrs, ctrl) {
  var gaugeByClass = elem.find('div#bubble-container')
  gaugeByClass.append('<div id="' + ctrl.containerDivId + '"></div>')

  var container = gaugeByClass[0].childNodes[1]
  ctrl.setContainer(container)

  function render() {
    if (!ctrl.series) {
      return
    }

    ctrl.data = ctrl.parseSeriesToJSON()
    ctrl.addBubbleChart()
  }
  this.events.on('render', () => {
    render()
    ctrl.renderingCompleted()
  })
}
```

```js
// 示例 2
link(scope, elem, attrs, ctrl) {
  //console.log("d3gauge inside link");
  var gaugeByClass = elem.find('.grafana-d3-gauge')
  //gaugeByClass.append('<center><div id="'+ctrl.containerDivId+'"></div></center>');
  gaugeByClass.append('<div id="' + ctrl.containerDivId + '"></div>')
  var container = gaugeByClass[0].childNodes[0]
  ctrl.setContainer(container)
  function render() {
    ctrl.renderGauge()
  }
  this.events.on('render', function() {
    render()
    ctrl.renderingCompleted()
  })
}
```

(看着上面两个例子好像是一个参照另一个写的)

可以重点参考的项目：

- piechart-panel - https://github.com/grafana/piechart-panel/blob/master/src/piechart_ctrl.ts
- grafana-gauge-panel - https://github.com/briangann/grafana-gauge-panel/blob/master/src/ctrl.js
- bubblechart-panel - https://github.com/digrich/bubblechart-panel/blob/master/src/bubble_ctrl.js

剩下还需要搞懂 seriesHanlder() 这个函数的作用是啥，打 log 调试。

```js
onDataReceived(dataList) {
    this.series = dataList.map(this.seriesHandler.bind(this));
    this.render();
}

seriesHandler(seriesData) {
    var series = new TimeSeries({
        datapoints: seriesData.datapoints,
        alias: seriesData.target
    });

    series.flotpairs = series.getFlotPairs(this.panel.nullPointMode);
    return series;
}
```

time series 的源码在这里：https://github.com/grafana/grafana/blob/master/public/app/core/time_series2.ts ，用来辅助计算出数组中的最大、最小和平均值，算是一个 util 类吧。

2019/11/07 在公司分享了如何写一个 grafana panel plugin，2019/11/15 总结了一篇指南放在 GitHub 上 - [my-simple-panel](https://github.com/baurine/my-simple-panel)。
