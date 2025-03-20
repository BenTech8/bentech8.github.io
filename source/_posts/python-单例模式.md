---
title: 单例模式
date: 2019-03-20 16:50:47
tags:
- Python
categories:
- Python
---

## 单例模式

单例模式(Singleton Pattern)， 是一种常用的软件设计模式。该模式的主要目的是确保某一个类只有一个实例存在。当你希望在整个系统中，某个类只能出现一个实例时，单例对象就能派上用场。

例如，某个服务器程序的配置信息存放在一个文件中，客户端通过一个AppConfig的类来读取配置文件的信息。如果在程序运行期间，有很多地方需要使用配置文件的内容，即很多地方都需要创建AppConfig对象的实例，这就导致系统中存在多个AppConfig的实例对象，而这样会严重浪费内存资源，尤其是在配置文件内容很多的情况下。事实上，类似AppConfig这样的类，我们希望在程序运行期间只存在一个实例对象。



## 创建单例对象

在Python中，可用多种方法实现单例模式：

- 使用模块
- 使用\__new__
- 使用装饰器(decorator)
- 使用元类(metaclass)



### 使用 \__new__

```python
class Singleton(object):
    _instance = None

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super().__new__(cls, *args, **kwargs)
        return cls._instance


one = Singleton()
two = Singleton()

print("one", one)  # one <__main__.Singleton object at 0x00000289BFAFA978>
print("two", two)  # two <__main__.Singleton object at 0x00000289BFAFA978>
```



### 使用模块

  Python的模块就是天然的单例模式，因为模块在第一次导入时，会生成.pyc文件，当第二次导入时，就会直接加载.pyc文件，而不会再次执行模块代码。因此，只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果想要一个单例类，可以考虑这样做：

```python
# mysingleton.py
class My_Singleton(object):
    def foo(self):
        pass


my_singleton = My_Singleton()
```

  将上面的代码保存在文件mysingleton.py中，然后这样使用：

```python
from mysingleton import my_singleton

my_singleton.foo()
```



## 单例模式优缺点

### 优点

实例控制

  单例模式会阻止其他对象实例化其自己的单例对象的副本，从而确保所有对象都访问唯一实例

灵活性

  因为类控制了实例化过程，所以类可以灵活更改实例化过程



### 缺点

开销

  虽然数量很少，但如果每对象请求引用时都要检查是否存在类的实例，将仍然需要一些开销。可以通过使用静态初始化解决此问题

可能的开发混淆

  使用单例对象(尤其在类库中定义的对象)时，开发人员必须记住自己不能使用new关键字实例化对象。因为可能无法访问库源代码，因此应用程序开发人员可能会意外发现自己无法直接实例化此类

对象生存期

  不能解决删除单个对象的问题。在提供内存管理的语言中(例如基于.NET Framework的语言)，只有单例类能够导致实例被取消分配，因为它包含对该实例的私有引用。在某些语言中(如c++)，其他类可以删除对象实例，但这样会导致单例类中出现悬浮引用
