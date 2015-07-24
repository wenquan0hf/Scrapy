# 借助 Firefox 来爬取

这里介绍一些使用 Firefox 进行爬取的点子及建议，以及一些帮助爬取的 Firefox 实用插件。

## 在浏览器中检查 DOM 的注意事项

Firefox 插件操作的是活动的浏览器 DOM(live browser DOM)，这意味着当您检查网页源码的时候， 其已经不是原始的 HTML，而是经过浏览器清理并执行一些 Javascript 代码后的结果。 Firefox 是个典型的例子，其会在 table 中添加 <\tbody> 元素。而 Scrapy 相反，其并不修改原始的 HTML，因此如果在 XPath 表达式中使用 <\tbody>，您将获取不到任何数据。

所以，当 XPath 配合 Firefox 使用时您需要记住以下几点:

- 当检查 DOM 来查找 Scrapy 使用的 XPath 时，禁用 Firefox 的 Javascrpit。
- 永远不要用完整的 XPath 路径。使用相对及基于属性(例如 `id`，`class`，`width` 等)的路径 或者具有区别性的特性例如 `contains(@href, 'image')`。
- 永远不要在 XPath 表达式中加入`<tbody>`元素，除非您知道您在做什么

## 对爬取有帮助的实用 Firefox 插件

### Firebug

[Firebug](http://getfirebug.com/) 是一个在 web 开发者间很著名的工具，其对抓取也十分有用。 尤其是[检查元素(Inspect Element)](http://www.youtube.com/watch?v=-pT_pDe54aA)特性对构建抓取数据的 XPath 十分方便。 当移动鼠标在页面元素时，您能查看相应元素的 HTML 源码。

查看`使用 Firebug 进行爬取`，了解如何配合 Scrapy 使用 Firebug 的详细教程。

### XPather

XPather 能让你在页面上直接测试 XPath 表达式。

### XPath Checker

XPath Checker 是另一个用于测试 XPath 表达式的 Firefox 插件。

### Tamper Data

Tamper Data 是一个允许您查看及修改 Firefox 发送的 header 的插件。Firebug 能查看 HTTP header，但无法修改。

### Firecookie

Firecookie 使得查看及管理 cookie 变得简单。您可以使用这个插件来创建新的 cookie，删除存在的 cookie，查看当前站点的 cookie，管理 cookie 的权限及其他功能。
