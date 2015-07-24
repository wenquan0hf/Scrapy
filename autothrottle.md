# 自动限速(AutoThrottle)扩展

该扩展能根据 Scrapy 服务器及您爬取的网站的负载自动限制爬取速度。

## 设计目标

1. 更友好的对待网站，而不使用默认的下载延迟 0。
2. 自动调整 scrapy 来优化下载速度，使得用户不用调节下载延迟及并发请求数来找到优化的值。 用户只需指定允许的最大并发请求数，剩下的都交给扩展来完成。

## 扩展是如何实现的

在 Scrapy 中，下载延迟是通过计算建立 TCP 连接到接收到 HTTP 包头(header)之间的时间来测量的。

注意，由于 Scrapy 可能在忙着处理 spider 的回调函数或者无法下载，因此在合作的多任务环境下准确测量这些延迟是十分苦难的。 不过，这些延迟仍然是对 Scrapy(甚至是服务器)繁忙程度的合理测量，而这扩展就是以此为前提进行编写的。

## 限速算法

算法根据以下规则调整下载延迟及并发数:

1. spider 永远以 1 并发请求数及 `AUTOTHROTTLE_START_DELAY` 中指定的下载延迟启动。
2. 当接收到回复时，下载延迟会调整到该回复的延迟与之前下载延迟之间的平均值。

> 注解
> 
> AutoThrottle 扩展尊重标准 Scrapy 设置中的并发数及延迟。这意味着其永远不会设置一个比 `DOWNLOAD_DELAY` 更低的下载延迟或者比 `CONCURRENT_REQUESTS_PER_DOMAIN` 更高的并发数 (或 `CONCURRENT_REQUESTS_PER_IP`，取决于您使用哪一个)。

## 设置

下面是控制 AutoThrottle 扩展的设置:

- AUTOTHROTTLE_ENABLED
- AUTOTHROTTLE_START_DELAY
- AUTOTHROTTLE_MAX_DELAY
- AUTOTHROTTLE_DEBUG
- CONCURRENT_REQUESTS_PER_DOMAIN
- CONCURRENT_REQUESTS_PER_IP
- DOWNLOAD_DELAY

更多内容请参考[限速算法](http://scrapy-chs.readthedocs.org/zh_CN/latest/topics/autothrottle.html#autothrottle-algorithm)。

### AUTOTHROTTLE_ENABLED

默认:`False`

启用 AutoThrottle 扩展。

### AUTOTHROTTLE_START_DELAY

默认:`5.0`

初始下载延迟(单位:秒)。

### AUTOTHROTTLE_MAX_DELAY

默认:`60.0`

在高延迟情况下最大的下载延迟(单位秒)。

### AUTOTHROTTLE_DEBUG

默认:`False`

起用 AutoThrottle 调试(debug)模式，展示每个接收到的 response。您可以通过此来查看限速参数是如何实时被调整的。
