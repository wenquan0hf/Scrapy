# Requests and Responses

Scrapy 使用 `Request` 和 Response 对象爬取 web 站点。

一般来说，`Request` 对象在 spiders 中被生成并且最终传递到下载器(Downloader)，下载器对其进行处理并返回一个 `Response` 对象， `Response` 对象还会返回到生成 `Request` 的 spider 中。

所有 `Request` and `Response` 的子类都会实现一些在基类中非必要的 功能。它们会在 `Request` subclasses 和 Response subclasses 两部分进行详细的说明。

## Request 对象

#### class scrapy.http.Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback])

一个 `Request` 对象代表一个 HTTP 请求，一般来讲， HTTP 请求是由 Spider 产生并被 Downloader 处理进而生成一个 Response。

**参数:**
    
- **url** (*string*) – 请求的 URL
- **callback** (*callable) – the function that will be called with the response of this request (once its d*ownloaded) as its first parameter. For more information see [Passing additional data to callback functions](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/request-response.html#topics-request-response-ref-request-callback-arguments) below. If a Request doesn’t specify a callback, the spider’s parse() method will be used. Note that if exceptions are raised during processing, errback is called instead.
- **method** (*string*) – 此请求的 HTTP 方法。默认是 'GET'。
- **meta** (*dict*) – `Request.meta` 属性的初始值。 一旦此参数被设置， 通过参数传递的字典将会被浅拷贝。
- **body** (*str or unicode*) – request 体。如果传进的参数是 unicode 类型，将会被编码为 str 类型。如果 body 参数没有给定，那么将会存储一个空的 string 类型，不管 这个参数是什么类型的，最终存储的都会是 str 类型(永远不会是 unicode 或是 None)。
- **headers** (*dict)* – 请求头。字典值的类型可以是 strings (for single valued headers) 或是 lists (for multi-valued headers)。如果传进的值是 None ，那么 HTTP 头将不会被发送。
- **cookies** (*dict or list*) –

请求的 cookies。可以被设置成如下两种形式。

1. Using a dict:

```
request_with_cookies = Request(url="http://www.example.com",
                               cookies={'currency': 'USD', 'country': 'UY'})
```			
2. Using a list of dicts:
 
```
request_with_cookies = Request(url="http://www.example.com",
                               cookies=[{'name': 'currency',
                                        'value': 'USD',
                                        'domain': 'example.com',
                                        'path': '/currency'}])
```

The latter form allows for customizing the domain and path attributes of the cookie. This is only useful if the cookies are saved for later requests.

When some site returns cookies (in a response) those are stored in the cookies for that domain and will be sent again in future requests. That’s the typical behaviour of any regular web browser. However, if, for some reason, you want to avoid merging with existing cookies you can instruct Scrapy to do so by setting the `dont_merge_cookies` key to True in the `Request.meta`.

Example of request without merging cookies:

```
request_with_cookies = Request(url="http://www.example.com",
                               cookies={'currency': 'USD', 'country': 'UY'},
                               meta={'dont_merge_cookies': True})
```

For more info see [CookiesMiddleware](downloader-middleware.md).

- **encoding** (*string*) – the encoding of this request (defaults to `'utf-8'`). This encoding will be used to percent-encode the URL and to convert the body to `str` (if given as `unicode`).
- **priority** (*int*) – the priority of this request (defaults to `0`). The priority is used by the scheduler to define the order used to process requests. Requests with a higher priority value will execute earlier. Negative values are allowed in order to indicate relatively low-priority.
- **dont_filter** (*boolean*) – indicates that this request should not be filtered by the scheduler. This is used when you want to perform an identical request multiple times, to ignore the duplicates filter. Use it with care, or you will get into crawling loops. Default to `False`.
- **errback** (*callable*) – a function that will be called if any exception was raised while processing the request. This includes pages that failed with 404 HTTP errors and such. It receives a [Twisted Failure](http://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html) instance as first parameter.

##### url

A string containing the URL of this request. Keep in mind that this attribute contains the escaped URL, so it can differ from the URL passed in the constructor.

This attribute is read-only. To change the URL of a Request use `replace()`.

##### method

A string representing the HTTP method in the request. This is guaranteed to be uppercase. Example: `"GET"`, `"POST"`, `"PUT"`, etc

##### headers

A dictionary-like object which contains the request headers.

##### body

A str that contains the request body.

This attribute is read-only. To change the body of a Request use `replace()`.

##### meta

A dict that contains arbitrary metadata for this request. This dict is empty for new Requests, and is usually populated by different Scrapy components (extensions, middlewares, etc). So the data contained in this dict depends on the extensions you have enabled.

See [Request.meta special keys](requests-and-responses.md) for a list of special meta keys recognized by Scrapy.

This dict is shallow copied when the request is cloned using the `copy()` or `replace()` methods, and can also be accessed, in your spider, from the `response.meta` attribute.

##### copy()

Return a new Request which is a copy of this Request. See also:[ Passing additional data to callback functions](requests-and-responses.md).

##### replace([url, method, headers, body, cookies, meta, encoding, dont_filter, callback, errback])

Return a Request object with the same members, except for those members given new values by whichever keyword arguments are specified. The attribute Request.meta is copied by default (unless a new value is given in the meta argument). See also [ Passing additional data to callback functions](requests-and-responses.md).

### Passing additional data to callback functions

The callback of a request is a function that will be called when the response of that request is downloaded. The callback function will be called with the downloaded Response object as its first argument.

Example:

```
def parse_page1(self, response):
    return scrapy.Request("http://www.example.com/some_page.html",
                          callback=self.parse_page2)

def parse_page2(self, response):
    # this would log http://www.example.com/some_page.html
    self.log("Visited %s" % response.url)
```

In some cases you may be interested in passing arguments to those callback functions so you can receive the arguments later, in the second callback. You can use the `Request.meta` attribute for that.

Here’s an example of how to pass an item using this mechanism, to populate different fields from different pages:

```
def parse_page1(self, response):
    item = MyItem()
    item['main_url'] = response.url
    request = scrapy.Request("http://www.example.com/some_page.html",
                             callback=self.parse_page2)
    request.meta['item'] = item
    return request

def parse_page2(self, response):
    item = response.meta['item']
    item['other_url'] = response.url
    return item
```

## Request.meta special keys

The `Request.meta` attribute can contain any arbitrary data, but there are some special keys recognized by Scrapy and its built-in extensions.

Those are:

- dont_redirect
- dont_retry
- handle_httpstatus_list
- dont_merge_cookies (see cookies parameter of Request constructor)
- cookiejar
- redirect_urls
- bindaddress
- dont_obey_robotstxt
- download_timeout

#### bindaddress

The IP of the outgoing IP address to use for the performing the request.

#### download_timeout

The amount of time (in secs) that the downloader will wait before timing out. See also: `DOWNLOAD_TIMEOUT`.

## Request subclasses

Here is the list of built-in Request subclasses. You can also subclass it to implement your own custom functionality.

## FormRequest objects

The FormRequest class extends the base Request with functionality for dealing with HTML forms. It uses [lxml.html ](http://lxml.de/lxmlhtml.html#forms)forms to pre-populate form fields with form data from Response objects.

##### class scrapy.http.FormRequest(url[, formdata, ...])

The `FormRequest` class adds a new argument to the constructor. The remaining arguments are the same as for the Request class and are not documented here.

**参数:**    

formdata (dict or iterable of tuples) – is a dictionary (or iterable of (key, value) tuples) containing HTML Form data which will be url-encoded and assigned to the body of the request.

The `FormRequest` objects support the following class method in addition to the standard Request methods:

###### classmethod from_response(response[, formname=None, formnumber=0, formdata=None, formxpath=None, clickdata=None, dont_click=False, ...])

Returns a new FormRequest object with its form field values pre-populated with those found in the HTML <form> element contained in the given response. For an example see `使用 FormRequest.from_response()方法模拟用户登录`.

The policy is to automatically simulate a click, by default, on any form control that looks clickable, like a `<input type="submit">`. Even though this is quite convenient, and often the desired behaviour, sometimes it can cause problems which could be hard to debug. For example, when working with forms that are filled and/or submitted using javascript, the default `from_response()` behaviour may not be the most appropriate. To disable this behaviour you can set the dont_click argument to True. Also, if you want to change the control clicked (instead of disabling it) you can also use the `clickdata` argument.

**参数:**    

- **response** (*Response object*) – the response containing a HTML form which will be used to pre-populate the form fields
- **formname** (*string*) – if given, the form with name attribute set to this value will be used.
- **formxpath** (*string*) – if given, the first form that matches the xpath will be used.
- **formnumber** *(integer*) – the number of form to use, when the response contains multiple forms. The first one (and also the default) is 0.
- **formdata** *(dict*) – fields to override in the form data. If a field was already present in the response <form> element, its value is overridden by the one passed in this parameter.
- **clickdata** (*dict*) – attributes to lookup the control clicked. If it’s not given, the form data will be submitted simulating a click on the first clickable element. In addition to html attributes, the control can be identified by its zero-based index relative to other submittable inputs inside the form, via the nr attribute.
- **dont_click** (*boolean*) – If True, the form data will be submitted without clicking in any element.

The other parameters of this class method are passed directly to the `FormRequest` constructor.

0.10.3 新版功能:The `formname` parameter.

0.17 新版功能:The `formxpat parameter.

### Request usage examples

#### Using FormRequest to send data via HTTP POST

If you want to simulate a HTML Form POST in your spider and send a couple of key-value fields, you can return a `FormRequest` object (from your spider) like this:

```
return [FormRequest(url="http://www.example.com/post/action",
                    formdata={'name': 'John Doe', 'age': '27'},
                    callback=self.after_post)]
```

#### 使用 FormRequest.from_response()方法模拟用户登录

通常网站通过`<input type="hidden">`实现对某些表单字段（如数据或是登录界面中的认证令牌等）的预填充。使用 Scrapy 抓取网页时，如果想要预填充或重写像用户名、用户密码这些表单字段，可以使用 `FormRequest.from_response()`方法实现。下面是使用这种方法的爬虫例子:

```
import scrapy

class LoginSpider(scrapy.Spider):
    name = 'example.com'
    start_urls = ['http://www.example.com/users/login.php']

    def parse(self, response):
        return scrapy.FormRequest.from_response(
            response,
            formdata={'username': 'john', 'password': 'secret'},
            callback=self.after_login
        )

    def after_login(self, response):
        # check login succeed before going on
        if "authentication failed" in response.body:
            self.log("Login failed", level=log.ERROR)
            return

        # continue scraping with authenticated session...
```

### Response objects

#### class scrapy.http.Response(url[, status=200, headers, body, flags])

A `Response` object represents an HTTP response, which is usually downloaded (by the Downloader) and fed to the Spiders for processing.

**参数:**   
 
- **url** (*string*) – the URL of this response
- **headers** (*dict*) – the headers of this response. The dict values can be strings (for single valued headers) or lists (for multi-valued headers).
-** status** (*integer*) – the HTTP status of the response. Defaults to `200`.
- **body** (*str*) – the response body. It must be str, not unicode, unless you’re using a encoding-aware Response subclass, such as `TextResponse`.
- **meta** (*dict*) – the initial values for the `Response.meta` attribute. If given, the dict will be shallow copied.
- **flags** (*[list](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/api.html#scrapy.spidermanager.SpiderManager.list)*) – is a list containing the initial values for the `Response.flag`s attribute. If given, the list will be shallow copied.

##### url

A string containing the URL of the response.

This attribute is read-only. To change the URL of a Response use `replace()`.

##### status

An integer representing the HTTP status of the response. Example: `200`, `404`.

##### headers

A dictionary-like object which contains the response headers.

##### body

A str containing the body of this Response. Keep in mind that Response.body is always a str. If you want the unicode version use `TextResponse.body_as_unicode()` (only available in `TextResponse` and subclasses).

This attribute is read-only. To change the body of a Response use `replace()`.

##### request

The `Request` object that generated this response. This attribute is assigned in the Scrapy engine, after the response and the request have passed through all [Downloader Middlewares](downloader-middleware.md). In particular, this means that:

- HTTP redirections will cause the original request (to the URL before redirection) to be assigned to the redirected response (with the final URL after redirection).
- Response.request.url doesn’t always equal Response.url
- This attribute is only available in the spider code, and in the [Spider Middlewares](spider-middleware.md), but not in Downloader Middlewares (although you have the Request available there by other means) and handlers of the `response_downloaded` signal.

##### meta

A shortcut to the `Request.meta` attribute of the `Response.request` object (ie. `self.request.meta`).

Unlike the `Response.request` attribute, the `Response.meta` attribute is propagated along redirects and retries, so you will get the original `Request.meta` sent from your spider.

> 参见
> 
> Request.meta attribute

##### flags

A list that contains flags for this response. Flags are labels used for tagging Responses. For example: ‘cached’, ‘redirected‘, etc. And they’re shown on the string representation of the Response (\_\_str__ method) which is used by the engine for logging.

##### copy()

Returns a new Response which is a copy of this Response.

##### replace([url, status, headers, body, request, flags, cls])

Returns a Response object with the same members, except for those members given new values by whichever keyword arguments are specified. The attribute `Response.meta` is copied by default.

## Response subclasses

Here is the list of available built-in Response subclasses. You can also subclass the Response class to implement your own functionality.

### TextResponse objects

#### class scrapy.http.TextResponse(url[, encoding[, ...]])

`TextResponse` objects adds encoding capabilities to the base `Response` class, which is meant to be used only for binary data, such as images, sounds or any media file.

`TextResponse` objects support a new constructor argument, in addition to the base `Response` objects. The remaining functionality is the same as for the `Response` class and is not documented here.

**参数:**   

encoding (string) – is a string which contains the encoding to use for this response. If you create a `TextResponse` object with a unicode body, it will be encoded using this encoding (remember the body attribute is always a string). If encoding is None (default value), the encoding will be looked up in the response headers and body instead.

`TextResponse` objects support the following attributes in addition to the standard Response ones:

##### encoding

A string with the encoding of this response. The encoding is resolved by trying the following mechanisms, in order:

1. the encoding passed in the constructor encoding argument
2. the encoding declared in the Content-Type HTTP header. If this encoding is not valid (ie. unknown), it is ignored and the next resolution mechanism is tried.
3. the encoding declared in the response body. The TextResponse class doesn’t provide any special functionality for this. However, `the HtmlResponse` and `XmlResponse` classes do.
4. the encoding inferred by looking at the response body. This is the more fragile method but also the last one tried.

##### selector

A `Selector` instance using the response as target. The selector is lazily instantiated on first access.

`TextResponse` objects support the following methods in addition to the standard `Response` ones:

##### body_as_unicode()

Returns the body of the response as unicode. This is equivalent to:

```
response.body.decode(response.encoding)
```

But not equivalent to:

```
unicode(response.body)
```

Since, in the latter case, you would be using you system default encoding (typically ascii) to convert the body to unicode, instead of the response encoding.

##### xpath(query)

A shortcut to `TextResponse.selector.xpath(query)`:

```
response.xpath('//p')
```

##### css(query)

A shortcut to `TextResponse.selector.css(query)`:

```
response.css('p')
```

### HtmlResponse objects

#### class scrapy.http.HtmlResponse(url[, ...])

The `HtmlResponse` class is a subclass of `TextResponse` which adds encoding auto-discovering support by looking into the HTML [meta http-equiv](http://www.w3schools.com/TAGS/att_meta_http_equiv.asp) attribute. See `TextResponse.encoding`.

### XmlResponse objects

#### class scrapy.http.XmlResponse(url[, ...])

The `XmlResponse` class is a subclass of `TextResponse` which adds encoding auto-discovering support by looking into the XML declaration line. See `TextResponse.encoding`.
