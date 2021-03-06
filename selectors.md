# 选择器(Selectors)

当抓取网页时，你做的最常见的任务是从 HTML 源码中提取数据。现有的一些库可以达到这个目的：

- [BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/) 是在程序员间非常流行的网页分析库，它基于 HTML 代码的结构来构造一个 Python 对象， 对不良标记的处理也非常合理，但它有一个缺点：慢。
- [lxml](http://lxml.de/) 是一个基于 [ElementTree](http://docs.python.org/library/xml.etree.elementtree.html)(不是 Python 标准库的一部分)的 python 化的 XML 解析库(也可以解析 HTML)。

Scrapy 提取数据有自己的一套机制。它们被称作选择器(seletors)，因为他们通过特定的 [XPath](http://www.w3.org/TR/xpath) 或者 [CSS](http://www.w3.org/TR/selectors) 表达式来“选择” HTML 文件中的某个部分。

[XPath](http://www.w3.org/TR/xpath) 是一门用来在 XML 文件中选择节点的语言，也可以用在 HTML 上。[CSS](http://www.w3.org/TR/selectors) 是一门将 HTML 文档样式化的语言。选择器由它定义，并与特定的 HTML 元素的样式相关连。

Scrapy 选择器构建于 [lxml]([lxml](http://lxml.de/)) 库之上，这意味着它们在速度和解析准确性上非常相似。

本页面解释了选择器如何工作，并描述了相应的 API。不同于 [lxml]([lxml](http://lxml.de/)) API 的臃肿，该 API 短小而简洁。这是因为 [lxml]([lxml](http://lxml.de/)) 库除了用来选择标记化文档外，还可以用到许多任务上。

选择器 API 的完全参考详见 Selector reference

## 使用选择器(selectors)

### 构造选择器(selectors)

Scrapy selector 是以 文字(text) 或 `TextResponse` 构造的 `Selector` 实例。 其根据输入的类型自动选择最优的分析方法(XML vs HTML):

```
>>> from scrapy.selector import Selector
>>> from scrapy.http import HtmlResponse
```

以文字构造:

```
>>> body = '<html><body><span>good</span></body></html>'
>>> Selector(text=body).xpath('//span/text()').extract()
[u'good']
```

以 response 构造:

```
>>> response = HtmlResponse(url='http://example.com', body=body)
>>> Selector(response=response).xpath('//span/text()').extract()
[u'good']
```

为了方便起见，response 对象以.selector 属性提供了一个 selector，您可以随时使用该快捷方法:

```
>>> response.selector.xpath('//span/text()').extract()
[u'good']
```

### 使用选择器(selectors)

我们将使用 Scrapy shell (提供交互测试)和位于 Scrapy 文档服务器的一个样例页面，来解释如何使用选择器：

[http://doc.scrapy.org/en/latest/_static/selectors-sample1.html](http://doc.scrapy.org/en/latest/_static/selectors-sample1.html)

这里是它的 HTML 源码:

```
<html>
 <head>
  <base href='http://example.com/' />
  <title>Example website</title>
 </head>
 <body>
  <div id='images'>
   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
   <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
  </div>
 </body>
</html>
```

首先，我们打开 shell:

```
scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html
```

接着，当 shell 载入后，您将获得名为 `response` 的 shell 变量，其为响应的 response，并且在其 `response.selector` 属性上绑定了一个 selector。

因为我们处理的是 HTML，选择器将自动使用 HTML 语法分析。

那么，通过查看 [HTML code](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/selectors.html#topics-selectors-htmlcode) 该页面的源码，我们构建一个 XPath 来选择 title 标签内的文字:

```
>>> response.selector.xpath('//title/text()')
[<Selector (text) xpath=//title/text()>]
```

由于在 response 中使用 XPath、CSS 查询十分普遍，因此，Scrapy 提供了两个实用的快捷方式: response.xpath()及 response.css():

```
>>> response.xpath('//title/text()')
[<Selector (text) xpath=//title/text()>]
>>> response.css('title::text')
[<Selector (text) xpath=//title/text()>]
```

如你所见，.xpath()及.css()方法返回一个类 SelectorList 的实例，它是一个新选择器的列表。这个 API 可以用来快速的提取嵌套数据。

为了提取真实的原文数据，你需要调用.extract() 方法如下:

```
>>> response.xpath('//title/text()').extract()
[u'Example website']
```

注意 CSS 选择器可以使用 CSS3 伪元素(pseudo-elements)来选择文字或者属性节点:

```
>>> response.css('title::text').extract()
[u'Example website']
```

现在我们将得到根 URL(base URL)和一些图片链接:

```
>>> response.xpath('//base/@href').extract()
[u'http://example.com/']

>>> response.css('base::attr(href)').extract()
[u'http://example.com/']

>>> response.xpath('//a[contains(@href, "image")]/@href').extract()
[u'image1.html',
 u'image2.html',
 u'image3.html',
 u'image4.html',
 u'image5.html']

>>> response.css('a[href*=image]::attr(href)').extract()
[u'image1.html',
 u'image2.html',
 u'image3.html',
 u'image4.html',
 u'image5.html']

>>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
[u'image1_thumb.jpg',
 u'image2_thumb.jpg',
 u'image3_thumb.jpg',
 u'image4_thumb.jpg',
 u'image5_thumb.jpg']

>>> response.css('a[href*=image] img::attr(src)').extract()
[u'image1_thumb.jpg',
 u'image2_thumb.jpg',
 u'image3_thumb.jpg',
 u'image4_thumb.jpg',
 u'image5_thumb.jpg']
```

### 嵌套选择器(selectors)

选择器方法( .xpath() or .css() )返回相同类型的选择器列表，因此你也可以对这些选择器调用选择器方法。下面是一个例子:

```
>>> links = response.xpath('//a[contains(@href, "image")]')
>>> links.extract()
[u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
 u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
 u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
 u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
 u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']

>>> for index, link in enumerate(links):
        args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
        print 'Link number %d points to url %s and image %s' % args

Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']
```

### 结合正则表达式使用选择器(selectors)

Selector 也有一个.re()方法，用来通过正则表达式来提取数据。然而，不同于使用.xpath()或者.css()方法，.re()方法返回 unicode 字符串的列表。所以你无法构造嵌套式的.re()调用。

下面是一个例子，从上面的 HTML code 中提取图像名字:

```
>>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
[u'My image 1',
 u'My image 2',
 u'My image 3',
 u'My image 4',
 u'My image 5']
```

### 使用相对 XPaths

记住如果你使用嵌套的选择器，并使用起始为`/`的 XPath，那么该 XPath 将对文档使用绝对路径，而且对于你调用的 `Selector` 不是相对路径。

比如，假设你想提取在`<div>`元素中的所有`<p>`元素。首先，你将先得到所有的`<div>`元素:

```
>>> divs = response.xpath('//div')
```

开始时，你可能会尝试使用下面的错误的方法，因为它其实是从整篇文档中，而不仅仅是从那些`<div>` 元素内部提取所有的`<p>`元素:

```
>>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
...     print p.extract()
```

下面是比较合适的处理方法(注意`.//p` XPath 的点前缀):

```
>>> for p in divs.xpath('.//p'):  # extracts all <p> inside
...     print p.extract()
```

另一种常见的情况将是提取所有直系`<p>`的结果:

```
>>> for p in divs.xpath('p'):
...     print p.extract()
```

更多关于相对 XPaths 的细节详见 XPath 说明中的 [Location Paths](http://www.w3.org/TR/xpath#location-paths) 部分。

### 使用 EXSLT 扩展

因建于 [lxml](http://lxml.de/) 之上，Scrapy 选择器也支持一些 [EXSLT](http://www.exslt.org/) 扩展，可以在 XPath 表达式中使用这些预先制定的命名空间：

<table>
<tr>
<td>前缀</td>
<td>命名空间</td>
<td> 用途</td>
</tr>
<tr>
<td>re</td>
<td> http://exslt.org/regular-expressions</td>
<td> <a href="http://exslt.org/regular-expressions">正则表达式</a></td>
</tr>
<tr>
<td>set</td>
<td>http://exslt.org/sets</td>
<td> <a href="http://exslt.org/sets">集合操作</a></td>
</tr>
</table>
     
### 正则表达式

例如在 XPath 的 `starts-with()`或 `contains()`无法满足需求时，`test()`函数可以非常有用。

例如在列表中选择有”class”元素且结尾为一个数字的链接:

```
>>> from scrapy import Selector
>>> doc = """
... <div>
...     <ul>
...         <li class="item-0"><a href="link1.html">first item</a></li>
...         <li class="item-1"><a href="link2.html">second item</a></li>
...         <li class="item-inactive"><a href="link3.html">third item</a></li>
...         <li class="item-1"><a href="link4.html">fourth item</a></li>
...         <li class="item-0"><a href="link5.html">fifth item</a></li>
...     </ul>
... </div>
... """
>>> sel = Selector(text=doc, type="html")
>>> sel.xpath('//li//@href').extract()
[u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
>>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
[u'link1.html', u'link2.html', u'link4.html', u'link5.html']
>>>
```

> 警告
> 
> C 语言库 `libxslt` 不原生支持 EXSLT 正则表达式，因此 [lxml](http://lxml.de/) 在实现时使用了 Python `re` 模块的钩子。 因此，在 XPath 表达式中使用 regexp 函数可能会牺牲少量的性能。

### 集合操作

集合操作可以方便地用于在提取文字元素前从文档树中去除一些部分。

例如使用 itemscopes 组和对应的 itemprops 来提取微数据(来自 http://schema.org/Product 的样本内容):

```
>>> doc = """
... <div itemscope itemtype="http://schema.org/Product">
...   <span itemprop="name">Kenmore White 17" Microwave</span>
...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
...   <div itemprop="aggregateRating"
...     itemscope itemtype="http://schema.org/AggregateRating">
...    Rated <span itemprop="ratingValue">3.5</span>/5
...    based on <span itemprop="reviewCount">11</span> customer reviews
...   </div>
...
...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
...     <span itemprop="price">$55.00</span>
...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
...   </div>
...
...   Product description:
...   <span itemprop="description">0.7 cubic feet countertop microwave.
...   Has six preset cooking categories and convenience features like
...   Add-A-Minute and Child Lock.</span>
...
...   Customer reviews:
...
...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
...     <span itemprop="name">Not a happy camper</span> -
...     by <span itemprop="author">Ellie</span>,
...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
...       <meta itemprop="worstRating" content = "1">
...       <span itemprop="ratingValue">1</span>/
...       <span itemprop="bestRating">5</span>stars
...     </div>
...     <span itemprop="description">The lamp burned out and now I have to replace
...     it. </span>
...   </div>
...
...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
...     <span itemprop="name">Value purchase</span> -
...     by <span itemprop="author">Lucas</span>,
...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
...       <meta itemprop="worstRating" content = "1"/>
...       <span itemprop="ratingValue">4</span>/
...       <span itemprop="bestRating">5</span>stars
...     </div>
...     <span itemprop="description">Great microwave for the price. It is small and
...     fits in my apartment.</span>
...   </div>
...   ...
... </div>
... """
>>>
>>> for scope in sel.xpath('//div[@itemscope]'):
...     print "current scope:", scope.xpath('@itemtype').extract()
...     props = scope.xpath('''
...                 set:difference(./descendant::*/@itemprop,
...                                .//*[@itemscope]/*/@itemprop)''')
...     print "    properties:", props.extract()
...     print
...

current scope: [u'http://schema.org/Product']
    properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

current scope: [u'http://schema.org/AggregateRating']
    properties: [u'ratingValue', u'reviewCount']

current scope: [u'http://schema.org/Offer']
    properties: [u'price', u'availability']

current scope: [u'http://schema.org/Review']
    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

current scope: [u'http://schema.org/Rating']
    properties: [u'worstRating', u'ratingValue', u'bestRating']

current scope: [u'http://schema.org/Review']
    properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

current scope: [u'http://schema.org/Rating']
    properties: [u'worstRating', u'ratingValue', u'bestRating']

>>>
```

在这里，我们首先在 `itemscope` 元素上迭代，对于其中的每一个元素，我们寻找所有的 `itemprops` 元素，并排除那些本身在另一个 `itemscope` 内的元素。

### Some XPath tips

Here are some tips that you may find useful when using XPath with Scrapy selectors, based on [this post from ScrapingHub’s blog](http://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches/). If you are not much familiar with XPath yet, you may want to take a look first at this [XPath tutorial](http://www.zvon.org/comp/r/tut-XPath_1.html).

#### Using text nodes in a condition

When you need to use the text content as argument to a [XPath string function](http://www.w3.org/TR/xpath/#section-String-Functions), avoid using .//text() and use just `.` instead.

This is because the expression `.//text()` yields a collection of text elements – a node-set. And when a node-set is converted to a string, which happens when it is passed as argument to a string function like `contains()` or `starts-with()`, it results in the text for the first element only.

Example:

```
>>> from scrapy import Selector
>>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')
```

Converting a node-set to string:

```
>>> sel.xpath('//a//text()').extract() # take a peek at the node-set
[u'Click here to go to the ', u'Next Page']
>>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
[u'Click here to go to the ']
```

A node converted to a string, however, puts together the text of itself plus of all its descendants:

```
>>> sel.xpath("//a[1]").extract() # select the first node
[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
>>> sel.xpath("string(//a[1])").extract() # convert it to string
[u'Click here to go to the Next Page']
```

**So, using the .//text() node-set won’t select anything in this case::**

```
>>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
[]
```

But using the . to mean the node, works:

```
>>> sel.xpath("//a[contains(., 'Next Page')]").extract()
[u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
```

#### Beware the difference between //node[1] and (//node)[1]

`//node[1]` selects all the nodes occurring first under their respective parents.

`(//node)[1]` selects all the nodes in the document, and then gets only the first of them.

Example:

```
>>> from scrapy import Selector
>>> sel = Selector(text="""
....:     <ul class="list">
....:         <li>1</li>
....:         <li>2</li>
....:         <li>3</li>
....:     </ul>
....:     <ul class="list">
....:         <li>4</li>
....:         <li>5</li>
....:         <li>6</li>
....:     </ul>""")
>>> xp = lambda x: sel.xpath(x).extract()
```

This gets all first <li\> elements under whatever it is its parent:

```
>>> xp("//li[1]")
[u'<li>1</li>', u'<li>4</li>']
```

And this gets the first <li\> element in the whole document:

```
>>> xp("(//li)[1]")
[u'<li>1</li>']
```

This gets all first <li\> elements under an <ul\> parent:

```
>>> xp("//ul/li[1]")
[u'<li>1</li>', u'<li>4</li>']
```

And this gets the first <li\> element under an <ul\> parent in the whole document:

```
>>> xp("(//ul/li)[1]")
[u'<li>1</li>']
```

#### When querying by class, consider using CSS

Because an element can contain multiple CSS classes, the XPath way to select elements by class is the rather verbose:

```
*[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]
```

If you use `@class='someclass'` you may end up missing elements that have other classes, and if you just use `contains(@class, 'someclass')` to make up for that you may end up with more elements that you want, if they have a different class name that shares the string `someclass`.

As it turns out, Scrapy selectors allow you to chain selectors, so most of the time you can just select by class using CSS and then switch to XPath when needed:

```
>>> from scrapy import Selector
>>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
>>> sel.css('.shout').xpath('./time/@datetime').extract()
[u'2014-07-23 19:00']
```

This is cleaner than using the verbose XPath trick shown above. Just remember to use the.in the XPath expressions that will follow.

## 内建选择器的参考

#### class scrapy.selector.Selector(response=None, text=None, type=None)

Selector 的实例是对选择某些内容响应的封装。

response 是 HtmlResponse 或 XmlResponse 的一个对象，将被用来选择和提取数据。

text 是在 response 不可用时的一个 unicode 字符串或 utf-8 编码的文字。将 text 和 response 一起使用是未定义行为。

type 定义了选择器类型，可以是 "html"，"xml" or None (默认).

如果 type 是 None ，选择器会根据 response 类型(参见下面)自动选择最佳的类型，或者在和 text 一起使用时，默认为 "html" 。

如果 type 是 None ，并传递了一个 response ，选择器类型将从 response 类型中推导如下：

- "html" for HtmlResponse type
- "xml" for XmlResponse type
- "html" for anything else

其他情况下，如果设定了 type ，选择器类型将被强制设定，而不进行检测。

##### xpath(query)

寻找可以匹配 xpath query 的节点，并返回 `SelectorList` 的一个实例结果，单一化其所有元素。列表元素也实现了 `Selector` 的接口。

query 是包含 XPATH 查询请求的字符串。

> 注解
> 
> 为了方便起见，该方法也可以通过 `response.xpath()`调用

##### css(query)

应用给定的 CSS 选择器，返回 SelectorList 的一个实例。

`query` 是一个包含 CSS 选择器的字符串。

在后台，通过 cssselect 库和运行 .xpath() 方法，CSS 查询会被转换为 XPath 查询。

> 注解
> 
> 为了方便起见，该方法也可以通过 response.css() 调用

##### extract()

串行化并将匹配到的节点返回一个 unicode 字符串列表。 结尾是编码内容的百分比。

##### re(regex)

应用给定的 regex，并返回匹配到的 unicode 字符串列表。

`regex` 可以是一个已编译的正则表达式，也可以是一个将被 `re.compile(regex)`编译为正则表达式的字符串。

##### register_namespace(prefix, uri)

注册给定的命名空间，其将在 `Selector` 中使用。 不注册命名空间，你将无法从非标准命名空间中选择或提取数据。参见下面的例子。

##### remove_namespaces()

移除所有的命名空间，允许使用少量的命名空间 xpaths 遍历文档。参加下面的例子。

##### \_\_nonzero__()

如果选择了任意的真实文档，将返回 True ，否则返回 False 。 也就是说， Selector 的布尔值是通过它选择的内容确定的。

### SelectorList 对象

#### class scrapy.selector.SelectorList

`SelectorList` 类是内建 list 类的子类，提供了一些额外的方法。

##### xpath(query)

对列表中的每个元素调用.xpath()方法，返回结果为另一个单一化的 SelectorList。

query 和 `Selector.xpath()`中的参数相同。

##### css(query)

对列表中的各个元素调用.css() 方法，返回结果为另一个单一化的 SelectorList。

query 和 Selector.css() 中的参数相同。

##### extract()

对列表中的各个元素调用.extract()方法，返回结果为单一化的 unicode 字符串列表。

##### re()

对列表中的各个元素调用 .re() 方法，返回结果为单一化的 unicode 字符串列表。

##### \_\_nonzero__()
列表非空则返回 True，否则返回 False。

### 在 HTML 响应上的选择器样例

这里是一些 `Selector` 的样例，用来说明一些概念。在所有的例子中，我们假设已经有一个通过 `HtmlResponse` 对象实例化的 `Selector`，如下:

```
sel = Selector(html_response)
```

1. 从 HTML 响应主体中提取所有的<h1\>元素，返回:class:Selector 对象(即 `SelectorList` 的一个对象)的列表:

```
sel.xpath("//h1")
```

2. 从 HTML 响应主体上提取所有<h1\>元素的文字，返回一个 unicode 字符串的列表:

```
sel.xpath("//h1").extract()         # this includes the h1 tag
sel.xpath("//h1/text()").extract()  # this excludes the h1 tag
```

3. 在所有<p\>标签上迭代，打印它们的类属性:

```
for node in sel.xpath("//p"):
    print node.xpath("@class").extract()
```

### 在 XML 响应上的选择器样例

这里是一些样例，用来说明一些概念。在两个例子中，我们假设已经有一个通过 `XmlResponse` 对象实例化的 `Selector` ，如下:

```
sel = Selector(xml_response)
```

1. 从 XML 响应主体中选择所有的 <product> 元素，返回 Selector 对象(即 SelectorList 对象)的列表:

```
sel.xpath("//product")
```

2. 从 Google Base XML feed 中提取所有的价钱，这需要注册一个命名空间:

```
sel.register_namespace("g", "http://base.google.com/ns/1.0")
sel.xpath("//g:price").extract()
```

### 移除命名空间

在处理爬虫项目时，完全去掉命名空间而仅仅处理元素名字，写更多简单/实用的 XPath 会方便很多。你可以为此使用 `Selector.remove_namespaces()` 方法。

让我们来看一个例子，以 Github 博客的 atom 订阅来解释这个情况。

首先，我们使用想爬取的 url 来打开 shell:

```
$ scrapy shell https://github.com/blog.atom
```

一旦进入 shell，我们可以尝试选择所有的 <link\> 对象，可以看到没有结果(因为 Atom XML 命名空间混淆了这些节点):

```
>>> response.xpath("//link")
[]
```

但一旦我们调用 `Selector.remove_namespaces()`方法，所有的节点都可以直接通过他们的名字来访问:

```
>>> response.selector.remove_namespaces()
>>> response.xpath("//link")
[<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
 <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
 ...
```

如果你对为什么命名空间移除操作并不总是被调用，而需要手动调用有疑惑。这是因为存在如下两个原因，按照相关顺序如下：

1. 移除命名空间需要迭代并修改文件的所有节点，而这对于 Scrapy 爬取的所有文档操作需要一定的性能消耗
2. 会存在这样的情况，确实需要使用命名空间，但有些元素的名字与命名空间冲突。尽管这些情况非常少见。
