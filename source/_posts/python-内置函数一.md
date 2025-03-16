---
title: 内置函数一
date: 2019-03-16 17:02:59
tags:
- Python
categories:
- Python
---

作用域相关

- locals()

  功能：返回当前作用域中的名字

- globals()

  功能：返回全局作用域中的名字



## 迭代器/生成器相关

- range()

  功能：生成数据

```
lst = [i for i in range(10)]
print(lst)  # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

- iter()

  功能：获取迭代器，内部实际使用的是__iter__()方法来获取迭代器

- next()

  功能：迭代器向下执行一次，内部实际使用了__next__()方法返回迭代器的下一个项目

```
lst = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

ret = iter(lst)  # 获取迭代器

print(next(ret)) # 0   执行迭代器
```



## 字符串类型代码的执行

- eval()

  功能：执行字符串类型的代码。并返回最终结果

```
print(eval("[11, 22, 33, 44]"))  # [11, 22, 33, 44]

print(eval("2+2"))  # 4

def func():
    print(666)
eval("func()")  # 666
```



- exec()

  功能：执行字符串类型的代码

```
exec("""
for i in range(10):
    print(i)
""")  # 0 1 2 3 4 5 6 7 8 9

exec("""
def func():
    print("我是赛利亚")

func()
""")  # 我是赛利亚
```

- compile()

  功能：将字符串类型的代码编译，有返回值的字符串形式代码用eval()求值，没有返回值的字符串形式代码用exec()执行

  参数说明：

  1）resource：要执行的代码，动态代码片段

  2）文件名：代码存放的文件名，当传入第一个参数的时候，这个参数给空就可以了

  3）模式，取值有3个：

​    a) exec：一般放一些流程语句的时候

​    b) eval：resource只存放一个求值表达式

​    c) single：resource存放的代码有交互的时候，mode应为single，但只允话代码只有一行，否则报错

```
code = """
for i in range(5):
    print(i)
"""
c1 = compile(code, "", mode="exec")
exec(c1)  # 0 1 2 3 4



code2 = """
name = input("请输入你的名字：")
"""
c2 = compile(code2, "", mode="single")
exec(c2)
print(name) # Tom
```

##  

## 输入输出相关

- input()

  功能：获取用户输入的内容

- print()

  功能：打印输出



## 内存相关

- hash()

  功能：获取对象的哈希值(int，bool，str，tuple)

  注意：

  \1) 数字的哈希值就是它本身

  \2) 每次调用hash()得到的哈希值不一样(除数字外)，所以哈希值不能做密码

```
num = 10
s = "Hello"

print(hash(num))  # 10

# 第一次调用
print(hash(s))  # 3718209680369926059

# 再次调用
print(hash(s))  # 263127024043316082
```

- id()

  功能：获取对象的内存地址



## 与文件相关

- open()

  功能：用于打开一个文件，创建一个文件句柄



## 模块相关

- __import__()

  功能：用于动态加载类和函数



## 帮助

- help()

  功能：函数用于查看函数或模块用途的详细说明



## 调用相关

- callable()

  功能：用于检查一个对象是否是可调用的。如果返回True， object有可能调用失败，但如果返回False，那么调用绝对不会成功

```
n = 10

def func():
    pass

print(callable(n)) # False

print(callable(func)) # True
```



## 查看内置属性

- dir()

  功能：查看对象的内置属性，方法。访问的是对象的__dir__()方法



## 基础数据类型相关

### 和数字相关

#### a) 数据类型

- bool()

  功能：将给定的数据转换成bool值。如果不给值，返回False

- int()

  功能：将给定的数据转换成int值。如果不给值，返回0

- float()

  功能：将给定的数据转换成float值，也就是小数

- complex()

  功能：创建一个复数，第一个参数为实部，第二个参数为虚部，或者第一个参数直接用字符串来描述复数

#### b) 进制转换

- bin()

  功能：将给的参数转换成二进制

- otc()

  功能：将给的参数转换成八进制

- hex()

  功能：将给的参数转换成十六进制

#### c) 数学运算

- abs()

  功能：返回绝对值

- divmod()

  功能：返回商和余数

- round()

  功能：四舍五入

  对于x.5，如果整数个位数为偶数靠近偶数进行舍弃，如果整数个位数为奇数靠近偶数进行舍弃

```
print(round(4.5)) # 4
print(round(5.5)) # 6
print(round(5.4)) # 5
```

- pow(a, b)

  功能：求a的b次幂，如果有三个参数，则求完次幂后对第三个数取余

```
print(pow(2, 3)) # 8

