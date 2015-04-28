---
layout: post
title: "[摘抄] 编写 Python 代码的 91 个建议"
description: ""
category: [Excerpt, Python]
tags: [Excerpt, Python]
---
{% include JB/setup %}

> Effective 系列丛书 《编写高质量代码：改善Python程序的91个建议》

- 理解 Pythonic 概念
- 编写 Pythonic 代码
- 理解Python与C语言的不同之处
- 在代码中适当添加注释
- 通过适当添加空行使代码布局更为优雅、合理
- 编写函数的 4 个原则
- 将常量集中到一个文件
- 使用 assert 语句来发现问题
- 数据交换值的时候不推荐使用中间变量
- 充分利用 lazy evaluation 的特性
- 理解枚举替代实现的缺陷
- 不推荐使用 type 来进行类型检查
- 尽量转为浮点类型后再做除法
- 警惕 eval 的安全漏洞
- 使用 enumerate 获取序列迭代的索引和值
- 分清 is 与 == 的使用场景(id 相等与值相等)
- 考虑兼容性 尽可能使用 unicode(python 2)/str(python 3)
- 构建合理的包层次来管理 module
- 有节制的使用 from...import 语句
- 优先使用 absulote import 来导入模块
- i+=1 不等于 ++i
- 使用 with 自动关闭资源
- 使用 else 子句简化循环（异常处理）
- 遵循异常处理的几项基本原则
- 避免 finally 中可能发生的陷阱
- 深入理解 None 正确判断对象是否为空
- 连接字符串应优先使用 join 而不是 +
- 格式化字符串时使用 format 而不是 %
- 区别对待可变对象和不可变对象
- \[\]\(\){} 一致的容器初始化形式
- [函数传参既不是传参也不是传引用](#函数传参既不是传参也不是传引用)
- [警惕默认参数潜在的问题](#警惕默认参数潜在的问题)
- [慎用变长参数](#慎用变长参数)
- 深入理解 str() 和 repr() 的区别
- 分清 staticmethod 和 classmethod 的适用场景
- [掌握字符串的基本用法](#掌握字符串的基本用法)
- 按需选择 sort() 或者 sorted()
- 使用 copy 模块深拷贝对象
- 使用 Counter 进行计数统计
- [深入掌握 ConfigParser](#深入掌握-configparser)
- 使用 argparse 处理命令行参数(docopt)
- [使用 pandas 处理大型 CSV 文件](#使用-pandas-处理大型-csv-文件)
- 一般情况使用 ElementTree 解析 XML
- [理解模块 pickle 优劣](#理解模块-pickle-优劣)
- [序列化的另一个不错的选择 JSON](#序列化的另一个不错的选择-json)
- 使用 traceback 获取栈信息
- [使用 logging 记录日志信息](#使用-logging-记录日志信息)
- 使用 threading 模块编写多线程程序
- 使用 Queue 使多线程编程更安全

设计模式

- 利用模块实现单例模式
- [用 mixin 模式让程序更加灵活](#用-mixin-模式让程序更加灵活)
- [用发布订阅模式实现松耦合](#用发布订阅模式实现松耦合)
- 使用状态模式美化代码

内部机制

- 理解 built-in objects
- `__init__()` 不是构造方法
- [理解名字查找机制](#理解名字查找机制)
- 为什么需要 self 参数
- [理解 MRO 与多继承](#理解-MRO-与多继承)
- [理解描述符机制](#理解描述符机制)
- [区别 `__getattr__()` 和 `__getattribute__()` 方法](#区别-getattr()-和-getattribute()-方法)
- 使用更为安全的 property
- 掌握 metaclass
- 熟悉 Python 对象协议
- [利用操作符重载实现中缀语法](#利用操作符重载实现中缀语法)
- 熟悉 Python 的迭代器协议 `__iter__`
- 熟悉 Python 的生成器 `yield`
- 基于生成器的协程及 greenlet
- [理解 GIL 的局限性](#理解-gil-的局限性)
- [对象的管理与垃圾回收](#对象的管理与垃圾回收)

使用工具辅助项目开发

- 从 pypi 安装包
- 使用 pip 和 yolk 安装管理包
- 做 paster 安装包
- [理解单元测试概念](#理解单元测试概念)
- 为包编写单元测试
- [利用测试驱动开发提高代码的可测性](#利用测试驱动开发提高代码的可测性)
- 使用 pylint 检查代码风格
- 进行高效的代码审查
- 将包发布到 Pypi

性能剖析与优化

- [了解代码优化的基本原则](#了解代码优化的基本原则)
- [借助性能优化工具](#借助性能优化工具)
- 利用 cProfile 定位性能瓶颈
- 使用 `memory_profile` 和 `objgraph` 剖析内存使用
- 努力降低算法复杂度
- 掌握循环优化的基本技巧
- 使用生成器提交效率 `yield`
- 使用不同的数据结构优化性能
- 充分利用 set 的优势
- 使用 multiprocessing 克服 GIL 的缺陷
- 使用线程池提高效率
- 使用 C/C++ 模块扩展提高性能
- 使用 Cython 编写扩展模块

## 理解 Pythonic 概念

> 充分利用 python 自身特性如变量交换

```py
a, b = b, a
```

> 充分理解内置函数和数据类型

```py
'%(fmt)s' % {'fmt': 'json'}
'{fmt}'.format({
    'fmt': 'json',
})
```

> 包和模块的命名采用短小的小写、单数形式
> 
> 包通常只作为命名空间，如只包含空的 `__init__` 文件

## 编写 Pythonic 代码

- 避免只用大小写来区分不同的对象
- 避免使用容易引起混淆的名称
- 不要害怕过长的变量名
- 全面掌握Python提供给我们的所有特性，包括语言特性和库特性。全面掌握Python提供给我们的所有特性，包括语言特性和库特性
- 深入学习业界公认的比较Pythonic的代码，比如Flask、gevent和requests等
- PEP8

## 理解Python与C语言的不同之处

- `缩进` 与 `{}`
- `与`
- 三元操作符 `:?` 与 `A if True else B`
- `switch case` 与 `{0: 'A', 1: 'B'}.get(0)`

## 在代码中适当添加注释

> 使用块或者行注释的时候仅仅注释那些复杂的操作、算法，还有可能别人难以理解的技巧或者不够一目了然的代码
> 
> 注释和代码隔开一定的距离，同时在块注释之后最好多留几行空白再写代码
> 
> 给外部可访问的函数和方法（无论是否简单）添加文档注释
> 
> 推荐在文件头中包含copyright申明、模块描述等，如有必要，可以考虑加入作者信息以及变更记录

## 通过适当添加空行使代码布局更为优雅、合理

- 在一组代码表达完一个完整的思路之后，应该用空白行进行间隔
- 尽量保持上下文语意的易理解性
- 避免过长的代码行，每行最好不要超过80个字符
- 不要为了保持水平对齐而使用多余的空格，其实使阅读者尽可能容易地理解代码所要表达的意义更重要

## 编写函数的 4 个原则

基本原则

- 函数设计要尽量短小，嵌套层次不宜过深
- 参数声明合理、简单、易于使用
- 函数参数设计应该考虑向下兼容
- 一个函数只做一件事

> Python中函数设计的好习惯还包括：不要在函数中定义可变对象作为默认值，使用异常替换返回错误，保证通过单元测试等

## 将常量集中到一个文件

```py
class _const:
    class ConstError(TypeError): pass
    class ConstCaseError(ConstError): pass
    def __setattr__(self, name, value):
        if self.__dict__.has_key(name):
            raise self.ConstError, "Can't change const.%s" % name
        if not name.isupper():
            raise self.ConstCaseError, \
                  'const name "%s" is not all uppercase' % name
        self.__dict__[name] = value
import sys
sys.modules[__name__]=_const()
```

使用

```py
import const
const.COMPANY = '测试'
```

## 使用 assert 语句来发现问题

> 断言是有代价的，它会对性能产生一定的影响，对于编译型的语言，如C/C++，这也许并不那么重要，因为断言只在调试模式下启用。但Python并没有严格定义调试和发布模式之间的区别，通常禁用断言的方法是在运行脚本的时候加上-O标志，这种方式带来的影响是它并不优化字节码，而是忽略与断言相关的语句

## 数据交换值的时候不推荐使用中间变量

## 充分利用 lazy evaluation 的特性

- `True if x > 5 else False`
- `yield`

## 理解枚举替代实现的缺陷

## 不推荐使用 type 来进行类型检查

> 基于内建类型扩展的用户自定义类型，函数并不能准确返回结果

应使用 `isinstance` 代替 `type(x) == **`

## 警惕 eval 的安全漏洞

> 如果使用对象不是信任源，应该尽量避免使用eval，在需要使用eval的地方可用安全性更好的ast.lit-eral_eval替代

## 避免 finally 中可能发生的陷阱

> 当try块中发生异常的时候，如果在ex-cept语句中找不到对应的异常处理，异常将会被临时保存起来，当finally执行完毕的时候，临时保存的异常将会再次被抛出，但如果finally语句中产生了新的异常或者执行了return或者break语句，那么临时保存的异常将会被丢失，从而导致异常屏蔽

```py
def func(x):
    try:
        if x <= 0:
            raise ValueError
        else:
            return x
    except ValueError as e:
        print(e)
    finally:
        print('finally')
        return -1
func(-5)
func(3)
```

以上的输出结果都是 -1

> 由于存在 finally 语句，执行 else 分支的 return 之前会先执行 finally 中的语句, 而 finally 中会直接返回 -1

## 函数传参既不是传参也不是传引用

> 函数参数在传递的过程中将整个对象传入，对可变对象的修改在函数外部以及内部都可见，调用者和被调用者之间共享这个对象，而对于不可变对象，由于并不能真正被修改，因此，修改往往是通过生成一个新对象然后赋值来实现的

## 警惕默认参数潜在的问题

def 在 python 是一个可执行的语句

> 如果不想让默认参数所指向的对象在所有的函数调用中被共享，而是在函数调用的过程中动态生成，可以在定义的时候使用None对象作为占位符

## 慎用变长参数

不要混用命名参数和 `**kwargs`

## 掌握字符串的基本用法

- 可以用未闭合的小括号拼接多行代码
- Python2 中判断变量是否是字符串时用 `isinstance(x, basestring)`

## 深入掌握 ConfigParser

- 支持定义默认 Section
- 支持字符串拼接

配置文件

```ini
[DEFAULT]
conn_url = %(dbn)s://%(user)s:%(pw)s@%(host)s:%(port)s/%(db)s
dbn = mysql
user = root
host = localhost
port = 3306
db = test
[db1]
user = aaa
pw = ppp
db = example
[db2]
host = 192.168.1.201
pwd = www
db = omb
```

读取配置

```py
import ConfigParser
conf = ConfigParser.ConfigParser()
conf.read('dev.conf')
print conf.get('db1', 'conn_url')
print conf.get('db2', 'conn_url')
```

结果分别是

- `mysql://aaa:ppp@localhost:3306/example`
- `mysql://root:www@192.168.1.201/omb`

## 使用 pandas 处理大型 CSV 文件

- 读取指定部分列和文件的行数
- 设置 CSV 文件与 Excel 兼容
- 对文件进行分块处理并返回一个可迭代对象
- 当文件格式相似的时候 支持多个文件合并处理

## 理解模块 pickle 优劣

- cPickle 速度大概是 pickle 的 1000 倍

## 序列化的另一个不错的选择 JSON

- simplejson 更新较快 可替代 json
- ujson 速度较快

## 使用 logging 记录日志信息

- logging 只是线程安全的 不支持多进程写入同一个文件
- [摘抄者注] logbook 是比较好的第三方扩展

## 用 mixin 模式让程序更加灵活

```py
def simple_tea_people():
    people = People()
    people.__bases__ += ('UseSimpleTeapot', 'UseCoffeepot')
    return people
```

每个类都有一个 `__bases__` 属性，它是一个用来存放所有基类的元组。
当我们向其中增加新的基类时，这个类就用有了新的方法，也就是所谓的混入 `minin`

## 用发布订阅模式实现松耦合

- blinker
- python-message

## 理解名字查找机制

作用域查找顺序

- 局部作用域 local
- 嵌套作用域 enclosing functions locals
- 全局作用域 global
- 内置作用域 build-in

## 理解 MRO 与多继承

菱形继承问题

```py
class A:
    def find(self):
        pass
    def search(self):
        pass
class B(A):
    def find(self):
        pass
class C(A):
    def search(self):
        pass
class D(B, C):
    pass
```

类 D 搜索 `find`/`search` 时根据 A 的定义方式有不同差别，即 MRO(Method Resolution Order) 的实现有差异

具体 MRO 顺序可以通过 `D.__mro__` 得到

设计时应尽量避免这种多继承问题

## 理解描述符机制

- 通过实例访问时，如果 x 是一个描述符，那么 `__getattribute__()` 会返回 `type(obj).__dict__['x'].__get__(obj, type(obj))`
- 通过类访问时，会被 `__getattribute__()` 转换为 `cls.__dict['x'].__get__(None, cls)`

## 区别 __getattr__() 和 __getattribute__() 方法

`__getattr__()` 方法被调用的情况

- 属性不在实例的 `__dict__` 中
- 属性不在基类及祖先类的 `__dict__` 中

`__getattribute__()` 总会被调用，`__getattr__()` 只有在 `__getattribute__()` 中引发异常的情况下才会被调用

## 利用操作符重载实现中缀语法

类似 bash 中的 pipe

```bash
ls | sort
```

通过重载 `__ror__` 方法来实现

## 理解 GIL 的局限性

全局解释锁，绕过 GIL 限制的方法

- multiprocessing 模块
- C 语言扩展
- ctypes

## 对象的管理与垃圾回收

Python 使用引用计数器的方式来管理内存中的对象，当引用计数为 0 时才会被 GC 回收

## 利用测试驱动开发提高代码的可测性

- 编写测试用例
- 运行测试用例
- 开始编码知道所有的测试用例都通过
- 重构

## 了解代码优化的基本原则

- 优先保证代码是可以工作的
- 权衡优化的代价
- 定义性能指标 集中力量解决首要问题
- 不要忽略可读性

## 借助性能优化工具

- psyco
- pypy
