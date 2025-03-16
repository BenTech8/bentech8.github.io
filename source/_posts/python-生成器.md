---
title: 生成器
date: 2025-03-16 17:15:20
tags:
- Python
categories:
- Python
---

## 简介

- 生成器本质就是迭代器
- 生成器对象可以直接进行for循环



### 生成器特点

- 省内存
- 惰性机制
- 只能向前，不能反复



### 生成器获取方式

- 通过生成器函数
- 通过生成器表达式



## 生成器函数

### yield

- 可以把函数分段运行
- 作用和return一样，也是返回数据



### 普通函数与生成器函数区别

普通函数：

```
def func():
    print("111")
    return 222

ret = func()
print(ret)


结果：
111
222
```

生成器函数：

```
def func():
    print("111")
    yield 222

ret = func()
print(ret)



结果：
<generator object func at 0x0000018007451C50>
```

所以：

- 当函数中存在yield，那么这个函数就是一个生成器函数
- 当执行生成器函数时，实则为获取这个生成器



### 获取生成器

```
def func():
    print("111")
    yield 222

ret = func()  # 获取到生成器
```



### 执行生成器

- 通过__next__()

```
def func():
    print("111")
    yield 222

gener = func()
ret = gener.__next__()  # 打印 111，222返回给ret
print(ret)
```

当程序运行完最后一个yield，那么后面继续执行__next__()，程序会报错，但后面内容还会执行。

```
def func():
    print("111")
    yield 222
    print("333")
    yield 444
    print("555")

gener = func()
print(gener.__next__())  # 111 222
print(gener.__next__())  # 333 444
print(gener.__next__())  # 555 StopIteration
```

- 通过send()

  send()和__next__()一样都可以让生成器执行到下一个yield，但send()可以给上一个yield的位置变量传递值。当执行完最后一个yield，再继续执行send()时，程序报错，但还可给最后一个yield位置变量传递值。

```
def func():
    print("111")
    a = yield 222
    print("a = ", a)
    print("333")
    b = yield 444
    print("b = ", b)

gener = func()
gener.__next__()  # 111
print(gener.send("1"))  # a = 1 333 444
print(gener.send("2"))  # b = 2  StopIteration
```

 send和__next__()区别：

- send和__next__()都是让生成器向下走一次
- send可以给上一个yield的位置传递值，不能给最后一个yield发送值。在第一次执行生成器代码的时候不能使用send()



### 利用for循环获取生成器内部元素

```
def func():
    yield 111
    yield 222
    yield 333
    yield 444

gen = func()
for i in gen:
    print(i)


结果：
111
222
333
444
```



### yield from

  可以直接把可迭代对象中的每一个数据作为生成器的结果进行返回

```
def gen():
    lst = [11, 22, 33, 44, 55, 66]
    yield from lst


ret = gen()
for i in ret:
    print(i)


结果：
11
22
33
44
55
66
```

此时，上述代码相当于：

```
def gen():
    lst = [11, 22, 33, 44, 55, 66]
    yield lst[0]
    yield lst[1]
    yield lst[2]
    yield lst[3]
    yield lst[4]
    yield lst[5]


ret = gen()
for i in ret:
    print(i)
```



## 生成器表达式

### 写法

- （结果 for 变量 in 可迭代对象 if 条件筛选）

```
gen = (i for i in range(10))
print(gen)  # <generator object <genexpr> at 0x000002CE52D91C50>
```



### 特点

- 本质是迭代器
- 省内存
- 惰性机制
- 只能向前，不能反复



### 应用

1）list ()可把传递进来的数据转化成列表，list里面包含for循环

```
g = (i for i in range(10))

print(list(g))  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

2)

```
def func():
    print(111)
    yield 222

g = func()
g1 = (i for i in g)
g2 = (i for i in g1)

print(list(g)) # 111 [222]
print(list(g1)) # []
print(list(g2)) # []
```

 3）

```
def add(a, b):
    return a + b

def test():
    for r_i in range(4):
        yield r_i

g = test()

for n in [2, 10]:
    g = (add(n, i) for i in g)

print(list(g))  # 20 21 22 23
```

此代码相当于：

```
def add(a, b):
    return a + b

def test():
    for r_i in range(4):
        yield r_i

g1 = test()

n = 2
g2 = (add(n, i) for i in g1)

n = 10
g3 = (add(n, i) for i in g2)

print(list(g3)) # n = 10, g3 = (add(n, i) for i in (add(n, i) for i in g1))
```

 



 
