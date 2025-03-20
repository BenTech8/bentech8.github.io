---
title: 区分函数与方法
date: 2019-03-20 16:40:36
tags:
- Python
categories:
- Python
---

## 通过print打印区分

- 函数在打印的时候，显示的是function
- 方法在打印的时候，显示的是method

```python
def func():
    pass

class Animal(object):

    def run(self):
        pass

print(func)  # <function func at 0x0000022343CD1F28>

c = Animal()
print(c.run)  # <bound method Animal.run of <__main__.Animal object at 0x0000022344128400>>
```



## MethodType与FunctionType

### 实例方法

- 通过**对象.方法名**，得到的是**方法**类型
- 通过**类名.方法名**，得到的是**函数**类型

```python
from types import FunctionType, MethodType


class Foo(object):

    def run(self):
        pass


fn = Foo()
print(isinstance(fn.run, MethodType))     # True
print(isinstance(Foo.run, FunctionType))  # True
```



### 静态方法

- 通过**对象.方法名**，得到的是**函数**类型
- 通过**类名.方法名**，得到的是**函数**类型

```python
from types import FunctionType, MethodType


class Foo(object):

    @staticmethod
    def run():
        pass

fn = Foo()
print(isinstance(fn.run, FunctionType))   # True
print(isinstance(Foo.run, FunctionType))  # True
```

 

### 类方法

- 通过**对象.方法名**，得到的是**方法**类型
- 通过**类名.方法名**，得到的是**方法**类型

```python
from types import FunctionType, MethodType


class Foo(object):

    @classmethod
    def run(cls):
        pass

fn = Foo()
print(isinstance(fn.run, MethodType))   # True
print(isinstance(Foo.run, MethodType))  # True
```
