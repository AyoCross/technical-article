　　网上现在有很多在django中的教程，不过基本都是你抄我我抄你这种，看到这篇，作者写的还算是很靠谱的，转过来，可以经常看一看

原作者：zni.feng

[原文链接](http://www.cnblogs.com/znicy/p/5626040.html "")

        
**一、Django中的异步请求**

　　Django Web中从一个http请求发起，到获得响应返回html页面的流程大致如下：http请求发起 -- http handling（request解析） -- url mapping（url正则匹配找到对应的View） -- 在View中进行逻辑的处理、数据计算（包括调用Model类进行数据库的增删改查）--将数据推送到template，返回对应的template/response。

![django顺序](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery1.png "")

同步请求：所有逻辑处理、数据计算任务在View中处理完毕后返回response。在View处理任务时用户处于等待状态，直到页面返回结果。

异步请求：View中先返回response，再在后台处理任务。用户无需等待，可以继续浏览网站。当任务处理完成时，我们可以再告知用户。

**二、关于Celery**

　　Celery是基于Python开发的一个分布式任务队列框架，支持使用任务队列的方式在分布的机器/进程/线程上执行任务调度。

![celery架构](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery2.png "")

　　Celery的架构，它采用典型的生产生-消费者模式，主要由三部分组成：broker（消息队列）、workers（消费者：处理任务）、backend（存储结果）。实际应用中，用户从Web前端发起一个请求，我们只需要将请求所要处理的任务丢入任务队列broker中，由空闲的worker去处理任务即可，处理的结果会暂存在后台数据库backend中。我们可以在一台机器或多台机器上同时起多个worker进程来实现分布式地并行处理任务。

**三、Django中Celery的实现**

　　在实际使用过程中，发现在Celery在Django里的实现与其在一般.py文件中的实现还是有很大差别，Django有其特定的使用Celery的方式。这里着重介绍Celery在Django中的实现方法，简单介绍与其在一般.py文件中实现方式的差别。

　　1. 建立消息队列

　　首先，我们必须拥有一个broker消息队列用于发送和接收消息。Celery官网给出了多个broker的备选方案：RabbitMQ、Redis、Database（不推荐）以及其他的消息中间件。在官网的强力推荐下，我们就使用RabbitMQ作为我们的消息中间人。在Linux上安装的方式如下：

```bash
sudo apt-get install rabbitmq-server
```

命令执行成功后，rabbitmq-server就已经安装好并运行在后台了。

　　另外也可以通过命令rabbitmq-server来启动rabbitmq server以及命令rabbitmqctl stop来停止server。

　　更多的命令可以参考rabbitmq官网的用户手册：https://www.rabbitmq.com/manpages.html

　　2. 安装django-celery

```bash
pip install celery
pip install django-celery
```

　　3. 配置settings.py

　　首先，在Django工程的settings.py文件中加入如下配置代码：
```bash
import djcelery
djcelery.setup_loader()
BROKER_URL= 'amqp://guest@localhost//'
CELERY_RESULT_BACKEND = 'amqp://guest@localhost//'
```

　　其中，当djcelery.setup_loader()运行时，Celery便会去查看INSTALLD_APPS下包含的所有app目录中的tasks.py文件，找到标记为task的方法，将它们注册为celery task。BROKER_URL和CELERY_RESULT_BACKEND分别指代你的Broker的代理地址以及Backend（result store）数据存储地址。在Django中如果没有设置backend，会使用其默认的后台数据库用来存储数据。注意，此处backend的设置是通过关键字CELERY_RESULT_BACKEND来配置，与一般的.py文件中实现celery的backend设置方式有所不同。一般的.py中是直接通过设置backend关键字来配置，如下所示：

```bash
app = Celery('tasks', backend='amqp://guest@localhost//', broker='amqp://guest@localhost//')
```

然后，在INSTALLED_APPS中加入djcelery：
```python
INSTALLED_APPS = (
    ……   
    'qv',
    'djcelery'
    ……   
)  
```

　　4. 在要使用该任务队列的app根目录下（比如qv），建立tasks.py，比如：

![3](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery3.png "")

　　在tasks.py中我们就可以编码实现我们需要执行的任务逻辑，在开始处import task，然后在要执行的任务方法开头用上装饰器@task。需要注意的是，与一般的.py中实现celery不同，tasks.py必须建在各app的根目录下，且不能随意命名。

　　5. 生产任务

　　在需要执行该任务的View中，通过build_job.delay的方式来创建任务，并送入消息队列。比如：

![4](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery4.png "")

　　6. 启动worker的命令

```bash
#先启动服务器
python manage.py runserver
#再启动worker 
python manage.py celery worker -c 4 --logievel=info
```

**四、补充**

　　Django下要查看其他celery的命令，包括参数配置、启动多worker进程的方式都可以通过python manage.py celery --help来查看:

![5](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery5.png "")

 　　另外，Celery提供了一个工具flower，将各个任务的执行情况、各个worker的健康状态进行监控并以可视化的方式展现，如下图所示：

![6](http://public-readonly.oss-cn-shanghai.aliyuncs.com/celery6.png "")

　　Django下实现的方式如下：　

　　1. 安装flower:

```bash
pip install flower
```

   2. 启动flower（默认会启动一个webserver，端口为5555）:

```bash
python manage.py celery flower
```
　　3. 进入http://localhost:5555即可查看。


