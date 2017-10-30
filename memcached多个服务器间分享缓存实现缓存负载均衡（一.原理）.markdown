　　近来遇到的一个问题：django在登陆时，将登陆信息写入本地缓存memcached，本地测试没有问题，但是部署到线上服务器时，经常会出现登陆认证失败。因为正式环境中使用的是Nginx与uWSGI来实现的多台服务器的负载均衡。怀疑是由于memcached没有共享导致。上网搜了一下，这篇博文写的非常透彻，从源码的角度上解释了memcache是如何实现缓存共享，以及多台性能不同的服务器间的负载均衡如何实现。转过来，供大家参考。

原文地址：[分析Memcached客户端如何把缓存数据分布到多个服务器上](http://www.cnblogs.com/liuguanghai/p/5386168.html "")

原作者：他乡客


 
　　Memcached客户端可以设多个memcached服务器,它是如何把数据分发到各个服务器上，而使各个服务器负载平衡的呢？

可以看看.net版中的客户端中的源码，就可以知道 先看代码：


```cpp
/// <summary>
        /// Returns appropriate SockIO object given
        /// string cache key and optional hashcode.
        /// 
        /// Trys to get SockIO from pool.  Fails over
        /// to additional pools in event of server failure.
        /// </summary>
        /// <param name="key">hashcode for cache key</param>
        /// <param name="hashCode">if not null, then the int hashcode to use</param>
        /// <returns>SockIO obj connected to server</returns>
        public SockIO GetSock(string key, object hashCode)
        {
            string hashCodeString = "<null>";
            if(hashCode != null)
                hashCodeString = hashCode.ToString();

            if(Log.IsDebugEnabled)
            {
                Log.Debug(GetLocalizedString("cache socket pick").Replace("$$Key$$", key).Replace("$$HashCode$$", hashCodeString));
            }

            if (key == null || key.Length == 0)
            {
                if(Log.IsDebugEnabled)
                {
                    Log.Debug(GetLocalizedString("null key"));
                }
                return null;
            }

            if(!_initialized)
            {
                if(Log.IsErrorEnabled)
                {
                    Log.Error(GetLocalizedString("get socket from uninitialized pool"));
                }
                return null;
            }

            // if no servers return null
            if(_buckets.Count == 0)
                return null;

            // if only one server, return it
            if(_buckets.Count == 1)
                return GetConnection((string)_buckets[0]);

            int tries = 0;

            // generate hashcode
            int hv;
            if(hashCode != null)
            {
                hv = (int)hashCode;
            }
            else
            {

                // NATIVE_HASH = 0
                // OLD_COMPAT_HASH = 1
                // NEW_COMPAT_HASH = 2
                switch(_hashingAlgorithm)
                {
                    case HashingAlgorithm.Native:
                        hv = key.GetHashCode();
                        break;

                    case HashingAlgorithm.OldCompatibleHash:
                        hv = OriginalHashingAlgorithm(key);
                        break;

                    case HashingAlgorithm.NewCompatibleHash:
                        hv = NewHashingAlgorithm(key);
                        break;

                    default:
                        // use the native hash as a default
                        hv = key.GetHashCode();
                        _hashingAlgorithm = HashingAlgorithm.Native;
                        break;
                }
            }

            // keep trying different servers until we find one
            while(tries++ <= _buckets.Count)
            {
                // get bucket using hashcode 
                // get one from factory
                int bucket = hv % _buckets.Count;
                if(bucket < 0)
                    bucket += _buckets.Count;

                SockIO sock = GetConnection((string)_buckets[bucket]);

                if(Log.IsDebugEnabled)
                {
                    Log.Debug(GetLocalizedString("cache choose").Replace("$$Bucket$$", _buckets[bucket].ToString()).Replace("$$Key$$", key));
                }

                if(sock != null)
                    return sock;

                // if we do not want to failover, then bail here
                if(!_failover)
                    return null;

                // if we failed to get a socket from this server
                // then we try again by adding an incrementer to the
                // current key and then rehashing 
                switch(_hashingAlgorithm)
                {
                    case HashingAlgorithm.Native:
                        hv += ((string)("" + tries + key)).GetHashCode();
                        break;

                    case HashingAlgorithm.OldCompatibleHash:
                        hv += OriginalHashingAlgorithm("" + tries + key);
                        break;

                    case HashingAlgorithm.NewCompatibleHash:
                        hv += NewHashingAlgorithm("" + tries + key);
                        break;

                    default:
                        // use the native hash as a default
                        hv += ((string)("" + tries + key)).GetHashCode();
                        _hashingAlgorithm = HashingAlgorithm.Native;
                        break;
                }
            }

            return null;
        }
```

　　上面代码是代码文件SockIOPool.cs中的一个方法，从方法签名上可以看出，获取一个socket连接是根据需要缓存数据的唯一键和它的哈希值，因为缓存的数据的键值是唯一的，所以它的哈希代码也是唯一的；

再看看上面方法中的以下代码：

         int bucket = hv % _buckets.Count;

                if(bucket < 0)

                    bucket += _buckets.Count;

 

                SockIO sock = GetConnection((string)_buckets[bucket]);

　　具体的选择服务器的算法是：唯一键值的哈希值与存放服务器列表中服务器（服务器地址记录不是唯一的）的数量进行模数运算来选择服务器的地址的。所以数据缓存在那台服务器取决于缓存数据的唯一键值所产生的哈希值和存放服务器列表中服务器的数量值，所以访问memcached服务的所有客户端操作数据时都必须使用同一种哈希算法和相同的服务器列表配置，否则就会或取不到数据或者重复存取数据。由于不同数据的唯一键所对应的哈希值不同，所以不同的数据就有可能分散到不同的服务器上，达到多个服务器负载平衡的目的。

　　如果几台服务器当中，负载能力各不同，想根据具体情况来配置各个服务器负载作用，也是可以做到的。看上面代码，可以知道程序是从_buckets中获取得服务器地址的，_buckets存放着服务器的地址信息，服务器地址在_bucket列表中并不是唯一的，它是可以有重复记录的。相同的服务器地址在_bucket重复记录越多，它被选中的机率就越大，相应负载作用也就越大。

　　怎么设置服务器让它发挥更大的负载作用,如下面代码：
        String[] serverlist = {"192.168.1.2:11211", "192.168.1.3:11211"};

        int[] weights   = new int[]{5, 2}; 

        SockIOPool pool = SockIOPool.GetInstance();

        pool.SetServers(serverlist);

        pool.SetWeights(weights);   

         pool.Initialize();

　　pool.SetWeights(weights)方法就是设配各个服务器负载作用的系数，系数值越大，其负载作用也就越大。如上面的例子，就设服务器192.168.1.2的负载系数为5,服务器192.168.1.3的负载系数为2,也就说服务器192.168.1.2 比192.168.1.3的负载作用大。      

　　程序中根据缓存数据中的唯一 键标识的哈希值跟服务器列表中服务器记录数量求模运算来确定数据的缓存的位置的方法，算法的优点：能够把数据匀均的分散到各个服务器上数据服务器负载平 衡，当然也可以通过配置使不同服务器有不同的负载作用。但也有缺点：使同类的数据过于分散，同个模块的数据都分散到不同的数据，不好统一管理和唯护；比 如：现在有A、B、C、D四台服务器一起来做缓存服务器，数月后C台服务器突然死掉不可用啦，那么按算法缓存在C台服务器的数据都不可用啦，但客户端还是按原来的四台服务器的算法来取操作数据，所以分布在C服务上的数据在C服务器恢复可用之前都不可用，都必须从数据库中读取数据，并且不能添加到缓存中，因为只要缓存数据的Key不变，它还是会被计算分配到C服务器上。如果想把分配到C服务器就必须全部初始化A、B、D三台服务器上的所有数据，并把C服务器从服务器列表中移除。

　　如果我们能够把数据分类分布到各个服务器中，同类型的数据分布到相同的服务器；比如说，A服务器存放用户日志模块信息，B服务器存放用户相册模块信息，C服务器存放音乐模块信息，D服务器存放用户基础信息。如果C服务器不可用后，就可以更改下配置使它存放在其它服务器当中，而且并不影响其它服务器的缓存信息。

解决方法1:不同的模块使用不同memcached客户端实例，这样不同模块就可以配置不同的服务器列表，这样不同模块的数据就缓存到了不同的服务器中。这样，当某台服务器不可用后，只会影响到相应memcached客户端实例的数据，而不会影响到其它客户端实例的数据。

解决方法2：修改或添加新的算法，并在数据唯一键中添加命名空间，算法根据配置和数据唯一键中命名空间来选择不同的Socket连接，也就是服务器啦。

　　数据项唯一键(key)的定义：命名空间.数据项ID，就跟编程中的” 命名空间”一样,经如说用户有一篇日志的ID是”999999”, 那么这条篇日志的唯一键就是:Sns.UserLogs.Log.999999,当然我们存贮的时候考虑性能问题，可以用一个短的数值来代替命名空间。这样在选择Socket的时候就可以根据数据项中的唯一键来选择啦。

