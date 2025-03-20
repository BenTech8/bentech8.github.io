---
title: 包
date: 2019-03-20 18:03:39
tags:
- Python
categories:
- Python
---

## package

  包是一种通过".模块名"来组织python模块名称空间的方式。我们创建的每个文件夹都可以被称之为包。python2中规定，包内必须存在\__init__.py文件。创建包的目的不是为了运行，而是被导入使用。包只是一种形式而已。

- 本质：一种模块

### 被称之为包所具备的条件：

- 文件夹
- 有__init__.py文件（python2中必须有，python3可省略，但最好保留）

### 导入包时执行的三个动作

- 判断文件夹是否已经被导入。如果已导入，则不会再重复导入，也就不会执行下面两个操作
- 开辟一片内存空间，在该空间中执行包的\__init__.py文件
- 创建包的名字。并使用该名称作为该包在当前模块中引用的名字

### 包的注意事项：

- 关于包相关的导入语句也分为import和from xxx import xxx两种，但无论使用哪种，无论在什么位置，在导入时都必须遵循一个原则：凡是在导入带点的，点左边都是一个包，否则报错。可以带一连串的点，如a.b.c
- import导入文件时，产生名称空间中的名字来源于文件。import包，产生的名称空间中的名字同样来源于文件。即包下的\__init__.py文件，导入包的本质就是导入该文件。
- 包A和包B下同名模块也不会冲突。如A.a和B.a来自两个不同的名称空间
- 导包时，需注意sys.path路径范围，否则易超出范围而报错。

```python
import os

os.makedirs('glance/api')
os.makedirs('glance/cmd')
os.makedirs('glance/db')

l = []

l.append(open('glance/__init__.py', 'w'))
l.append(open('glance/api/__init__.py', 'w'))
l.append(open('glance/api/policy.py', 'w'))
l.append(open('glance/api/versions.py', 'w'))
l.append(open('glance/cmd/__init__.py', 'w'))
l.append(open('glance/cmd/manage.py', 'w'))
l.append(open('glance/db/__init__.py', 'w'))
l.append(open('glance/db/models.py', 'w'))

map(lambda f: f.close(), l)
```

各文件中写有如下内容：

```python
# policy.py
def get():
    print("from policy.py")

# versions.py
def create_resource(conf):
    print("from version.py: ", conf)

# manage.py
def main():
    print("from manage.py")

# models.py
def register_models(engine):
    print("from models.py: ", engine)
```



## 包的导入方式

### import xxx

```python
# 与glance文件夹同目录下test.py
import glance.db.models

glance.db.models.register_models("mysql")


结果：
from models.py:  mysql
```



### from xxx import xxx

  采用from xxx import xxx这种形式，import后面不可以出现"."。也就是说，from a.b import c是正确的，而from a import b.c是错误的

```python
# 与glance文件夹同目录下test.py
from glance.db.models import register_models

register_models("mysql")

结果：
from models.py:  mysql
```



##  \__init__.py

  不论使用哪种方式导入一个包，只要是第一次导入包或者是包的任何其它部分。都会先执行\__init__.py文件。

```python
# glance文件夹下的__init__.py文件
print("我是glance的__init__.py文件")
```

```python
# 与glance文件夹同目录下test.py
import glance


结果：
我是glance的__init__.py文件
```



## 绝对导入

  绝对导入，是按sys.path路径寻找包，如果包不在sys.path列表中，可将包移至路径中。也可append包路径到sys.path中，但一般不这样做。

```python
# policy.py
from glance.api.versions import create_resource

def get():
    create_resource("mysql")
    print("from policy.py")
```



```python
# 与glance文件夹同目录下的test.py
from glance.api.policy import get

get()

结果：
我是glance的__init__.py文件
from version.py:  mysql
from policy.py
```



## 相对导入

  在相对导入中，python不允许你运行的程序导包的时候超过当前包的范围。

```python
# policy.py
from ..cmd import manage

def get():
    manage.main()
    print("from policy.py")

get()

结果：
ValueError: attempted relative import beyond top-level package
```

如果在包内使用了相对导入，那么在使用该包内信息的时候，只能在包外面导入

在包内导入则报错：

```python
# policy.py
from . import versions

def get():
    versions.create_resource("mysql")
    print("from policy.py")

get()

结果：
ImportError: cannot import name 'versions'
```

在包外导入，运行成功：

```python
# policy.py
from . import versions

def get():
    versions.create_resource("mysql")
    print("from policy.py")
```

```python
# 与glance文件夹同目录下的test.py
from glance.api.policy import get

get()


结果：
我是glance的__init__.py文件
from version.py:  mysql
from policy.py
```



##  单独导入一个包

  单独导入一个包glance,，如果在包中\__init__.py文件中并没有关于子包的加载，此时导入的glance什么都做不了。

  在\__init__.py中加载子包可以使用绝对路径加载，也可使用相对路径加载

```python
# glance文件夹下的__init__.py文件
from glance import api
# glance/api/__init__.py文件

from glance.api import policy
# glance/api/policy.py

def get():
    print("from policy.py")
```

```python
# 与glance文件夹同目录下的test.py
import glance

glance.api.policy.get()


结果：
from policy.py
```
