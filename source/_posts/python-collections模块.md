---
title: collections模块
date: 2019-03-20 17:30:26
tags:
- Python
categories:
- Python
---

## collections

  collections模块主要封装了一些关于集合类的相关操作。比如，iterable，Iteratort等等，除此之外，collections还提供了一些除基本数据类型以外的数据集合类型。Counter，deque，OrderDict，defaultdict以及namedtuple



### Counter

- 计数器，主要用来计数

```python
from collections import Counter

s = "Hello word"
print(Counter(s))  # Counter({'l': 2, 'o': 2, 'H': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})

lst = [11, 22, 33, 44, 55, 66, 55, 44, 44]
el = Counter(lst)

print(el[44])  # 3
```



###  双向队列

- 既可从左边添加数据，也可从右边添加数据
- 既可从左边获取数据，也可从右边获取数据

```python
from collections import deque

q = deque()

q.append(11)
q.append(22)
q.appendleft(33)

print(q)              # deque([33, 11, 22])
print(q[0])           # 33

print(q.pop())        # 22
print(q.popleft())    # 33
```



### namedtuple(命名元组)

- 给元组内的元素进行命名

```python
from collections import namedtuple

nt = namedtuple("Point", ["x", "y"])
p = nt(1, 2)

print(p.x)  # 1
print(p.y)  # 2
```



### OrderedDict

- 有序字典，按照存储的顺序保存



### defaultdict

- 给字典设置默认值。当key不存在时，直接获取默认值
- defaultdict括号内对象必须为可调用对象

```python
from collections import defaultdict

dic1 = defaultdict(list)
print(dic1["name"])       # []
print(dic1)               # defaultdict(<class 'list'>, {'name': []})


def func():
    return 'Tom'

dic = defaultdict(func)
print(dic["name"])        # Tom
```

 应用：

```python
# 将lst = [11, 22, 33, 44, 55, 66, 77, 88, 99]大于等于66放入字典"key1"中，小于66放入字典"key2"中
from collections import defaultdict

lst = [11, 22, 33, 44, 55, 66, 77, 88, 99]
dic = defaultdict(list)
for el in lst:
    if el < 66:
        dic["key2"].append(el)
    else:
        dic["key1"].append(el)

dic = dict(dic)
print(dic)  # {'key2': [11, 22, 33, 44, 55], 'key1': [66, 77, 88, 99]}
```
