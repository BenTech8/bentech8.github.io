---
title: Set
date: 2025-03-16 16:51:30
tags:
- Python
categories:
- Python
---

## 简介

- set中的元素：可hash(int, str, tuple, bool)，不重复，底层存储无序
- set集合中的元素必须是可hash，但set本身是不可hash，是可变的数据类型



## 应用(重点)

  利用set中元素不可重复，可给list去重

```
lst = [11, 11, 22, 22, 33, 33, 44, 44, 55, 55]

lst = list(set(lst))
print(lst)  # [33, 11, 44, 22, 55]  结果无序，但已去重
```



## 增删改查

### 增

- add

  重复的内容不会被添加到set集合中，新增对象须为可hash数据类型

```
s = {11, 22, 33, 44}

s.add("hello")
print(add)  # {33, 11, 44, 'hello', 22}

s.add(11)
print(s)  # {33, 11, 44, 'hello', 22}
```

- update

  迭代更新，新增对象可为可hash或不可hash数据类型

```
s = {11, 22, 33, 44}

s.update("hello")
print(s)  # {33, 'l', 11, 44, 'e', 22, 'o', 'h'}
s = {11, 22, 33, 44}

s.update([55, 66, 77])
print(s)  # {33, 66, 11, 44, 77, 22, 55}
```



### 删

- pop

  随机弹出一个

```
s = {11, 22, 33, 44}

num = s.pop()
print(s) # {11, 44, 22}
print(num) # 33
```

- remove

  根据元素内容进行删除，若元素不存在，则报错

```
s = {11, 22, 33, 44}

s.remove(11)
print(s)  # {33, 44, 22}
```

- clear

  清空set集合，如果set集合是空的，打印出来是set()，须与dict区分

```
s = {11, 22, 33, 44}

s.clear()
print(s)  # set()
```



### 改

- set集合中数据没有索引，也没法定位一个元素，所以没有办法进行直接修改
- 可采用先删除后添加方式完成修改

```
s = {11, 22, 33, 44}

s.remove(11)
s.add(55)
print(s)  # {33, 44, 22, 55}
```



### 查

  set是一个可迭代对象，可进行for循环

```
for el in s:
    print(el)
```



## 常用操作

### 交集

- &

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1 & s2)  # {33, 44}
```

- intersection

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1.intersection(s2)) # {33, 44}
```



### 并集

- |

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1 | s2)  # {33, 22, 55, 11, 44}
```

- union

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1.union(s2))  # {33, 22, 55, 11, 44}
```



### 差集

  s1 - s2：得到s1中单独存在的

- \-

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1 - s2)  # {11, 22}
```

- difference

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1.difference(s2))  # {11, 22}
```



### 反交集

  s1 ^ s2：得到一个新的集合，里面元素为两集合（s1, s2）不相同的元素

- ^

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1 ^ s2)  # {22, 55, 11}
```

- symmetric_difference

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1.symmetric_difference(s2))  # {22, 55, 11}
```



### 子集

  s1 < s2:判断s1是否为s2子集，返回True or False

- <

```
s1 = {11, 22, 33, 44}
s2 = {33, 44, 55}

print(s1 < s2)  # False
```

- issubset

```
s1 = {11, 22, 33, 44}
s2 = {11, 22, 33, 44, 55}

print(s1.issubset(s2)) # True
```



### 超集

  s1 > s2：判断s1是否为s2的超集，也即s2是否为s1的子集，返回True or False

- \>

```
s1 = {11, 22, 33, 44}
s2 = {11, 22, 33}

print(s1 > s2)  # True
```

- issuperset

```
s1 = {11, 22, 33, 44}
s2 = {11, 22, 33}

print(s1.issuperset(s2))  # True
```



### frozenset

- frozenset是一种不可变，可hash的集合
- 通过frozenset可将可变的数据类型转换成不可变，可hash的frozenset集合

```
s = [44, 55, 66]

s1 = {11, 22, 33, s}  # 报错  TypeError: unhashable type: 'list'
s = frozenset([44, 55, 66])print(s)  # frozenset({66, 44, 55})

s1 = {11, 22, 33, s}  # 正常
 
s = frozenset([1,2,3,4,5])
print(s)  # frozenset({1, 2, 3, 4, 5})
for el in s:
    print(el)  # 1 2 3 4 5
```

 
