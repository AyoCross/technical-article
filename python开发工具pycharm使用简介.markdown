
　　pycharm是一款常用的python开发工具，功能十分强大，并且多平台支持（Windows/MacOS/Linux），官方提供社区开源版本：[pycharm Community免费版本下载地址](https://www.jetbrains.com/pycharm/download/ "pycharm下载地址")。


# pycharm在日常开发中常用功能简介：

1. 断点调试，类似于Visual Studio，实时查看变量/内存使用情况，支持django/flask等工程的调试:
![django调试图](https://raw.githubusercontent.com/AyoCross/usual_pics/master/django%E8%B0%83%E8%AF%95%E5%9B%BE.png "")
2. 自动检查代码风格，内嵌PEP8标准（详情见附录），代码风格统一，便于管理。配置方法：

  >(1) pip install autopep8；
  
  >(2) 选择菜单「File」–>「Settings」–>「Tools」–>「External Tools」–>点击加号添加工具
  ![安装autopep8](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%AE%89%E8%A3%85autopep8.png "")
  
  >(3)填写配置项
  ```
  Name：Autopep8 (可随意填写)
  Tools settings:      
  Programs：autopep8
  Parameters：--in-place --aggressive --ignore=E123,E133,E50 $FilePath$
  Working directory：$ProjectFileDir$
  ```
  ![autopep8配置项](https://raw.githubusercontent.com/AyoCross/usual_pics/master/autopep8%E9%85%8D%E7%BD%AE%E9%A1%B9.png "")

3. 内嵌git功能，方便进行版本控制。
4. 支持模块模板，便于工程内统一注释风格，标明作者等。
5. 自动分析模块全局变量、函数位置，位置切换十分方便。
![文件structure](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E6%96%87%E4%BB%B6structure.png "")
6. 代码单元测试功能，更快捷得进行单元测试，提高代码鲁棒性。

以下为开发过程中常用快捷键举例：
1. Ctrl+Click(或Ctrl+B)轻松查看源码，了解代码原理，查看函数定义后，选中函数名称，Ctrl+Click即可查看调用该函数的位置，轻松返回。
2. 自动引入，Alt+Enter，自动提示，引入模块神器。需先选中Settings->general->autoimport->python :show popup；![自动引入](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E8%87%AA%E5%8A%A8%E5%BC%95%E5%85%A5.png "")
3. 插入常用代码Ctrl+J，不用重复劳动。
![常用代码](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%B8%B8%E7%94%A8%E4%BB%A3%E7%A0%81.png "")

4. 4.	快速查找文件Ctrl+E打开最近访问的文件，Ctrl+Shift+E打开最近编辑过的文件，尤其是在大型工程中，传统的按层查找文件非常低效；配合Ctrl+Tab切换前一个打开的文件，切换效率提升显著。
![快速查找文件](https://raw.githubusercontent.com/AyoCross/usual_pics/master/%E5%BF%AB%E9%80%9F%E6%9F%A5%E6%89%BE%E6%96%87%E4%BB%B6.png "")
10. 万能搜索，双击Shift，可以搜索文件名、类名、方法名，还可以搜索目录名，搜索目录的技巧是在在关键字前面加斜杠/。
11. 历史粘贴板，Ctrl+Shift+V可访问历史粘贴板。
12. 多屏显示代码，Settings-Keymap中进行自定义。
13. 任意换行， Shfit+Enter。
14. 9.	优化导入，Ctrl+Alt+O，去掉没有使用的引入，规范引用顺序。
15. 注释选中行，Ctrl+/
16. 

　　另外，pycharm支持多python共存，支持virtualenv进行配置。多版本下同时开发更加方便。

　　其他方面，pycharm还能自定义字体，语法高亮颜色等一系列自主化配置，可以使写代码更加称心入手。

---
# 附录1：Win/Mac快捷键（原图来自麻瓜编程）
![pycharmWindows下客户端的快捷键](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEWin.png "")

[Win快捷键高清图链接](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEWin.png "")

![pycharm Mac下客户端的快捷键](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEMac.jpg "")

[Mac快捷键高清图链接](https://raw.githubusercontent.com/AyoCross/usual_pics/master/pycharm%E5%BF%AB%E6%8D%B7%E9%94%AEMac.jpg "")

---
# 附录2：PEP8标准
[PEP 8标准-英文原版](https://www.python.org/dev/peps/pep-0008/ "PEP 8标准-英文原版")

## PEP8标准中文摘要

### 代码布局

> 1.缩进固定为4个空格，防止不同平台之间的table缩进不一致；

> 2.每行的长度最多为79个字节；

> 3．import导入时，标准库在前，接着是第三方库，以及自定义的文件导入；

### 注释

> 	模块顶部要有模块注释；类以及自定义函数要有其功能，以及入参出参注释，待解决的问题处，句末可以有注释，诸如# FIXME/TODO等高亮注释；

### 命名风格

> 1.避免使用库文件名、或者易混淆的常用名；

> 2.类名或异常名：类名要用首字母大写的规则。内部类，要加上前导下划线；

> 3.函数名与变量名：小写，为了增加可读性可以用下划线分隔；

> 4.函数与方法的参数名：用self作为实例方法的第一个参数，cls作为类方法的第一个参数；

> 5.常量：大写，为了增加可读性可以用下划线分隔；

### 程序设计建议

> 1.与诸如None这样的字符比较时，要使用is or is not，不要用等于操作；

> 2.当捕获一个异常的时候，要用详细异常声明代替except Exception（一个空的except:语句将会捕获 SystemExit 和 KeyboardInterrrupt 异常。这会使得很难用Ctrl+C来中断一个程序，并且还会隐藏其他的问题）；

> 3.try中的代码功能越明确越好，不要用一个try来包含所有内容；

> 4.用isinstance(object, type)代替类型比较；


