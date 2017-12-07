# 亚马逊云EC2搭建SS科学上网

作为一个程序猿，如果开发过程中不能使用Stack Overflow或者Google来查问题，那么解决BUG的能力会瞬间降低好几个层次。
毕竟中文互联网比起英文还是范围少很多。虽然CSDN、博客园，以及其他独立的博客也能提供很多解决问题的思路，但是很多也都是直接翻译的Stack Overflow原文。

一般大公司都会给提供有限制的VPS，以供查阅Google资料，但是如果是小的创业公司，或者经常在家码代码的话，那还是自己搭建一个最方便了。

亚马逊提供的免费一年EC2服务器的使用权，就非常符合这个需求，尤其是如果你跟我一样还是个后端开发的话，还能在上面部署自己的博客网站，做很多的尝试。

好了，废话不多说，开始EC2搭建教程了：

1. 首先你得先获取服务器----这不废话么。。具体的申请流程百度一大堆，先登陆亚马逊账户，绑定一个信用卡，然后就是一步一步往下来。需要注意的点：  亚马逊的服务器提供了多个节点，有美国的/欧洲的/东南亚的等等，我选择的是位于新加坡的节点，ping较低，而且我也只是为了上Google，如果你还有其他需求的话，可以自行选择节点； 服务器的系统看你喜好，我用的是Ubuntu，你选Debian或者CentOS也行，没啥大的区别。
2. 获取服务器后，登录到服务器（可以使用SSH工具使用Xshell5或者MobaXterm都行），开始安装SS转发工具。可以看[这位的GitHub](https://github.com/RockerFlower/shadowsocks "")，里面也有具体的安装方法。因为GitHub在国内不稳定，经常打不开，你需要多试几次。或者你就直接看我下面的方法也行。。：

Debian/Ubuntu
```bash
apt-get install python-pip
pip install shadowsocks
```
CentOS:
```bash
yum install python-setuptools && easy_install pip
pip install shadowsocks
```
安装完毕之后，几条命令，你的SS转发服务器就搭建起来啦：

```bash
ssserver -p 443 -k password -m aes-256-cfb
```
当然，你肯定不想一直开着终端，关掉终端之后服务器就关了那就没意思了。后台运行：
```bash
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start
```
关闭服务：
```bash
sudo ssserver -d stop
```
检查日志：
```bash
sudo less /var/log/shadowsocks.log
```
至此，你的SS转发服务器就搭建起来了。当然，可能你在搭建的过程中会遇到各种各样的问题，欢迎跟帖讨论，或者直接在这个[GitHub地址](https://github.com/shadowsocks/shadowsocks/issues?page=1&q=isAissue+isAopen "")开贴讨论，当然，最好是先挨个看下其他的帖子，很可能别人早已给出解决方案了。

下一步，就要配置你的账号密码，以便连上你的服务器，进行上网了。

-----------

配置你的配置文件。
配置文件位置默认是位于：/etc目录下，shadowsocks.json文件，当然，也有其他版本位于/etc/shadowsocks/shadowsocks.json。你都试试。
以下为示例，server监听IP,0.0.0.0为监听所有IP；端口号和密码你自己设定。
```json
{
    "server": "0.0.0.0",
    "port_password": {
        "8888": "123qwe456",
    },
    "timeout": 300,
    "method": "aes-256-cfb"
}

```
**要注意的是**：端口号设定之后，要在亚马逊的服务器管理界面，把该端口设为白名单（创建安全组，协议规为自定义TCP规则），不然防火墙直接给拦下了。
而且在安全组里面你还可以限定IP（如果你上网位置固定的话，强烈推荐），只允许从你的IP进行访问，防止有人暴力破解你的服务器，或者GFW嗅探你的SS。。。嘿嘿嘿


配置完毕之后，你就可以在你的电脑（神马！你连SS客户端都没有？[Windows客户端，需要下载.NET](https://github.com/shadowsocks/shadowsocks-windows/wiki/Shadowsocks-Windows-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E "")，其他系统的客户端自己百度。。）或者路由器里配置SS客户端，填进去账号密码IP端口号，开始你的Google/Stack Overflow之旅吧~~

------------------------------------------丑陋的分割线

我用的就是去年京东免费领的那个斐讯路由器，刷了华硕的固件，支持SS功能，然后路由器下的所有设备都能用了，不然IOS11不支持SS贼蛋疼。。

每天晚上回去看YouTube 1080P视频，简直美滋滋..唯一一点比较可惜的就是瞬时值不够稳定，吃鸡还是得开加速器，不过最起码比免费的或者一年100的那种加速强多了。。。对了，最开始搭SS是为了干啥子来着？



