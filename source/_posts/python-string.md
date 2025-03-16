---
title: String
date: 2025-03-16 16:30:33
tags:
- Python
categories:
- Python
---

## 简介

-  把字符连成串，在python中用', '', ''', """引起来的内容被称为字符串
-  字符串是不可变对象，所有操作都是产生新字符串返回



## 切片和索引

### 索引

 索引就是下标，下标从**0**开始

```
s1 = 'python'

print(s1[0]) # 获取第一个
print(s1[6]) #越界，报错
print(s1[-1]) # 获取倒数第一个
print(s1[-2]) # 获取倒数第二个
```



### 切片

-  使用下标来截取部分字符串的内容
-  **语法：str[start: end]**
-  规则：顾头不顾尾，从start 开始截取，截取到end位置，但不包括end

```
s2 = "python"
print(s2[0:3]) # pyt
print(s2[4:6]) # on
print(s2[3:]) # hon
print(s2[-1:-3]) # 从-1获取到-5这样是获取不到任何结果的，默认从左往右数
print(s2[-3:-1]) # ho
print(s2[:]) # python
 
```

**步长：**

 步长如果是正数，则从左往右取，如果是负数，则从右往左取。默认为1

```
s3 = ""python"

print(s3[1:5:2]) # yh
print(s3[-1:-5:-1]) #noht
print(s3[-2::-2]) # otp
```



## 方法

### 大小写

- capitalize

**功能**：对第一个单词首字母大写，对于非第一个单词首字母进行检查，若大写则改为小写。

```
s1 = "hello world"

print(s1.capitalize()) # Hello worlds2 = "heLLo"print(s2.capitalize()) # Hello 
```

- lower

**功能**：将字符串全部转换成小写

```
s1 = "HeLLo WoRld"

print(s1.lower()) # hello world
```

- upper

**功能**：将字符串全部转换成大写

```
s1 = "hello world"

print(s1.upper()) # HELLO WORLD
```

- swapcase

**功能**：大小写互相转换

```
s1 = "Hello World"

print(s1.swapcase()) # hELLO wORLD
```

- casefold

**功能**：转换成小写

和lower的区别：lower()对某些字符支持不够好。casefold()对所有的字母都有效。比如东欧的一些字母。

```
s2 = "БBß" # 俄美德字符

print(s2.casefold()) # 6bss
print(s2.lower()) # 6bß
```

- title

**功能**：每个被特殊字符隔开的字母首字母大写

```
s1 = "hello world"
s2 = "pyt你好hon"

print(s1.title()) # Hello World
print(s2.title()) # Pyt你好Hon
```

###  

### 切来切去

- center

**功能**：居中

```
s1 = "python"

print(s1.center(10)) #  python  
print(s1.center(10, "*")) # **python**
```

- expandtabs

**功能**：更改tab的长度，默认长度改为8

```
s1 = “pyth\ton”

print(s1.expandtabs()) # pyth    on
```

- strip

**功能**：默认去掉左右两端的空白(空格，\n, \t)

```
s1 = "  \n  python  \t  "

print(s1.strip()) # pythons2 = "hello world"print(s2.lstrip("h").rstrip("d") # ello worl
```

- lstrip

**功能**：默认去掉左端的空白(空格，\n, \t)

- rstrip

**功能**：默认去掉右端的空白(空格，\n, \t)

- replace

**功能**：字符串替换

```
s1 = "hello world"

ret = s1.replace("o", "p") # hellp wprld
ret = s1.replace("o", "p",1) # hellp world
```

- split

**功能**：字符串切割

- 当括号内为无时即s.split()，此时根据空白切割
- s.split("|", 1),根据字符串s中第一个"|"切割

```
s1 = "python, java, c, c++, c#"

print(s1.split(",") # ["python", "java", "c", "c++", "c#"
s = "py|th|on"
print(s.split("|", 1))  # ['py', 'th|on']
print(int("  1  ")) # 1 int()会自动去除左右两端的空白
```

- join

**功能**：将列表名元素按给定字符连接

```
s = "hello"

ss = "_".join(s)
print(ss)  # h_e_l_l_o
lst = ["hello", "world"]

s = "_".join(lst)
print(s) # hello_world

```



### 查找

- startswith

**功能**：判断是否以某个/些字符开头

```
s1 = "Beautiful is better than ugly"

print(s1.startswith("Beautiful")) # True
```

- endswith

**功能**：判数是否以某个/些字符结尾

```
s1 = "Beautiful is better than ugly"

print(s1.startswith("ugly")) # True
```

- count

**功能**：查找某个/些字符出现的次数

```
s1 = "python"

print(s1.count("h")) # 1print("hello".count("l", 0, 3)) # 1  判断"hello"[0:3]中"l"出现多少次
```

- find

**功能**：查找某个/些字符出现的位置，如果没有返回-1

如果查找的某个字符在字符串中出现多次，则查找所得的结果是第一个字符出现的位置。

```
s1 = "python, java, c, c++, php"

print(s1.find("java")) # 8
```

切片找：

```
s1 = "python, java, c, c++, php"

print(s1.find("java", 12, 25)) # -1 
```

- rfind

从字符串末尾开始查找

- index

**功能**：求索引位置，如果找不到索引，程序会报错

```
s1 = "python, java, c, c++, php"

print(s1.index("java")) # 8
```

- rindex

从字符串末尾开始查找



### 格式化输出 

- format

```
print("我叫%s, 今年%d岁了， 我喜欢%s"%("赛利亚"， 18， "玩"))

print("我叫{}, 今年{}岁了， 我喜欢{}".format("赛利亚"， 18， "玩"))

print("我叫{0}, 今年{2}岁了， 我喜欢{1}".format("赛利亚", "玩", 18))

print("我叫{name}, 今年{age}岁了， 我喜欢{hobby}".format(name="赛利亚", hobby="玩", age=18))
```



### 条件判断

- isalnum

**功能**：是否由字母和数字(int)组成，可以理解为包含isalpha和isnumeric的功能

```
s1 = "123.16"
s2= "abc"
s3 = "_abc!@"s4 = "adfgsddfrd123456789壹贰叁肆伍陆柒一二三"

print(s1.isalnum()) # False
print(s2.isalnum()) # True
print(s3.isalnum()) # Falseprint(s4.isalnum()) # True
```

- isalpha

**功能**：是否由字母组成

```
s1 = "123.16"
s2= "abc"
s3 = "_abc!@"s4 = "中国"

print(s1.isalpha()) # False

print(s2.isalpha()) # True

print(s3.isalpha()) # Falseprint(s4.isalpha()) # True
```

- isdigit

**功能**：是否由阿拉伯数字(%d)组成

```
s1 = "123"
s2 = "123.16"

print(s1.isdigit()) # True
print(s2.isdigit()) # False
```

- isdecimal

 

 

- isnumeric

```
a = "123456789壹贰叁肆伍陆柒一二三四五"

print(a.isnumeric()) # True
```



### 计算字符串长度

- len

 len()是python的内置函数。

```
s1 = "python"

print(len(s1)) # 6
```



### 迭代

-  可以使用for循环来遍历字符串中的每一个字符
-  可迭代对象：可以一个一个往外取值的对象

```
s1 = "hello world"

for c in s1:    
    print(c)
```
