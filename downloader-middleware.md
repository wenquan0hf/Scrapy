# 下载器中间件(Downloader Middleware)

下载器中间件是介于 Scrapy 的 request/response 处理的钩子框架。 是用于全局修改 Scrapy request 和 response 的一个轻量、底层的系统。

## 激活下载器中间件     

要激活下载器中间件组件，将其加入到 `DOWNLOADER_MIDDLEWARES` 设置中。该设置是一个字典(dict)，键为中间件类的路径，值为其中间件的顺序(order)。

这里是一个例子:

```
DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.CustomDownloaderMiddleware': 543,
}
```

`DOWNLOADER_MIDDLEWARES` 设置会与 Scrapy 定义的 `DOWNLOADER_MIDDLEWARES_BASE` 设置合并(但不是覆盖)，而后根据顺序(order)进行排序，最后得到启用中间件的有序列表: 第一个中间件是最靠近引擎的，最后一个中间件是最靠近下载器的。

关于如何分配中间件的顺序请查看 `DOWNLOADER_MIDDLEWARES_BASE` 设置，而后根据您想要放置中间件的位置选择一个值。由于每个中间件执行不同的动作，您的中间件可能会依赖于之前(或者之后)执行的中间件，因此顺序是很重要的。

如果您想禁止内置的(在 `DOWNLOADER_MIDDLEWARES_BASE` 中设置并默认启用的)中间件，您必须在项目的 DOWNLOADER_MIDDLEWARES 设置中定义该中间件，并将其值赋为 *None*。例如，如果您想要关闭 user-agent 中间件:

```
DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.CustomDownloaderMiddleware': 543,
    'scrapy.contrib.downloadermiddleware.useragent.UserAgentMiddleware': None,
}
```

最后，请注意，有些中间件需要通过特定的设置来启用。更多内容请查看相关中间件文档。

## 编写您自己的下载器中间件

编写下载器中间件十分简单。每个中间件组件是一个定义了以下一个或多个方法的 Python 类:

#### class scrapy.contrib.downloadermiddleware.DownloaderMiddleware

##### process_request(request, spider)

当每个 request 通过下载中间件时，该方法被调用。

process_request() 必须返回其中之一: 返回 `None` 、返回一个 Response 对象、返回一个 Request 对象或 raise IgnoreRequest。

如果其返回 `None`，Scrapy 将继续处理该 request，执行其他的中间件的相应方法，直到合适的下载器处理函数(download handler)被调用，该 request 被执行(其 response 被下载)。

如果其返回 Response 对象，Scrapy 将不会调用 任何 其他的 process_request()或 process_exception()方法，或相应地下载函数； 其将返回该 response。已安装的中间件的 process_response()方法则会在每个 response 返回时被调用。

如果其返回 Request 对象，Scrapy 则停止调用 process_request 方法并重新调度返回的 request。当新返回的 request 被执行后， 相应地中间件链将会根据下载的 response 被调用。

如果其 raise 一个 IgnoreRequest 异常，则安装的下载中间件的 process_exception() 方法会被调用。如果没有任何一个方法处理该异常， 则 request 的 errback(`Request.errback`)方法会被调用。如果没有代码处理抛出的异常， 则该异常被忽略且不记录(不同于其他异常那样)。

**参数:**	

- request (`Request` 对象) – 处理的 request
- spider (`Spider` 对象) – 该 request 对应的 spider

#### process_response(request, response, spider)

process_request() 必须返回以下之一: 返回一个 Response 对象、返回一个 Request 对象或 raise 一个 IgnoreRequest 异常。

如果其返回一个 Response (可以与传入的 response 相同，也可以是全新的对象) 该 response 会被在链中的其他中间件的 process_response()方法处理。

如果其返回一个 Request 对象，则中间件链停止，返回的 request 会被重新调度下载。处理类似于 process_request()返回 request 所做的那样。

如果其抛出一个 IgnoreRequest 异常，则调用 request 的 errback(`Request.errback`)。如果没有代码处理抛出的异常，则该异常被忽略且不记录(不同于其他异常那样)。

**参数:**	

- **request** (`Request` 对象) – response 所对应的 request
- **response** (`Response` 对象) – 被处理的 response
- **spider** (`Spider` 对象) – response 所对应的 spider

#### process_exception(request, exception, spider)

当下载处理器(download handler)或 process_request()(下载中间件)抛出异常(包括 IgnoreRequest 异常)时，Scrapy 调用 process_exception() 。

process_exception()应该返回以下之一: 返回 `None`、一个 Response 对象、或者一个 Request 对象。

