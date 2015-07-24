# 常见问题(FAQ)

## Scrapy 相 BeautifulSoup 或 lxml 比较，如何呢？

[BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/) 及 [lxml](http://lxml.de/) 是 HTML 和 XML 的分析库。Scrapy 则是 编写爬虫，爬取网页并获取数据的应用框架(application framework)。

Scrapy 提供了内置的机制来提取数据(叫做 选择器(selectors))。 但如果您觉得使用更为方便，也可以使用 [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/)(或 [lxml](http://lxml.de/))。 总之，它们仅仅是分析库，可以在任何 Python 代码中被导入及使用。

换句话说，拿 Scrapy 与 BeautifulSoup (或 lxml) 比较就好像是拿 [jinja2](http://jinja.pocoo.org/2/) 与 [Django](http://www.djangoproject.com/) 相比。

## Scrapy 支持那些 Python 版本？

Scrapy 仅仅支持 Python 2.7。Python2.6 的支持从 Scrapy 0.20 开始被废弃了。

## Scrapy 支持 Python 3 么？

不。但是 Python 3.3+的支持已经在计划中了。现在，Scrapy 支持 Python 2.7。

> 参见
> 
> Scrapy 支持那些 Python 版本？。

## Scrapy 是否从 Django 中”剽窃”了 X 呢？

也许吧，不过我们不喜欢这个词。我们认为 [Django](http://www.djangoproject.com/) 是一个很好的开源项目，同时也是 一个很好的参考对象，所以我们把其作为 Scrapy 的启发对象。

我们坚信，如果有些事情已经做得很好了，那就没必要再重复制造轮子。这个想法，作为 开源项目及免费软件的基石之一，不仅仅针对软件，也包括文档，过程，政策等等。所以，与其自行解决每个问题，我们选择从其他已经很好地解决问题的项目中复制想法(copy idea) ，并把注意力放在真正需要解决的问题上。

如果 Scrapy 能启发其他的项目，我们将为此而自豪。欢迎来抄(steal)我们！

## Scrapy 支持 HTTP 代理么？

是的。(从 Scrapy 0.8 开始)通过 HTTP 代理下载中间件对 HTTP 代理提供了支持。参考 `HttpProxyMiddleware`。

## 如何爬取属性在不同页面的 item 呢？

参考 [Passing additional data to callback functions](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/request-response.html#topics-request-response-ref-request-callback-arguments)。

## Scrapy 退出，ImportError: Nomodule named win32api

[这是个 Twisted bug](http://twistedmatrix.com/trac/ticket/3707)，您需要安装 [pywin32](http://sourceforge.net/projects/pywin32/)。

## 我要如何在 spider 里模拟用户登录呢?

参考 [使用 FormRequest.from_response()方法模拟用户登录](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/request-response.html#topics-request-response-ref-request-userlogin)。

## Scrapy 是以广度优先还是深度优先进行爬取的呢？

默认情况下，Scrapy 使用 LIFO 队列来存储等待的请求。简单的说，就是[深度优先顺序](http://en.wikipedia.org/wiki/Depth-first_search)。深度优先对大多数情况下是更方便的。如果您想以 广度优先顺序 进行爬取，你可以设置以下的设定:

```
DEPTH_PRIORITY = 1
SCHEDULER_DISK_QUEUE = 'scrapy.squeue.PickleFifoDiskQueue'
SCHEDULER_MEMORY_QUEUE = 'scrapy.squeue.FifoMemoryQueue'
```

## 我的 Scrapy 爬虫有内存泄露，怎么办?

参考`调试内存溢出`。

另外，Python 自己也有内存泄露，在 `Leaks without leaks` 有所描述。

## 如何让 Scrapy 减少内存消耗?

参考上一个问题。

## 我能在 spider 中使用基本 HTTP 认证么？

可以。参考 `HttpAuthMiddleware`.

## 为什么 Scrapy 下载了英文的页面，而不是我的本国语言？

尝试通过覆盖 `DEFAULT_REQUEST_HEADERS` 设置来修改默认的 [Accept-Language](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) 请求头。

## 我能在哪里找到 Scrapy 项目的例子？

参考`例子`。

## 我能在不创建 Scrapy 项目的情况下运行一个爬虫(spider)么？

是的。您可以使用 `runspider` 命令。例如，如果您有个 spider 写在 `my_spider.py` 文件中，您可以运行:

```
scrapy runspider my_spider.py
```

详情请参考 `runspider` 命令。

## 我收到了 “Filtered offsite request” 消息。如何修复？

这些消息(以 DEBUG 所记录)并不意味着有问题，所以你可以不修复它们。

这些消息由 Offsite Spider 中间件(Middleware)所抛出。 该(默认启用的)中间件筛选出了不属于当前 spider 的站点请求。

更多详情请参见:`OffsiteMiddleware`。

## 发布 Scrapy 爬虫到生产环境的推荐方式？

参见 `Scrapyd`。

## 我能对大数据(large exports)使用 JSON 么？

这取决于您的输出有多大。参考 `JsonItemExporter` 文档中的`这个警告`。

我能在信号处理器(signal handler)中返回(Twisted)引用么？
有些信号支持从处理器中返回引用，有些不行。参考`内置信号参考手册(Built-in signals reference)`来了解详情。

## reponse 返回的状态值 999 代表了什么?

999 是雅虎用来控制请求量所定义的返回值。 试着减慢爬取速度，将 spider 的下载延迟改为 2 或更高:

```
class MySpider(CrawlSpider):

    name = 'myspider'

    download_delay = 2

    # [ ... rest of the spider code ... ]
```

或在 `DOWNLOAD_DELAY` 中设置项目的全局下载延迟。

## 我能在 spider 中调用 `pdb.set_trace()` 来调试么？

可以，但你也可以使用 Scrapy 终端。这能让你快速分析(甚至修改) spider 处理返回的返回(response)。通常来说，比老旧的 `pdb.set_trace()`有用多了。

更多详情请参考 在 `spider 中启动 shell 来查看 response`。

## 将所有爬取到的 item 转存(dump)到 JSON/CSV/XML 文件的最简单的方法?

dump 到 JSON 文件:

```
scrapy crawl myspider -o items.json
```

dump 到 CSV 文件:

```
scrapy crawl myspider -o items.csv
```

dump 到 XML 文件:

```
scrapy crawl myspider -o items.xml
```

更多详情请参考 `Feed exports`

## 在某些表单中巨大神秘的`__VIEWSTATE` 参数是什么？

__VIEWSTATE 参数存在于 ASP.NET/VB.NET 建立的站点中。关于这个参数的作用请参考[这篇文章](http://search.cpan.org/~ecarroll/HTML-TreeBuilderX-ASP_NET-0.09/lib/HTML/TreeBuilderX/ASP_NET.pm)。这里有一个爬取这种站点的[样例爬虫](http://github.com/AmbientLighter/rpn-fas/blob/master/fas/spiders/rnp.py)。

## 分析大 XML/CSV 数据源的最好方法是?

使用 XPath 选择器来分析大数据源可能会有问题。选择器需要在内存中对数据建立完整的 DOM 树，这过程速度很慢且消耗大量内存。

为了避免一次性读取整个数据源，您可以使用 `scrapy.utils.iterators` 中的 `xmliter` 及 `csviter` 方法。 实际上，这也是 feed spider(参考 `Spiders`)中的处理方法。

## Scrapy 自动管理 cookies 么？

是的，Scrapy 接收并保持服务器返回来的 cookies，在之后的请求会发送回去，就像正常的网页浏览器做的那样。

更多详情请参考 `Requests and Responses` 及 `CookiesMiddleware`。

## 如何才能看到 Scrapy 发出及接收到的 Cookies 呢？

启用 `COOKIES_DEBUG` 选项。

## 要怎么停止爬虫呢?

在回调函数中 raise `CloseSpider` 异常。 更多详情请参见:`CloseSpider`。

## 如何避免我的 Scrapy 机器人(bot)被禁止(ban)呢？

参考`避免被禁止(ban)`。

## 我应该使用 spider 参数(arguments)还是设置(settings)来配置 spider 呢？

`spider 参数`及`设置(settings)`都可以用来配置您的 spider。没有什么强制的规则来限定要使用哪个，但设置(settings)更适合那些一旦设置就不怎么会修改的参数，而 spider 参数则意味着修改更为频繁，在每次 spider 运行都有修改，甚至是 spider 运行所必须的元素 (例如，设置 spider 的起始 url)。

这里以例子来说明这个问题。假设您有一个 spider 需要登录某个网站来 爬取数据，并且仅仅想爬取特定网站的特定部分(每次都不一定相同)。 在这个情况下，认证的信息将写在设置中，而爬取的特定部分的 url 将是 spider 参数。

## 我爬取了一个 XML 文档但是 XPath 选择器不返回任何的 item

也许您需要移除命名空间(namespace)。参见`移除命名空间`。
