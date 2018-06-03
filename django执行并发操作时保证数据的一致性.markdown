# django执行并发操作时保证数据的一致性

------

[TOC]

------

| 修改时间   |  修改说明  |  修改人  |
| --------   | :----: | :----:  |
| 2018-06-03     | 初次成稿 | AyoCross     |


### 1. 情景复现

本周一，我负责维护的一个公司内部django项目，出现了两次数据库死锁导致的系统不可用的情况，报错信息如下：

    OperationalError: (1205, 'Lock wait timeout exceeded; try restarting transaction')

![报错描述][1]
即：数据库资源被占用时间过长，导致超时。
出现问题之后赶紧找PE把数据库里的锁kill，然后再来定位问题，看sql语句执行的都没什么异常，而且系统最近也没有进行过更新。

项目使用的版本号：
django=1.8.3
mysql=5.5.47，死锁涉及的表为InnoDB


###  2. 问题分析

我们都知道，InnoDB数据表在执行update操作时，可以添加行锁，来确保修改数据时不会同时被其他人修改。我印象中，django中的ORM，应该是有对这个锁的封装的，所以一直都没有管过update数据时的可靠性问题。

出了这个问题以后，Google了一下，发现好像并没有相关的说明，互联网中绝大多数都是说，想要确保数据的一致性，需要使用select_for_update 来实现加锁。而且大都指向同一篇文章：[国内翻译文][2]/[英文原文][3]。但是我明明记得以前好像看过，django会自动为操作加锁，防止不同用户之间产生冲突。难道是我记忆出现了偏差？


### 3. 原因梳理

好吧，既然网上找的方案行不通，那就看源码吧，还好有pycharm这么好用的看源码工具，随便找到项目中一个`Bean.save()`的位置，摁住Ctrl，点击鼠标，进入调用源码的位置。然后就是找到生成SQL的位置了。经过一番的定位，最终，在

> C:\Python27\Lib\site-packages\django\db\models\sql\compiler.py

这个文件里找到：
![as_sql][4]

在这个函数里，找到select for update相关的说明：

![此处输入图片的描述][5]

![此处输入图片的描述][6]

看这个源码的意思，还真是，除非你在代码里显式的注明，调用select_for_update，则会给执行的SQL语句加上`FOR UPDATE`，否则就不会有相关的操作。本以为自己想的肯定没问题，网上说的都是错的，结果被拍拍打脸，果然是自己的记忆出现了偏差，django对于并发数据的保护默认是不存在的，需要人为去干预才行，否则如果多人同时操作同一行数据，就会出问题。

###  4. 问题解决

至于这个问题应当如何解决，在上面两篇文章的下面都做了详细的解释，我在这就不赘述了，遇到类似问题的同学，可以跟着上面文章里的两种乐观/悲观的思路，去尝试解决。


  [1]: https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E2%80%94select_for_update/Lock%20wait%20timeout%20exceeded.png
  [2]: http://ikcaz.whflyjk.com/archives/59/
  [3]: https://medium.com/@hakibenita/how-to-manage-concurrency-in-django-models-b240fed4ee2
  [4]: https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E2%80%94select_for_update/as_sql.png
  [5]: https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E2%80%94select_for_update/select_for_update.png
  [6]: https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E2%80%94select_for_update/for_update_sql_func.png