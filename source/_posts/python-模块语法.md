---
title: 模块语法
date: 2019-03-20 17:06:56
tags:
- Python
categories:
- Python
---

## 模块

  模块就是一个包含python定义和声明的文件，文件名就是模块的名字加上.py后缀。换句话说，所有的py文件都可以看成是一个模块。

  模块名有两个：py文件名， \__main__



### 模块的四个通用类别

- 使用python编写的py文件
- 已被编译为共享库或者DLL或C或者C++的扩展
- 包好一组模块的包
- 使用c编写并连接到python解释器的内置模块



### 使用模块的好处

- 可以把代码进行分类
- 可以实现代码的重用



### 导入模块时执行的三个动作

- 先在sys.modules中查看当前导入的模块是否已经被导入。如果已导入，则不会再重复导入，也就不会执行下面两个操作
- 开辟一片内存空间，在该空间中执行模块中的代码
- 创建模块的名字。并使用该名称作为该模块在当前模块中引用的名字

使用globals()查看模块的名称空间

```python
import random


print(globals())


结果:
{'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x00000275FF8DC2B0>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__file__': 'E:/python_个人/day 024 模块的语法/临时/临时.py', '__cached__': None, 'random': <module 'random' from 'C:\\Python36\\lib\\random.py'>}
```

 

### 导入模块的顺序

- 先引入内置模块
- 再引入第三方模块
- 最后引入自己定义的模块



## 导入模块的两种方式

### import xxx

- 应用场景：当不确定要使用模块中哪些功能时。
- 设置别名：import xxx as xxx

模块在导入的时候会创建其自己的名称空间。所以，在使用模块中的变量的时候一般是不会产生冲突的

```python
test模块：

main_person_one = "Jhon"


def foo():
    print(main_person_one) 
```

```python
# 调用方
import test

main_person_one = "Tom"

test.foo()  # Jhon
print(main_person_one)  # Tom
```

在模块中使用global

- global表示把全局的内容引入到局部，但这个全局指的是py文件。换句话说，global指向的是模块内部，并不会改变外部模块的内容

```python
# 模块test中

main_person_one = "Jhon"


def foo():
    global main_person_one
    main_person_one = "Tom"
    print(main_person_one)
```

```python
# 调用方
import test

main_person_one = "Linda"

test.foo()  # Tom

print(test.main_person_one)  # Tom
print(main_person_one)  # Linda
```



### from xxx import xxx

  使用from xxx import xxx 时，与import方式一样，python也会给模块创建名称空间。但不同的是，import会把整个模块内容引入过来，而from方式则是部分引入。当一个模块中的内容过多时，可以采用此方式选择性的导入要使用的内容

- 应用场景：当确定要使用模块中哪些功能时
- 设置别名：from xxx import xxx as xxx
- from xxx import *：导入所有内容

当从一个模块中引入一个变量时，如果当前文件中出现了重名的变量时，会覆盖掉模块引入的那个变量

```python
# test模块：

main_person_one = "Jhon"


def foo():
    print(main_person_one)
```

```python
# 调用方
from test import main_person_one, foo

main_person_one = "Tom"

foo()  # Jhon
print(main_person_one)  # Tom
```



## 修改模块变量的方式(慎用)

- 方式：模块名.变量名 = 值 ， 只能通过import方式修改
- 后果：当某一调用方对模块变量进行修改后，所有调用方拿到的这个变量的值都发生变化，因为模块只会导入一次，共享名称空间。

```python
# test模块：

main_person_one = "Jhon"


def foo():
    print(main_person_one)
```

```python
# 调用方
import test

test.main_person_one = "Tom"

print(test.main_person_one)  # Tom
```



## \__name__

  每个模块在完成代码编写后，都需对其功能进行测试，此时需要对其编写测试代码。但被调用方导入时不需执行测试代码。此时可以利用\__name__过滤掉测试代码的执行

  在python中，每个模块都有自己的\__name\_\_，但\_\_name\_\_的值是不定的。当把一个模块作为程序的入口时，此时该模块的\_\_name\_\_是"\_\_main\_\_"，而如果把模块导入时，此时模块内部的\_\_name__就是该模块自身的名字

运行test模块时：

```python
# test模块：

def foo():
    print('----test----')


print(__name__)  # __main__

if __name__ == '__main__':
    foo()  # ----test----
```

运行调用方时：

```
# 调用方
import test结果：test  # 即 test模块中，__name__ = test
```



## \__all__

  在模块中设置__all__， 可以限制模块可允许的导入内容，只对from xxx import * 有效

```python
# test模块
__all__ = ["x", "y"]
x = 2
y = 3
z = 5


def test():
    print("-----test-----")
```

```python
# 调用方
from test import *

print(x)  # 2
print(z)  # NameError: name 'z' is not defined
```
