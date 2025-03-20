---
title: 初识面向象
date: 2019-03-20 16:27:44
tags:
- Python
categories:
- Python
---

## 面向对象与面向过程

### 面向过程

- 特点：一切以事物的流程为核心
- 优点：负责的问题流程化，编写相对简单
- 缺点：可扩展性差



### 面向对象

- 特点：一切以对象为中心
- 优点：可扩展性强
- 缺点：编程的复杂度高于面向过程

## 类与对象

### 类

- 类是对事物的总结，抽象的概念，类用来描述对象
- __init__():称为初始化方法，对对象进行初始化操作，创建对象的时候自动调用这个方法



### 对象

- 对象是类的实例化的结果
- 对象能执行哪些方法，都由类来决定，类中定义了什么，对象就拥有什么

```
class User(object):
    def __init__(self, username, password):
        self.username = username
        self.password = password

    def login(self, uname, pwd):
        if self.username == uname and self.password == pwd:
            return True
        else:
            return False


u1 = User("尝试了就好", "123")
ret = u1.login(input("username:").strip(), input("password:").strip())
print(ret)
```



### 创建对象

xxx = 类名()

- 先自动执行__new__()来开辟内存，此时新开辟出来的内存区域是空的
- 再自动调用__init__()来完成对象的初始化工作



## 面向对象三大特征

### 封装

- 在面向对象思想中，是把一些看似无关紧要的内容组合到一起统一进行存储和使用。面向对象中表现为对属性和方法进行封装



### 继承

- 子类可以自动拥有父类中除了私有属性外的其他所有内容
- 一个类可以同时继承多个父类

```
class Animal(object):

    def run(self):
        print("----run----")


class Cat(Animal):
    pass

cat = Cat()
cat.run()  # ----run----
```



### 多态

- 同一个对象，多种形态

```
class Animal(object):
    def run(self):
        print("Animal is running...")


class Dog(Animal):
    def run(self):
        print("Dog is running...")


class Cat(Animal):
    def run(self):
        print("Cat is running...")


class Observer():
    def observe(self, animal):
        animal.run()


dg = Dog()
ct = Cat()

ob = Observer()
ob.observe(dg)
ob.observe(ct)

结果：
Dog is running...
Cat is running...
```
