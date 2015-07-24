# 核心 API

新版功能。

该节文档讲述 Scrapy 核心 API，目标用户是开发 Scrapy 扩展(extensions)和中间件(middlewares)的开发人员。

## Crawler API

Scrapy API 的主要入口是 `Crawler` 的实例对象，通过类方法 `from_crawler` 将它传递给扩展(extensions)。该对象提供对所有 Scrapy 核心组件的访问，也是扩展访问 Scrapy 核心组件和挂载功能到 Scrapy 的唯一途径。

Extension Manager 负责加载和跟踪已经安装的扩展，它通过 `EXTENSIONS` 配置，包含一个所有可用扩展的字典， 字典的顺序跟你在 [configure the downloader middlewares ](downloader-middleware.md)配置的顺序一致。

#### class scrapy.crawler.Crawler(spidercls, settings)

Crawler 必须使用 `scrapy.spider.Spider` 子类及 `scrapy.settings.Settings` 的对象进行实例化

##### settings

crawler 的配置管理器。

扩展(extensions)和中间件(middlewares)使用它用来访问 Scrapy 的配置。

关于 Scrapy 配置的介绍参考这里 [Settings](settings.md)。

API 参考 `Settings`。

##### signals

crawler 的信号管理器。

扩展和中间件使用它将自己的功能挂载到 Scrapy。

关于信号的介绍参考[信号(Signals)](signals.md)。

API 参考 `SignalManager`。

##### stats

crawler 的统计信息收集器。

扩展和中间件使用它记录操作的统计信息，或者访问由其他扩展收集的统计信息。

关于统计信息收集器的介绍参考[数据收集(Stats Collection)](stats-collection.md)。

API 参考类 `StatsCollector class`。

##### extensions

扩展管理器，跟踪所有开启的扩展。

大多数扩展不需要访问该属性。

关于扩展和可用扩展列表器的介绍参考[扩展(Extensions)](extensions.md)。

##### engine

执行引擎，协调 crawler 的核心逻辑，包括调度，下载和 spider。

某些扩展可能需要访问 Scrapy 的引擎属性，以修改检查(modify inspect)或修改下载器和调度器的行为， 这是该 API 的高级使用，但还不稳定。

##### spider 正在爬取的 spider。该 spider 类的实例由创建 crawler 时所提供， 在调用 :meth:\`crawl\` 方法是所创建。

##### crawl(*args, **kwargs)

根据给定的 args , kwargs 的参数来初始化 spider 类，启动执行引擎，启动 crawler。

返回一个延迟 deferred 对象，当爬取结束时触发它。

##### class scrapy.crawler.CrawlerRunner(settings)

