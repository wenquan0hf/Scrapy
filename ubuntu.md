# Ubuntu 软件包

新版功能。

[Scrapinghub](http://scrapinghub.com/) 发布的 apt-get 可获取版本通常比 [Ubuntu](https://github.com/scrapy/scrapy) 里更新，并且在比 Github 仓库 (master & stable branches)稳定的同时还包括了最新的漏洞修复。

用法:

- 把 Scrapy 签名的 GPG 密钥添加到 APT 的钥匙环中:  

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 627220E7
```  

- 执行如下命令，创建/etc/apt/sources.list.d/scrapy.list 文件:

```
echo 'deb http://archive.scrapy.org/ubuntu scrapy main' | sudo tee /etc/apt/sources.list.d/scrapy.list
```

- 更新包列表并安装 scrapy-0.25:

```
sudo apt-get update && sudo apt-get install scrapy-0.25
```

> 注解
> 
> 如果你要升级 Scrapy，请重复步骤 3。
  
> 警告
> 
> debian 官方源提供的 python-scrapy 是一个非常老的版本且不再获得 Scrapy 团队支持。