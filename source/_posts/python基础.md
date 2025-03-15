---
title: python基础
date: 2025-03-15 22:35:14
tags: Python
categories: Python
---

## python简介

 python是一门动态、解释型、强类型高级开发编程语言



## 变量

 把程序运行过程中产生的值存储起来，方便后面程序调用



### 变量的命名规范

- 由数字，字母，下划线组成
- 不能数字开头，更不能是纯数字
- 不能是python的关键字
- 不要太长
- 要有意义
- 区分大小写
- 不要用中文

推荐：1）驼峰体：每个单词的首字母大写

​      2）下划线：单词用下划线连接



## 基本数据类型(int, bool)

### int

- 操作：+，-，*，/，%，//
- **xxx.bit_length()：**返回xxx的二进制长度

```python
a = 10
print(a.bit_length()) # 4
```

### bool

- True
- False



### 类型转换

- bool ---> int

```python
bs = False
print(type(bs)) # <class 'bool'>

x = int(bs)
print(x) # 0
print(type(x)) # <class 'int'>
```

- int ---> bool

```python
print(bool(-1)) # True
print(bool(0))  # False
```

结论：

a) 把x转换成y类型：y(x)

b) 空的东西都是False， 非空的东西是True

c) False：0，''，[]， {}，set()，tuple()，None(真空)



## 用户交互

-  变量 = input("提示语")
-  input默认得到str类型数据



## 格式化输出

- %s：字符串占位符，任何数据类型都适用
- %d：数字占位符，映变量类型必须为int，否则程序报错
- 在字符串如果使用了%s这样的占位符，那么所有的%都将变成占位符，此时需要使用%%来表示字符串的%
- 如果字符串没有使用%s，%d占位，则无需使用%%来表示字符串的%

```python
print("我叫%s, 今年%s岁" % ("赛利亚", 56))  # 我叫赛利亚, 今年56岁

print("我叫%s, 我已经拥有了全国0.01%%的财产了" % ("赛利亚")) # 我叫赛利亚, 我已经拥有了全国0.01%的财产了

print("我叫赛利亚， 我已经学习了2%的python了") # 我叫赛利亚， 我已经学习了2%的python了
```



## 运算符

- and：并且，左右两端都为真，结果为真，否则为假
- or：或者，左右两端有一个为真，结果为真，左右两端都为假，则结果为假
- not：取反，非真即假，非假即真



### 运算优先级

- () ---> not ---> and ---> or

```
print(3 > 2 and 4 > 6 and 5 < 7 and 7 > 8) # False
print(4 > 6 or 7 < 5 or 5 > 8 or 7 > 9 or 5 > 3) # True
```



### 当左右两端是数字时

- x or y，若x为真，则值为x，否则为y
- x and y，与or相反

```
print(3 and 0 or 5 and 4 or 6 and 8) # 4
```



## 注释

- 单行注释：#
- 多行注释：

```
"""多行注释
写多少行都行
"""
```



## if语句

### if结构：

```
if 条件：
    if-语句块
```



### if-else结构：

```
if 条件：
    if-语句块
else:
    else-语句块
```



### if-elif-else结构：

```
if 条件：
    if-语句块
elif 条件：
    elif-语句块
...
else:
    else-语句块
```

if语句里可以嵌套if语句，但不要超过3层，最多5层。



## 循环

### while循环

while 结构：

```
while 结构：
    while语句块(循环体)
```

 执行顺序：判断条件是否为真，如果为真则执行循环体，否则跳出循环。执行完循环体之后再次判断条件是否为真，直到为假为止。



break and continue

- break：结束本层循环
- continue：结束本层本次循环，继续执行下一次循环



while-else结构

```
while 条件：
    while语句块(循环体)
else:
    pass
```

- 执行顺序：当条件成立时执行循环体，当条件不成立时执行else里的代码。
- 如果循环是通过break退出的，那么while后的else将不会执行，只有在while条件判断是假的时候才会执行这个else。
