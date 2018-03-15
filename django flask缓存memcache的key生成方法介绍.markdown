# django flask缓存memcache的key生成方法介绍

标签： python

---
去年的一个django项目中，使用了memcache作为系统缓存，并实现多台机器上的缓存共享。配置的cache如下图所示：

![django settings](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/memcache_set.png "")

最近在项目调试过程中，发现memcache在进行缓存时，使用的key并不是实际写入的key，一度我还以为是不是缓存的位置出BUG了。。想找下到底是存的是什么key：

```bash
telnet 10.200.75.11:11211
    
trying 10.200.75.11...
Connected to 10.200.75.11.
Escape character is '^]'.
    
get somekey
END
```

    

发现memcache中并没有django中写入的key(somekey)，但是在django中却能取出来，看来并不是数据过期的原因。

好，那就单步调试，看django里的源码到底是如何写key value的：

![set](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/memcache_set.png "")

![make key](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/BaseCache_make_key.png "")

![init](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/BaseCache_init.png "")

![key func](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/BaseCache_key_func.png "")

至此，我们已经发现了，原来在存储之前，给key加了个前缀，格式如下：

>  :1:somekey

现在再去Telnet中获取key:

![bash](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/bash_get.png "")


至此，django中关于memcache的key问题终于解释清楚了，原来是加了一些前缀。

掉过头来看flask，猜测也是类似的操作，

flask中的cache配置：

![flask_settings](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/flask_settings.png "")

单步跟着进去，发现竟然没有修改key的操作，再到Telnet中取一下，还真是没动key:

![flask key](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%20memcache/flask_key.png "")

---

至此，我们可以发现：django在存储key的时候，加了一点前缀，而flask却没有。

希望能对你有所帮助。



AyoCross
2018年03月14日下午