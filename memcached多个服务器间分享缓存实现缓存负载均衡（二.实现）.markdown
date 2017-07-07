具体实现篇

此处以django为例，介绍memcached如何在django中使用

首先，在settings.py文件中，对CACHES模块进行设置，多个IP/PORT之间用逗号间隔。

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': [
            '10.111.75.24:11211',
            '10.111.75.25:11211',
        ]

    }
}
```

很多文章都止步于此，所以我也一开始以为只需要这样就行了，但是配置好Nginx负载均衡后，还是没有实现memcached的共享。于是在这台机器(本机IP为10.111.75.25)上试了下：

```bash
telnet 10.111.75.24 11211
```

发现并不能访问。因此发现memcached竟然还对IP做了限定。。这和我Google的答案不一样啊（网上都是说memcached不对IP做限制，只能通过系统防火墙实现）。。于是就找到memcached配置文件（/etc/memcached.conf），打开看了下，发现有这么一行，才知道原来memcached默认限定本地IP来访问。。：

```bash
# Specify which IP address to listen on. The default is to listen on all IP addresses
# This parameter is one of the only security measures that memcached has, so make sure
# it's listening on a firewalled interface.
-l 127.0.0.1

# Limit the number of simultaneous incoming connections. The daemon default is 1024
# -c 1024

```

找到问题所在，改起来就容易了，把这一行改成：

-l 10.111.75.25, 127.0.0.1

:wq保存，重启下memcached，再telnet，就没问题了：/etc/init.d/memcached restart

加上10.111.75.25这个IP，就能让外网IP能够访问到本地的memcached服务了。

至此，memcached不能共享数据的问题解决，实现负载均衡下的缓存共享。


AyoCross