This is a convenient helper class that creates, configures and runs crawlers inside an already setup Twisted [reactor](http://twistedmatrix.com/documents/current/core/howto/reactor-basics.html).

The CrawlerRunner object must be instantiated with a `Settings` object.

This class shouldn’t be needed (since Scrapy is responsible of using it accordingly) unless writing scripts that manually handle the crawling process. See [在脚本中运行 Scrapy](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/practices.html#run-from-script) for an example.

##### crawlers

Set of `crawlers` created by the `crawl()` method.

##### crawl_deferreds

Set of the [deferreds](http://twistedmatrix.com/documents/current/core/howto/defer.html) return by the `crawl()` method. This collection it’s useful for keeping track of current crawling state.

##### crawl(spidercls, *args, **kwargs)

This method sets up the crawling of the given spidercls with the provided arguments.

It takes care of loading the spider class while configuring and starting a crawler for it.

Returns a deferred that is fired when the crawl is finished.

**参数:**    

- **spidercls** (`Spider` subclass or str) – spider class or spider’s name inside the project
- **args** (*[list](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/api.html#scrapy.spidermanager.SpiderManager.list)*) – arguments to initializate the spider
- **kwargs** (dict) – keyword arguments to initializate the spider

##### stop()

Stops simultaneously all the crawling jobs taking place.

Returns a deferred that is fired when they all have ended.

## 设置(Settings) API

#### scrapy.settings.SETTINGS_PRIORITIES

Dictionary that sets the key name and priority level of the default settings priorities used in Scrapy.

Each item defines a settings entry point, giving it a code name for identification and an integer priority. Greater priorities take more precedence over lesser ones when setting and retrieving values in the `Settings` class.

```
SETTINGS_PRIORITIES = {
    'default': 0,
    'command': 10,
    'project': 20,
    'spider': 30,
    'cmdline': 40,
}
```

For a detailed explanation on each settings sources, see: [Settings](settings.md).

class scrapy.settings.Settings(values={}, priority='project')
This object stores Scrapy settings for the configuration of internal components, and can be used for any further customization.

After instantiation of this class, the new object will have the global default settings described on [内置设定参考手册](settings.md). already populated.

Additional values can be passed on initialization with the `values` argument, and they would take the `priority` level. If the latter argument is a string, the priority name will be looked up in `SETTINGS_PRIORITIES`. Otherwise, a expecific integer should be provided.

Once the object is created, new settings can be loaded or updated with the set() method, and can be accessed with the square bracket notation of dictionaries, or with the get() method of the instance and its value conversion variants. When requesting a stored key, the value with the highest priority will be retrieved.

##### set(name, value, priority='project')

Store a key/value attribute with a given priority. Settings should be populated before configuring the Crawler object (through the configure() method), otherwise they won’t have any effect.

**参数:**    

- **name** (string) – the setting name
- **value** (any) – the value to associate with the setting
- **priority** (string or int) – the priority of the setting. Should be a key of    	`SETTINGS_PRIORITIES` or an integer


##### setdict(values, priority='project')

Store key/value pairs with a given priority.

This is a helper function that calls `set()` for every item of `values` with the provided `priority`.

**参数:**    

- **values** (dict) – the settings names and values
- **priority** (string or int) – the priority of the settings. Should be a key of 	`SETTINGS_PRIORITIES` or an integer

##### setmodule(module, priority='project')

Store settings from a module with a given priority.

This is a helper function that calls `set()` for every globally declared uppercase variable of `module` with the provided `priority`.

**参数:**    

- **module** (module object or string) – the module or the path of the module
- **priority** (string or int) – the priority of the settings. Should be a key of 	`SETTINGS_PRIORITIES` or an integer

##### get(name, default=None)

获取某项配置的值，且不修改其原有的值。

**参数:**    

- **name** (*字符串*) – 配置名
- **default** (*任何*) – 如果没有该项配置时返回的缺省值

##### getbool(name, default=False)

return `False` 将某项配置的值以布尔值形式返回。比如，`1` 和`'1'`，`True` 都返回``True``， 而 `0`，`'0'`，`False` 和 `None` 返回 `False`。

比如，通过环境变量计算将某项配置设置为 `'0'`，通过该方法获取得到 `False`。

**参数:**    

- **name** (*字符串*) – 配置名
-** default** (*任何*) – 如果该配置项未设置，返回的缺省值

##### getint(name, default=0)

将某项配置的值以整数形式返回

**参数:**    

- **name** (*字符串*) – 配置名
-** default** (*任何*) – 如果该配置项未设置，返回的缺省值

##### getfloat(name, default=0.0)

将某项配置的值以浮点数形式返回

**参数:**    

- **name** (*字符串*) – 配置名
-** default** (*任何*) – 如果该配置项未设置，返回的缺省值

##### getlist(name, default=None)

将某项配置的值以列表形式返回。如果配置值本来就是 list 则将返回其拷贝。如果是字符串，则返回被 ”,” 分割后的列表。

比如，某项值通过环境变量的计算被设置为`'one,two'`，该方法返回`[‘one’, ‘two’]`。

**参数:** 
   
- **name** (*字符串*) – 配置名
-** default** (*任何*) – 如果该配置项未设置，返回的缺省值

##### getdict(name, default=None)

Get a setting value as a dictionary. If the setting original type is a dictionary, a copy of it will be returned. If it’s a string it will evaluated as a json dictionary.

**参数:**  
  
- **name** (*字符串*) – 配置名
-** default** (*任何*) – 如果该配置项未设置，返回的缺省值

##### copy()

Make a deep copy of current settings.

This method returns a new instance of the `Settings` class, populated with the same values and their priorities.

Modifications to the new object won’t be reflected on the original settings.

##### freeze()

Disable further changes to the current settings.

After calling this method, the present state of the settings will become immutable. Trying to change values through the `set()` method and its variants won’t be possible and will be alerted.

##### frozencopy()

Return an immutable copy of the current settings.

Alias for a `freeze()` call in the object returned by `copy()`

## SpiderManager API

#### class scrapy.spidermanager.SpiderManager

This class is in charge of retrieving and handling the spider classes defined across the project.

Custom spider managers can be employed by specifying their path in the `SPIDER_MANAGER_CLASS` project setting. They must fully implement the `scrapy.interfaces.ISpiderManager` interface to guarantee an errorless execution.

##### from_settings(settings)

This class method is used by Scrapy to create an instance of the class. It’s called with the current project settings, and it loads the spiders found in the modules of the SPIDER_MODULES setting.

**参数:** 	

settings (`Settings` instance) – project settings

##### load(spider_name)

Get the Spider class with the given name. It’ll look into the previously loaded spiders for a spider class with name spider_name and will raise a KeyError if not found.

**参数:**    

spider_name (*str*) – spider class name

**list()**

Get the names of the available spiders in the project.

##### find_by_request(request)

List the spiders’ names that can handle the given request. Will try to match the request’s url against the domains of the spiders.

**参数:** 
   
request (`Request` instance) – queried request

## 信号(Signals) API

#### class scrapy.signalmanager.SignalManager

##### connect(receiver, signal)

链接一个接收器函数(receiver function) 到一个信号(signal)。

signal 可以是任何对象，虽然 Scrapy 提供了一些预先定义好的信号， 参考文档[信号(Signals)](signals.md)。

**参数:**    

- **receiver** (*可调用对象*) – 被链接到的函数
- **signal** (*对象*) – 链接的信号

##### send_catch_log(signal, **kwargs)

发送一个信号，捕获异常并记录日志。

关键字参数会传递给信号处理者(signal handlers)(通过方法 `connect()`关联)。

##### send_catch_log_deferred(signal, **kwargs)

跟 `send_catch_log()`相似但支持返回 [deferreds](http://twistedmatrix.com/documents/current/core/howto/defer.html) 形式的信号处理器。

返回一个 [deferred](http://twistedmatrix.com/documents/current/core/howto/defer.html)，当所有的信号处理器的延迟被触发时调用。发送一个信号，处理异常并记录日志。

关键字参数会传递给信号处理者(signal handlers)(通过方法 `connect()`关联)。

##### disconnect(receiver, signal)

解除一个接收器函数和一个信号的关联。这跟方法 `connect()`有相反的作用，参数也相同。

##### disconnect_all(signal)

取消给定信号绑定的所有接收器。

**参数:**    

- **signal** (*object*) – 要取消绑定的信号

## 状态收集器(Stats Collector) API

模块 scrapy.statscol 下有好几种状态收集器， 它们都实现了状态收集器 API 对应的类 `Statscollector` (即它们都继承至该类)。

#### class scrapy.statscol.StatsCollector

##### get_value(key, default=None)

返回指定 key 的统计值，如果 key 不存在则返回缺省值。

##### get_stats()

以 dict 形式返回当前 spider 的所有统计值。

##### set_value(key, value)

设置 key 所指定的统计值为 value。

##### set_stats(stats)

使用 dict 形式的 stats 参数覆盖当前的统计值。

##### inc_value(key, count=1, start=0)

增加 key 所对应的统计值，增长值由 count 指定。如果 key 未设置，则使用 start 的值设置为初始值。

##### max_value(key, value)

如果 key 所对应的当前 value 小于参数所指定的 value，则设置 value。如果没有 key 所对应的 value，设置 value。

##### min_value(key, value)

如果 key 所对应的当前 value 大于参数所指定的 value，则设置 value。如果没有 key 所对应的 value，设置 value。

##### clear_stats()

清除所有统计信息。

以下方法不是统计收集 api 的一部分，但实现自定义的统计收集器时会使用到：

##### open_spider(spider)

打开指定 spider 进行统计信息收集。

##### close_spider(spider)

关闭指定 spider。调用后，不能访问和收集统计信息。
