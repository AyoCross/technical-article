接触django已经很长时间了，但是使用QuerySet查询集的方式一直比较低端，只会使用filter/Q函数/exclude等方式来查询，数据量比较小的时候还可以，但是如果数据量很大，而且查询比较复杂，那么如果还是使用多个filter进行查询效率就会很低。就趁着清明放假的时间，跑来公司干点私活。输出成这篇文档，一是加深印象，提高熟练度；二是分享出来，造福大家~

提高查询数据库效率的方案有两种：

第一种，是使用原生的SQL语句来进行查询，这样的优点在于能够完全按照开发者的意图来执行，效率会很高，但是缺点也很明显：1.开发者需要非常熟悉SQL语句，加大开发者的工作量，同时，夹杂着SQL的项目也不利于以后程序的维护，增大程序的耦合度。2.若查询条件是动态变化的，则会使开发变得更加困难。

django为了解决这一难题，提供了aggregate(聚合函数)和annotate(在aggregate的基础上进行GROUP BY操作)。

下面，就来介绍第二种方法。

#一. aggregate的使用方法#

今天在同事的指点下，仔细看了django中annotate的使用方法，会根据查询条件来动态生成SQL语句，提高组合查询的效率。

理解aggregate的关键在于理解SQL中的聚合函数：以下摘自百度百科：SQL基本函数，聚合函数对一组值执行计算，并返回单个值。除了 COUNT 以外，聚合函数都会忽略空值。常见的聚合函数有AVG / COUNT / MAX / MIN /SUM 等。

aggregate就是在django中实现聚合函数的。先来看aggregate的使用场景：在项目中有时候你想要从数据库中取出一个汇总的集合。我们使用django官方的例子：

```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()

class Publisher(models.Model):
    name = models.CharField(max_length=300)
    num_awards = models.IntegerField()

class Book(models.Model):
    name = models.CharField(max_length=300)
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.FloatField()
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher)
    pubdate = models.DateField()

class Store(models.Model):
    name = models.CharField(max_length=300)
    books = models.ManyToManyField(Book)
    registered_users = models.PositiveIntegerField()
```

如果我们使用aggregate来进行计数：
```bash
>>> from django.db.models import Count
>>> pubs = Publisher.objects.aggregate(num_books=Count('book'))
>>> pubs
{'num_books': 27}

```
而且aggregate不单单可以求和，还可以求平均Avg，最大最小等等。
```bash
>>> from django.db.models import Avg
>>> Book.objects.all().aggregate(Avg('price'))
{'price__avg': 34.35}
```

```
# Cost per page  输出的名字同样可以指定，比如price_per_page
>>> from django.db.models import F, FloatField, Sum
>>> Book.objects.all().aggregate(
... price_per_page=Sum(F('price')/F('pages'), output_field=FloatField()))
{'price_per_page': 0.4470664529184653}
```

通过上面的介绍，我们可以知道，aggregate的逻辑比较简单，应用场景比较窄，如果你想要对数据进行分组（GROUP BY）后再聚合的操作，则需要使用annotate来实现。

#二. annotate的使用方法#

首先，假设有这么一个models:
```python
# python:2.7.9
# django:1.7.8

class MessageTab(models.Model):
    msg_sn = models.CharField(max_lenth=20, verbose_name=u'编号')
    msg_name = models.CharField(max_length=50, verbose_name=u'消息名称')
    message_time = models.DateTimeField(verbose_name=u'消息出现时间')
    msg_status = models.CharField(max_length=50, default='未处理', verbose_name=u'消息状态')
    class Meta:
        db_table = 'message_tab'
```

如果在开发过程中，有这么一个需求：查询各个消息状态的数量。那么我们经常会使用filter(...).count(...)来进行查询。现在我们可以使用：
```python
    msgS = MessageTab.objects.values_list('msg_status').annotate(Count('id'))
```
其中，id为数据库自动生成的自增字段。values_list的用法自行Google，或者print出来看一看。

此时，数据库实际执行的代码，可以通过：
```python
    print msgS.query
```
打印出来。可以看到：

```sql
SELECT `message_tab`.`msg_status`, COUNT(`message_tab`.`id`) AS `id__count` FROM `message_tab` GROUP BY `message_tab`.`msg_status` ORDER BY NULL
```
很直观明了。通过msg\_status来进行group by。如果想自定义`id__count`，比如指定为msg_num，则可以使用：annotate(msg_num=Count('id'))

当存在多个查询条件时，比如查询最近7天内，message\_name属于某个分组内的消息，则可以使用Q函数：
```python
    date_end = now().date() + timedelta(days=1)
    date_start = date_end - timedelta(days=7)
    messageTimeRange = (date_start, date_end)
    GroupList = getGroupIdLis(request.user)  # 返回当前用户能查询的group的一个列表。。仅做参考用
    qQueryList = [Q(message_time__range=messageTimeRange), Q(message_name__in=GroupList)] # 可以有多个Q函数查询条件
    
    msgS = MessageTab.objects.filter(reduce(operator.and_, qQueryList)).values_list('msg_status').annotate(msg_num=Count('id'))
```

再次调用print msgS.query可看到SQL语句：
```sql
SELECT `message_tab`.`msg_status`, COUNT(`message_tab`.`id`) AS `msg_num` FROM `message_tab` WHERE (`message_tab`.`message_time` BETWEEN 2017-03-27 00:00:00 AND 2017-04-03 00:00:00 AND `message_tab`.`message_name` IN (1785785, 78757, 285889, 2727333, 7272957, 786767)) GROUP BY
 `message_tab`.`msg_status` ORDER BY NULL
```

是不是很完美！！

