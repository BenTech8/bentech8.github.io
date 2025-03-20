---
title: 异常
date: 2019-03-20 17:05:02
tags: 
- Python
categories:
- Python
---

## 异常

### 定义

- 程序在运行过程中产生的错误

### 结构

- 产生异常：python解释器调用代码，出现错误，系统自动产生一个异常对象，并把错误返回给调用方
- 捕获异常：通过try...except可以对系统产生的异常进行捕获
- 处理异常：如果except捕获到了相对应的异常，则在except代码块里处理异常，否则python解释器将错误返回调用者
- 别名：指接收到的异常对象

 

```
try:
    代码块
except 异常名称 as 别名: 
    代码块
except 异常名称 as 别名:
    代码块
...
else:
    不产生异常时执行这里的代码
finally:
    不管有无异常，都执行这里的代码
```



###  Exception

  Exception是所有异常的基类。换句话说，所有的错误都是Exception的子类对象。

## 抛出异常（raise）

```python
import traceback


def cul(a, b):
    if (type(a) == int or type(a) == float) and (type(b) == int or type(b) == float):
        return a + b
    else:
        raise Exception("参数数据类型有误")

try:
    print(cul(1, "a"))
except Exception as e:
    print(e)  # 参数数据类型有误
    print(traceback.format_exc())  # 获取堆栈信息
    print("出现了错误")  # 出现了错误


# print(traceback.format_exc())结果：
Traceback (most recent call last):
  File "E:/python_个人/day 019/临时.py", line 11, in <module>
    print(cul(1, "a"))
  File "E:/python_个人/day 019/临时.py", line 8, in cul
    raise Exception("参数数据类型有误")
```

 

## 自定义异常（慎用）

- 自定义一个类，并继承Exception类，类中内容全部继承Exception。就自定义了一个异常
- 名字须符合规范

```python
class GenderError(Exception):
    pass


class Person(object):
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender


def gender_judge(obj):
    if obj.gender != "男":
        raise GenderError("性别不对")


p1 = Person("Tom", "男")
p2 = Person("Linda", "女")

try:
    gender_judge(p1)
    gender_judge(p2)
except GenderError as e:
    print(e)  # 性别不对
```
