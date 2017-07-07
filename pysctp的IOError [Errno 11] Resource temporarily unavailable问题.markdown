**记录一次异常的SCTP连接错误**

最近公司的一个项目，需要用到SCTP协议来传输diameter协议内容。于是最近研究了下SCTP协议在python中的实现。
SCTP协议的python实现非常少，搜了下，只有[pysctp](https://github.com/philpraxis/pysctp "")这个库，而且只能运行于Linux系统。于是就按照对于TCP的理解，写了一个简单的服务器和客户端的实现。

开发环境：Ubuntu16.04,；python2.7；Pycharm。

在pycharm的自带终端Terminal中debug过程中，我发现client客户端在连接上服务器之后，服务器端经常会出现一个错误，错误异常为：IOError: [Errno 11] Resource temporarily unavailable ：

```bash
Traceback (most recent call last):
  File "/home/test/Downloads/pycharm-community-2017.1.1/helpers/pydev/pydevd.py", line 1578, in <module>
    globals = debugger.run(setup['file'], None, None, is_module)
  File "/home/test/Downloads/pycharm-community-2017.1.1/helpers/pydev/pydevd.py", line 1015, in run
    pydev_imports.execfile(file, globals, locals)  # execute the script
  File "/home/test/PycharmProjects/sctp_socket/server.py", line 47, in <module>
    fromaddr, flags, msg, notif = conn_sock.sctp_recv(BUFSIZE)
  File "/home/test/sctp_env/local/lib/python2.7/site-packages/sctp.py", line 1206, in sctp_recv
    (fromaddr, flags, msg, _notif) = _sctp.sctp_recv_msg(self._sk.fileno(), maxlen)
IOError: [Errno 11] Resource temporarily unavailable
```

虽然问题定位到sctp_recv_msg这个函数，但是我发现只要connect成功，并不需要收发数据，就会触发这个错误。而且不是百分之百，会概率出现，而且有时连接成功后，发送一个data块，服务器再往回返，会收到无限多个重复的data块。使用wireshark抓包也看不出来有什么问题。

百思不得其解啊，CSDN / stackoverflow什么的都搜了个遍也没找到解决问题的方案，本来已经想要放弃了，打算底层socket的收发使用C语言/C++来封装，使用python来调用socket接口，虽然实现起来可能比全python稍微麻烦一点，但起码能跑起来。。。


最后，我分别使用命令行重新试了下，

```bash
python server.py
```

```bash
python client.py
```

一切正常，问题就这么解决了。WTF...

事后回想，问题大概率是出在pycharm的终端上；但是同事使用debian命令行也曾出现过无限多次重复接收的问题。。不知道为啥了...反正现在已经好了

#--------
**0502更新部分**：经过多次测试，多次重复接收问题是因为：当socket关闭时，recv端会接收到无限多次msg为空的消息，如果没有对msg进行判断的话，会无线循环接收。。因此，解决办法就是加一个判断句：if not msg: break

而Errno 11的问题，还没有明确解决方案。暂时定位为不能使用非阻塞，即setblocking(1)，不能为0...

#--------

最后贴一段简单的server和client代码，copy下来就能跑。当然需要先安装pysctp这个库（这个库需要先安装Debian/Ubuntu sudo apt-get install libsctp-dev lksctp-tools，不然会报错找不到文件）。。如果下面程序运行报错的话把中文去了试试。

```python
# -*- coding: utf-8 -*-
# 实现简单的socket服务器功能
import socket
import sctp
from time import ctime

HOST = ''
PORT = 3868
BUFSIZE = 1024
ADDR = (HOST, PORT)

socket_serv = sctp.sctpsocket(socket.AF_INET, socket.SOCK_STREAM,None)  # , sctp.IPPROTO_SCTP

socket_serv.initparams.max_instreams = 3
socket_serv.initparams.num_ostreams = 3
#socket_serv.initparams.max_attempts = 2
#socket_serv.initparams.set_max_init_timeo(0)

socket_serv.bindx([ADDR])
socket_serv.listen(5)
# socket_serv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
# socket_serv.setblocking(0)

socket_serv.events.data_io = 1
socket_serv.events.clear()

while True:
    print '等待用户连接...'
    conn_sock, addr = socket_serv.accept()
    print '有用户接入：', addr
    msg = ''
    while True:
        # msg= conn_sock.recv(BUFSIZE)
        fromaddr, flags, msg, notif = conn_sock.sctp_recv(BUFSIZE)
        print 'recv: %s' % msg
        if not msg:
            break
        conn_sock.sctp_send('当前时间：%s...发过来的是%s' % (ctime(), msg))

    conn_sock.close()
socket_serv.close()
```

```python
# -*- coding:utf-8
import _sctp
from sctp import *

server = "localhost"
tcpport = 3868

if _sctp.getconstant("IPPROTO_SCTP") != 132:
    raise "getconstant failed"
client_sock = sctpsocket(socket.AF_INET, socket.SOCK_STREAM, None)

saddr = (server, tcpport)
print "TCP ", saddr, " ----------------------------------------------"

client_sock.initparams.max_instreams = 3
client_sock.initparams.num_ostreams = 3
#client_sock.initparams.max_attempts = 2
#client_sock.initparams.set_max_init_timeo(0)

# tcp.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
# client_sock.setblocking(0)
client_sock.events.clear()
client_sock.events.data_io = 1  # 要获取消息的流编号，SCTP 需要启用套接字选项 sctp_data_io_event

client_sock.connect(saddr)

while 1:

    while True:
        data = raw_input('INPUT: ')
        if not data:
            break
        client_sock.sctp_send(data)
        fromaddr, flags, msg, notif = client_sock.sctp_recv(1024)
        # print "	Msg arrived, flag %d" % flags
        #if flags & FLAG_NOTIFICATION:
        #    raise "We did not subscribe to receive notifications!"
        #else:
        #    print "%s____" % msg, cont
        print "%s____" % msg
    break
client_sock.close()

```

如果还有问题，欢迎回帖/发邮件讨论。ayocross1@gmail.com
