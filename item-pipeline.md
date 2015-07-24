# Item Pipeline

当 Item 在 Spider 中被收集之后，它将会被传递到 Item Pipeline，一些组件会按照一定的顺序执行对 Item 的处理。

每个 item pipeline 组件(有时称之为“Item Pipeline”)是实现了简单方法的 Python 类。他们接收到 Item 并通过它执行一些行为，同时也决定此 Item 是否继续通过 pipeline，或是被丢弃而不再进行处理。

以下是 item pipeline 的一些典型应用：

- 清理 HTML 数据
- 验证爬取的数据(检查 item 包含某些字段)
- 查重(并丢弃)
- 将爬取结果保存到数据库中

## 编写你自己的 item pipeline

编写你自己的 item pipeline 很简单，每个 item pipeline 组件是一个独立的 Python 类，同时必须实现以下方法:

#### process_item(self, item, spider)

每个 item pipeline 组件都需要调用该方法，这个方法必须返回一个 Item (或任何继承类)对象， 或是抛出 DropItem 异常，被丢弃的 item 将不会被之后的 pipeline 组件所处理。

**参数: **   
- item (Item 对象) – 被爬取的 item
- spider (Spider 对象) – 爬取该 item 的 spider

此外，他们也可以实现以下方法:

#### open_spider(self, spider)

当 spider 被开启时，这个方法被调用。

**参数:**   
 
- spider (Spider 对象) – 被开启的 spider

#### close_spider(spider)

当 spider 被关闭时，这个方法被调用

**参数:**    

spider (Spider 对象) – 被关闭的 spider

#### from_crawler(cls, crawler)

If present, this classmethod is called to create a pipeline instance from a Crawler. It must return a new instance of the pipeline. Crawler object provides access to all Scrapy core components like settings and signals; it is a way for pipeline to access them and hook its functionality into Scrapy.

**参数:**
    
crawler (Crawler object) – crawler that uses this pipeline

## Item pipeline 样例

### 验证价格，同时丢弃没有价格的 item

让我们来看一下以下这个假设的 pipeline，它为那些不含税(`price_excludes_vat` 属性)的 item 调整了 `price` 属性，同时丢弃了那些没有价格的 item:

```
from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item['price']:
            if item['price_excludes_vat']:
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)
```

### 将 item 写入 JSON 文件

以下 pipeline 将所有(从所有 spider 中)爬取到的 item，存储到一个独立地 items.jl 文件，每行包含一个序列化为 JSON 格式的 item:

```
import json

class JsonWriterPipeline(object):

    def __init__(self):
        self.file = open('items.jl', 'wb')

    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item
```

> 注解
> 
> JsonWriterPipeline 的目的只是为了介绍怎样编写 item pipeline，如果你想要将所有爬取的 item 都保存到同一个 JSON 文件， 你需要使用 Feed exports 。

### Write items to MongoDB

In this example we’ll write items to MongoDB using pymongo. MongoDB address and database name are specified in Scrapy settings; MongoDB collection is named after item class.

The main point of this example is to show how to use from_crawler() method and how to clean up the resources properly.

> 注解
> 
> Previous example (JsonWriterPipeline) doesn’t clean up resources properly. Fixing it is left as an exercise for the reader.
> import pymongo

```
class MongoPipeline(object):

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        collection_name = item.__class__.__name__
        self.db[collection_name].insert(dict(item))
        return item
```

### 去重

一个用于去重的过滤器，丢弃那些已经被处理过的 item。让我们假设我们的 item 有一个唯一的 id，但是我们 spider 返回的多个 item 中包含有相同的 id:

```
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):

    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
```

## 启用一个 Item Pipeline 组件

为了启用一个 Item Pipeline 组件，你必须将它的类添加到 ITEM_PIPELINES 配置，就像下面这个例子:

```
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
```

分配给每个类的整型值，确定了他们运行的顺序，item 按数字从低到高的顺序，通过 pipeline，通常将这些数字定义在 0-1000 范围内。
