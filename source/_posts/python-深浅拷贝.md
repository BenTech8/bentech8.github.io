---
title: 深浅拷贝
date: 2019-03-15 22:50:49
tags: Python
categories: Python
---

## =

- 没有产生新对象，都是内存地址的赋值

```
lst1 = [11, 22, 33, 44]
lst2 = lst1
print(id(lst1)) # 1916361712712
print(id(lst2)) # 1916361712712

lst1.append("hello")
print(lst1)  # [11, 22, 33, 44, 'hello']
print(lst2)  # [11, 22, 33, 44, 'hello']
```

```
dic1 = {"name": "Tom", "age": 18}
dic2 = dic1
print(id(dic1)) # 2020711820888
print(id(dic2)) # 2020711820888
dic1["hobby"]  = "football"

print(dic1)  # {'name': 'Tom', 'age': 18, 'hobby': 'football'}
print(dic2)  # {'name': 'Tom', 'age': 18, 'hobby': 'football'}
```

  对于list，set，dict来说，变量1 = 变量2，其实就相当于把变量2内容的内存地址交给变量1，并不是复制一份内容，所以一变，都变。 



## 浅拷贝

- 优点：省内存
- 缺点：只拷贝第一层内容
- 浅拷贝方法：copy()，对于list，还有lst[:]

```
lst1 = [11, 22, 33]
lst2 = lst1[:]

print(id(lst1))  # 1744651035720
print(id(lst2))  # 1744651035848

lst1.append(44)

print(lst1) # [11, 22, 33, 44]
print(lst2) # [11, 22, 33]
```

```
lst1 = [11, 22, 33, [44, 55]]
lst2 = lst1.copy()

print(id(lst1))  # 2081315824712
print(id(lst2))  # 2081316649864

lst1[3].append("hello")

print(lst1) # [11, 22, 33, [44, 55, 'hello']]
print(lst2) # [11, 22, 33, [44, 55, 'hello']]
```



## 深拷贝

- 深拷贝把内部元素完全进行拷贝复制。

  深拷贝方法：

```
import copy

s2 = copy.deepcopy(s1)
```

```
import copy

lst1 = [11, 22, 33, [44, 55]]
lst2 = copy.deepcopy(lst1)

print(id(lst1))  # 1236356428872
print(id(lst2))  # 1236356430152

lst1[3].append("hello")

print(lst1) # [11, 22, 33, [44, 55, 'hello']]
print(lst2) # [11, 22, 33, [44, 55]]
```
