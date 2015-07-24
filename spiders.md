# Spiders

Spider 类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取 item)。换句话说，Spider 就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。

对 spider 来说，爬取的循环类似下文:

1. 以初始的 URL 初始化 Request，并设置回调函数。 当该 request 下载完毕并返回时，将生成 response，并作为参数传给该回调函数。

    spider 中初始的 request 是通过调用 `start_requests()`来获取的。`start_requests()`读取 `start_url`s 中的 URL， 并以 `parse` 为回调函数生成 `Request`。

2. 在回调函数内分析返回的(网页)内容，返回 Item 对象或者 Request 或者一个包括二者的可迭代容器。 返回的 Request 对象之后会经过 Scrapy 处理，下载相应的内容，并调用设置的 callback 函数(函数可相同)。

3. 在回调函数内，您可以使用 选择器(Selectors) (您也可以使用 BeautifulSoup, lxml 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成 item。

4. 最后，由 spider 返回的 item 将被存到数据库(由某些 Item Pipeline 处理)或使用 Feed exports 存入到文件中。

虽然该循环对任何类型的 spider 都(多少)适用，但 Scrapy 仍然为了不同的需求提供了多种默认 spider。 之后将讨论这些 spider。

## Spider 参数

Spider 可以通过接受参数来修改其功能。 spider 参数一般用来定义初始 URL 或者指定限制爬取网站的部分。 您也可以使用其来配置 spider 的任何功能。

在运行 `crawl` 时添加`-a` 可以传递 Spider 参数:

```
scrapy crawl myspider -a category=electronics
```

Spider 在构造器(constructor)中获取参数:

```
import scrapy

class MySpider(Spider):
    name = 'myspider'

    def __init__(self, category=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        # ...
```

Spider 参数也可以通过 Scrapyd 的 `schedule.json API` 来传递。 参见 [Scrapyd documentation](http://scrapyd.readthedocs.org/)。

## 内置 Spider 参考手册

Scrapy 提供多种方便的通用 spider 供您继承使用。 这些 spider 为一些常用的爬取情况提供方便的特性， 例如根据某些规则跟进某个网站的所有链接、根据 [Sitemaps](http://www.sitemaps.org/) 来进行爬取，或者分析 XML/CSV 源。

下面 spider 的示例中，我们假定您有个项目在 `myproject.items` 模块中声明了 `TestItem`:

```
import scrapy

class TestItem(scrapy.Item):
    id = scrapy.Field()
    name = scrapy.Field()
    description = scrapy.Field()
```

### Spider

#### class scrapy.spider.Spider

Spider 是最简单的 spider。每个其他的 spider 必须继承自该类(包括 Scrapy 自带的其他 spider 以及您自己编写的 spider)。Spider 并没有提供什么特殊的功能。其仅仅请求给定的 start\_urls/start_requests，并根据返回的结果(resulting responses)调用 spider 的 parse 方法。

##### name

定义 spider 名字的字符串(string)。spider 的名字定义了 Scrapy 如何定位(并初始化)spider，所以其必须是唯一的。 不过您可以生成多个相同的 spider 实例(instance)，这没有任何限制。 name 是 spider 最重要的属性，而且是必须的。

如果该 spider 爬取单个网站(single domain)，一个常见的做法是以该网站(domain)(加或不加[后缀](http://en.wikipedia.org/wiki/Top-level_domain))来命名 spider。 例如，如果 spider 爬取 `mywebsite.com`，该 spider 通常会被命名为 mywebsite 。

##### allowed_domains

可选。包含了 spider 允许爬取的域名(domain)列表(list)。 当 `OffsiteMiddleware` 启用时， 域名不在列表中的 URL 不会被跟进。

##### start_urls

URL 列表。当没有制定特定的 URL 时，spider 将从该列表中开始进行爬取。 因此，第一个被获取到的页面的 URL 将是该列表之一。后续的 URL 将会从获取到的数据中提取。

##### custom_settings

A dictionary of settings that will be overridden from the project wide configuration when running this spider. It must be defined as a class attribute since the settings are updated before instantiation.

For a list of available built-in settings see: [内置设定参考手册](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/settings.html#topics-settings-ref)。 

##### crawler

This attribute is set by the from_crawler() class method after initializating the class, and links to the Crawler object to which this spider instance is bound.

Crawlers encapsulate a lot of components in the project for their single entry access (such as extensions, middlewares, signals managers, etc). See Crawler API to know more about them.

##### settings

Configuration on which this spider is been ran. This is a Settings instance, see the Settings topic for a detailed introduction on this subject.

##### from_crawler(crawler, *args, **kwargs)

This is the class method used by Scrapy to create your spiders.

You probably won’t need to override this directly, since the default implementation acts as a proxy to the __init__() method, calling it with the given arguments args and named arguments kwargs.

Nonetheless, this method sets the crawler and settings attributes in the new instance, so they can be accessed later inside the spider’s code.

**参数:**   

   - **crawler** (Crawler instance) – crawler to which the spider will be bound 
   - **args** (list) – arguments passed to the __init__() method
   - **kwargs** (dict) – keyword arguments passed to the __init__() method

##### start_requests()

该方法必须返回一个可迭代对象(iterable)。该对象包含了 spider 用于爬取的第一个 Request。

当 spider 启动爬取并且未制定 URL 时，该方法被调用。 当指定了 URL 时，`make_requests_from_url()`将被调用来创建 Request 对象。 该方法仅仅会被 Scrapy 调用一次，因此您可以将其实现为生成器。

该方法的默认实现是使用 `start_urls` 的 url 生成 Request。

如果您想要修改最初爬取某个网站的 Request 对象，您可以重写(override)该方法。 例如，如果您需要在启动时以 POST 登录某个网站，你可以这么写:

```
def start_requests(self):
    return [scrapy.FormRequest("http://www.example.com/login",
                               formdata={'user': 'john', 'pass': 'secret'},
                               callback=self.logged_in)]

def logged_in(self, response):
    # here you would extract links to follow and return Requests for
    # each of them, with another callback
    pass
```

##### make_requests_from_url(url)

该方法接受一个 URL 并返回用于爬取的 Request 对象。该方法在初始化 request 时被 start_requests()调用，也被用于转化 url 为 request。

默认未被复写(overridden)的情况下，该方法返回的 Request 对象中，`parse()`作为回调函数，dont_filter 参数也被设置为开启。(详情参见 `Request`)。

##### parse(response)

当 response 没有指定回调函数时，该方法是 Scrapy 处理下载的 response 的默认方法。

parse 负责处理 response 并返回处理的数据以及(/或)跟进的 URL。 Spider 对其他的 Request 的回调函数也有相同的要求。

该方法及其他的 Request 回调函数必须返回一个包含 Request 及(或) Item 的可迭代的对象。

**参数:**    response (`Response`) – 用于分析的 response

##### log(message[, level, component])

使用 scrapy.log.msg() 方法记录(log)message。log 中自动带上该 spider 的 name 属性。 更多数据请参见 Logging。

##### closed(reason)

当 spider 关闭时，该函数被调用。 该方法提供了一个替代调用 signals.connect()来监听 spider_closed 信号的快捷方式。

### Spider 样例

让我们来看一个例子:

```
import scrapy

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        self.log('A response from %s just arrived!' % response.url)
```

另一个在单个回调函数中返回多个 Request 以及 Item 的例子:

```
import scrapy
from myproject.items import MyItem

class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = [
        'http://www.example.com/1.html',
        'http://www.example.com/2.html',
        'http://www.example.com/3.html',
    ]

    def parse(self, response):
        sel = scrapy.Selector(response)
        for h3 in response.xpath('//h3').extract():
            yield MyItem(title=h3)

        for url in response.xpath('//a/@href').extract():
            yield scrapy.Request(url, callback=self.parse)
```

### CrawlSpider

#### class scrapy.contrib.spiders.CrawlSpider

爬取一般网站常用的 spider。其定义了一些规则(rule)来提供跟进 link 的方便的机制。 也许该 spider 并不是完全适合您的特定网站或项目，但其对很多情况都使用。因此您可以以其为起点，根据需求修改部分方法。当然您也可以实现自己的 spider。

除了从 Spider 继承过来的(您必须提供的)属性外，其提供了一个新的属性:

##### rules

一个包含一个(或多个) Rule 对象的集合(list)。 每个 Rule 对爬取网站的动作定义了特定表现。 Rule 对象在下边会介绍。 如果多个 rule 匹配了相同的链接，则根据他们在本属性中被定义的顺序，第一个会被使用。

该 spider 也提供了一个可复写(overrideable)的方法:

##### parse_start_url(response)

当 start_url 的请求返回时，该方法被调用。 该方法分析最初的返回值并必须返回一个 Item 对象或者 一个 Request 对象或者 一个可迭代的包含二者对象。

### 爬取规则(Crawling rules)

####class scrapy.contrib.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

link_extractor 是一个 Link Extractor 对象。其定义了如何从爬取到的页面提取链接。

callback 是一个 callable 或 string(该 spider 中同名的函数将会被调用)。从 link_extractor 中每获取到链接时将会调用该函数。该回调函数接受一个 response 作为其第一个参数，并返回一个包含 Item 以及(或) Request 对象(或者这两者的子类)的列表(list)。

> 警告
> 
> 当编写爬虫规则时，请避免使用 parse 作为回调函数。 由于 CrawlSpider 使用 parse 方法来实现其逻辑，如果 您覆盖了 parse 方法，crawl spider 将会运行失败。
> 

`cb_kwargs` 包含传递给回调函数的参数(keyword argument)的字典。

`follow` 是一个布尔(boolean)值，指定了根据该规则从 response 提取的链接是否需要跟进。 如果 `callback` 为 None，`follow` 默认设置为 `True`，否则默认为 `False`。

`process_links` 是一个 callable 或 string(该 spider 中同名的函数将会被调用)。 从 link_extractor 中获取到链接列表时将会调用该函数。该方法主要用来过滤。

process_request 是一个 callable 或 string(该 spider 中同名的函数将会被调用)。 该规则提取到每个 request 时都会调用该函数。该函数必须返回一个 request 或者 None。 (用来过滤 request)

### CrawlSpider 样例

接下来给出配合 rule 使用 CrawlSpider 的例子:

```
import scrapy
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor

class MySpider(CrawlSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com']

    rules = (
        # 提取匹配 'category.php' (但不匹配 'subsection.php') 的链接并跟进链接(没有 callback 意味着 follow 默认为 True)
        Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        # 提取匹配 'item.php' 的链接并使用 spider 的 parse_item 方法进行分析
        Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    def parse_item(self, response):
        self.log('Hi, this is an item page! %s' % response.url)

        item = scrapy.Item()
        item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        return item
```

该 spider 将从 example.com 的首页开始爬取，获取 category 以及 item 的链接并对后者使用 parse_item 方法。当 item 获得返回(response)时，将使用 XPath 处理 HTML 并生成一些数据填入 Item 中。

### XMLFeedSpider

#### class scrapy.contrib.spiders.XMLFeedSpider

XMLFeedSpider 被设计用于通过迭代各个节点来分析 XML 源(XML feed)。迭代器可以从 iternodes，xml，html 选择。鉴于 xml 以及 html 迭代器需要先读取所有 DOM 再分析而引起的性能问题，一般还是推荐使用 iternodes。不过使用 html 作为迭代器能有效应对错误的 XML。

您必须定义下列类属性来设置迭代器以及标签名(tag name):

##### iterator

用于确定使用哪个迭代器的 string。可选项有:

- 'iternodes' -一个高性能的基于正则表达式的迭代器
- 'html' -使用 Selector 的迭代器。 需要注意的是该迭代器使用 DOM 进行分析，其需要将所有的 DOM 载入内存， 当数据量大的时候会产生问题。
- 'xml' -使用 Selector 的迭代器。 需要注意的是该迭代器使用 DOM 进行分析，其需要将所有的DOM 载入内存，当数据量大的时候会产生问题。

默认值为 iternodes 。

##### itertag

一个包含开始迭代的节点名的 string。例如:

```
itertag = 'product'
```

##### namespaces

一个由 (prefix, url) 元组(tuple)所组成的 list。 其定义了在该文档中会被 spider 处理的可用的 namespace。 prefix 及 uri 会被自动调用 `register_namespace()`生成 namespace。

您可以通过在 `itertag` 属性中制定节点的 namespace。

例如:

```
class YourSpider(XMLFeedSpider):

    namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
    itertag = 'n:url'
    # ...
```

除了这些新的属性之外，该 spider 也有以下可以覆盖(overrideable)的方法:

##### adapt_response(response)

该方法在 spider 分析 response 前被调用。您可以在 response 被分析之前使用该函数来修改内容(body)。该方法接受一个 response 并返回一个 response(可以相同也可以不同)。

##### parse_node(response, selector)

当节点符合提供的标签名时(itertag)该方法被调用。 接收到的 response 以及相应的 Selector 作为参数传递给该方法。 该方法返回一个 Item 对象或者 Request 对象 或者一个包含二者的可迭代对象(iterable)。

##### process_results(response, results)

当 spider 返回结果(item 或 request)时该方法被调用。设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理，例如设定 item 的 ID。其接受一个结果的列表(list of results)及对应的 response。其结果必须返回一个结果的列表(list of results)(包含 Item 或者 Request 对象)。

### XMLFeedSpider 例子

该 spider 十分易用。下边是其中一个例子:

```
from scrapy import log
from scrapy.contrib.spiders import XMLFeedSpider
from myproject.items import TestItem

class MySpider(XMLFeedSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com/feed.xml']
    iterator = 'iternodes' # This is actually unnecessary, since it's the default value
    itertag = 'item'

    def parse_node(self, response, node):
        log.msg('Hi, this is a <%s> node!: %s' % (self.itertag, ''.join(node.extract())))

        item = TestItem()
        item['id'] = node.xpath('@id').extract()
        item['name'] = node.xpath('name').extract()
        item['description'] = node.xpath('description').extract()
        return item
```

简单来说，我们在这里创建了一个 spider，从给定的 start_urls 中下载 feed，并迭代 feed 中每个 item 标签，输出，并在 Item 中存储有些随机数据。

### CSVFeedSpider

#### class scrapy.contrib.spiders.CSVFeedSpider

该 spider 除了其按行遍历而不是节点之外其他和 XMLFeedSpider 十分类似。 而其在每次迭代时调用的是 `parse_row()`。

##### delimiter

在 CSV 文件中用于区分字段的分隔符。类型为 string。 默认为 ',' (逗号)。

##### quotechar

A string with the enclosure character for each field in the CSV file Defaults to '"' (quotation mark).

##### headers

在 CSV 文件中包含的用来提取字段的行的列表。参考下边的例子。

##### parse_row(response, row)

该方法接收一个 response 对象及一个以提供或检测出来的 header 为键的字典(代表每行)。 该 spider 中，您也可以覆盖 `adapt_response` 及 `process_results` 方法来进行预处理(pre-processing)及后(post-processing)处理。

### CSVFeedSpider 例子

下面的例子和之前的例子很像，但使用了 `CSVFeedSpider`:

```
from scrapy import log
from scrapy.contrib.spiders import CSVFeedSpider
from myproject.items import TestItem

class MySpider(CSVFeedSpider):
    name = 'example.com'
    allowed_domains = ['example.com']
    start_urls = ['http://www.example.com/feed.csv']
    delimiter = ';'
    quotechar = "'"
    headers = ['id', 'name', 'description']

    def parse_row(self, response, row):
        log.msg('Hi, this is a row!: %r' % row)

        item = TestItem()
        item['id'] = row['id']
        item['name'] = row['name']
        item['description'] = row['description']
        return item
```

### SitemapSpider

#### class scrapy.contrib.spiders.SitemapSpider

SitemapSpider 使您爬取网站时可以通过 Sitemaps 来发现爬取的 URL。

其支持嵌套的 sitemap，并能从 robots.txt 中获取 sitemap 的 url。

##### sitemap_urls

包含您要爬取的 url 的 sitemap 的 url 列表(list)。您也可以指定为一个 robots.txt，spider 会从中分析并提取 url。

##### sitemap_rules

一个包含 (regex, callback) 元组的列表(list):

- `regex` 是一个用于匹配从 sitemap 提供的 url 的正则表达式。`regex` 可以是一个字符串或者编译的正则对象(compiled regex object)。
- callback 指定了匹配正则表达式的 url 的处理函数。`callback` 可以是一个字符串(spider 中方法的名字)或者是 callable。

例如:

```
sitemap_rules = [('/product/', 'parse_product')]
```

规则按顺序进行匹配，之后第一个匹配才会被应用。

如果您忽略该属性，sitemap 中发现的所有 url 将会被 parse 函数处理。

##### sitemap_follow

一个用于匹配要跟进的 sitemap 的正则表达式的列表(list)。其仅仅被应用在 使用 *Sitemap index files* 来指向其他 sitemap 文件的站点。

默认情况下所有的 sitemap 都会被跟进。

##### sitemap_alternate_links

指定当一个 url 有可选的链接时，是否跟进。 有些非英文网站会在一个 url 块内提供其他语言的网站链接。

例如:

```
<url>
    <loc>http://example.com/</loc>
    <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
</url>
```

当 `sitemap_alternate_links` 设置时，两个 URL 都会被获取。 当 `sitemap_alternate_links` 关闭时，只有 `http://example.com/` 会被获取。

默认 `sitemap_alternate_links` 关闭。

### SitemapSpider 样例

简单的例子:使用 parse 处理通过 sitemap 发现的所有 url:

```
from scrapy.contrib.spiders import SitemapSpider

class MySpider(SitemapSpider):
    sitemap_urls = ['http://www.example.com/sitemap.xml']

    def parse(self, response):
        pass # ... scrape item here ...
```

用特定的函数处理某些 url，其他的使用另外的 callback:

```
from scrapy.contrib.spiders import SitemapSpider

class MySpider(SitemapSpider):
    sitemap_urls = ['http://www.example.com/sitemap.xml']
    sitemap_rules = [
        ('/product/', 'parse_product'),
        ('/category/', 'parse_category'),
    ]

    def parse_product(self, response):
        pass # ... scrape product ...

    def parse_category(self, response):
        pass # ... scrape category ...
```

跟进 [robots.txt](http://www.robotstxt.org/) 文件定义的 sitemap 并只跟进包含有`..sitemap_shop` 的 url:

```
from scrapy.contrib.spiders import SitemapSpider

class MySpider(SitemapSpider):
    sitemap_urls = ['http://www.example.com/robots.txt']
    sitemap_rules = [
        ('/shop/', 'parse_shop'),
    ]
    sitemap_follow = ['/sitemap_shops']

    def parse_shop(self, response):
        pass # ... scrape shop here ...
```

在 SitemapSpider 中使用其他 url:

```
from scrapy.contrib.spiders import SitemapSpider

class MySpider(SitemapSpider):
    sitemap_urls = ['http://www.example.com/robots.txt']
    sitemap_rules = [
        ('/shop/', 'parse_shop'),
    ]

    other_urls = ['http://www.example.com/about']

    def start_requests(self):
        requests = list(super(MySpider, self).start_requests())
        requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
        return requests

    def parse_shop(self, response):
        pass # ... scrape shop here ...

    def parse_other(self, response):
        pass # ... scrape other here ...
```