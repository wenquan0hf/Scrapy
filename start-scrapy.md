# 初窥 Scrapy

Scrapy 是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

其最初是为了[页面抓取](https://en.wikipedia.org/wiki/Data_scraping#Screen_scraping) (更确切来说，[网络抓取](https://en.wikipedia.org/wiki/Web_scraping))所设计的， 也可以应用在获取 API 所返回的数据(例如 [Amazon Associates Web Services ](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html)) 或者通用的网络爬虫。

本文档将通过介绍 Scrapy 背后的概念使您对其工作原理有所了解， 并确定Scrapy是否是您所需要的。

当您准备好开始您的项目后，您可以参考[入门教程](scrapy-tutorial.md)。

## 选择一个网站

当您需要从某个网站中获取信息，但该网站未提供API或能通过程序获取信息的机制时，Scrapy 可以助你一臂之力。

以 [Mininova](http://www.mininova.org/) 网站为例，我们想要获取今日添加的所有种子的 URL、名字、描述以及文件大小信息。

今日添加的种子列表可以通过这个页面找到:  
[http://www.mininova.org/today](http://www.mininova.org/today)  

## 定义您想抓取的数据

第一步是定义我们需要爬取的数据。在 Scrapy 中， 这是通过 Scrapy Items 来完成的。(在本例子中为种子文件)

我们定义的 Item:

```
import scrapy

class TorrentItem(scrapy.Item):
    url = scrapy.Field()
    name = scrapy.Field()
    description = scrapy.Field()
    size = scrapy.Field()
```

## 编写提取数据的 Spider

第二步是编写一个 spider。其定义了初始 URL([http://www.mininova.org/today](http://www.mininova.org/today))、针对后续链接的规则以及从页面中提取数据的规则。

通过观察页面的内容可以发现，所有种子的URL都类似 `http://www.mininova.org/tor/NUMBER` 。 其中，`NUMBER` 是一个整数。 根据此规律，我们可以定义需要进行跟进的链接的正则表达式: `/tor/\d+ `。

我们使用 XPath 来从页面的HTML源码中选择需要提取的数据。 以其中一个种子文件的页面为例:

[http://www.mininova.org/tor/2676093](http://www.mininova.org/tor/2676093)

观察 HTML 页面源码并创建我们需要的数据(种子名字，描述和大小)的 XPath 表达式。

通过观察，我们可以发现文件名是包含在 `<h1>` 标签中的:

```
<h1>Darwin - The Evolution Of An Exhibition</h1>
```

与此对应的 XPath 表达式:

```
//h1/text()
```

种子的描述是被包含在 `id="description"` 的`<div>`标签中:

```
<h2>Description:</h2>

<div id="description">
Short documentary made for Plymouth City Museum and Art Gallery regarding the setup of an exhibit about Charles Darwin in conjunction with the 200th anniversary of his birth.

...
```

对应获取描述的 XPath 表达式:

```
//div[@id='description']
```

文件大小的信息包含在 `id=specifications` 的`<div>`的第二个`<p>`标签中:

```
<div id="specifications">

<p>
<strong>Category:</strong>
<a href="/cat/4">Movies</a> &gt; <a href="/sub/35">Documentary</a>
</p>

<p>
<strong>Total size:</strong>
150.62&nbsp;megabyte</p>
```

选择文件大小的 XPath 表达式:

```
//div[@id='specifications']/p[2]/text()[2]
```

关于 XPath 的详细内容请参考[ XPath 参考](http://www.w3.org/TR/xpath/) 。

最后，结合以上内容给出 spider 的代码:

```
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor

class MininovaSpider(CrawlSpider):

    name = 'mininova'
    allowed_domains = ['mininova.org']
    start_urls = ['http://www.mininova.org/today']
    rules = [Rule(LinkExtractor(allow=['/tor/\d+']), 'parse_torrent')]

    def parse_torrent(self, response):
        torrent = TorrentItem()
        torrent['url'] = response.url
        torrent['name'] = response.xpath("//h1/text()").extract()
        torrent['description'] = response.xpath("//div[@id='description']").extract()
        torrent['size'] = response.xpath("//div[@id='info-left']/p[2]/text()[2]").extract()
        return torrent
```

`TorrentItem` 的定义在 上面 。

## 执行 spider，获取数据

终于，我们可以运行 spider 来获取网站的数据，并以 JSON 格式存入到 scraped_data.json 文件中:

```
scrapy crawl mininova -o scraped_data.json
```

命令中使用了 [Feed 导出](feed-exports.md) 来导出 JSON 文件。您可以修改导出格式(XML 或者 CSV)或者存储后端(FTP 或者 [Amazon S3](http://aws.amazon.com/cn/s3/))，这并不困难。

同时，您也可以编写 item 管道 将 item 存储到数据库中。

## 查看提取到的数据

执行结束后，当您查看 `scraped_data.json`，您将看到提取到的 item:

```
[{"url": "http://www.mininova.org/tor/2676093", "name": ["Darwin - The Evolution Of An Exhibition"], "description": ["Short documentary made for Plymouth ..."], "size": ["150.62 megabyte"]},
# ... other items ...
]
```

由于 [选择器(Selectors)](selectors.md) 返回 list，所以值都是以 list 存储的(除了 `url` 是直接赋值之外)。 如果您想要保存单个数据或者对数据执行额外的处理，那将是 [Item Loaders](item-loaders.md) 发挥作用的地方。

## 还有什么？

您已经了解了如何通过 Scrapy 提取存储网页中的信息，但这仅仅只是冰山一角。Scrapy 提供了很多强大的特性来使得爬取更为简单高效，例如:

- HTML，XML 源数据 选择及提取 的内置支持
- 提供了一系列在spider之间共享的可复用的过滤器(即 [Item Loaders](item-loaders.md))，对智能处理爬取数据提供了内置支持。
- 通过 feed 导出 提供了多格式(JSON、CSV、XML)，多存储后端(FTP、S3、本地文件系统)的内置支持
- 提供了 media pipeline，可以 自动下载 爬取到的数据中的图片(或者其他资源)。
- 高扩展性。您可以通过使用 signals，设计好的API(中间件，extensions，pipelines)来定制实现您的功能。
- 内置的中间件及扩展为下列功能提供了支持:
	- cookies and session 处理
	- HTTP 压缩
	- HTTP 认证
	- HTTP 缓存
	- user-agent 模拟
	- robots.txt
	- 爬取深度限制
	- 其他
- 针对非英语语系中不标准或者错误的编码声明, 提供了自动检测以及健壮的编码支持。
- 支持根据模板生成爬虫。在加速爬虫创建的同时，保持在大型项目中的代码更为一致。详细内容请参阅 [genspider](command-line-tools.md) 命令。
- 针对多爬虫下性能评估、失败检测，提供了可扩展的[状态收集工具](stats-collection.md)。
- 提供[交互式 shell 终端](scrapy-shell.md)，为您测试 XPath 表达式，编写和调试爬虫提供了极大的方便
- 提供 [System service](scrapyd.md)，简化在生产环境的部署及运行
- 内置 Telnet 终端 ，通过在 Scrapy 进程中钩入 Python 终端，使您可以查看并且调试爬虫
- Logging 为您在爬取过程中捕捉错误提供了方便
- 支持 Sitemaps 爬取
- 具有缓存的 DNS 解析器

## 接下来

下一步当然是 [下载 Scrapy](http://scrapy.org/download/) 了， 您可以阅读[Scrapy 入门教程](scrapy-tutorial.md)并加入[社区](http://scrapy.org/community/)。感谢您的支持!