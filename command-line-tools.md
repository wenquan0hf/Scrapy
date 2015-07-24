# 命令行工具(Command line tools)

新版功能.

Scrapy 是通过 scrapy 命令行工具进行控制的。 这里我们称之为 “Scrapy tool” 以用来和子命令进行区分。对于子命令，我们称为 “command” 或者 “Scrapy commands”。

Scrapy tool 针对不同的目的提供了多个命令，每个命令支持不同的参数和选项。

## 默认的 Scrapy 项目结构

在开始对命令行工具以及子命令的探索前，让我们首先了解一下 Scrapy 的项目的目录结构。

虽然可以被修改，但所有的 Scrapy 项目默认有类似于下边的文件结构:

```
scrapy.cfg
myproject/
    __init__.py
    items.py
    pipelines.py
    settings.py
    spiders/
        __init__.py
        spider1.py
        spider2.py
        ...
```

`scrapy.cfg` 存放的目录被认为是 项目的根目录 。该文件中包含 python 模块名的字段定义了项目的设置。例如:

```
[settings]
default = myproject.settings
```

## 使用 `scrapy` 工具

您可以以无参数的方式启动 Scrapy 工具。该命令将会给出一些使用帮助以及可用的命令:

```
Scrapy X.Y - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  crawl         Run a spider
  fetch         Fetch a URL using the Scrapy downloader
[...]
```

如果您在 Scrapy 项目中运行，当前激活的项目将会显示在输出的第一行。上面的输出就是响应的例子。如果您在一个项目中运行命令将会得到类似的输出:

```
Scrapy X.Y - project: myproject

Usage:
  scrapy <command> [options] [args]

[...]
```

### 创建项目

一般来说，使用 `scrapy` 工具的第一件事就是创建您的 Scrapy 项目:

```
scrapy startproject myproject
```

该命令将会在 myproject 目录中创建一个 Scrapy 项目。

接下来，进入到项目目录中:

```
cd myproject
```

这时候您就可以使用 scrapy 命令来管理和控制您的项目了。

### 控制项目

您可以在您的项目中使用 `scrapy` 工具来对其进行控制和管理。

比如，创建一个新的 spider:

```
scrapy genspider mydomain mydomain.com
```

有些 Scrapy 命令(比如 `crawl`)要求必须在 Scrapy 项目中运行。 您可以通过下边的 `commands reference` 来了解哪些命令需要在项目中运行，哪些不用。

另外要注意，有些命令在项目里运行时的效果有些许区别。 以 fetch 命令为例，如果被爬取的 url 与某个特定 spider 相关联， 则该命令将会使用 spider 的动作(spider-overridden behaviours)。 (比如 spider 指定的 `user_agent`)。 该表现是有意而为之的。一般来说， `fetch` 命令就是用来测试检查 spider 是如何下载页面。

## 可用的工具命令(tool commands)

该章节提供了可用的内置命令的列表。每个命令都提供了描述以及一些使用例子。您总是可以通过运行命令来获取关于每个命令的详细内容:

```
scrapy <command> -h
```

您也可以查看所有可用的命令:

```
scrapy -h
```

Scrapy 提供了两种类型的命令。一种必须在 Scrapy 项目中运行(针对项目(Project-specific)的命令)，另外一种则不需要(全局命令)。全局命令在项目中运行时的表现可能会与在非项目中运行有些许差别(因为可能会使用项目的设定)。

全局命令:


- `startproject`
- `settings`
- `runspider`
- `shell`
- `fetch`
- `view`
- `version`

项目(Project-only)命令:


- `crawl`
- `check`
- `list`
- `edit`
- `parse`
- `genspider`
- `deploy`
- `bench`

### startproject
- 语法: `scrapy startproject <project_name>`  
- 是否需要项目: no

在 project\_name 文件夹下创建一个名为 project\_name 的 Scrapy 项目。

例子:

```
$ scrapy startproject myproject
```

### genspider

- 语法: `scrapy genspider [-t template] <name> <domain>`
- 是否需要项目: yes

在当前项目中创建 spider。

这仅仅是创建 spider 的一种快捷方法。该方法可以使用提前定义好的模板来生成 spider。您也可以自己创建 spider 的源码文件。

例子:

```
$ scrapy genspider -l
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed

$ scrapy genspider -d basic
import scrapy

class $classname(scrapy.Spider):
    name = "$name"
    allowed_domains = ["$domain"]
    start_urls = (
        'http://www.$domain/',
        )

    def parse(self, response):
        pass

$ scrapy genspider -t basic example example.com
Created spider 'example' using template 'basic' in module:
  mybot.spiders.example
```

### crawl

- 语法: `scrapy crawl <spider>`
- 是否需要项目: yes

使用 spider 进行爬取。

例子:

```
$ scrapy crawl myspider
[ ... myspider starts crawling ... ]
```

### check

- 语法: `scrapy check [-l] <spider>`
- 是否需要项目: yes

运行 contract 检查。

例子:

```
$ scrapy check -l
first_spider
  * parse
  * parse_item
second_spider
  * parse
  * parse_item

$ scrapy check
[FAILED] first_spider:parse_item
>>> 'RetailPricex' field is missing

[FAILED] first_spider:parse
>>> Returned 92 requests, expected 0..4
```

### list

- 语法: `scrapy list`
- 是否需要项目: yes

