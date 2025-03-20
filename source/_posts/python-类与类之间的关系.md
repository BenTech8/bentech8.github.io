---
title: 类与类之间的关系
date: 2019-03-20 16:36:48
tags:
- Python
categories:
- Python
---

## 依赖关系

- 在方法中给方法传递一个对象
- 此种关系，类与类之间的关系是最轻的

```
# 植物大战僵尸
class Plant(object):
    """
    植物类
    """
    def __init__(self, name, attack, hp):
        """
        初始化
        :param name:植物名
        :param attack: 攻击力
        :param hp: 血量
        """
        self.name = name
        self.attack = attack
        self.hp = hp

    def fight(self, zombie):
        """
        攻击js,使js掉血
        :param js: 攻击对象
        :return: None
        """
        zombie.hp -= self.attack


class Zombie(object):
    """
    僵尸类
    """
    def __init__(self, name, hp, attack):
        """
        初始化
        :param name: 僵尸名
        :param hp: 血量
        :param attack: 攻击力
        """
        self.name = name
        self.hp = hp
        self.attack = attack

    def eat(self, plant):
        """
        僵尸吃plant
        :param plant: 吃的对象
        :return: None
        """
        plant.hp -= self.attack


lvluo = Plant("绿萝", 20, 100)
js1 = Zombie("僵尸1号", 100, 5)

lvluo.fight(js1)
js1.eat(lvluo)

print(lvluo.hp)  # 95
print(js1.hp)    # 80
```



## 关联关系

- 两种事物必须是互相关联的，但是在某些特殊情况下是可以更改和更换的
- 一对多：一的一方埋集合，多的一方埋实体

### 一对一关系

```
class Person(object):
    """
    Person类
    """
    def __init__(self, name, id=None):
        """
        初始化
        :param name: 姓名
        :param id: 身份证号码
        """
        self.name = name
        self.id = id

    def display(self):
        """
        展示有无身份证
        :return: None
        """
        if self.id:
            print("我叫%s, 我的身份证号码为%s" % (self.name, self.id))
        else:
            print("我未成年，没有身份证！！！")


class Id(object):
    """
    Id类
    """
    def __init__(self, name, num):
        """
        初始化
        :param name: 姓名
        :param num: 身份证号
        """
        self.name = name
        self.num = num


p_id = Id("Tom", 123456789)
p1 = Person("Tom", p_id.num)
p1.display()  # 我叫Tom, 我的身份证号码为123456789
```



### 一对多关系

```
class Teacher(object):
    """
    Teacher类
    """
    def __init__(self, name, lst=None):
        """
        初始化
        :param name: 姓名
        :param lst: 学生列表
        """
        self.name = name
        if lst == None:
            self.lst = []
        else:
            self.lst = lst

    def add(self, student):
        """
        添加学生
        :param student: 学生对象
        :return: None
        """
        self.lst.append(student)

    def display(self):
        """
        打印学生姓名
        :return: None
        """
        for el in self.lst:
            print(el.name)


class Student(object):
    """
    Student类
    """
    def __init__(self, name, num):
        """
        初始化
        :param name: 姓名
        :param num: 学号
        """
        self.name = name
        self.num = num

# 创建对象
t = Teacher("Tom")
s1 = Student("andy", 1)
s2 = Student("Linda", 2)

# 把学生添加给老师
t.add(s1)
t.add(s2)

t.display()

结果：
andy
Linda
```



## 继承关系

- 在具体继承关系中，访问方法时，self永远都是指向创建对象时的对象。

```
class Base(object):
    def __init__(self, num):
        self.num = num
    
    def func1(self):
        print(self.num)


class Foo(Base):
    pass

obj = Foo(123)  # 调用__init__方法，自己没有，找父类
obj.func1()  # 123
```

 Next：

```
class Base(object):
    def __init__(self, num):
        self.num = num

    def func1(self):
        print(self.num)
        self.func2()

    def func2(self):
        print("Base.func2")


class Foo(Base):

    def func2(self):
        print("Foo.func2")


obj = Foo(123)
obj.func1()


结果：
123
Foo.func
```

 Next：

```
class Base(object):
    def __init__(self, num):
        self.num = num

    def func1(self):
        print(self.num)
        self.func2()

    def func2(self):
        print(111, self.num)


class Foo(Base):

    def func2(self):
        print(222, self.num)


lst = [Base(1), Base(2), Foo(3)]
for obj in lst:
    obj.func1()


结果：
1
111 1
2
111 2
3
222 3
```
