---
title: 类的约束
date: 2019-03-20 16:43:11
tags:
- Python
categories:
- Python
---

## 提取父类

- 在父类中定义好方法，在这个方法中什么都不用干。就抛出一个异常就可以了。这样所有的子类都必须重写这个方法。否则，访问时就会报错

```python
class Base(object):
    def login(self):
        raise NotImplementedError("子类没有实现该方法")   # NotImplementedError 没有实现的错误


class Normal(Base):
    def login(self):
        pass


class Member(Base):
    def denglu(self):
        pass


class Admin(Base):
    def login(self):
        pass


# 项目经理总入口
def login(obj):
    print("准备验证码...")
    obj.login()
    print("进入主页...")


n = Normal()
m = Member()
a = Admin()

login(n)
login(m)  # 报错
login(a)
```



##  使用抽象类（不推荐）

- 使用抽象类描述父类，在抽象类中给出一个抽象方法。这样子类就不得不给出抽象方法的具体实现，否则报错，也可以起到约束的效果

### 抽象类

- python不能直接实现抽象类，需借助abc模块实现，ABCMeta是实现抽象基础类的元类，由它生成抽象类，只能被继承，不能实例化。
- 抽象类不能创建对象
- 抽象类中可以有正常的方法
- 子类必须重写父类中的抽象方法，否则子类也是一个抽象类
- 接口：类中全部方法都是抽象方法
- 如果一个类中包含了抽象方法。那么这个类一定是一个抽象类

```python
from abc import ABCMeta, abstractmethod


class Base(metaclass=ABCMeta):
    @abstractmethod
    def login(self):
        pass


class Normal(Base):

    def login(self):
        pass


class Member(Base):

    def denglu(self):
        pass


class Admin(Base):

    def login(self):
        pass


n = Normal()
n.login()

m = Member()  # 报错 Can't instantiate abstract class Member with abstract methods login  
m.login()

a = Admin()
a.login()
```
