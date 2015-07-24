# 发送 email

虽然 Python 通过 [smtplib](http://docs.python.org/library/smtplib.html) 库使得发送 email 变得很简单，Scrapy 仍然提供了自己的实现。 该功能十分易用，同时由于采用了 [Twisted 非阻塞式(non-blocking)IO](http://twistedmatrix.com/documents/current/core/howto/defer-intro.html)，其避免了对爬虫的非阻塞式 IO 的影响。 另外，其也提供了简单的 API 来发送附件。 通过一些 settings 设置，您可以很简单的进行配置。

## 简单例子

有两种方法可以创建邮件发送器(mail sender)。您可以通过标准构造器(constructor)创建:

```
from scrapy.mail import MailSender
mailer = MailSender()
```

或者您可以传递一个 Scrapy 设置对象，其会参考 settings:

```
mailer = MailSender.from_settings(settings)
```

这是如何来发送邮件了(不包括附件):

```
mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])
```

## MailSender 类参考手册

在 Scrapy 中发送 email 推荐使用 MailSender。其同框架中其他的部分一样，使用了 [Twisted 非阻塞式(non-blocking)IO](http://twistedmatrix.com/documents/current/core/howto/defer-intro.html)。

#### class scrapy.mail.MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)

**参数:**    

- smtphost (str) – 发送 email 的 SMTP 主机(host)。如果忽略，则使用 MAIL_HOST 。
- mailfrom (str) – 用于发送 email 的地址(address)(填入 From:) 。如果忽略，则使用 MAIL_FROM 。
- smtpuser – SMTP 用户。如果忽略,则使用 MAIL_USER 。如果未给定，则将不会进行 SMTP 认证(authentication)。
- smtppass (str) – SMTP 认证的密码
- smtpport (int) – SMTP 连接的短裤
- smtptls – 强制使用 STARTTLS
- smtpssl (boolean) – 强制使用 SSL 连接

##### classmethod from_settings(settings)

使用 Scrapy 设置对象来初始化对象。其会参考`这些 Scrapy 设置`。

**参数:**    

settings (scrapy.settings.Settings object) – the e-mail recipients

##### send(to, subject, body, cc=None, attachs=(), mimetype='text/plain')

发送 email 到给定的接收者。

**参数:**    

- to (list) – email 接收
- subject (str) – email 内容
- cc (list) – 抄送的人
- body (str) – email 的内容
- attachs (iterable) – 可迭代的元组 (attach_name, mimetype, file_object)
attach_name 是一个在 email 的附件中显示的名字的字符串，mimetype 是附件的 mime 类型， file\_object 是包含附件内容的可读的文件对象。
- mimetype (str) – email 的 mime 类型

## Mail 设置

这些设置定义了 MailSender 构造器的默认值。其使得在您不编写任何一行代码的情况下，为您的项目配置实现 email 通知的功能。

### MAIL_FROM

默认值: `'scrapy@localhost'`

用于发送 email 的地址(address)(填入 `From`:) 。

### MAIL_HOST

默认值:`'localhost'`

发送 email 的 SMTP 主机(host)。

### MAIL_PORT

默认值:`25`

发用邮件的 SMTP 端口。

### MAIL_USER

默认值:`None`

SMTP 用户。如果未给定，则将不会进行 SMTP 认证(authentication)。

### MAIL_PASS

默认值:`None`

用于 SMTP 认证，与 `MAIL_USER` 配套的密码。

### MAIL_TLS

默认值:`False`

强制使用 STARTTLS。STARTTLS 能使得在已经存在的不安全连接上，通过使用 SSL/TLS 来实现安全连接。

### MAIL_SSL

默认值: `False`

强制使用 SSL 加密连接。
