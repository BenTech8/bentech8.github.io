---
title: pickle模块
date: 2019-03-20 17:45:49
tags:
- Python
categories:
- Python
---

## 序列化

  在存储数据或者网络传输数据的时候，需要对对象进行处理。把对象处理成方便存储和传输的数据格式。这个过程叫**序列化**。

  不同的序列化，结果也不同，但目的是一样的。都是为了存储和传输。

在python中存在三种序列化的方案：

- pickle：可将python中的任意数据类型转化成bytes并写入到文件中，同样也可以把文件中写好的bytes转换回我们python的数据
- shelve：简单另类的一种序列化的方案。有点儿类似redis，可以作为一种小型的数据库来使用
- json：将python中常见的字典、列表转化成字符串。是目前前后端数据交互使用频率最高的一种数据格式



## pickle

  把python对象写入到文件中的一种解决方案，但写入到文件的是bytes.

### dumps

  将python对象序列化为bytes类型

```python
import pickle

class Cat(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print("%s抓老鼠" % self.name)


c = Cat("jerry", 5)
bs = pickle.dumps(c)
print(bs)  # b'\x80\x03c__main__\nCat\nq\x00)\x81q\x01}q\x02(X\x04\x00\x00\x00nameq\x03X\x05\x00\x00\x00jerryq\x04X\x03\x00\x00\x00ageq\x05K\x05ub.'
```

 

### loads

  将bytes反序列化为python对象。注：反序列化为对象后，对象定义代码必须存在，否则无法使用对象

```python
import pickle

class Cat(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print("%s抓老鼠" % self.name)

bs = b'\x80\x03c__main__\nCat\nq\x00)\x81q\x01}q\x02(X\x04\x00\x00\x00nameq\x03X\x05\x00\x00\x00jerryq\x04X\x03\x00\x00\x00ageq\x05K\x05ub.'
cc = pickle.loads(bs)
print(cc.name, cc.age)  # jerry 5
```



 

### dump &load

- dump：将python对象序列化后写入到文件中
- load：从文件中读取bytes,并反序列化为对象
- 对于读写多对象，由于读取时不知文件中写入了多少个对象，导致不好读取，所以一般采用把对象先装进list，然后把list写入至文件中，读取后遍历list即可拿到对象

```python
import pickle

class Person(object):

    def __init__(self, name, pwd):
        self.name = name
        self.pwd = pwd


lst = [Person("Tom", "123"), Person("Linda", "123"), Person("john", "123")]

# 写
with open("login", mode="wb") as f:
    for el in lst:
        pickle.dump(el, f)

# 读
with open("login", mode="rb") as f:
    for i in range(len(lst)):  # 已知对象个数，对文件进行遍历读取
        ret = pickle.load(f)
        print(ret.name, ret.pwd)


结果：
Tom 123
Linda 123
john 123
```

 

```python
import pickle

class Cat(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def eat(self):
        print("%s抓老鼠" % self.name)


lst = [Cat("jerry", 5), Cat("tomy", 6), Cat("alpha", 7)]

with open("cat", mode="wb") as f:
    pickle.dump(lst, f)


with open("cat", mode="rb") as f:
    ret = pickle.load(f)
    for el in ret:
        el.eat()

结果：
jerry抓老鼠
tomy抓老鼠
alpha抓老鼠
```

 

### 应用

  注册登录程序

```python
import pickle

class User(object):
    def __init__(self, username, password):
        self.username = username
        self.password = password


class Client(object):

    def regist(self):
        uname = input("username: ")
        pwd = input("password: ")
        user = User(uname, pwd)
        pickle.dump(user, open("userinfo", mode="wb"))
        print("regist successful!")

    def login(self):
        uname = input("username: ")
        pwd = input("password: ")
        f = open("userinfo", mode="rb")
        while 1:
            try:
                u = pickle.load(f)
                if u.username == uname and u.password == pwd:
                    print("login successful!")
                    break
            except Exception as e:
                print("login failed!")
                break

    def run(self):
        self.regist()
        self.regist()
        self.login()


if __name__ == "__main__":
    c = Client()
    c.run()
```

