---
title: 类成员
date: 2019-03-20 16:30:06
tags:
- Python
categories:
- Python
---

## 变量

### 实例变量(字段)

- 每个实例都应该拥有的变量
- 实例变量中隐含着一个创建这个对象的类。通过这个类就能找到类中定义的全部内容，包括方法和属性信息等
- 访问：对象.变量名

```
class Person(object):

    def __init__(self, name, age):
        self.name = name  # 实例变量
        self.age = age    # 实例变量


p1 = Person("Tom", 18)
print(p1.name)  # Tom  访问实例变量

p1.name = "Andy"  # 修改实例变量的值
print(p1.name)  # Andy
```



### 类变量(静态变量)

- 直接写在类中的变量，所有对象共享
- 访问：类名.变量名，对象.变量名(不推荐，且只能访问，不能修改)

```
class Foo(object):
    count = 0  # 类变量

    def __init__(self):
        Foo.count += 1


print(Foo.count)    # 0 访问类变量
Foo()
Foo()
Foo()
print(Foo.count)  # 3
```



## 方法

### 成员方法(实例方法）

- 对象直接访问的方法
- 访问：对象.方法名()

```
class User(object):

    def login(self):  # 实例方法
        print("欢迎登录")


u = User()
u.login()  # 调用实例方法
```



### 类方法

- cls指类本身，不管用对象还是用类去访问类方法，默认传递进去的是类
- 调用：类名.方法名()，对象.方法名()(不推荐)

语法：

```
@classmethod
def 方法名(cls):
    pass
```

```
class Foo(object):

    @classmethod
    def add(cls, a, b):  # 类方法
        print(cls)
        return a + b


print(Foo)
print(Foo.add(1, 2))

结果：
<class '__main__.Foo'>
<class '__main__.Foo'>
3
```



### 静态方法

- 当出现一个方法不需要使用到实例变量和类变量时，就可以选择使用静态方法
- 调用：类名.方法名()，对象.方法名()(不推荐)

语法：

```
@staticmethod
def 方法名.():
    pass
```

```
class Foo(object):

    @staticmethod
    def welcome():
        print("---Welcoming---")


Foo.welcome()  # ---Welcoming---
```



## 私有

  在Python中使用"__"作为方法或者变量的前缀，那么这个方法或者变量就是一个私有的

  如果类中存在继承关系，子类是无法继承父类的私有内容的

### 私有变量

- 私有变量不能直接访问，但可通过在公共方法中访问私有变量的值
- 实例变量，类变量都可为私有

```
class Person(object):

    def __init__(self, salary):
        self.__salary = salary

    def sal(self):
        print(self.__salary)


p1 = Person(50000)
print(p1.__salary)  # 报错
p1.sal()  # 50000
```



###  私有方法

- 私有方法只能在类中自己调用，类外面不能直接访问，可定义公共方法，在公共方法中调用私有方法
- 实例方法，类方法，静态方法都可变为私有

```
class Person(object):

    def __init__(self, name, salary):
        self.name = name
        self.__salary = salary

    def __sal(self):
        print(self.__salary)

    def display(self):
        self.__sal()


p1 = Person("Tom", 50000)
p1.display()  # 50000
```



## 属性(@property)

- 通过方法改造过来的一种变量的写法

注意：

- 方法参数只能有一个self
- 调用时不需要写括号，直接当成属性变量
- 需要有返回值

```
class Person(object):

    def __init__(self, name, birthday):
        self.name = name
        self.birthday = birthday

    @property
    def age(self):
        return 2018 - self.birthday


p = Person("Tom", 1994)
print(p.age)  # 24
```

增加方法，使之可以赋值：

```
class Test(object):

    def __init__(self):
        self.__num = 100

    @property
    def num(self):
        return self.__num

    @num.setter
    def num(self, new_num):
        self.__num = new_num


t = Test()
print(t.num)  # 100
t.num = 50
print(t.num)  # 50
```

 property另一种表达方式：

  **property(fget=None, fset=None, fdel=None, doc=None)**

- fget:获取属性值的函数。
- fset:设置属性值的函数。
- fdel:删除属性值的函数。
- doc:为该属性创建一个docstring。

```
class Test(object):

    def __init__(self):
        self.__num = 100

    def getnum(self):
        return self.__num

    def setnum(self, new_num):
        self.__num = new_num

    num = property(getnum, setnum)


p1 = Test()
print(p1.num)  # 100
p1.num = 50
print(p1.num)  # 50
```



## 类特殊成员

### \__new__()

- 构造方法
- 在创建对象时，系统自动先执行__new__()来开辟内存，此时新开辟出来的内存区域是空的

### \__init__()

- 初始化
- 在__new__()调用完后，紧接着系统自动调用__init__()来完成对象的初始化工作

### \__str__()

- 获取对象的描述信息
- 触发：**print(对象)**

### **\__repr__()**

- 调用：**print(repr(对象))**

### **\__call__()**

- 触发：**对象()**

### **\__del__()**

- 在删除对象或者程序结束，对象内存空间删除时，系统都会自动调用一次del方法

### \__getitem__()

- 触发：**对象[key]**

### \__setitem__()

- 触发：**对象[key] = value**

### **\__delitem__()**

- 触发：**del 对象[key]**

### **\__add__()**

- 触发：**对象 + 对象** 

### **\__enter__()**

- 触发：**with 对象 as 变量：**

### **\__exit__()**

- 触发：**with 对象 as 变量：**退出时

###  \__iter__()

- 触发：当遍历对象时

### \__hash__()

- 调用：**print(hash(对象))**

### **\__gt__()**

- 触发：**对象1 > 对象2**

### **\__lt__()**

- 触发：**对象1 < 对象2**

### **\__ge__()**

- 触发：**对象1 >= 对象2**
