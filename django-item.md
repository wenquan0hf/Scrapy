# DjangoItem

`DjangoItem` 是一个 item 的类，其从 Django 模型中获取字段(field)定义。 您可以简单地创建一个 `DjangoItem` 并指定其关联的 `Django` 模型。

除了获得您 item 中定义的字段外， DjangoItem 提供了创建并获得一个具有 item 数据的 Django 模型实例(Django model instance)的方法。

## 使用 DjangoItem

DjangoItem 使用方法与 Django 中的 ModelForms 类似。您创建一个子类，并定义其 django_model 属性。这样，您就可以得到一个字段与 Django 模型字段(model field)一一对应的 item 了。

另外，您可以定义模型中没有的字段，甚至是覆盖模型中已经定义的字段。

让我们来看个例子:

创造一个 Django 模型:

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=255)
    age = models.IntegerField()
```

定义一个基本的 `DjangoItem`:

```
from scrapy.contrib.djangoitem import DjangoItem

class PersonItem(DjangoItem):
    django_model = Person
```

`DjangoItem` 的使用方法和 `Item` 类似:

```
>>> p = PersonItem()
>>> p['name'] = 'John'
>>> p['age'] = '22'
```

要从 item 中获取 Django 模型，调用 `DjangoItem` 中额外的方法 `save()`:

```
>>> person = p.save()
>>> person.name
'John'
>>> person.age
'22'
>>> person.id
1
```

当我们调用 `save()`时，模型已经保存了。我们可以在调用时带上 `commit=False` 来避免保存， 并获取到一个未保存的模型:

```
>>> person = p.save(commit=False)
>>> person.name
'John'
>>> person.age
'22'
>>> person.id
None
```

正如之前所说的，我们可以在 item 中加入字段:

```
import scrapy
from scrapy.contrib.djangoitem import DjangoItem

class PersonItem(DjangoItem):
    django_model = Person
    sex = scrapy.Field()
```

```
>>> p = PersonItem()
>>> p['name'] = 'John'
>>> p['age'] = '22'
>>> p['sex'] = 'M'
```

> 注解
> 
> 当执行 save()时添加到 item 的字段不会有作用(taken into account)。

并且我们可以覆盖模型中的字段:

```
class PersonItem(DjangoItem):
    django_model = Person
    name = scrapy.Field(default='No Name')
```

这在提供字段属性时十分有用，例如您项目中使用的默认或者其他属性一样。

## DjangoItem 注意事项

DjangoItem 提供了在 Scrapy 项目中集成 DjangoItem 的简便方法，不过需要注意的是，如果在 Scrapy 中爬取大量(百万级)的 item 时，Django ORM 扩展得并不是很好(not scale well)。这是因为关系型后端对于一个密集型(intensive)应用(例如 web 爬虫)并不是一个很好的选择，尤其是具有大量的索引的数据库。

## 配置 Django 的设置

在 Django 应用之外使用 Django 模型(model)，您需要设置 `DJANGO_SETTINGS_MODULE` 环境变量以及 –大多数情况下– 修改 `PYTHONPATH` 环境变量来导入设置模块。

完成这个配置有很多方法，具体选择取决您的情况及偏好。 下面详细给出了完成这个配置的最简单方法。

假设您项目的名称为 `mysite`，位于`/home/projects/mysite` 且用 `Person` 模型创建了一个应用 `myapp`。 这意味着您的目录结构类似于:

```
/home/projects/mysite
├── manage.py
├── myapp
│   ├── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

接着您需要将`/home/projects/mysite` 加入到 `PYTHONPATH` 环境变量中并将 `mysite.settings` 设置为 `DJANGO_SETTINGS_MODULE` 环境变量。 这可以在 Scrapy 设置文件中添加下列代码:

```
import sys
sys.path.append('/home/projects/mysite')

import os
os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'
```

注意，由于我们在 python 运行环境中，所以我们修改 `sys.path` 变量而不是 `PYTHONPATH` 环境变量。 如果所有设置正确，您应该可以运行 `scrapy shell` 命令并且导入 `Person` 模型(例如 `from myapp.models import Person`)。
