---
title: random模块
date: 2019-03-20 17:36:06
tags:
- Python
categories:
- Python
---

## random

  所有关于随机相关的内容都在random模块中

## 相关功能

### random()

- 产生(0, 1)之间的小数

```
import random

print(random.random())  # 0.2622839912972649
```



### uniform(a, b)

- 产生(a, b)之间的小数

```
import random

print(random.uniform(3, 10))  # 3.1025846246337134
```



### randint(a, b)

- 产生[a, b]之间的整数

```
import random

print(random.randint(1, 10)) # 7
```



### randrange(a, b, step)

- 产生[a:b:step]里的一个数
- step默认等于1

```
import random

print(random.randrange(1, 10, 2)) # 3
```



### choice(string/tuple/list)

- 产生对象里的其中一个元素

```
import random

print(random.choice([11, 22, 33, 44, [55, 66, 77]]))  # 44
```



### sample(string/tuple/list/set)

- 从对象中任选两个元素，以列表形式返回

```
import random

print(random.sample([11, 22, 33, 44], 2))  # [44, 22]
```



### shuffle(list)

- 随机打乱顺序
- 返回None

```
import random

lst = [11, 22, 33, 44, 55]
random.shuffle(lst)
print(lst)  # [11, 55, 44, 33, 22]
```
