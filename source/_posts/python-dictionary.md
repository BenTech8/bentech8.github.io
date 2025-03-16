---
title: Dictionary
date: 2025-03-16 16:47:32
tags:
- Python
categories:
- Python
---

## 字典的简单介绍

- 字典(dict)是python中唯一的一个映射类型。是以{ }括起来的键值对组成。
- 字典(dict)中key必须是不可变的，而value没有要求，可以保存任意类型的数据。

### 字典的保存原理：

  字典在保存的时候，采用的是hash算法：根据key来计算出一个内存地址，然后将key-value保存在这个地址中。所以，在dict中存储的key-value中的key必须是可hash的。

  对于可hash，暂可以理解为可以改变的都是不可hash的，那么可哈希就意味着不可变，这个是为了能准确的计算内存地址而规定的。

- 已知的可hash(不可变)的数据类型：int, str, tuple, bool
- 不可hash(可变)的数据类型：list, dict, set
- dict保存的数据不是按照我们添加进去的顺序保存的，是按照hash表的顺序保存的，而hash表不是连续的。所以不能进行切片工作，只能通过key来获取dict中的数据。

```
dic = {"name": "Tom", "age": 18}

print(dic[name]) # Tom
```



## 字典的增删改查

### 增

- dic[key] = value

   若字典中无此key,则添加键值对，若字典中有此key,则为修改value内容

```
dic = {"name": "Tom", "age":18}

dic["hobby"] = "football" 
print(dic)  # {"name": "Tom", "age": 18, "hobby": "football"}  增加键-值对
dic = {"name": "Tom", "age": 18}

dic["age"] = 20
print(dic) # {"name": "Tom", "age": 20} 修改value内容

```

- setdefault

流程：

a) 判断key是否存在，如果存在，就不执行新增，返回key对应value内容

b) 如果不存在，执行新增，并返回key对应value内容

```
dic = {"name": "Tom", "age": 18}

ret = dic.setdefault("hobby", "football")
print(dic)  # {"name": "Tom", "age": 18, "hobby": "football"}
print(ret)  # football
dic = {"name": "Tom", "age": 18}

ret = dic.setdefault("name", "Andy")
print(dic)  # {"name": "Tom", "age": 18}
print(ret)  # Tom
```

当采用setdefault方法新增键值对时，若只提供key,则默认value为None

```
dic = {"name": "Tom", "age": 18}

dic.setdefault("hobby")
print(dic) # {"name": "Tom", "age": 18, "hobby": None}
```



### 删

- pop

根据key删除，有返回值

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

ret = dic.pop("hobby")
print(dic)  # {"name": "Tom", "age": 18}print(ret) # football
```

- popitem

随机删除(3.5以下为随机删除，3.5以上为删除末尾的，但python底层为随机删除)

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

dic.popitem()
print(dic)  # {"name": "Tom", "age": 18}
```

- del

```
根据key删除
dic = {"name": "Tom", "age": 18, "hobby": "football"}

del dic["hobby"]
print(dic)  # {"name": "Tom", "age": 18}
```

- clear

清空字典

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

dic.clear()
print(dic)  # {}
```



### 改

- dic[key] = value

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

dic["age"] = 20
print(dic) # {"name": "Tom", "age": 20, "hobby": "football"}
```

- update

 dic.update(dic1)：把dic1中的内容更新到dic中。如果key重名，则修改替换，如果不存在，则新增

```
dic = {"name": "Tom", "age": 18}
dic1 = {"name": "Andy", "hobby": "football"}

dic.update(dic1)

print(dic) # {"name": "Andy", "age": 18, "hobby": "football"}
print(dic1) # {"name": "Andy", "hobby": "football"}
```



### 查

- print(dic[key])

当key不存在时，报错

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

print(dic["name"])  # Tom
```

- get

当key不存在时，返回None

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}


print(dic.get("name")) # Tom
print(dic.get("addr")) # None
```

- setdefault

当key存在时，则返回value值，但当key不存在时，则新增key-value对。

```
dic = {"name": "Tom", "age": 18, "hobby": "football"}

print(dic.setdefault("name")) # Tom
```

##  

## 其他相关操作

### dic.keys()

遍历字典所有key:

```
for k in dic.keys():
    print(k)
for k in dic:
    print(k）
    print(dic[k]) 
```



### dic.values()

遍历字典所有value:

```
for v in dic.values():
    print(v)
```



### dic.items()

遍历字典的最好方案：

```
for k, v in dic.items():
    print(k, v)
```



###  fromkeys

- fromkeys属于类dict的一个静态方法
- 创建新字典，不是在原有基础上增加键值对
- 如果value为可变数据数据。则所有key都可改动这个数据，一旦改动，所有的value跟着改变

```
dic = dict.fromkeys(["name_1", "name_2"], [11, 22, 33])

print(dic) # {'name_1': [11, 22, 33], 'name_2': [11, 22, 33]}
dic = dict.fromkeys([``"name_1"``, ``"name_2"``], [11, 22, 33])` `dic[``'name_1'``].append(``"hello"``)` `print(dic) # {``'name_1'``: [11, 22, 33, ``'hello'``], ``'name_2'``: [11, 22, 33, ``'hello'``]}
```



## dict在迭代过程中删除元素问题

### dict中元素在迭代过程中删除，则报错

```
dic = {"name": "Tom", "age": 18}

for k in dic:
    del dic[k]  # RuntimeError: dictionary changed size during iteration
```



### 解决方案

  可以先把要删除的元素保存在一个list中，然后循环list，删除字典中元素

```
dic = {'k1': 11, "k2": 22, "s1": 33, "s2": 44}

lst = []
for k in dic:
    if "k" in k:
        lst.append(k)

for i in lst:
    del dic[i]

print(dic)  # {'s1': 33, 's2': 44}
```



## 字典嵌套

字典里可以嵌套多层列表、字典

```python
dic = {
        "name": "Tom",
        "age": 30,    
        "hobby": ["football", "baseball"],
        "wife": {
                "name": "Andy",
                "age": "28",
        }
}

print(dic.get("wife").get("name")) # Andy
print(dic["hobby"][1])  # baseball
```
