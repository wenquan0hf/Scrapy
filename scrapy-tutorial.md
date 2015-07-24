# Scrapy 入门教程

在本篇教程中，我们假定您已经安装好 Scrapy。 如若不然，请参考`安装指南` 。

接下来以 [Open Directory Project(dmoz) (dmoz)](http://www.dmoz.org/) 为例来讲述爬取。

本篇教程中将带您完成下列任务:


1. 创建一个 Scrapy 项目
2. 定义提取的 Item
3. 编写爬取网站的 `spider` 并提取 `Item`
4. 编写 `Item Pipeline` 来存储提取到的 Item(即数据)

Scrapy 由 [Python](https://www.python.org/) 编写。如果您刚接触并且好奇这门语言的特性以及 Scrapy 的详情，对于已经熟悉其他语言并且想快速学习 Python 的编程老手，我们推荐 [Learn Python The Hard Way](http://learnpythonthehardway.org/book/)，对于想从 Python 开始学习的编程新手，[非程序员的 Python 学习资料列表](http://wiki.python.org/moin/BeginnersGuide/NonProgrammers)将是您的选择。

## 创建项目

在开始爬取之前，您必须创建一个新的 Scrapy 项目。 进入您打算存储代码的目录中，运行下列命令:

```
scrapy startproject tutorial
```

该命令将会创建包含下列内容的 tutorial 目录:

```
tutorial/
    scrapy.cfg
    tutorial/
        __init__.py
        items.py
        pipelines.py
        settings.py
        spiders/
            __init__.py
            ...
```

这些文件分别是:

- scrapy.cfg: 项目的配置文件
- tutorial/: 该项目的 python 模块。之后您将在此加入代码。
- tutorial/items.py: 项目中的 item 文件。
- tutorial/pipelines.py: 项目中的 pipelines 文件。
- tutorial/settings.py: 项目的设置文件。
- tutorial/spiders/: 放置 spider 代码的目录。

## 定义 Item

Item 是保存爬取到的数据的容器；其使用方法和 python 字典类似， 并且提供了额外保护机制来避免拼写错误导致的未定义字段错误。

类似在 ORM 中做的一样，您可以通过创建一个 `scrapy.Item` 类， 并且定义类型为 `scrapy.Field` 的类属性来定义一个 Item。 (如果不了解 ORM, 不用担心，您会发现这个步骤非常简单)

首先根据需要从 dmoz.org 获取到的数据对 item 进行建模。 我们需要从 dmoz 中获取名字，url，以及网站的描述。 对此，在 item 中定义相应的字段。编辑 `tutorial` 目录中的 `items.py` 文件:

```
import scrapy

class DmozItem(scrapy.Item):
    title = scrapy.Field()
    link = scrapy.Field()
    desc = scrapy.Field()
```

一开始这看起来可能有点复杂，但是通过定义 item， 您可以很方便的使用 Scrapy 的其他方法。而这些方法需要知道您的 item 的定义。

## 编写第一个爬虫(Spider)

Spider 是用户编写用于从单个网站(或者一些网站)爬取数据的类。

其包含了一个用于下载的初始 URL，如何跟进网页中的链接以及如何分析页面中的内容， 提取生成 `item` 的方法。

为了创建一个 Spider，您必须继承 `scrapy.Spider` 类， 且定义以下三个属性:


- `name`: 用于区别 Spider。 该名字必须是唯一的，您不可以为不同的 Spider 设定相同的名字。
- `start_urls`: 包含了 Spider 在启动时进行爬取的 url 列表。 因此，第一个被获取到的页面将是其中之一。 后续的 URL 则从初始的 URL 获取到的数据中提取。
- `parse()` 是 spider 的一个方法。 被调用时，每个初始 URL 完成下载后生成的 `Response` 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成 item)以及生成需要进一步处理的 URL 的 `Request 对象。

以下为我们的第一个 Spider 代码，保存在 tutorial/spiders 目录下的 dmoz_spider.py 文件中:

```
import scrapy

class DmozSpider(scrapy.spiders.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        filename = response.url.split("/")[-2]
        with open(filename, 'wb') as f:
            f.write(response.body)
```

### 爬取

进入项目的根目录，执行下列命令启动 spider:

```
scrapy crawl dmoz
```

`crawl dmoz` 启动用于爬取 `dmoz.org` 的 spider，您将得到类似的输出:

```
2014-01-23 18:13:07-0400 [scrapy] INFO: Scrapy started (bot: tutorial)
2014-01-23 18:13:07-0400 [scrapy] INFO: Optional features available: ...
2014-01-23 18:13:07-0400 [scrapy] INFO: Overridden settings: {}
2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled extensions: ...
2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled downloader middlewares: ...
2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled spider middlewares: ...
2014-01-23 18:13:07-0400 [scrapy] INFO: Enabled item pipelines: ...
2014-01-23 18:13:07-0400 [dmoz] INFO: Spider opened
2014-01-23 18:13:08-0400 [dmoz] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/> (referer: None)
2014-01-23 18:13:09-0400 [dmoz] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
2014-01-23 18:13:09-0400 [dmoz] INFO: Closing spider (finished)
```

查看包含 `[dmoz]` 的输出，可以看到输出的 log 中包含定义在 `start_urls` 的初始 URL，并且与 spider 中是一一对应的。在 log 中可以看到其没有指向其他页面( (`referer:None`) )。

除此之外，更有趣的事情发生了。就像我们 parse 方法指定的那样，有两个包含 url 所对应的内容的文件被创建了: Book，Resources 。

### 刚才发生了什么？

Scrapy 为 Spider 的 start_urls 属性中的每个 URL 创建了 scrapy.Request 对象，并将 parse 方法作为回调函数(callback)赋值给了 Request。

Request 对象经过调度，执行生成 `scrapy.http.Response` 对象并送回给 `spider parse()` 方法。

### 提取 Item

#### Selectors 选择器简介

从网页中提取数据有很多方法。Scrapy 使用了一种基于 [XPath](http://www.w3.org/TR/xpath) 和 [CSS](http://www.w3.org/TR/selectors) 表达式机制: `Scrapy Selectors`。 关于 selector 和其他提取机制的信息请参考 `Selector` 文档。

这里给出 XPath 表达式的例子及对应的含义:


- /html/head/title: 选择 HTML 文档中<head\>标签内的<title\>元素
- /html/head/title/text(): 选择上面提到的<title\>元素的文字
- //td: 选择所有的<td\>元素
- //div[@class="mine"]: 选择所有具有 class="mine"属性的 div 元素

上边仅仅是几个简单的 XPath 例子，XPath 实际上要比这远远强大的多。 如果您想了解的更多，我们推荐[这篇 XPath 教程](http://www.w3school.com.cn/xpath/index.asp)。

为了配合 XPath，Scrapy 除了提供了 `Selector` 之外，还提供了方法来避免每次从 response 中提取数据时生成 selector 的麻烦。

Selector 有四个基本的方法(点击相应的方法可以看到详细的 API 文档):


- `xpath()`: 传入 xpath 表达式，返回该表达式所对应的所有节点的 selector list 列表 。
- `css()`: 传入 CSS 表达式，返回该表达式所对应的所有节点的 selector list 列表.
- `extract()`: 序列化该节点为 unicode 字符串并返回 list。
- `re()`: 根据传入的正则表达式对数据进行提取，返回 unicode 字符串 list 列表。

#### 在 Shell 中尝试 Selector 选择器

为了介绍 Selector 的使用方法，接下来我们将要使用内置的 Scrapy shell。Scrapy Shell 需要您预装好 IPython(一个扩展的 Python 终端)。

您需要进入项目的根目录，执行下列命令来启动 shell:

```
scrapy shell "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/"
```

> 注解
> 
> 当您在终端运行 Scrapy 时，请一定记得给 url 地址加上引号，否则包含参数的 url(例如 & 字符)会导致 Scrapy 运行失败。

shell 的输出类似:

```
[ ... Scrapy log here ... ]

2015-01-07 22:01:53+0800 [domz] DEBUG: Crawled (200) <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/> (referer: None)
[s] Available Scrapy objects:
[s]   crawler    <scrapy.crawler.Crawler object at 0x02CE2530>
[s]   item       {}
[s]   request    <GET http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
[s]   response   <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
[s]   sel        <Selector xpath=None data=u'<html lang="en">\r\n<head>\r\n<meta http-equ'>
[s]   settings   <CrawlerSettings module=<module 'tutorial.settings' from 'tutorial\settings.pyc'>>
[s]   spider     <DomzSpider 'domz' at 0x302e350>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser

>>>
```

当 shell 载入后，您将得到一个包含 response 数据的本地 response 变量。输入 response.body 将输出 response 的包体，输出 response.headers 可以看到 response 的包头。

更为重要的是，当输入 response.selector 时， 您将获取到一个可以用于查询返回数据的 selector(选择器)， 以及映射到 response.selector.xpath() 、 response.selector.css() 的 快捷方法(shortcut): response.xpath() 和 response.css() 。

同时，shell 根据 response 提前初始化了变量 sel 。该 selector 根据 response 的类型自动选择最合适的分析规则(XML vs HTML)。

让我们来试试:

```
In [1]: sel.xpath('//title')
Out[1]: [<Selector xpath='//title' data=u'<title>Open Directory - Computers: Progr'>]

In [2]: sel.xpath('//title').extract()
Out[2]: [u'<title>Open Directory - Computers: Programming: Languages: Python: Books</title>']

In [3]: sel.xpath('//title/text()')
Out[3]: [<Selector xpath='//title/text()' data=u'Open Directory - Computers: Programming:'>]

In [4]: sel.xpath('//title/text()').extract()
Out[4]: [u'Open Directory - Computers: Programming: Languages: Python: Books']

In [5]: sel.xpath('//title/text()').re('(\w+):')
Out[5]: [u'Computers', u'Programming', u'Languages', u'Python']
```

#### 提取数据

现在，我们来尝试从这些页面中提取些有用的数据。

您可以在终端中输入 `response.body` 来观察 HTML 源码并确定合适的 XPath 表达式。不过，这任务非常无聊且不易。您可以考虑使用 Firefox 的 Firebug 扩展来使得工作更为轻松。详情请参考 使用 [Firebug 进行爬取](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/firebug.html#topics-firebug)和[借助 Firefox 来爬取](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/firefox.html#topics-firefox)。

在查看了网页的源码后，您会发现网站的信息是被包含在 第二个 <ul\> 元素中。

我们可以通过这段代码选择该页面中网站列表里所有 <li\> 元素:

```
sel.xpath('//ul/li')
```

网站的描述:

```
sel.xpath('//ul/li/text()').extract()
```

网站的标题:

```
sel.xpath('//ul/li/a/text()').extract()
```

以及网站的链接:

```
sel.xpath('//ul/li/a/@href').extract()
```

之前提到过，每个`.xpath()`调用返回 selector 组成的 list，因此我们可以拼接更多的`.xpath()`来进一步获取某个节点。我们将在下边使用这样的特性:

```
for sel in response.xpath('//ul/li'):
    title = sel.xpath('a/text()').extract()
    link = sel.xpath('a/@href').extract()
    desc = sel.xpath('text()').extract()
    print title, link, desc
```

> 注解
> 
> 关于嵌套 selctor 的更多详细信息，请参考 嵌套选择器(selectors) 以及 选择器(Selectors) 文档中的 使用相对 XPaths 部分。

在我们的 spider 中加入这段代码:

```
import scrapy

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            title = sel.xpath('a/text()').extract()
            link = sel.xpath('a/@href').extract()
            desc = sel.xpath('text()').extract()
            print title, link, desc
```

现在尝试再次爬取 dmoz.org，您将看到爬取到的网站信息被成功输出:

```
scrapy crawl dmoz
```

### 使用 item

Item 对象是自定义的 python 字典。您可以使用标准的字典语法来获取到其每个字段的值。(字段即是我们之前用 Field 赋值的属性):

```
>>> item = DmozItem()
>>> item['title'] = 'Example title'
>>> item['title']
'Example title'
```

一般来说，Spider 将会将爬取到的数据以 Item 对象返回。所以为了将爬取的数据返回，我们最终的代码将是:

```
import scrapy

from tutorial.items import DmozItem

class DmozSpider(scrapy.Spider):
    name = "dmoz"
    allowed_domains = ["dmoz.org"]
    start_urls = [
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Books/",
        "http://www.dmoz.org/Computers/Programming/Languages/Python/Resources/"
    ]

    def parse(self, response):
        for sel in response.xpath('//ul/li'):
            item = DmozItem()
            item['title'] = sel.xpath('a/text()').extract()
            item['link'] = sel.xpath('a/@href').extract()
            item['desc'] = sel.xpath('text()').extract()
            yield item
```

> 注解
> 
> 您可以在 [dirbot](https://github.com/scrapy/dirbot) 项目中找到一个具有完整功能的 spider。该项目可以通过 [https://github.com/scrapy/dirbot](https://github.com/scrapy/dirbot) 找到。

现在对 dmoz.org 进行爬取将会产生 `DmozItem` 对象:

```
[dmoz] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
     {'desc': [u' - By David Mertz; Addison Wesley. Book in progress, full text, ASCII format. Asks for feedback. [author website, Gnosis Software, Inc.\n],
      'link': [u'http://gnosis.cx/TPiP/'],
      'title': [u'Text Processing in Python']}
[dmoz] DEBUG: Scraped from <200 http://www.dmoz.org/Computers/Programming/Languages/Python/Books/>
     {'desc': [u' - By Sean McGrath; Prentice Hall PTR, 2000, ISBN 0130211192, has CD-ROM. Methods to build XML applications fast, Python tutorial, DOM and SAX, new Pyxie open source XML processing library. [Prentice Hall PTR]\n'],
      'link': [u'http://www.informit.com/store/product.aspx?isbn=0130211192'],
      'title': [u'XML Processing with Python']}
```

## 保存爬取到的数据

最简单存储爬取的数据的方式是使用 `Feed exports`:

```
scrapy crawl dmoz -o items.json
```

该命令将采用 [JSON](http://en.wikipedia.org/wiki/JSON) 格式对爬取的数据进行序列化，生成 `items.json` 文件。

在类似本篇教程里这样小规模的项目中，这种存储方式已经足够。 如果需要对爬取到的 item 做更多更为复杂的操作，您可以编写 `Item Pipeline` 。 类似于我们在创建项目时对 Item 做的，用于您编写自己的 `tutorial/pipelines.py` 也被创建。 不过如果您仅仅想要保存 item，您不需要实现任何的 pipeline。

## 下一步

本篇教程仅介绍了 Scrapy 的基础，还有很多特性没有涉及。请查看 初窥 Scrapy 章节中的 还有什么？ 部分,大致浏览大部分重要的特性。

接着，我们推荐您把玩一个例子(查看`例子`)，而后继续阅读`基本概念` 。