print(pow(2, 3, 5)) # 3
```

- sum(可迭代对象)

  功能：求和

- min()

  功能：求最小值

- max()

  功能：求最大值 



### 和数据结构相关

#### a) 列表和元组

- list()

  功能：将一个可迭代对象转换成列表

- tuple()

  功能：将一个可迭代对象转换成元组

- reversed()

  功能：将一个序列翻转，返回翻转序列的迭代器

```
lst = [1, 2, 3, 4, 5, 6]
ret = reversed(lst)
print(ret)  # <list_reverseiterator object at 0x000001B589038208>

for i in ret:
    print(i)  # 6 5 4 3 2 1
```

- slice

  功能：列表的切片

```
lst = [1, 2, 3, 4, 5, 6]

s = slice(1, 5, 2)
print(lst[s])  # [2, 4]
```

#### b） 字符串相关

- str()

  功能：将数据转化成字符串

- format()

  功能：与具体数据相关，用于计算各种小数，精算等

```
# 字符串
print(format('test', '<20'))    # 左对齐 
print(format('test', '>20'))    # 右对齐 
print(format('test', '^20'))    # 居中 

# 数值 
print(format(3, 'b'))           # ⼆进制 
print(format(97, 'c'))          # 转换成unicode字符
print(format(11, 'd'))          # ⼗进制 
print(format(11, 'o'))          # ⼋进制 
print(format(11, 'x'))          # ⼗六进制(⼩写字母) 
print(format(11, 'X'))          # ⼗六进制(⼤写字母) 
print(format(11, 'n'))          # 和d一样 
print(format(11))               # 和d一样 

# 浮点数 
print(format(123456789, 'e'))           # 科学计数法. 默认保留6位小数 
print(format(123456789, '0.2e'))        # 科学计数法. 保留2位小数(小写) 
print(format(123456789, '0.2E'))        # 科学计数法. 保留2位小数(大写) 
print(format(1.23456789, 'f'))          # 小数点计数法. 保留6位小数 
print(format(1.23456789, '0.2f'))       # 小数点计数法. 保留2位小数 
print(format(1.23456789, '0.10f'))      # 小数点计数法. 保留10位小数 
print(format(1.23456789e+10000, 'F'))   # 小数点计数法.
```

- bytes()

  功能：把字符串转化成bytes类型

```
s = "你好"

bs = bytes(s, encoding="utf-8")
print(bs)  # b'\xe4\xbd\xa0\xe5\xa5\xbd'
```

- bytearray()

  功能：返回一个新字节数组，这个数组里的元素是可变的，并且每个元素的值的范围是[0, 256]

```
ret = bytearray('alex', encoding='utf-8')
print(ret)  # bytearray(b'alex')

ret[0] = 65
print(ret) # bytearray(b'Alex')
```

- memoryview()

  功能：查看bytes在内存中的情况

```
s = "你好"
ret = memoryview(s.encode("utf-8"))
print(ret)  # <memory at 0x0000022FC0071048>
```

- ord()

  功能：输入字符找带字符编码的位置

- chr()

  功能：输入位置数字找出对应的字符

- ascii()

  功能：在ascii码中就返回这个值，如里不在就返回\u...

```
s = "你好"
print(ascii(s))  # '\u4f60\u597d'
```

- repr()

  功能：返回一个对象的官方表示形式

```
s = "你好\我叫赛利亚"
print(repr(s)) # '你好\\我叫赛利亚'
print(s) #你好\我叫赛利亚
```

#### c） 数据集合

- dict()

  功能：创建一个字典

- set()

  功能：创建一个集合

- frozenset()

  功能：创建一个冻结的集合，冻结的集合不能进行添加和删除操作

#### d) 其他相关

- len()

  功能：返回一个对象中的元素的个数

- enumerate(可迭代对象，start=0)

  功能：获取集合的枚举对象

```
lst = [11, 22, 33]
for index, el in enumerate(lst):
    print(index, el)

结果：
0 11
1 22
2 33
```

- all()

  功能：可迭代对象中全部是True，结果才是True

```
print(all([1, 2, 0, True])) # False
```

- any()

  功能：可迭代对象中有一个是True， 结果就是True

```
print(any([1, 2, 0, True])) # True
```

- zip()

  功能：函数用于将可迭代对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的一个可迭代对象zip object。如果各个迭代器的元素不一致，则返回对象长度与最短的对象相同

```
lst1 = [1, 2, 3]
lst2 = ["a", "b", "c", "d"]
lst3 = (11, 22, 33, 44, 55)

print(zip(lst1, lst2, lst3))       #  <zip object at 0x000001E7D275BF48>
print(list(zip(lst1, lst2, lst3))) #[(1, 'a', 11), (2, 'b', 22), (3, 'c', 33)]

for i in zip(lst1, lst2, lst3):
    print(i)                       # (1, 'a', 11)  (2, 'b', 22) (3, 'c', 33)
```
