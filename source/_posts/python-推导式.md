---
title: 推导式
date: 2025-03-16 16:54:43
tags:
- Python
categories:
- Python
---

## 列表推导式

- 写点：[结果 for 变量 in 可迭代对象 if 判断]

```
lst = [i for i in range(1, 20) if i % 2 == 0]
print(lst) # [2, 4, 6, 8, 10, 12, 14, 16, 18]
```



## 字典推导式

- 写法：[结果 for 变量 in 可迭代对象 if 判断]

```
lst = [11, 22, 33]
dic = {i: lst[i] for i in range(len(lst))}
print(dic)  # {0: 11, 1: 22, 2: 33}
```



## 集合推导式

- 写法：[结果 for 变量 in 可迭代对象 if 判断]

```
lst = [1, 8, 33, 44, -1, -8, 12]
s = {abs(i) for i in lst}
print(s)  # {1, 33, 8, 44, 12}
```

结论：

- 推导式比较耗内存。一次加载。而生成器表达式几乎不占用内存。使用的时候才分配和使用内存。