如果其返回 `None`，Scrapy 将会继续处理该异常，接着调用已安装的其他中间件的 process_exception()方法，直到所有中间件都被调用完毕，则调用默认的异常处理。

如果其返回一个 Response 对象，则已安装的中间件链的 process_response()方法被调用。Scrapy 将不会调用任何其他中间件的 process_exception() 方法。

如果其返回一个 Request 对象， 则返回的 request 将会被重新调用下载。这将停止中间件的 process_exception()方法执行，就如返回一个 response 的那样。

**参数:**
	
- **request **(是 `Request` 对象) – 产生异常的 request
- **exception **(`Exception` 对象) – 抛出的异常
- **spider **(`Spider` 对象) – request 对应的 spider

## 内置下载中间件参考手册

本页面介绍了 Scrapy 自带的所有下载中间件。关于如何使用及编写您自己的中间件，请参考 [downloader middleware usage guide](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/downloader-middleware.html#topics-downloader-middleware)。

关于默认启用的中间件列表(及其顺序)请参考 `DOWNLOADER_MIDDLEWARES_BASE` 设置。

### CookiesMiddleware

#### class scrapy.contrib.downloadermiddleware.cookies.CookiesMiddleware

该中间件使得爬取需要 cookie(例如使用 session)的网站成为了可能。其追踪了 web server 发送的 cookie，并在之后的 request 中发送回去，就如浏览器所做的那样。

以下设置可以用来配置 cookie 中间件:

- COOKIES_ENABLED
- COOKIES_DEBUG

### 单 spider 多 cookie session

Scrapy 通过使用 `cookiejar` Request meta key 来支持单 spider 追踪多 cookie session。 默认情况下其使用一个 cookie jar(session)，不过您可以传递一个标示符来使用多个。

例如:

```
for i, url in enumerate(urls):
    yield scrapy.Request("http://www.example.com", meta={'cookiejar': i},
        callback=self.parse_page)
```

需要注意的是 cookiejar meta key 不是”黏性的(sticky)”。您需要在之后的 request 请求中接着传递。例如:

```
def parse_page(self, response):
    # do some processing
    return scrapy.Request("http://www.example.com/otherpage",
        meta={'cookiejar': response.meta['cookiejar']},
        callback=self.parse_other_page)
```

### COOKIES_ENABLED

默认: `True`

是否启用 cookies middleware。如果关闭，cookies 将不会发送给 web server。

### COOKIES_DEBUG

默认: `False`

如果启用，Scrapy 将记录所有在 request(Cookie 请求头)发送的 cookies 及 response 接收到的 cookies(Set-Cookie 接收头)。

下边是启用 `COOKIES_DEBUG` 的记录的样例:

```
2011-04-06 14:35:10-0300 [diningcity] INFO: Spider opened
2011-04-06 14:35:10-0300 [diningcity] DEBUG: Sending cookies to: <GET http://www.diningcity.com/netherlands/index.html>
        Cookie: clientlanguage_nl=en_EN
2011-04-06 14:35:14-0300 [diningcity] DEBUG: Received cookies from: <200 http://www.diningcity.com/netherlands/index.html>
        Set-Cookie: JSESSIONID=B~FA4DC0C496C8762AE4F1A620EAB34F38; Path=/
        Set-Cookie: ip_isocode=US
        Set-Cookie: clientlanguage_nl=en_EN; Expires=Thu, 07-Apr-2011 21:21:34 GMT; Path=/
2011-04-06 14:49:50-0300 [diningcity] DEBUG: Crawled (200) <GET http://www.diningcity.com/netherlands/index.html> (referer: None)
[...]
```

### DefaultHeadersMiddleware

##### class scrapy.contrib.downloadermiddleware.defaultheaders.DefaultHeadersMiddleware

该中间件设置 `DEFAULT_REQUEST_HEADERS` 指定的默认 request header。

### DownloadTimeoutMiddleware

#### class scrapy.contrib.downloadermiddleware.downloadtimeout.DownloadTimeoutMiddleware

该中间件设置 `DOWNLOAD_TIMEOUT` 或 spider 的 `download_timeout` 属性指定的 request 下载超时时间。

> 注解
> 
> 您也可以使用 `download_timeout` Request.meta key 来对每个请求设置下载超时时间。这种方式在 DownloadTimeoutMiddleware 被关闭时仍然有效。

### HttpAuthMiddleware

#### class scrapy.contrib.downloadermiddleware.httpauth.HttpAuthMiddleware

该中间件完成某些使用 [Basic access authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) (或者叫 HTTP 认证)的 spider 生成的请求的认证过程。

在 spider 中启用 HTTP 认证，请设置 spider 的 `http_user` 及 `http_pass` 属性。

样例:

```
from scrapy.contrib.spiders import CrawlSpider

class SomeIntranetSiteSpider(CrawlSpider):

    http_user = 'someuser'
    http_pass = 'somepass'
    name = 'intranet.example.com'

    # .. rest of the spider code omitted ...
```

### HttpCacheMiddleware

#### class scrapy.contrib.downloadermiddleware.httpcache.HttpCacheMiddleware

该中间件为所有 HTTP request 及 response 提供了底层(low-level)缓存支持。其由 cache 存储后端及 cache 策略组成。

Scrapy 提供了两种 HTTP 缓存存储后端:

- Filesystem storage backend (默认值)
- DBM storage backend

您可以使用 `HTTPCACHE_STORAGE` 设定来修改 HTTP 缓存存储后端。您也可以实现您自己的存储后端。

Scrapy 提供了两种了缓存策略:

- RFC2616 策略
- Dummy 策略(默认值)

您可以使用 `HTTPCACHE_POLICY` 设定来修改 HTTP 缓存存储后端。您也可以实现您自己的存储策略。

### Dummy 策略(默认值)

该策略不考虑任何 HTTP Cache-Control 指令。每个 request 及其对应的 response 都被缓存。 当相同的 request 发生时，其不发送任何数据，直接返回 response。

Dummpy 策略对于测试 spider 十分有用。其能使 spider 运行更快(不需要每次等待下载完成)， 同时在没有网络连接时也能测试。其目的是为了能够回放 spider 的运行过程，*使之与之前的运行过程一模一样*。

使用这个策略请设置:

- `HTTPCACHE_POLICY` 为 `scrapy.contrib.httpcache.DummyPolicy`

### RFC2616 策略

该策略提供了符合 RFC2616 的 HTTP 缓存，例如符合 HTTP Cache-Control，针对生产环境并且应用在持续性运行环境所设置。该策略能避免下载未修改的数据(来节省带宽，提高爬取速度)。

实现了:

- 当 no-store cache-control 指令设置时不存储 response/request。
- 当 no-cache cache-control 指定设置时不从 cache 中提取 response，即使 response 为最新。
- 根据 max-age cache-control 指令中计算保存时间(freshness lifetime)。
- 根据 Expires 指令来计算保存时间(freshness lifetime)。
- 根据 response 包头的 *Last-Modified* 指令来计算保存时间(freshness lifetime)。(Firefox 使用的启发式算法)。
- 根据 response 包头的 *Age* 计算当前年龄(current age)。
- 根据 *Date* 计算当前年龄(current age)。
- 根据 response 包头的 *Last-Modified* 验证老旧的 response。
- 根据 response 包头的 *ETag* 验证老旧的 response。
- 为接收到的 response 设置缺失的 *Date* 字段。


目前仍然缺失:

- Pragma: no-cache 支持 [http://www.mnot.net/cache_docs/#PRAGMA](http://www.mnot.net/cache_docs/#PRAGMA)
- Vary 字段支持 [http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6)
- 当 update 或 delete 之后失效相应的 response [http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10)
- ... 以及其他可能缺失的特性 ..

使用这个策略，设置:

- `HTTPCACHE_POLICY` 为 `scrapy.contrib.httpcache.RFC2616Policy`

### Filesystem storage backend (默认值)

文件系统存储后端可以用于 HTTP 缓存中间件。

使用该存储端，设置:

- `HTTPCACHE_STORAGE` 为 `scrapy.contrib.httpcache.FilesystemCacheStorage`

每个 request/response 组存储在不同的目录中，包含下列文件:

- `request_body` - the plain request body
- `request_headers` - the request headers (原始 HTTP 格式)
- `response_body` - the plain response body
- `response_headers` - the request headers (原始 HTTP 格式)
- `meta` - 以 Python `repr()`格式(grep-friendly 格式)存储的该缓存资源的一些元数据。
- `pickled_meta` - 与 `meta` 相同的元数据，不过使用 pickle 来获得更高效的反序列化性能。

目录的名称与 request 的指纹(参考 `scrapy.utils.request.fingerprint`)有关，而二级目录是为了避免在同一文件夹下有太多文件 (这在很多文件系统中是十分低效的)。目录的例子:

```
/path/to/cache/dir/example.com/72/72811f648e718090f041317756c03adb0ada46c7
```

### DBM storage backend


同时也有 [DBM](http://en.wikipedia.org/wiki/Dbm) 存储后端可以用于 HTTP 缓存中间件。

默认情况下，其采用 [anydbm](http://docs.python.org/library/anydbm.html) 模块，不过您也可以通过 `HTTPCACHE_DBM_MODULE` 设置进行修改。

使用该存储端，设置:

`HTTPCACHE_STORAGE 为 scrapy.contrib.httpcache.DbmCacheStorage`

### LevelDB storage backend

A [LevelDB](http://code.google.com/p/leveldb/) storage backend is also available for the HTTP cache middleware.

This backend is not recommended for development because only one process can access LevelDB databases at the same time, so you can’t run a crawl and open the scrapy shell in parallel for the same spider.

In order to use this storage backend:

- set `HTTPCACHE_STORAGE` to `scrapy.contrib.httpcache.LeveldbCacheStorage`
- install `LevelDB python bindings` like `pip install leveldb`

### HTTPCache 中间件设置

`HttpCacheMiddleware` 可以通过以下设置进行配置:

### HTTPCACHE_ENABLED

新版功能。

默认: `False`

HTTP 缓存是否开启。

在 0.11 版更改: 在 0.11 版本前，是使用 `HTTPCACHE_DIR` 来开启缓存。

### HTTPCACHE_EXPIRATION_SECS

默认: `0`

缓存的 request 的超时时间，单位秒。

超过这个时间的缓存 request 将会被重新下载。如果为 0，则缓存的 request 将永远不会超时。

在 0.11 版更改: 在 0.11 版本前，0 的意义是缓存的 request 永远超时。

### HTTPCACHE_DIR

默认: `'httpcache'`

存储(底层的)HTTP 缓存的目录。如果为空，则 HTTP 缓存将会被关闭。 如果为相对目录，则相对于项目数据目录(project data dir)。更多内容请参考[默认的 Scrapy 项目结构](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/commands.html#topics-project-structure)。

### HTTPCACHE_IGNORE_HTTP_CODES

新版功能。

默认: `[]`

不缓存设置中的 HTTP 返回值(code)的 request。

### HTTPCACHE_IGNORE_MISSING

默认: `False`

如果启用，在缓存中没找到的 request 将会被忽略，不下载。

### HTTPCACHE_IGNORE_SCHEMES

新版功能。

默认: `['file']`

不缓存这些 URI 标准(scheme)的 response。

### HTTPCACHE_STORAGE

默认: `'scrapy.contrib.httpcache.FilesystemCacheStorage'`

实现缓存存储后端的类。

### HTTPCACHE_DBM_MODULE

新版功能。

默认: `'anydbm'`

在 `DBM 存储后端`的数据库模块。 该设定针对 DBM 后端。

### HTTPCACHE_POLICY

新版功能。

默认: `'scrapy.contrib.httpcache.DummyPolicy'`

实现缓存策略的类。

### HttpCompressionMiddleware

#### class scrapy.contrib.downloadermiddleware.httpcompression.HttpCompressionMiddleware

该中间件提供了对压缩(gzip, deflate)数据的支持。

### HttpCompressionMiddleware Settings

### COMPRESSION_ENABLED

默认:`True`

Compression Middleware(压缩中间件)是否开启。

### ChunkedTransferMiddleware

#### class scrapy.contrib.downloadermiddleware.chunked.ChunkedTransferMiddleware

该中间件添加了对 [chunked transfer encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding) 的支持。

### HttpProxyMiddleware

新版功能。

#### class scrapy.contrib.downloadermiddleware.httpproxy.HttpProxyMiddleware

该中间件提供了对 request 设置 HTTP 代理的支持。您可以通过在 `Request` 对象中设置 `proxy` 元数据来开启代理。

类似于 Python 标准库模块 [urllib](http://docs.python.org/library/urllib.html) 及 [urllib2](http://docs.python.org/library/urllib2.html)，其使用了下列环境变量:

- http_proxy
- https_proxy
- no_proxy

### RedirectMiddleware

#### class scrapy.contrib.downloadermiddleware.redirect.RedirectMiddleware

该中间件根据 response 的状态处理重定向的 request。

通过该中间件的(被重定向的)request 的 url 可以通过 `Request.meta` 的` redirect_urls` 键找到。

`RedirectMiddleware` 可以通过下列设置进行配置(更多内容请参考设置文档):

- REDIRECT_ENABLED
- REDIRECT_MAX_TIMES

如果 `Request.meta` 中 `dont_redirect` 设置为 True，则该 request 将会被此中间件忽略。

### RedirectMiddleware settings

### REDIRECT_ENABLED

新版功能。

默认: `True`

是否启用 Redirect 中间件。

### REDIRECT_MAX_TIMES

默认:`20`

单个 request 被重定向的最大次数。

### MetaRefreshMiddleware

#### class scrapy.contrib.downloadermiddleware.redirect.MetaRefreshMiddleware

该中间件根据 meta-refresh html 标签处理 request 重定向。

`MetaRefreshMiddleware` 可以通过以下设定进行配置 (更多内容请参考设置文档)。

- METAREFRESH_ENABLED
- METAREFRESH_MAXDELAY

该中间件遵循 `RedirectMiddleware` 描述的 `REDIRECT_MAX_TIMES` 设定，`dont_redirect `及 redirect_urls meta key。

### MetaRefreshMiddleware settings

### METAREFRESH_ENABLED

新版功能。

默认: `True`

Meta Refresh 中间件是否启用。

### REDIRECT_MAX_METAREFRESH_DELAY

默认: `100`

跟进重定向的最大 meta-refresh 延迟(单位:秒)。

### RetryMiddleware

#### class scrapy.contrib.downloadermiddleware.retry.RetryMiddleware

该中间件将重试可能由于临时的问题，例如连接超时或者 HTTP 500 错误导致失败的页面。

爬取进程会收集失败的页面并在最后，spider 爬取完所有正常(不失败)的页面后重新调度。 一旦没有更多需要重试的失败页面，该中间件将会发送一个信号(retry_complete)， 其他插件可以监听该信号。

`RetryMiddleware` 可以通过下列设定进行配置 (更多内容请参考设置文档):

- RETRY_ENABLED
- RETRY_TIMES
- RETRY_HTTP_CODES

关于 HTTP 错误的考虑:

如果根据 HTTP 协议，您可能想要在设定 `RETRY_HTTP_CODES` 中移除 400 错误。该错误被默认包括是由于这个代码经常被用来指示服务器过载(overload)了。而在这种情况下，我们想进行重试。

如果 `Request.meta` 中 `dont_retry` 设为 True，该 request 将会被本中间件忽略。

### RetryMiddleware Settings

### RETRY_ENABLED

新版功能。

默认: `True`

Retry Middleware 是否启用。

### RETRY_TIMES

默认:`2`

包括第一次下载，最多的重试次数

### RETRY_HTTP_CODES

默认: `[500, 502, 503, 504, 400, 408]`

重试的 response 返回值(code)。其他错误(DNS 查找问题、连接失败及其他)则一定会进行重试。

### RobotsTxtMiddleware

#### class scrapy.contrib.downloadermiddleware.robotstxt.RobotsTxtMiddleware

该中间件过滤所有 robots.txt eclusion standard 中禁止的 request。

确认该中间件及 `ROBOTSTXT_OBEY ` 设置被启用以确保 Scrapy 尊重 robots.txt。

> 警告
> 
> 记住，如果您在一个网站中使用了多个并发请求，Scrapy 仍然可能下载一些被禁止的页面。这是由于这些页面是在 robots.txt 被下载前被请求的。这是当前 robots.txt 中间件已知的限制，并将在未来进行修复。

### DownloaderStats

#### class scrapy.contrib.downloadermiddleware.stats.DownloaderStats

保存所有通过的 request、response 及 exception 的中间件。

您必须启用 `DOWNLOADER_STATS` 来启用该中间件。

### UserAgentMiddleware

#### class scrapy.contrib.downloadermiddleware.useragent.UserAgentMiddleware

用于覆盖 spider 的默认 user agent 的中间件。

要使得 spider 能覆盖默认的 user agent，其 user_agent 属性必须被设置。

### AjaxCrawlMiddleware

#### class scrapy.contrib.downloadermiddleware.ajaxcrawl.AjaxCrawlMiddleware

根据 meta-fragment html 标签查找 ‘AJAX 可爬取’ 页面的中间件。查看 [https://developers.google.com/webmasters/ajax-crawling/docs/getting-started](https://developers.google.com/webmasters/ajax-crawling/docs/getting-started) 来获得更多内容。

> 注解
> 
> 即使没有启用该中间件，Scrapy 仍能查找类似于 `'http://example.com/!#foo=bar'` 这样的’AJAX 可爬取’页面。AjaxCrawlMiddleware 是针对不具有 `'!#'` 的 URL，通常发生在’index’或者’main’页面中。

### AjaxCrawlMiddleware 设置

### AJAXCRAWL_ENABLED

新版功能。

默认:`False`

AjaxCrawlMiddleware 是否启用。您可能需要针对[通用爬虫](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/broad-crawls.html#topics-broad-crawls)启用该中间件。
