# python定时任务/异步任务工具celery与Scheduler

------

[TOC]

------

| 修改时间   |  修改说明  |  修改人  |
| --------   | :----: | :----:  |
| 2018-07-01     | 初次成稿 | AyoCross     |


本文以django/flask为例，介绍如何使用celery or Scheduler 来实现python的定时任务计划，并比较二者在使用过程中的不同之处，区分不同场景来选择适当的工具。

### 1. why 异步 or 定时

异步任务，指的是与同步任务相反的一种逻辑处理流程，在执行某任务A的过程中，需要执行任务B，但是B任务比较耗时，而且B的结果对A无影响。这种情况下，就需要考虑使用异步来实现任务B，从而使A的处理速度加快。一个典型的例子就是用户操作过程中的邮件提醒，比如业务逻辑如下：当用户修改web项目中的某几个敏感参数的时候，需要向该用户以及管理员用户发送通知邮件，并完成参数的修改。那么用户的修改操作即可看作是A任务，邮件发送即为B任务，B任务的发送结果（邮件发送成功还是失败）对修改任务A是否成功执行无影响。此时就需要考虑将B邮件发送任务异步实现，减少修改参数的服务器处理速度，提高用户体验。

定时任务，就更好理解了，比如需要每隔5分钟检查一次数据库里的当天有效数据，如果发生变更，则需要执行某些逻辑。此时就需要定时任务来执行响应的任务计划。

###  how to

#### 1. Scheduler
Scheduler作为一个轻量级的**定时任务工具**，在小型项目或者轻量级异步任务中经常会用到，最常见的使用方法如下：
首先，安装Scheduler：`pip install APScheduler`；
然后，在项目启动的位置，调用以下函数：

```python
from apscheduler.schedulers.background import BackgroundScheduler

def initialize_cron():
    apsched = BackgroundScheduler()
    apsched.add_job(isp_sync , 'cron' , kwargs = {} , hour = 2 , minute = 10)
    apsched.add_job(kafka_sync,'cron', hour = 6 , minute = 10)
    apsched.add_job(workflow,'cron', minute = '*/10')
    apsched.add_job(detect_isp_access,'cron', hour='*')
    apsched.add_job(mass_report,'cron', minute = '*/5')
    apsched.start()
```

感觉似乎没什么好介绍的，只要部署上去，就完事了，它会每隔多少时间，或者每天的定点时间，去执行响应的任务。比较高阶的用法，会出现在某些特殊的任务，可能需要在某个时间段内暂停任务，然后再恢复任务。就会需要用到pause / resume，详情可以查看[这篇博文][1]，写的很详细。



#### 2. celery

只要是项目上了点规模，或者说，既有定时任务，也有异步任务需要；再或者，待执行的任务占资源比较多，那就最好使用celery来实现，可以把任务从主任务中剥离出来，减少项目占用资源过多出问题的几率。
在django/flask中使用celery，也是非常简单的，首先，安装celery，
```python
pip install celery==3.1.25
pip install redis==2.10.5  # Broker,用于存放celery任务队列，除celery之外，也可使用rabbitMQ/Zookeeper等
```
安装完毕后，需要在项目中配置celery，以flask为例：在创建flask_app时，对celery进行配置：
```python

app = Flask("cdn_console",static_folder='', static_url_path='')
app.config.from_object('DefaultConfig')
cdn_celery = configure_celery(app)

def configure_celery(app):
    platforms.C_FORCE_ROOT = True
    celery = Celery(app.import_name, broker=app.config['CELERY_BROKER_URL'])
    celery.conf.update(app.config)
    Taskbase = celery.Task

    class ContextTask(Taskbase):
        abstract = True

    def __call__(self, *args, **kw):
        with app.app_context():
            return Taskbase.__call__(self, *args, **kw)

    celery.Task = ContextTask
    return celery


class DefaultConfig(object):
    DEBUG = True
    TESTING = False

    _basedir = os.path.join(os.path.abspath(os.path.dirname(os.path.dirname(
                            os.path.dirname(__file__)))))

    if DEBUG==True:
        redis_host='127.0.0.1'
    else:
        redis_host='redis.test.org'

    REDIS_URL = "redis://%s:6379/1"%redis_host
    CELERY_BROKER_URL = 'redis://%s:6379'%redis_host,  # Broker 地址
    # CELERY_RESULT_BACKEND = 'redis://localhost:6379',  # 结果存储地址
    
    # 定时任务，
    CELERYBEAT_SCHEDULE = {
        'task1': {  # 定时任务1，每隔5分钟执行一次
            'task': 'tasks.api_delay_func_1',
            "schedule": timedelta(minutes=5),
            "args": '',
        },
        #'task2': {
        #    'task': 'tasks.api_delay_func_2',
        #    "schedule": timedelta(minutes=30),
        #    "args": '',
        #},
    }
} 


# tasks.py中的定时任务：
@cdn_celery.task
def api_delay_func_1(log_type='5'):
    """
    定时任务
    """
    from run import app
    app_ctx = app.app_context()
    app_ctx.push()
    try:
        print '定时任务'
        app_ctx.pop()
    except Exception as e:
        app_ctx.pop()
        print e   
```

配置中之所有有个ContextTask函数，是因此flask的应用上下文，这种配置也是flask官方推荐的配置模块。

如果某个函数需要执行异步任务，添加一个装饰器即可：
```python
@cdn_celery.task
def flow_converge():
    """
    流量/带宽  汇聚
    """
    print '异步任务'
    return


# 在需要调用该异步函数的位置，使用delay来调用
from tasks import flow_converge
flow_converge.delay()
```

项目中添加完毕之后，在项目所在的目录运行脚本来执行celery：
```bash
nohup celery worker -A tasks --loglevel=info -c 2 >../celery_worker.log 2>&1 &
nohup celery beat -A tasks --loglevel=info >../celery_beat.log 2>&1 &
```


### 3. which one

只要不是太小的项目，或者你确定项目以后不会有其他的定时任务出现了，那么就使用scheduler来实现，否则，加入celery的怀抱吧！毕竟celery真的是太好用了。。


AyoCross    20180701

[1]: http://debugo.com/apscheduler/