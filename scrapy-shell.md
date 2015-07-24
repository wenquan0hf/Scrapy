# Scrapy 终端(Scrapy shell)

Scrapy 终端是一个交互终端，供您在未启动 spider 的情况下尝试及调试您的爬取代码。 其本意是用来测试提取数据的代码，不过您可以将其作为正常的 Python 终端，在上面测试任何的 Python 代码。

该终端是用来测试 XPath 或 CSS 表达式，查看他们的工作方式及从爬取的网页中提取的数据。 在编写您的 spider 时，该终端提供了交互性测试您的表达式代码的功能，免去了每次修改后运行 spider 的麻烦。

一旦熟悉了 Scrapy 终端后，您会发现其在开发和调试 spider 时发挥的巨大作用。

如果您安装了 [IPython](http://ipython.org/)，Scrapy 终端将使用 [IPython](http://ipython.org/) (替代标准 Python 终端)。 IPython 终端与其他相比更为强大，提供智能的自动补全，高亮输出，及其他特性。

我们强烈推荐您安装 [IPython](http://ipython.org/)，特别是如果您使用 Unix 系统([IPython](http://ipython.org/) 在 Unix 下工作的很好)。 详情请参考 IPython installation guide 。

## 启动终端

您可以使用 shell 来启动 Scrapy 终端:

```
scrapy shell <url>
```

<url\> 是您要爬取的网页的地址。

## 使用终端

Scrapy 终端仅仅是一个普通的 Python 终端(或 [IPython](http://ipython.org/))。其提供了一些额外的快捷方式。

### 可用的快捷命令(shortcut)

- `shelp()` - 打印可用对象及快捷命令的帮助列表
- `fetch(request_or_url)` - 根据给定的请求(request)或 URL 获取一个新的 response，并更新相关的对象
- `view(response)` - 在本机的浏览器打开给定的 response。 其会在 response 的 body 中添加一个 <base> tag ，使得外部链接(例如图片及 css)能正确显示。 注意，该操作会在本地创建一个临时文件，且该文件不会被自动删除。

### 可用的 Scrapy 对象

Scrapy 终端根据下载的页面会自动创建一些方便使用的对象，例如 Response 对象及 Selector 对象(对 HTML 及 XML 内容)。

这些对象有:

- crawler - 当前 Crawler 对象。
- spider - 处理 URL 的 spider。对当前 URL 没有处理的 Spider 时则为一个 Spider 对象。
- request - 最近获取到的页面的 Request 对象。您可以使用 replace() 修改该 request。或者 使用 fetch 快捷方式来获取新的 request。
- response - 包含最近获取到的页面的 Response 对象。
- sel - 根据最近获取到的 response 构建的 Selector 对象。
- settings - 当前的 Scrapy settings

## 终端会话(shell session)样例

下面给出一个典型的终端会话的例子。 在该例子中，我们首先爬取了 [http://scarpy.org](http://scarpy.org) 的页面，而后接着爬取 [http://slashdot.org](http://scarpy.org) 的页面。 最后，我们修改了(Slashdot)的请求，将请求设置为 POST 并重新获取， 得到 HTTP 405(不允许的方法)错误。 之后通过 Ctrl-D(Unix)或 Ctrl-Z(Windows)关闭会话。

需要注意的是，由于爬取的页面不是静态页，内容会随着时间而修改， 因此例子中提取到的数据可能与您尝试的结果不同。 该例子的唯一目的是让您熟悉 Scrapy 终端。

首先，我们启动终端:

```
scrapy shell 'http://scrapy.org' --nolog
```

接着该终端(使用 Scrapy 下载器(downloader))获取 URL 内容并打印可用的对象及快捷命令(注意到以`[s]`开头的行):

```
[s] Available Scrapy objects:
[s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
[s]   item       {}
[s]   request    <GET http://scrapy.org>
[s]   response   <200 http://scrapy.org>
[s]   sel        <Selector xpath=None data=u'<html>\n  <head>\n    <meta charset="utf-8'>
[s]   settings   <scrapy.settings.Settings object at 0x2bfd650>
[s]   spider     <Spider 'default' at 0x20c6f50>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser

>>>
```

之后，您就可以操作这些对象了:

```
>>> sel.xpath("//h2/text()").extract()[0]
u'Welcome to Scrapy'

>>> fetch("http://slashdot.org")
[s] Available Scrapy objects:
[s]   crawler    <scrapy.crawler.Crawler object at 0x1a13b50>
[s]   item       {}
[s]   request    <GET http://slashdot.org>
[s]   response   <200 http://slashdot.org>
[s]   sel        <Selector xpath=None data=u'<html lang="en">\n<head>\n\n\n\n\n<script id="'>
[s]   settings   <scrapy.settings.Settings object at 0x2bfd650>
[s]   spider     <Spider 'default' at 0x20c6f50>
[s] Useful shortcuts:
[s]   shelp()           Shell help (print this help)
[s]   fetch(req_or_url) Fetch request (or URL) and update local objects
[s]   view(response)    View response in a browser

>>> sel.xpath('//title/text()').extract()
[u'Slashdot: News for nerds, stuff that matters']

>>> request = request.replace(method="POST")

>>> fetch(request)
[s] Available Scrapy objects:
[s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
...

>>>
```

## 在 spider 中启动 shell 来查看 response

有时您想在 spider 的某个位置中查看被处理的 response， 以确认您期望的 response 到达特定位置。

这可以通过 `scrapy.shell.inspect_response` 函数来实现。

以下是如何在 spider 中调用该函数的例子:

```
import scrapy

class MySpider(scrapy.Spider):
    name = "myspider"
    start_urls = [
        "http://example.com",
        "http://example.org",
        "http://example.net",
    ]

    def parse(self, response):
        # We want to inspect one specific response.
        if ".org" in response.url:
            from scrapy.shell import inspect_response
            inspect_response(response, self)

        # Rest of parsing code.
```

当运行 spider 时，您将得到类似下列的输出:

```
2014-01-23 17:48:31-0400 [myspider] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
2014-01-23 17:48:31-0400 [myspider] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
[s] Available Scrapy objects:
[s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
...

>>> response.url
'http://example.org'
```

接着测试提取代码:

```
>>> sel.xpath('//h1[@class="fn"]')
[]
```

呃，看来是没有。您可以在浏览器里查看 response 的结果，判断是否是您期望的结果:

```
>>> view(response)
True
```

最后您可以点击 Ctrl-D(Windows 下 Ctrl-Z)来退出终端，恢复爬取:

```
>>> ^D
2014-01-23 17:50:03-0400 [myspider] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
...
```

注意: 由于该终端屏蔽了 Scrapy 引擎，您在这个终端中不能使用 `fetch` 快捷命令(shortcut)。 当您离开终端时，spider 会从其停下的地方恢复爬取，正如上面显示的那样。
