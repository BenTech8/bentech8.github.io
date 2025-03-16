---
title: 迭代器
date: 2025-03-16 17:12:03
tags:
- Python
categories:
- Python
---

## 迭代器

- Iterable：可迭代对象，内部包含__iter__()函数
- Iterator：迭代器，内部包含__iter__()和__next__()函数

特点：

- 节省内存
- 惰性机制
- 不能反复，只能向下执行



### 可迭代对象

#### 判断一个对象是否为可迭代对象

- 通过dir函数来查看类中定义的方法中是否有__iter__方法

如果__iter__能找到，那么这个类的对象就是一个可迭代对象。

```python
print(dir(str))print(dir(list))


结果：
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'clear', 'copy', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']
```

- 通过isinstance()函数

如果返回值为True,则该对象为可迭代对象，否则不是。

```
from collections import Iterable

lst = [11, 22, 33]

print(isinstance(lst, Iterable))  # True
```



### 迭代器

- 迭代器本身是可迭代的，所以类中有__iter__方法。

#### 判断一个对象是否为迭代器

- 通过dir函数来查看类中定义的方法中是否有__next__方法

```
lst = [11, 22, 33]
ret = lst.__iter__()

print("__next__" in dir(ret))  # True
```

- 通过isinstance()函数

```
from collections import Iterator

lst = [11, 22, 33]
ret = lst.__iter__()

print(isinstance(ret, Iterator))  # True
```



####  迭代器取值

  通过__next__获取迭代器数据

```
lst = [11, 22, 33]
ret = lst.__iter__()

print(ret.__next__())
print(ret.__next__())
print(ret.__next__())


结果：
11
22
33
```

  如果获取元素个数超过迭代器元素的长度，则报错StopIteration

```
lst = [11, 22, 33]
ret = lst.__iter__()

print(ret.__next__())
print(ret.__next__())
print(ret.__next__())
print(ret.__next__())  # StopIteration
```



#### for循环内部代码

```
ret = lst.__iter__()

while 1:
    try:
        el = ret.__next__()
        print(el)
    except StopIteration:
        break
```



### 可迭代对象 -----> 迭代器

- 可迭代对象不一定是迭代器，但可通过__iter__方法转换成迭代器

```
lst = [11, 22, 33]
ret = lst.__iter__()

print("__next__" in dir(ret))  # True
```

 
