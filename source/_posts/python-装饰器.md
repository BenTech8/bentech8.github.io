---
title: 装饰器
date: 2025-03-16 17:24:16
tags:
- Python
categories:
- Python
---

## 定义

  装饰器本质上就是一个python函数，可以让其他函数在不需要做任何代码变动的前提下，增加额外的功能。装饰器的返回值也是一个函数对象。



## 装饰器种类

### 不带参数的装饰器

```python
import time


def timer(func):
    def inner():
        start = time.time()
        time.sleep(0.2)
        func()
        print(time.time() - start)
    return inner


@timer          # ===> func1 = timer(func1）
def func1():
    print("in func1")


func1()
```



### 带参数的装饰器

```python
import time


def timer(func):
    def inner(*args, **kwargs):
        start = time.time()
        time.sleep(0.2)
        func(*args, **kwargs)
        print(time.time() - start)
    return inner


@timer        # ===> func1 = timer(func1)
def func1(a):
    print(a)


func1("hello")
```



### 带返回值的装饰器

```python
import time


def timer(func):
    def inner(*args, **kwargs):
        start = time.time()
        time.sleep(0.2)
        ret = func(*args, **kwargs)
        print(time.time() - start)
        return ret
    return inner


@timer        # ===> func1 = timer(func1)
def func1(a, b):
    print("in func1")


@timer        # ===> func2 = timer(func2)
def func2(a):
    print("in func2 and get a:%s" % (a))
    return "func2 over"


func1("hello", "world")
print(func2("aaaa"))
```

  当查看函数信息的方法时，在此处失效：

```python
import time


def timer(func):
    def inner(*args, **kwargs):
        start = time.time()
        time.sleep(0.2)
        ret = func(*args, **kwargs)
        print(time.time() - start)
        return ret
    return inner

@timer
def index():
    """这是一个主页信息"""
    print("from index")


print(index.__doc__)
print(index.__name__)


# 结果：
None
inner
```

  解决方案：

```python
import time
from functools import wraps


def timer(func):
    @wraps(func)
    def inner(*args, **kwargs):
        start = time.time()
        time.sleep(0.2)
        ret = func(*args, **kwargs)
        print(time.time() - start)
        return ret
    return inner


@timer
def index():
    """这是一个主页信息"""
    print("from index")


print(index.__doc__)
print(index.__name__)


# 结果：
这是一个主页信息
index
```



## 开放封闭原则

  装饰器完美的遵循了 开放封闭原则



### 对扩展开放

  任何一个程序，不可能在设计之初就已经想好了所有的功能并且未来不做任何更新和修改。所以必须允许代码扩展、添加新功能。



### 对修改封闭

  我们写的一个函数，很有可能已经交付给其他人使用了，如果这个时候对其进行修改，很有可能影响其他已经在使用该函数的用户。



## 装饰器的固定结构

```python
def timer(func):
    def inner(*args, **kwargs):
        """执行函数之前要做的"""
        ret = func(*args, **kwargs)
        """执行函数之后要做的"""
        return ret
    return inner
```

```python
from functools import wraps


def timer(func):
    @wraps(func)
    def inner(*args, **kwargs):
        """执行函数之前要做的"""
        ret = func(*args, **kwargs)
        """执行函数之后要做的"""
        return ret
    return inner
```



## 装饰器带参数

  应用场景：如果有成千上万个函数使用了一个装饰器，现在想把这些装饰器取消掉

```python
def outer(flag):
    def timer(func):
        def inner(*args, **kwargs):
            if flag:
                print("执行函数之前要做的")
            ret = func(*args, **kwargs)
            if flag:
                print("执行函数之后要做的")
            return ret
        return inner
    return timer


@outer(False)
def func():
    print(111)


func()
```



## 多个装饰器装饰一个函数

```python
def wrapper1(func):   # func = f
    def inner1():
        print("wrapper1, before func")
        func()
        print("wrapper1, after func")
    return inner1


def wrapper2(func):  # func = inner1
    def inner2():
        print("wrapper2, before func")
        func()
        print("wrapper2, after func")
    return inner2


@wrapper2    # f = wrapper2(f) ===> f = inner2
@wrapper1    # 先装饰wrapper1 ---> f = wrapper1(f) ===> f = inner1
def f():
    print("in f")


f()   # ===> inner2()
```

  规律：

  对于n个装饰器装饰同一个函数，例如靠近函数名f的为@outer_1，最上层为@outer_n，那么执行f()时，执行顺序为：

- 第n个装饰器执行f()前要做的 ---> 第n-1个装饰器执行f()前要做的 ---> ...--->第1个装饰器执行f()前要做的 --->
- f() --->
- 第1个装饰器执行f()要做的 ---> 第二个装饰器执行f()后要做的 --->...--->第n个装饰器执行f()后要做的
