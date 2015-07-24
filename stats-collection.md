# 数据收集(Stats Collection)

Scrapy 提供了方便的收集数据的机制。数据以 key/value 方式存储，值大多是计数值。 该机制叫做数据收集器(Stats Collector)，可以通过 [Crawler API](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/api.html#topics-api-crawler) 的属性 `stats` 来使用。在下面的章节[常见数据收集器使用方法](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/stats.html#topics-stats-usecases)将给出例子来说明。

无论数据收集(stats collection)开启或者关闭，数据收集器永远都是可用的。因此您可以 import 进自己的模块并使用其 API(增加值或者设置新的状态键(stat keys))。该做法是为了简化数据收集的方法: 您不应该使用超过一行代码来收集您的 spider，Scrpay 扩展或任何您使用数据收集器代码里头的状态。

数据收集器的另一个特性是(在启用状态下)很高效，(在关闭情况下)非常高效(几乎察觉不到)。

数据收集器对每个 spider 保持一个状态表。当 spider 启动时，该表自动打开，当 spider 关闭时，自动关闭。

## 常见数据收集器使用方法

通过 stats 属性来使用数据收集器。 下面是在扩展中使用状态的例子:

```
class ExtensionThatAccessStats(object):

    def __init__(self, stats):
        self.stats = stats

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.stats)
```

设置数据:

```
stats.set_value('hostname', socket.gethostname())
```

增加数据值:

```
stats.inc_value('pages_crawled')
```

当新的值比原来的值大时设置数据:

```
stats.max_value('max_items_scraped', value)
```

当新的值比原来的值小时设置数据:

```
stats.min_value('min_free_memory_percent', value)
```

获取数据:

```
>>> stats.get_value('pages_crawled')
8
```

获取所有数据:

```
>>> stats.get_stats()
{'pages_crawled': 1238, 'start_time': datetime.datetime(2009, 7, 14, 21, 47, 28, 977139)}
```

## 可用的数据收集器

除了基本的 `StatsCollector`，Scrapy 也提供了基于 `StatsCollector` 的数据收集器。 您可以通过 `STATS_CLASS` 设置来选择。默认使用的是 `MemoryStatsCollector`。

### MemoryStatsCollector

#### class scrapy.statscol.MemoryStatsCollector

一个简单的数据收集器。其在 spider 运行完毕后将其数据保存在内存中。数据可以通过 spider_stats 属性访问。该属性是一个以 spider 名字为键(key)的字典。

这是 Scrapy 的默认选择。

##### spider_stats

保存了每个 spider 最近一次爬取的状态的字典(dict)。该字典以 spider 名字为键，值也是字典。

### DummyStatsCollector

#### class scrapy.statscol.DummyStatsCollector

该数据收集器并不做任何事情但非常高效(因为什么都不做(写文档的人真调皮 o(╯□╰)o))。 您可以通过设置 STATS_CLASS 启用这个收集器，来关闭数据收集，提高效率。 不过，数据收集的性能负担相较于 Scrapy 其他的处理(例如分析页面)来说是非常小的。
