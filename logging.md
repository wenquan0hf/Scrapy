# Logging

crapy 提供了 log 功能。您可以通过 `scrapy.log` 模块使用。当前底层实现使用了 [Twisted logging](http://twistedmatrix.com/projects/core/documentation/howto/logging.html)，不过可能在之后会有所变化。

log 服务必须通过显示调用 `scrapy.log.start()`来开启，以捕捉顶层的 Scrapy 日志消息。 在此之上，每个 crawler 都拥有独立的 log 观察者(observer)(创建时自动连接(attach)),接收其 spider 的日志消息。

## Log levels

Scrapy 提供 5 层 logging 级别:

- CRITICAL - 严重错误(critical)
- ERROR - 一般错误(regular errors)
- WARNING - 警告信息(warning messages)
- INFO - 一般信息(informational messages)
- DEBUG - 调试信息(debugging messages)

## 如何设置 log 级别

您可以通过终端选项(command line option) –loglevel/-L 或 `LOG_LEVEL` 来设置 log 级别。

## 如何记录信息(log messages)

下面给出如何使用 `WARNING` 级别来记录信息的例子:

```
from scrapy import log
log.msg("This is a warning", level=log.WARNING)
```

## 在 Spider 中添加 log(Logging from Spiders)

在 spider 中添加 log 的推荐方式是使用 Spider 的 `log()`方法。该方法会自动在调用 `scrapy.log.msg()`时赋值 spider 参数。其他的参数则直接传递给 `msg()`方法。

## scrapy.log 模块

#### scrapy.log.start(logfile=None, loglevel=None, logstdout=None)

启动 Scrapy 顶层 logger。该方法必须在记录任何顶层消息前被调用 (使用模块的 msg() 而不是 Spider.log 的消息)。否则，之前的消息将会丢失。

**参数:**    

- logfile (str) – 用于保存 log 输出的文件路径。如果被忽略，LOG_FILE 设置会被使用。如果两个参数都是 None，log 将会被输出到标准错误流(standard error)。
- loglevel – 记录的最低的 log 级别。可用的值有: CRITICAL，ERROR，WARNING，INFO and DEBUG。
- logstdout (boolean) – 如果为 True，所有您的应用的标准输出(包括错误)将会被记录(logged instead)。 例如，如果您调用 “print ‘hello’”，则’hello’会在 Scrapy 的 log 中被显示。 如果被忽略，则 LOG_STDOUT 设置会被使用。

#### scrapy.log.msg(message, level=INFO, spider=None)

记录信息(Log a message)

**参数:**    
- message (str) – log 的信息
- level – 该信息的 log 级别. 参考 Log levels.
- spider (Spider 对象) – 记录该信息的 spider. 当记录的信息和特定的 spider 有关联时，该参数必须被使用。

#### scrapy.log.CRITICAL

严重错误的 Log 级别

#### scrapy.log.ERROR

错误的 Log 级别 Log level for errors

#### scrapy.log.WARNING

警告的 Log 级别 Log level for warnings

#### scrapy.log.INFO

记录信息的 Log 级别(生产部署时推荐的 Log 级别)

#### scrapy.log.DEBUG

调试信息的 Log 级别(开发时推荐的 Log 级别)

## Logging 设置

以下设置可以被用来配置 logging:

- LOG_ENABLED
- LOG_ENCODING
- LOG_FILE
- LOG_LEVEL
- LOG_STDOUT
