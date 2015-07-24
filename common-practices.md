# 实践经验(Common Practices)

本章节记录了使用 Scrapy 的一些实践经验(common practices)。 这包含了很多使用不会包含在其他特定章节的的内容。

## 在脚本中运行 Scrapy

除了常用的 `scrapy crawl` 来启动 Scrapy，您也可以使用 `API` 在脚本中启动 Scrapy。

需要注意的是，Scrapy 是在 Twisted 异步网络库上构建的，因此其必须在 Twisted reactor 里运行。

另外，在 spider 运行结束后，您必须自行关闭 Twisted reactor。这可以通过在 `CrawlerRunner.crawl` 所返回的对象中添加回调函数来实现。

下面给出了如何实现的例子，使用 [testspiders](https://github.com/scrapinghub/testspiders) 项目作为例子。

```
from twisted.internet import reactor
from scrapy.crawler import CrawlerRunner
from scrapy.utils.project import get_project_settings

runner = CrawlerRunner(get_project_settings())

# 'followall' is the name of one of the spiders of the project.
d = runner.crawl('followall', domain='scrapinghub.com')
d.addBoth(lambda _: reactor.stop())
reactor.run() # the script will block here until the crawling is finished
Running spiders outside projects it’s not much different. You have to create a generic Settings object and populate it as needed (See 内置设定参考手册 for the available settings), instead of using the configuration returned by get_project_settings.

Spiders can still be referenced by their name if SPIDER_MODULES is set with the modules where Scrapy should look for spiders. Otherwise, passing the spider class as first argument in the CrawlerRunner.crawl method is enough.

from twisted.internet import reactor
from scrapy.spider import Spider
from scrapy.crawler import CrawlerRunner
from scrapy.settings import Settings

class MySpider(Spider):
    # Your spider definition
    ...

settings = Settings({'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'})
runner = CrawlerRunner(settings)

d = runner.crawl(MySpider)
d.addBoth(lambda _: reactor.stop())
reactor.run() # the script will block here until the crawling is finished
```

> 参见
> 
> Twisted Reactor Overview.

## 同一进程运行多个 spider

默认情况下，当您执行 `scrapy crawl` 时，Scrapy 每个进程运行一个 spider。 当然，Scrapy 通过`内部(internal)API` 也支持单进程多个 spider。

下面以 [testspiders](https://github.com/scrapinghub/testspiders) 作为例子来说明如何同时运行多个 spider:

```
from twisted.internet import reactor, defer
from scrapy.crawler import CrawlerRunner
from scrapy.utils.project import get_project_settings

runner = CrawlerRunner(get_project_settings())
dfs = set()
for domain in ['scrapinghub.com', 'insophia.com']:
    d = runner.crawl('followall', domain=domain)
    dfs.add(d)

defer.DeferredList(dfs).addBoth(lambda _: reactor.stop())
reactor.run() # the script will block here until all crawling jobs are finished
```

相同的例子，不过通过链接(chaining) deferred 来线性运行 spider:

```
from twisted.internet import reactor, defer
from scrapy.crawler import CrawlerRunner
from scrapy.utils.project import get_project_settings

runner = CrawlerRunner(get_project_settings())

@defer.inlineCallbacks
def crawl():
    for domain in ['scrapinghub.com', 'insophia.com']:
        yield runner.crawl('followall', domain=domain)
    reactor.stop()

crawl()
reactor.run() # the script will block here until the last crawl call is finished
```

> 参见
> 
> 在脚本中运行 Scrapy。

## 分布式爬虫(Distributed crawls)

Scrapy 并没有提供内置的机制支持分布式(多服务器)爬取。不过还是有办法进行分布式爬取，取决于您要怎么分布了。

如果您有很多 spider，那分布负载最简单的办法就是启动多个 Scrapyd，并分配到不同机器上。

如果想要在多个机器上运行一个单独的 spider，那您可以将要爬取的 url 进行分块，并发送给 spider。 例如:

首先，准备要爬取的 url 列表，并分配到不同的文件 url 里:

```
http://somedomain.com/urls-to-crawl/spider1/part1.list
http://somedomain.com/urls-to-crawl/spider1/part2.list
http://somedomain.com/urls-to-crawl/spider1/part3.list
```

接着在 3 个不同的 Scrapd 服务器中启动 spider。spider 会接收一个(spider)参数 part，该参数表示要爬取的分块:

```
curl http://scrapy1.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=1
curl http://scrapy2.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=2
curl http://scrapy3.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=3
```

## 避免被禁止(ban)

有些网站实现了特定的机制，以一定规则来避免被爬虫爬取。 与这些规则打交道并不容易，需要技巧，有时候也需要些特别的基础。 如果有疑问请考虑联系[商业支持](http://scrapy.org/support/)。

下面是些处理这些站点的建议(tips):

- 使用 user agent 池，轮流选择之一来作为 user agent。池中包含常见的浏览器的 user agent(google 一下一大堆)
- 禁止 cookies(参考 `COOKIES_ENABLED`)，有些站点会使用 cookies 来发现爬虫的轨迹。
- 设置下载延迟(2 或更高)。参考 `DOWNLOAD_DELAY` 设置。
- 如果可行，使用 [Google cache](http://www.googleguide.com/cached_pages.html) 来爬取数据，而不是直接访问站点。
- 使用 IP 池。例如免费的 [Tor 项目](https://www.torproject.org/)或付费服务([ProxyMesh](http://proxymesh.com/))。
- 使用高度分布式的下载器(downloader)来绕过禁止(ban)，您就只需要专注分析处理页面。这样的例子有:[Crawlera](http://crawlera.com/)

如果您仍然无法避免被 ban，考虑联系[商业支持](http://scrapy.org/support/)。

## 动态创建 Item 类

对于有些应用，item 的结构由用户输入或者其他变化的情况所控制。您可以动态创建 class。

```
from scrapy.item import DictItem, Field

def create_item_class(class_name, field_list):
fields = {field_name: Field() for field_name in field_list}

return type(class_name, (DictItem,), {'fields': fields})
```