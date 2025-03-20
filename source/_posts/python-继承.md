---
title: 继承
date: 2019-03-20 16:46:41
tags:
- Python
categories:
- Python
---

## 判断继承

### issubclass

- 判断xxx类是否是yyy类型的子类
- 可以隔代判数

```python
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



##  多继承

- python支持多继承，一个类可以拥有多个父类
- 经典类：在python2.2之前版本使用的是经典类，经典类在基类的根如果什么都不写，表示继承xxx
- 新式类：在python2.2之后版本使用的是新式类，新式类的特点是基类的根是object
- python3使用的都是新式类，如果基类谁都不继承。那这个类会默认继承object



## MRO

  在多继承中，当多个父类出现了重名方法时，此时就涉及到如何查找父类方法的问题，即MRO(method resolution order)问题。

  在python中，不同的版本中使用的是不同的算法来完成MRO的

### 经典类的MRO

- 采用的是树型结构的深度优先遍历（先序）,即根-左-右。

```python
class A:
    pass

class B(A):
    pass

class C(A):
    pass

class D(B, C):
    pass

class E:
    pass

class F(E, B):
    pass

class G(F, D):
    pass

class H:
    pass

class Foo(H, G):
    pass
```

 继承关系图如下：

 ![1427763-20181113194621417-491962360](python-%E7%BB%A7%E6%89%BF.assets/1427763-20181113194621417-491962360.png)

​                                                                                                                                     图1 继承关系图

**类的MRO：Foo ---> H ---> G ---> F ---> E ---> B ---> A --->D ---> C**



### 新式类的MRO—C3算法

官方网址：https://www.python.org/download/releases/2.3/mro/?tdsourcetag=s_pctim_aiomsg

步骤：

- 先拆分
- 再合并：从下向上合并，拿出每一项的头和后一项的身体进行比较，如果出现了，就过，从后一项的头继续去比较，如果不出现就拿出来

```
class A:
    pass
class B(A):
    pass
class C(A):
    pass
class D(B, C):
    pass
class E(C, A):
    pass
class F(D, E):
    pass
class M(F, E):
    pass
class N:
    pass
class P(M,N):
    pass
class G(P):
    pass
class O:
    pass
class X(O):
    pass
class H(G, X, F):
    pass
print(H.__mro__)结果：(<class '__main__.H'>, <class '__main__.G'>, <class '__main__.P'>, <class '__main__.M'>, <class '__main__.X'>, <class '__main__.F'>, <class '__main__.D'>, <class '__main__.B'>, <class '__main__.E'>, <class '__main__.C'>, <class '__main__.A'>, <class '__main__.N'>, <class '__main__.O'>, <class 'object'>)
```

**1) 先拆分：设L为查找方法的MRO顺序**

L(H) = H + L(G) + L(X) + L(F) + GXF

 

L(G) = G + L(P) + P

L(X) = X + L(O) + O

L(F) = F + L(D) + L(E) + DE

 

L(P) = P + L(M) + L(N) + MN

L(D) = D + L(B) + L(C) + BC

L(E) = E + L(C) + A + CA

 

L(M) = M + L(F) + L(E) + FE

**2) 合并：**

L(H) = H + L(G) + L(X) + L(F) + GXF   ====>HGPMXFDBECANO

 

L(G) = G + L(P) + P                 ====>GPMFDBECAN

L(X) = X + L(O) + O                 ====>XO

L(F) = F + L(D) + L(E) + DE          ====>FDBECA

 

L(P) = P + L(M) + L(N) + MN         ====>PMFDBECAN

L(D) = D + L(B) + L(C) + BC          ====>DBCA

L(E) = E + L(C) + A + CA            ====>ECA

 

L(M) = M + L(F) + L(E) + FE          ====>MFDBECA

L(C) = C + L(A)                   ====>CA

L(N) = N                        ====>N

L(B) = B + L(A) + A                ====>BA

**类的MRO：H--->G--->P--->M--->X--->F--->D--->B--->E--->C--->A--->N--->O**



## super

### 访问父类的初始化方法

  这样子类和父类初始化方法中相同的代码可不必都写，直接用父类的就好

```python
class Base(object):

    def __init__(self, a, b, c):
        self.a = a
        self.b = b
        self.c = c


class Foo(Base):

    def __init__(self, a, b, c, d):
        super().__init__(a, b, c)
        self.d = d


f = Foo(1, 2, 3, 4)
print(vars(f))    # {'a': 1, 'b': 2, 'c': 3, 'd': 4}
```

 

### 实现子类方法调用父类(MRO)中的方法

- **super().方法名()：**执行MRO中下一个方法
- **super(类名，self).方法名()：**定位到该类，执行MRO中该类下一个类的方法

```python
class Biology(object):
    def move(self):
        print("----biology-----")


class Animal(Biology):
    def move(self):
        print("-----Animal-----")


class Cat(Animal):
    def move(self):
        super().move()   # 找MRO中的下一个
        print("-----Cat-----")


c = Cat()
c.move()


# 结果：
# -----Animal-----
# -----Cat-----
```

```python
class Biology(object):
    def move(self):
        print("----biology-----")


class Animal(Biology):
    def move(self):
        print("-----Animal-----")


class Cat(Animal):
    def move(self):
        super(Animal, self).move()   # 定位到Animal，找Animal下一个
        print("-----Cat-----")


c = Cat()
c.move()


# 结果：
# ----biology-----
# -----Cat-----
```

 

### 应用

```python
class Init(object):
    def __init__(self, v):
        print("init")
        self.val = v


class Add2(Init):
    def __init__(self, val):
        print("Add2")
        super(Add2, self).__init__(val)
        print(self.val)
        self.val += 2


class Mult(Init):
    def __init__(self, val):
        print("Mult")
        super(Mult, self).__init__(val)
        self.val *= 5


class HaHa(Init):
    def __init__(self, val):
        print("哈哈")
        super(HaHa, self).__init__(val)
        self.val /= 5


class Pro(Add2,Mult,HaHa): #
    pass


class Incr(Pro):
    def __init__(self, val):
        super(Incr, self).__init__(val)
        self.val += 1


p = Incr(5)
print(p.val)   # Add2 Mult 哈哈 init 5.0 8.0c = Add2(2) print(c.val)  # Add2 init 2 4
```

 

**MRO：**

**Incr**： Incr ---> Pro ---> Add2 ---> Mult ---> HaHa ---> Init

**Add2**： Add2 ---> Init
