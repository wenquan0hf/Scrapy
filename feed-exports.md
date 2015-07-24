# Feed exports

新版功能。

实现爬虫时最经常提到的需求就是能合适的保存爬取到的数据，或者说，生成一个带有爬取数据的”输出文件”(通常叫做”输出 feed”)，来供其他系统使用。

Scrapy 自带了 Feed 输出，并且支持多种序列化格式(serialization format)及存储方式(storage backends)。

## 序列化方式(Serialization formats)

feed 输出使用到了 Item exporters 。其自带支持的类型有:

- [JSON](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/feed-exports.html#topics-feed-format-json)
- [JSON lines](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/feed-exports.html#topics-feed-format-jsonlines)
- [CSV](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/feed-exports.html#topics-feed-format-csv)
- [XML](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/feed-exports.html#topics-feed-format-xml)

您也可以通过 FEED_EXPORTERS 设置扩展支持的属性。

### JSON

- FEED_FORMAT: json
- 使用的 exporter: JsonItemExporter
- 大数据量情况下使用 JSON 请参见 这个警告

### JSON lines

- FEED_FORMAT: jsonlines
- 使用的 exporter: JsonLinesItemExporter

### CSV

- FEED_FORMAT: csv
- 使用的 exporter: CsvItemExporter

### XML

- FEED_FORMAT: xml
- 使用的 exporter: XmlItemExporter

### Pickle

- FEED_FORMAT: pickle
- 使用的 exporter: PickleItemExporter

### Marshal

- FEED_FORMAT: marshal
- 使用的 exporter: MarshalItemExporter

## 存储(Storages)

使用 feed 输出时您可以通过使用 [URI](http://en.wikipedia.org/wiki/Uniform_Resource_Identifier)(通过 FEED_URI 设置) 来定义存储端。feed 输出支持 URI 方式支持的多种存储后端类型。

自带支持的存储后端有:

- 本地文件系统
- FTP
- S3 (需要 boto)
- 标准输出

有些存储后端会因所需的外部库未安装而不可用。例如，S3 只有在 boto 库安装的情况下才可使用。

## 存储 URI 参数

存储 URI 也包含参数。当 feed 被创建时这些参数可以被覆盖:

- `%(time)s` - 当 feed 被创建时被 timestamp 覆盖
- `%(name)s` - 被 spider 的名字覆盖

其他命名的参数会被 spider 同名的属性所覆盖。例如， 当 feed 被创建时， `%(site_id)s` 将会被 `spider.site_id` 属性所覆盖。

下面用一些例子来说明:

- 存储在 FTP，每个 spider 一个目录:

	- `ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json`  

- 存储在 S3，每一个 spider 一个目录:

   - `s3://mybucket/scraping/feeds/%(name)s/%(time)s.json`

## 存储端(Storage backends)

### 本地文件系统

将 feed 存储在本地系统。

- URI scheme: `file`
- URI 样例: `file:///tmp/export.csv`
- 需要的外部依赖库:`none`

注意: (只有)存储在本地文件系统时，您可以指定一个绝对路径 /tmp/export.csv 并忽略协议(scheme)。不过这仅仅只能在 Unix 系统中工作。

### FTP

将 feed 存储在 FTP 服务器。

- URI scheme:`ftp`
- URI 样例:`ftp://user:pass@ftp.example.com/path/to/export.csv`
- 需要的外部依赖库:`none`

### S3

将 feed 存储在 Amazon S3 。

- URI scheme: s3
- URI 样例:
	- s3://mybucket/path/to/export.csv
	- s3://aws_key:aws_secret@mybucket/path/to/export.csv
- 需要的外部依赖库: boto

您可以通过在 URI 中传递 user/pass 来完成 AWS 认证，或者也可以通过下列的设置来完成:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

### 标准输出

feed 输出到 Scrapy 进程的标准输出。

- URI scheme: stdout
- URI 样例: stdout:
- 需要的外部依赖库: none

## 设定(Settings)

这些是配置 feed 输出的设定:

- FEED_URI (必须)
-  FEED_FORMAT
-  FEED_STORAGES
-  FEED_EXPORTERS
-  FEED_STORE_EMPTY

### FEED_URI

Default:`None`

输出 feed 的 URI。支持的 URI 协议请参见`存储端(Storage backends)`。

为了启用 feed 输出，该设定是必须的。

### FEED_FORMAT

输出 feed 的序列化格式。可用的值请参见`序列化方式(Serialization formats)`。

### FEED_STORE_EMPTY

Default:`False`

是否输出空 feed(没有 item 的 feed)。

### FEED_STORAGES

Default::`{}`

包含项目支持的额外 feed 存储端的字典。 字典的键(key)是 URI 协议(scheme)，值是存储类(storage class)的路径。

### FEED_STORAGES_BASE

Default:

```
{
    '': 'scrapy.contrib.feedexport.FileFeedStorage',
    'file': 'scrapy.contrib.feedexport.FileFeedStorage',
    'stdout': 'scrapy.contrib.feedexport.StdoutFeedStorage',
    's3': 'scrapy.contrib.feedexport.S3FeedStorage',
    'ftp': 'scrapy.contrib.feedexport.FTPFeedStorage',
}
```

包含 Scrapy 内置支持的 feed 存储端的字典。

### FEED_EXPORTERS

Default::`{}`

包含项目支持的额外输出器(exporter)的字典。 该字典的键(key)是 URI 协议(scheme)，值是 Item 输出器(exporter) 类的路径。

### FEED_EXPORTERS_BASE

Default:

```
FEED_EXPORTERS_BASE = {
    'json': 'scrapy.contrib.exporter.JsonItemExporter',
    'jsonlines': 'scrapy.contrib.exporter.JsonLinesItemExporter',
    'csv': 'scrapy.contrib.exporter.CsvItemExporter',
    'xml': 'scrapy.contrib.exporter.XmlItemExporter',
    'marshal': 'scrapy.contrib.exporter.MarshalItemExporter',
}
```

包含 Scrapy 内置支持的 feed 输出器(exporter)的字典。
