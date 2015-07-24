# 异常(Exceptions)

## 内置异常参考手册(Built-in Exceptions reference)

下面是 Scrapy 提供的异常及其用法。

### DropItem

#### exception scrapy.exceptions.DropItem

该异常由 item pipeline 抛出，用于停止处理 item。详细内容请参考 [Item Pipeline](item-pipeline.md)。

### CloseSpider

#### exception scrapy.exceptions.CloseSpider(reason='cancelled')

该异常由 spider 的回调函数(callback)抛出，来暂停/停止 spider。支持的参数:

**参数:**    

- **reason** (*str*) – 关闭的原因

样例:

```
def parse_page(self, response):
    if 'Bandwidth exceeded' in response.body:
        raise CloseSpider('bandwidth_exceeded')
```

### IgnoreRequest

#### exception scrapy.exceptions.IgnoreRequest

该异常由调度器(Scheduler)或其他下载中间件抛出，声明忽略该 request。

### NotConfigured

#### exception scrapy.exceptions.NotConfigured

该异常由某些组件抛出，声明其仍然保持关闭。这些组件包括:

- Extensions
- Item pipelines
- Downloader middlwares
- Spider middlewares

该异常必须由组件的构造器(constructor)抛出。

### NotSupported

#### exception scrapy.exceptions.NotSupported

该异常声明一个不支持的特性。
