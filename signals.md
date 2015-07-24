# 信号(Signals)

Scrapy 使用信号来通知事情发生。您可以在您的 Scrapy 项目中捕捉一些信号(使用 [extension](extensions.md))来完成额外的工作或添加额外的功能，扩展 Scrapy。

虽然信号提供了一些参数，不过处理函数不用接收所有的参数 - 信号分发机制(singal dispatching mechanism)仅仅提供处理器(handler)接受的参数。

您可以通过[信号(Signals) API](api.md) 来连接(或发送您自己的)信号。

## 延迟的信号处理器(Deferred signal handlers)

有些信号支持从处理器返回 [Twisted deferreds](http://twistedmatrix.com/documents/current/core/howto/defer.html)，参考下边的`内置信号参考手册(Built-in signals reference)`来了解哪些支持。

## 内置信号参考手册(Built-in signals reference)

以下给出 Scrapy 内置信号的列表及其意义。

### engine_started

#### scrapy.signals.engine_started()

当 Scrapy 引擎启动爬取时发送该信号。

该信号支持返回 deferreds。

> 注解
> 
> 该信号可能会在信号 `spider_opened` 之后被发送，取决于 spider 的启动方式。 所以不要 依赖 该信号会比 `spider-opened` 更早被发送。

### engine_stopped

#### scrapy.signals.engine_stopped()

当 Scrapy 引擎停止时发送该信号(例如，爬取结束)。

该信号支持返回 deferreds。

### item_scraped

#### scrapy.signals.item_scraped(item, response, spider)

当 item 被爬取，并通过所有 Item Pipeline 后(没有被丢弃(dropped)，发送该信号。

该信号支持返回 deferreds。

**参数:** 
   
- **item** (`Item` 对象) – 爬取到的 item
- **spider** (`Spider` 对象) – 爬取 item 的 spider
- **response** (`Response` 对象) – 提取 item 的 response

### item_dropped

#### scrapy.signals.item_dropped(item, exception, spider)

当 item 通过 [Item Pipeline](item-pipeline.md)，有些 pipeline 抛出 DropItem 异常，丢弃 item 时，该信号被发送。

该信号支持返回 deferreds。

**参数:** 
   
- **item** (`Item` 对象) – 爬取到的 item
- **spider** (`Spider` 对象) – 爬取 item 的 spider
- **exception** (`DropItem` 异常) – 导致 item 被丢弃的异常(必须是 DropItem 的子类)

### spider_closed

#### scrapy.signals.spider_closed(spider, reason)

当某个 spider 被关闭时，该信号被发送。该信号可以用来释放每个 spider 在 `spider_opened` 时占用的资源。

该信号支持返回 deferreds。

**参数:**    

- spider (`Spider` 对象) – 关闭的 spider
- reason (*str*) – 描述 spider 被关闭的原因的字符串。如果 spider 是由于完成爬取而被关闭，则其为`'finished'`。否则，如果 spider 是被引擎的 `close_spider` 方法所关闭，则其为调用该方法时传入的 `reason` 参数(默认为`'cancelled'`)。如果引擎被关闭(例如， 输入 Ctrl-C)，则其为`'shutdown'`。

### spider_opened

#### scrapy.signals.spider_opened(spider)

当 spider 开始爬取时发送该信号。该信号一般用来分配 spider 的资源，不过其也能做任何事。

该信号支持返回 deferreds。

**参数:**    

- **spider** (`Spider 对象`) – 开启的 spider

### spider_idle

#### scrapy.signals.spider_idle(spider)

当 spider 进入空闲(idle)状态时该信号被发送。空闲意味着:

- requests 正在等待被下载
- requests 被调度
- items 正在 item pipeline 中被处理

当该信号的所有处理器(handler)被调用后，如果 spider 仍然保持空闲状态，引擎将会关闭该 spider。当 spider 被关闭后，`spider_close`d 信号将被发送。

您可以，比如，在 `spider_idle` 处理器中调度某些请求来避免 spider 被关闭。

该信号**不支持**返回 deferreds。

**参数:**    

- **spider** (`Spider` 对象) – 空闲的 spider

### spider_error

#### scrapy.signals.spider_error(failure, response, spider)

当 spider 的回调函数产生错误时(例如，抛出异常)，该信号被发送。

**参数:**    

- **failure** ([Failure](http://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html) 对象) – 以 [Twisted Failure](http://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html) 对象抛出的异常
- **response** (`Response 对象`) – 当异常被抛出时被处理的 response
- **spider** (`Spider 对象`) – 抛出异常的 spider

### request_scheduled

#### scrapy.signals.request_scheduled(request, spider)

当引擎调度一个 Request 对象用于下载时，该信号被发送。

该信号 不支持 返回 deferreds。

**参数:**
    
- **request** (`Request` 对象) – 到达调度器的 request
- **spider** (`Spider` 对象) – 产生该 request 的 spider

### request_dropped

#### scrapy.signals.request_dropped(request, spider)

Sent when a `Request`, scheduled by the engine to be downloaded later, is rejected by the scheduler.

The signal does not support returning deferreds from their handlers.

**参数:**    

- **request** (`Request` object) – the request that reached the scheduler
- **spider** (`Spider` object) – the spider that yielded the request

### response_received

#### scrapy.signals.response_received(response, request, spider)

当引擎从 downloader 获取到一个新的 Response 时发送该信号。

该信号 不支持 返回 deferreds。

**参数:**  
  
- **response** (`Response` 对象) – 接收到的 response
- **request** (`Request` 对象) – 生成 response 的 request
- **spider** (`Spider 对象`) – response 所对应的 spider

### response_downloaded

#### scrapy.signals.response_downloaded(response, request, spider)

当一个 HTTPResponse 被下载时，由 downloader 发送该信号。

该信号 不支持 返回 deferreds。

**参数:**    

- **response** (`Response` 对象) – \下载的 response
- **request** (`Request` 对象) – 生成 response 的 request
- **spider** (`Spider 对象`) – response 所对应的 spider