列出当前项目中所有可用的 spider。每行输出一个 spider。

使用例子:

```
$ scrapy list
spider1
spider2
```

### edit

- 语法: `scrapy edit <spider>`
- 是否需要项目: yes

使用 EDITOR 中设定的编辑器编辑给定的 spider

该命令仅仅是提供一个快捷方式。开发者可以自由选择其他工具或者 IDE 来编写调试 spider。

例子:

```
$ scrapy edit spider1
```

### fetch

- 语法:`scrapy fetch <url>`
- 是否需要项目: no

使用 Scrapy 下载器(downloader)下载给定的 URL，并将获取到的内容送到标准输出。

该命令以 spider 下载页面的方式获取页面。例如，如果 spider 有 `USER_AGENT` 属性修改了 User Agent，该命令将会使用该属性。

因此，您可以使用该命令来查看 spider 如何获取某个特定页面。

该命令如果非项目中运行则会使用默认 Scrapy downloader 设定。

例子:

```
$ scrapy fetch --nolog http://www.example.com/some/page.html
[ ... html content here ... ]

$ scrapy fetch --nolog --headers http://www.example.com/
{'Accept-Ranges': ['bytes'],
 'Age': ['1263   '],
 'Connection': ['close     '],
 'Content-Length': ['596'],
 'Content-Type': ['text/html; charset=UTF-8'],
 'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
 'Etag': ['"573c1-254-48c9c87349680"'],
 'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
 'Server': ['Apache/2.2.3 (CentOS)']}
```

### view

- 语法:`scrapy view <url>`
- 是否需要项目: no

在浏览器中打开给定的 URL，并以 Scrapy spider 获取到的形式展现。 有些时候 spider 获取到的页面和普通用户看到的并不相同。 因此该命令可以用来检查 spider 所获取到的页面，并确认这是您所期望的。

例子:

```
$ scrapy view http://www.example.com/some/page.html
[ ... browser starts ... ]
```

### shell

- 语法: `scrapy shell [url]`
- 是否需要项目: no

以给定的 URL(如果给出)或者空(没有给出 URL)启动 Scrapy shell。查看 Scrapy 终端(Scrapy shell) 获取更多信息。

例子:

```
$ scrapy shell http://www.example.com/some/page.html
[ ... scrapy shell starts ... ]
```

### parse

- 语法: scrapy parse <url> [options]
- 是否需要项目: yes

获取给定的 URL 并使用相应的 spider 分析处理。如果您提供`--callback` 选项，则使用 spider 的该方法处理，否则使用 `parse`。

支持的选项:


- `--spider=SPIDER`: 跳过自动检测 spider 并强制使用特定的 spider
- `--a NAME=VALUE`: 设置 spider 的参数(可能被重复)
- `--callback or -c`: spider 中用于解析返回(response)的回调函数
- `--pipelines`: 在 pipeline 中处理 item
- `--rules or -r`: 使用 CrawlSpider 规则来发现用来解析返回(response)的回调函数
- `--noitems`: 不显示爬取到的 item
- `--nolinks`: 不显示提取到的链接
- `--nocolour`: 避免使用 pygments 对输出着色
- `--depth or -d`: 指定跟进链接请求的层次数(默认:1)
- `--verbose or -v`: 显示每个请求的详细信息

例子:

```
$ scrapy parse http://www.example.com/ -c parse_item
[ ... scrapy log lines crawling example.com spider ... ]

>>> STATUS DEPTH LEVEL 1 <<<
# Scraped Items  ------------------------------------------------------------
[{'name': u'Example item',
 'category': u'Furniture',
 'length': u'12 cm'}]

# Requests  -----------------------------------------------------------------
[]
```

### settings

- 语法:`scrapy settings [options]`
- 是否需要项目: no

获取 Scrapy 的设定

在项目中运行时，该命令将会输出项目的设定值，否则输出 Scrapy 默认设定。

例子:

```
$ scrapy settings --get BOT_NAME
scrapybot
$ scrapy settings --get DOWNLOAD_DELAY
0
```

### runspider

- 语法:`scrapy runspider <spider_file.py>`
- 是否需要项目: no

在未创建项目的情况下，运行一个编写在 Python 文件中的 spider。

例子:

```
$ scrapy runspider myspider.py
[ ... spider starts crawling ... ]
```

### version

- 语法:`scrapy version [-v]`
- 是否需要项目: no

输出 Scrapy 版本。配合 -v 运行时，该命令同时输出 Python，Twisted 以及平台的信息，方便 bug 提交。

### deploy

新版功能。

- 语法:`scrapy deploy [ <target:project> | -l <target> | -L ]`
- 是否需要项目: yes

将项目部署到 Scrapyd 服务。查看[部署您的项目](http://scrapyd.readthedocs.org/en/latest/deploy.html)。

### bench

新版功能。

- 语法: scrapy bench
- 是否需要项目: no

运行 benchmark 测试。Benchmarking。

## 自定义项目命令

您也可以通过 COMMANDS_MODULE 来添加您自己的项目命令。您可以以 [scrapy/commands](https://github.com/scrapy/scrapy/tree/master/scrapy/commands) 中 Scrapy commands 为例来了解如何实现您的命令。

## COMMANDS_MODULE

```
Default: '' (empty string)
```

用于查找添加自定义 Scrapy 命令的模块。

例子:

```
COMMANDS_MODULE = 'mybot.commands'
```