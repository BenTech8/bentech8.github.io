---
title: shelve模块
date: 2019-03-20 17:43:47
tags:
- Python
categories:
- Python
---

## shelve

- shelve提供python的持久化操作。
- 持久化：把数据写到硬盘上
- shelve操作与字典非常类似

```python
import shelve

d = shelve.open("tom")
d["name"] = "Tom"
print(d["name"])  # Tom
d.close()
```

文件关闭后无法读取其内容

```python
import shelve

d = shelve.open("tom")
d["name"] = "Tom"
d.close()

print(d["name"])  # ValueError: invalid operation on closed shelf
```

 遍历：

```python
import shelve

d = shelve.open("namelist", writeback=True)

d["name"] = "Tom"
d["age"] = 18
d.close()


d = shelve.open("namelist")

for k in d:  # 遍历所有的key
    print(k)  # name age

for k in d.keys():  # 遍历所有key
    print(k)  # name age

for k, v in d.items():  # 遍历所有的键-值对
    print(k, v)  # name Tom   age 18
d.close()
```



## writeback

- 把修改的内容自动回写到文件中

### 修改内容

shelve.open()默认writeback=False，所以，在默认情况下修改字典的数据，修改内容不会写入到文件中：

```python
import shelve

d = shelve.open("namedict")
d["one"] = {"name": "Tom", "age": 18, "hobby": "football"}
d.close()
```

```python
import shelve

d = shelve.open("namedict")
d["one"]["name"] = "Linda"
d.close()


d = shelve.open("namedict")
print(d["one"])  # {'name': 'Tom', 'age': 18, 'hobby': 'football'}
d.close()
```

 当writeback=True时，修改的数据保存在文件中，再次读取为修改后的结果

```python
import shelve

d = shelve.open("namedict", writeback=True)
d["one"]["name"] = "Linda"
d.close()


d = shelve.open("namedict")
print(d["one"])  # {'name': 'Linda', 'age': 18, 'hobby': 'football'}
d.close()
```



###  删除内容

  在writeback=True条件下，删除内容，文件中内容也会被删除

```python
import shelve

d = shelve.open("namedict", writeback=True)
del d["one"]
d.close()


d = shelve.open("namedict")
print(d["one"])  # KeyError: b'one' 报错
d.close()  
```

