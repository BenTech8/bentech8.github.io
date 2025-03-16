---
title: 小数据池
date: 2019-03-15 22:45:42
tags:
categories: Python
---

## 小数据池

### 代码块

  python程序是由代码块构成的，一个代码块的文本作为pythont程序执行的单元

```
官方文档：
     A Python program is constructed from code blocks. A block is a piece of Python program text that is executed as a unit. The following are blocks: a module, a function body, and a class definition. Each command typed interactively is a block. A script file (a file given as standard input to the interpreter or specified as a command line argument to the interpreter) is a code block. A script command (a command specified on the interpreter command line with the ‘-c‘ option) is a code block. The string argument passed to the built-in functions eval() and exec() is a code block. A code block is executed in an execution frame. A frame contains some administrative information (used for debugging) and determines where and how execution continues after the code block’s execution has completed.
```

一个代码块：

- 一个模块(module)
- 一个函数(function)
- 一个类(class)
- 每一个command命令
- 一个文件(file)
- eval()
- exec()



### id()

  通过id()可以查看到一个变量表示的值在内存中的地址

```
s = "hello"
print(id(s)) # 2305859175064
```



### is 和 == 区别

- ==：判断左右两端的值是否相等
- is：判断左右两端内容的内存地址是否一致。如果返回True,那可以确定这两个变量使用的是同一个对象

  如果内存地址相同，则值一定是相等的，如果值相等，则不一定同一对象

```python
a = 100
b = 100
print(a is b)  # True
print(a == b)  # True
```

```python
a = 1000
b = 1000

print(a == b) # True
print(a is b) # False 在command命令下为False, 在.py文件中（例如pycharm中）得到的结果为True。（详情见下面）
```



### 小数据池

- 一种缓存机制，也被称为驻留机制。各大编程语言中都有类似的东西。网上搜索常量池，小数据池指的是同一个内容
- 小数据池只针对：int(整数)， string(字符串)， bool(布尔值)。其他数据类型不存在驻留机制

优点：能够提高字符串、整数的处理速度。省略了创建对象的过程。

缺点：在"池"中创建或者插入新的内容会花费更多的时间。

1.整数

```
官方文档：
    The current implementation keeps an array of integer objects for all integers between -5 and 256, when you create an int in that range you actually just get back a reference to the existing object. So it should be possible to change the value of 1. I suspect the behavior of Python in this case is undefined.
```

- 在python中，-5~256会被加到小数据池中，每次使用都是同一个对象
- 在使用的时候，内存中只会创建一个该数据的对象，保存在小数据池中。当使用的时候直接从小数据池中获取对象的内存引用，而不需要重新创建一个新的数据，这样会节省更多的内存区域。

2.字符串

```
Incomputer science, string interning is a method of storing only onecopy of each distinct string value, which must be immutable. Interning strings makes 
some stringprocessing tasks more time- or space-efficient at the cost of requiring moretime when the string is created or interned. The distinct values are
 stored ina string intern pool. –引⾃自维基百科
```

- 如果字符串的长度是0或者1，都会默认进行缓存。（中文字符无效）
- 字符串长度大于1，但是字符串中只包含数字，字母，下划线时会被缓存。
- 用乘法得到的字符串：1）乘数为1，仅包含数字，字母，下划线时会被缓存。如果包含其他字符，而长度<= 1也会被驻存(中文字符除外)。2）乘数大于1，仅包含数字，字母，下划线时会被缓存，但字符串长度不能大于20
- 指定驻留：可以通过sys模块中的intern()函数来指定要驻留的内容。(详情见sys模块相关内容)



### 代码块缓存机制

  在代码块内缓存机制是不一样的：

- 在执行同一个代码块的初始化对象的命令时，会检查其值是否已经存在，如果存在，会将其重用。换句话说：执行同一个代码块时，遇到初始化对象的命令时，他会将初始化的这个变量与值存储在一个字典中，再遇到新的初始化对象命令时，先在字典中查询其值是否已经存在，如果存在，那么它会重复使用这个字典中的之前的这个值。即两变量指向同一个内存地址。
- 如果是不同的代码块，则判断这两个变量是否满足小数据池的数据，如果满足，则两变量指向同一个地址。如果不满足，则得到两个不同的对象，即两变量指向的是不同的内存地址。

注意：对于同一个代码块，只针对单纯创建变量，才会采用缓存机制，对于创建变量并同时做相关运算，则无。

```
a = 1000
b = 1000

print(id(a)) # 2135225709680
print(id(b)) # 2135225709680
print(a is b)  # True  .py文件运行
```

```
a = 1000
b = 10*100

print(id(a)) # 1925536396400
print(id(b)) # 1925536643952
print(a is b) # False  .py文件运行
```



### 小数据池与代码块缓存机制区别与联系

- 小数据池与代码块对缓存数据类型要求不一致，代码块只针对单纯创建变量有效，而对整数大小，字符串字符要求及长度无限制。
- 对于代码块缓存机制：如果不满足代码块缓存机制，则判断是否满足小数据池数据，如果满足，则采用小数据池缓存机制。

```
a = 5*5
b = 25

print(id(a))  # 1592487712
print(id(b))  # 1592487712

print(a is b)  # True  .py文件运行
```

```
a = "Incomputer science, string interning is a method of storing only onecopy of each distinct string value"
b = "Incomputer science, string interning is a method of storing only onecopy of each distinct string value"

print(id(a)) # 2926961023256
print(id(b)) # 2926961023256

print(a is b) # True  .py文件运行
```
