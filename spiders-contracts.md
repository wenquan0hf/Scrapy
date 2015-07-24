# Spiders Contracts

新版功能。

> 注解
> 
> 这是一个新引入(Scrapy 0.15)的特性，在后续的功能/API 更新中可能有所改变，查看 [release notes](http://scrapy-chs.readthedocs.org/zh_CN/latest/news.html#news) 来了解更新。

测试 spider 是一件挺烦人的事情，尤其是只能编写单元测试(unit test)没有其他办法时，就更恼人了。 Scrapy 通过合同(contract)的方式来提供了测试 spider 的集成方法。

您可以硬编码(hardcode)一个样例(sample)url， 设置多个条件来测试回调函数处理 repsponse 的结果，来测试 spider 的回调函数。 每个 contract 包含在文档字符串(docstring)里，以`@`开头。 查看下面的例子:

```
def parse(self, response):
    """ This function parses a sample response. Some contracts are mingled
    with this docstring.

    @url http://www.amazon.com/s?field-keywords=selfish+gene
    @returns items 1 16
    @returns requests 0 0
    @scrapes Title Author Year Price
    """
```

该回调函数使用了三个内置的 contract 来测试:

#### class scrapy.contracts.default.UrlContract

该 constract(@url)设置了用于检查 spider 的其他 constract 状态的样例 url。该 contract 是必须的，所有缺失该 contract 的回调函数在测试时将会被忽略:

```
@url url
```

#### class scrapy.contracts.default.ReturnsContract

该 contract(@returns)设置 spider 返回的 items 和 requests 的上界和下界。上界是可选的:

```
@returns item(s)|request(s) [min [max]]
```

#### class scrapy.contracts.default.ScrapesContract

该 contract(@scrapes)检查回调函数返回的所有 item 是否有特定的 fields:

```
@scrapes field_1 field_2 ...
```

使用 `check` 命令来运行 contract 检查。

## 自定义 Contracts

如果您想要比内置 scrapy contract 更为强大的功能，可以在您的项目里创建并设置您自己的 contract，并使用 `SPIDER_CONTRACTS` 设置来加载:

```
SPIDER_CONTRACTS = {
    'myproject.contracts.ResponseCheck': 10,
    'myproject.contracts.ItemValidate': 10,
}
```

每个 contract 必须继承 `scrapy.contracts.Contract` 并覆盖下列三个方法:

#### class scrapy.contracts.Contract(method, *args)
**参数:**    

- method (function) – contract 所关联的回调函数
- args (list) – 传入 docstring 的(以空格区分的)argument 列表(list)

##### adjust_request_args(args)

接收一个`字典(dict)`作为参数。该参数包含了所有 `Request` 对象 参数的默认值。该方法必须返回相同或修改过的字典。

##### pre_process(response)

该函数在 sample request 接收到 response 后，传送给回调函数前被调用，运行测试。

##### post_process(output)

该函数处理回调函数的输出。迭代器(Iterators)在传输给该函数前会被列表化(listified)。

该样例 contract 在 response 接收时检查了是否有自定义 header。 在失败时 Raise `scrapy.exceptions.ContractFaild` 来展现错误:

```
from scrapy.contracts import Contract
from scrapy.exceptions import ContractFail

class HasHeaderContract(Contract):
    """ Demo contract which checks the presence of a custom header
        @has_header X-CustomHeader
    """

    name = 'has_header'

    def pre_process(self, response):
        for header in self.args:
            if header not in response.headers:
                raise ContractFail('X-CustomHeader not present')
```