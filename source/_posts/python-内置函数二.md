---
title: 内置函数二
date: 2025-03-16 17:03:03
tags:
- Python
categories:
- Python
---

## lambda

- 表示匿名函数
- 为了解决一些简单的需求而设计的一句话函数

语法： **函数名** = **lambda 参数：返回值**

- 函数的参数可以有多个，多个参数之间用逗号隔开
- 匿名函数不管多复杂，只能写一行，且逻辑结束后直接返回数据
- 返回值和正常的函数一样，可以是任意数据类型

```
f = lambda n: n ** 2
print(f(10))  # 100
print(f.__name__)  # <lambda>
```

注：匿明函数并不是说一定没有名字。上面f就是一个函数名。说他是匿明原因是通过__name__查看的时候是没有名字的。统一叫lambda。在调用时没有特别之处，像正常函数调用即可。



## sorted()

- 功能：排序

语法：**sorted(Iterable, key=None, reverse=False)**

- **key:**排序规则(排序函数)，在sorted内部会将可迭代对象中的每一个元素传递给这个函数的参数。根据这个函数运算的结果进行排序
- **reverse**:是否倒叙，默认值为False

```
lst = [1, 4, 6, 8, 4, 9, 10]

print(sorted(lst))  # [1, 4, 4, 6, 8, 9, 10]
```

 

与函数组合使用

```
lst = [
    {"name": "电脑", "price": 6000},
    {"name": "手表", "price": 4000},
    {"name": "耳机", "price": 1000}
]

def func(dic):
    return dic["price"]

print(sorted(lst, key=func))


结果：
[{'name': '耳机', 'price': 1000}, 
 {'name': '手表', 'price': 4000}, 
 {'name': '电脑', 'price': 6000}]
```

 

与lambda组合使用

```
lst = [
    {"name": "电脑", "price": 6000},
    {"name": "手表", "price": 4000},
    {"name": "耳机", "price": 1000}
]


print(sorted(lst, key=lambda dic:dic["price"]))


结果：
[{'name': '耳机', 'price': 1000},
 {'name': '手表', 'price': 4000},
 {'name': '电脑', 'price': 6000}]
```



## filter()

- 功能：筛选

语法：**filter(function, Iterable)**

- **function:**用来筛选的函数。在filter中会自动的把Iterable中的元素传递给function。然后根据function返回的True or False来判断是否保留此项数据

```
 portfolio = [
     {'name': 'IBM', 'shares': 100, 'price': 91.1},
     {'name': 'AAPL', 'shares': 50, 'price': 543.22},
     {'name': 'FB', 'shares': 200, 'price': 21.09},
     {'name': 'HPQ', 'shares': 35, 'price': 31.75},
     {'name': 'YHOO', 'shares': 45, 'price': 16.35},
     {'name': 'ACME', 'shares': 75, 'price': 115.65}
 ]


lst = list(filter(lambda dic: dic["price"] > 100, portfolio))
print(lst)


结果：
[{'name': 'AAPL', 'shares': 50, 'price': 543.22}, 
{'name': 'ACME', 'shares': 75, 'price': 115.65}
```

 

## map()

- 映射函数

语法：**map(function, Iterable)**

- 可以对可迭代对象中的每一个元素进行映射。分别去执行function,保留最后的执行结果

```
lst1 = [1, 2, 3, 4, 5]
lst2 = [2, 4, 6, 8, 10]

print(list(map(lambda x, y: x + y, lst1, lst2)))



结果：

[3, 6, 9, 12, 15]
```



##  issubclass()

- 判断xxx类是否是yyy类型的子类
- 可以隔代判断

```
class Base:
    pass

class Foo(Base):
    pass

class Bar(Foo):
    pass

print(issubclass(Bar, Foo))   # True
print(issubclass(Foo, Bar))   # False
print(issubclass(Bar, Base))  # True
```



## type()

- 返回xxx是什么类型
- 可以精准的返回数据类型

```python
print(type("hello"))  # <class 'str'>

class Person(object):
    pass

obj = Person()

print(type(Person))  # <class 'type'>
print(type(obj))     # <class '__main__.Person'>
```



## isinstance()

- 判断xxx对象是否是xxx类型
- 可以判断该对象是否是xxx家族体系中的(只能往上判断)

```
class Animal:
    pass


class Cat(Animal):
    pass


class Persian_Cat(Cat):
    pass

kitty = Cat()
print(isinstance(kitty, Cat))           # True
print(isinstance(kitty, Persian_Cat))   # False
print(isinstance(kitty, Animal))        # True
```

