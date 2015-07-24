# Telnet 终端(Telnet Console)

Scrapy 提供了内置的 telnet 终端，以供检查，控制 Scrapy 运行的进程。 telnet 仅仅是一个运行在 Scrapy 进程中的普通 python 终端。因此您可以在其中做任何事。

telnet 终端是一个[自带的 Scrapy 扩展](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/extensions.html#topics-extensions-ref)。 该扩展默认为启用，不过您也可以关闭。 关于扩展的更多内容请参考[Telnet console 扩展](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/extensions.html#topics-extensions-ref-telnetconsole)。

## 如何访问 telnet 终端

telnet 终端监听设置中定义的 `TELNETCONSOLE_PORT`，默认为 6023。 访问 telnet 请输入:

```
telnet localhost 6023
>>>
```

Windows 及大多数 Linux 发行版都自带了所需的 telnet 程序。

## telnet 终端中可用的变量

telnet 仅仅是一个运行在 Scrapy 进程中的普通 python 终端。因此您可以做任何事情，甚至是导入新终端。

telnet 为了方便提供了一些默认定义的变量:

<table>
<tr>
<td>快捷名称 
</td>
<td>描述
</td>
</tr>
<tr>
<td><code>crawler</code>    
</td>
<td>Scrapy Crawler (<code>scrapy.crawler.Crawler</code> 对象)    
</td>
</tr>
<tr>
<td><code> engine </code>    
</td>
<td>Crawler.engine属性    
</td>
</tr>
<tr>
<td><code>spider </code>    
</td>
<td>当前激活的爬虫(spider)    
</td>
</tr>
<tr>
<td><code> slot</code>    
</td>
<td>the engine slot    
</td>
</tr>
<tr>
<td><code>extensions </code>    
</td>
<td>扩展管理器(manager) (Crawler.extensions属性)    
</td>
</tr>
<tr>
<td><code>stats </code>    
</td>
<td>状态收集器 (Crawler.stats属性)    
</td>
</tr>
<tr>
<td><code>settings </code>    
</td>
<td>Scrapy设置(setting)对象 (Crawler.settings属性)    
</td>
</tr>
<tr>
<td><code>est </code>    
</td>
<td>打印引擎状态的报告    
</td>
</tr>
<tr>
<td><code>prefs </code>    
</td>
<td> 针对内存调试 (参考调试内存溢出)
</td>
</tr>
<tr>
<td><code>p </code>    
</td>
<td>pprint.pprint 函数的简写    
</td>
</tr>
<tr>
<td><code>hpy </code>    
</td>
<td>针对内存调试 (参考 调试内存溢出)    
</td>
</tr>
</table>

## Telnet console usage examples

下面是使用 telnet 终端的一些例子:

### 查看引擎状态

在终端中您可以使用 Scrapy 引擎的 est()方法来快速查看状态:

```
telnet localhost 6023
>>> est()
Execution engine status

time()-engine.start_time                        : 8.62972998619
engine.has_capacity()                           : False
len(engine.downloader.active)                   : 16
engine.scraper.is_idle()                        : False
engine.spider.name                              : followall
engine.spider_is_idle(engine.spider)            : False
engine.slot.closing                             : False
len(engine.slot.inprogress)                     : 16
len(engine.slot.scheduler.dqs or [])            : 0
len(engine.slot.scheduler.mqs)                  : 92
len(engine.scraper.slot.queue)                  : 0
len(engine.scraper.slot.active)                 : 0
engine.scraper.slot.active_size                 : 0
engine.scraper.slot.itemproc_size               : 0
engine.scraper.slot.needs_backout()             : False
```

### 暂停，恢复和停止 Scrapy 引擎

暂停:

```
telnet localhost 6023
>>> engine.pause()
>>>
```

恢复:

```
telnet localhost 6023
>>> engine.unpause()
>>>
```

停止:

```
telnet localhost 6023
>>> engine.stop()
Connection closed by foreign host.
```

## Telnet 终端信号

#### scrapy.telnet.update_telnet_vars(telnet_vars)

在 telnet 终端开启前发送该信号。您可以挂载(hook up)该信号来添加，移除或更新 telnet 本地命名空间可用的变量。您可以通过在您的处理函数(handler)中更新 telnet_vars 字典来实现该修改。

**参数:**    telnet_vars (dict) – telnet 变量的字典

## Telnet 设定

以下是终端的一些设定:

### TELNETCONSOLE_PORT

Default:`[6023, 6073]`

telnet 终端使用的端口范围。如果设为 `None` 或 `0`， 则动态分配端口。

### TELNETCONSOLE_HOST

默认: `'127.0.0.1'`

telnet 终端监听的接口(interface)。
