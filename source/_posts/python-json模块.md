---
title: json模块
date: 2019-03-20 17:48:10
tags:
- Python
categories:
- Python
---

## 定义

- json全称：javascript object notation
- 前后端交互的枢纽：后端通过将程序产生的字典转化成json格式的json字符串(串)，然后通过网络传输，给到前端，前端解析json文件，完成数据交互



## python字典与json字符串相互转换

### python字典 ------> json字符串：dumps

- ensure_ascii：默认为True。如果ensure_ascii为False, 那么写入的字符串中可以包含非ASCII字符

```python
import json

dic = {"name": "Tom", "age": 18, "hobby": "篮球", 1: False, 2: None}
s = json.dumps(dic, ensure_ascii=False)
print(s)  # {"name": "Tom", "age": 18, "hobby": "篮球", "1": false, "2": null}
```



### json字符串 ------> python字典：loads

```python
s = '{"name": "Tom", "age": 18, "hobby": "篮球", "1": false, "2": null}'
dic = json.loads(s)
print(dic)   # {'name': 'Tom', 'age': 18, 'hobby': '篮球', '1': False, '2': None}
```



## 读、写json文件

### dump

- 将python字典转换成json字符串，并写入文件中
- indent：缩进

```python
import json

dic = {"name": "Tom", "age": 18, "hobby": "篮球", 1: False, 2: None}
f = open("1.json", "w", encoding="utf-8")  # 把对象打散成json写入到文件中
json.dump(dic, f, ensure_ascii=False, indent=4)
f.close()结果：
```



### load

- 读取json文件，并解析成字典

```python
import json
f = open("1.json", encoding="utf-8")
dic = json.load(f)
f.close()
print(dic)  # {'name': 'Tom', 'age': 18, 'hobby': '篮球', '1': False, '2': None}
```



## 读、写多个json字符串

### 问题

  我们可以向同一文件中写入多个json字符串,但是json文件中内容是一行内容。此时读取时无法读取。

```python
import json

lst = [{"a": 1}, {"b": 2}, {"c": 3}]
f = open("2.json", "w")
for el in lst:
    json.dump(el, f)
f.close()结果：
```

```python
f = open("2.json")
dic = json.load(f)

结果：
json.decoder.JSONDecodeError: Extra data: line 1 column 9 (char 8)
```



### 解决方案

  改用dumps和loads，对每一行分别做处理

```python
import json

lst = [{"a": 1}, {"b": 2}, {"c": 3}]

f = open("2.json", "w")
for el in lst:
    s = json.dumps(el) + "\n"
    f.write(s)
f.close()结果：
```

 

```python
f = open("2.json")
for line in f:
    dic = json.loads(line.strip())
    print(dic)
f. close()结果：{'a': 1}{'b': 2}{'c': 3}
```



##  json处理类

### 将类转换成json字符串

```python
import json


class Person(object):

    def __init__(self, name, age):
        self.name = name
        self.age = age


p = Person("Tom", 18)


def func(obj):
    return {"name": obj.name, "age": obj.age}


s = json.dumps(p, default=func)
print(s)  # {"name": "Tom", "age": 18}
```

### 将json字符串赋给类

```python
class Person(object):

    def __init__(self, name, age):
        self.name = name
        self.age = age


s = '{"name": "Tom", "age": 18}'

def func(dic):
    return Person(dic["name"], dic["age"])

p = json.loads(s, object_hook=func)
print(p.name, p.age)  # Tom  18
```

