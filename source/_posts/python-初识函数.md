---
title: 初识函数
date: 2025-03-16 16:57:01
tags:
- Python
categories:
- Python
---

## 函数定义及调用

- 函数：对功能和动作的封装
- 使用def关键字来定义函数
- 使用"函数名()"可调用函数

```
def 函数名():
    函数体函数名()  # 调用函数
```



### 函数名

- 函数名就是变量名，其命名规范与变量名命名规范一致
- 函数名存储的是函数的内存地址

```
def func():
    pass
print(func) # <function func at 0x0000023A55019A60>
```

- 函数名可以赋值给其他变量

```
def func():
    print("Hello World!")

ret = func
ret()  # Hello World!
```

- 函数名可以当做容器类的元素

```
def func_1():
    print("-----1-----")

def func_2():
    print("-----2-----")

def func_3():
    print("-----3-----")

def func_4():
    print("-----4-----")

lst = [func_1, func_2, func_3, func_4]
for i in lst:
    i()

结果：
-----1-----
-----2-----
-----3-----
-----4-----
```

- 函数名可以为函数的参数

```
def func_1():
    print("-----1-----")

def func_2():
    print("-----2-----")

def func(fn):
    fn()

func(func_1)  # -----1-----
```

- 函数名可以为函数的返回值

```
def func():
    def func_1():
        print("-----func_1-----")
    return func_1

func()()  # -----func_1-----
```

 

打印函数名

```
def func():
    def func_1():
        print("-----func_1-----")
    return func_1

print(func.__name__)  # func
```

 

## return

  执行完函数后，可以使用return来返回结果

- return作用：终止函数运行
- 如果函数体无return，则调用函数返回None
- 如果函数体有return，但return后未有值，则返回None
- 如果函数体有return，且return后有一个值或变量名，则返回值或变量名所指向的值
- 如果函数体有return，且return后有多个值，则多个值以元组形式返回给调用者，调用者可直接解构成多个变量

```
def operation(a, b):
    sum = a + b
    sub = a - b
    qud = a * b
    qut = a / b
    return sum, sub, qud, qut

x, y, z, w = operation(1, 2)
    
```



## 参数

### 形参

  写在函数声明的位置的变量

#### 位置参数

```
def regist(id, name, age, gender):
    print(id, name, age, gender)

regist(1, "Tom", 18, "男")  # 1 Tom 18 男
```



#### 默认值参数

  默认值参数可以把函数共性的参数问题进行提炼，以减少输入，提高输入效率

```
def regist(id, name, age, gender="男"):
    print(id, name, age, gender)

regist(1, "Tom", 18) # 1 Tom 18 男
```



#### 混合参数(位置参数+默认值参数)

  位置参数在前，默认值参数在后，否则报错

```
def regist(id, name, gender="男", age):
    print(id, name, age, gender)

regist(1, "Tom", 18) # SyntaxError: non-default argument follows default argument
```



#### 动态参数

a.动态接收位置参数——*args

- *表示动态传参，可以接受所有的位置参数
- 传参的时候自动的把实参打包成元组，交给形参
- 动态传参可以不传参数

```
def func(*args):
    print(args)

func(11, 22, 33, 44, 55, 66)  # (11, 22, 33, 44, 55, 66)
def func(*args):
    print(args)

func()  # ()
```

b.动态接收关键字参数——**kwargs

- 接收到的内容放在字典里

```
def func(**kwargs):
    print(kwargs)


func(a=11, b=22, c=33, d=44, e=55) # {'a': 11, 'b': 22, 'c': 33, 'd': 44, 'e': 55}
```

 

#### 形参顺序

顺序：位置参数，*args，默认值参数，**kwargs

```
def func(a, *args, b="", **kwargs):
    print(a, args, b, kwargs)


func(11, 22, 33, 44, b="男", c=66, d=77,) # 11 (22, 33, 44) 男 {'c': 66, 'd': 77}
```

 

### 打散

- \* + string/list/

```
print(*[1, 2, 3, 4, 5, 6]) # 1 2 3 4 5 6
```

- ** + dict

```
def func(**kwargs):
    print(kwargs)

func(**{"name": "Tom", "age": 18, "hobby": "football"})  # {'name': 'Tom', 'age': 18, 'hobby': 'football'}
```

 

### 无敌传参

```
def func(*args, **kwargs):
    print(args, kwargs)


func(11, 22, 33, 44, a=55, b=66, c=77, d=88)  #(11, 22, 33, 44) {'a': 55, 'b': 66, 'c': 77, 'd': 88}
```

 

### 实参

  在函数调用的时候给函数传递的值。实际执行的时候给函数传递的信息

#### 位置参数

  按照形参顺序给把各实参赋值给相应的形参

```
def introduction(name, age):
    print("My name is %s, I'm %s years old." % (name, age))

introduction("Tom", 18)  # My name is Tom, I'm 18 years old.
def introduction(name, age):
    print("My name is %s, I'm %s years old." % (name, age))

introduction(18, "Tom")  # My name is 18, I'm Tom years old.

```



#### 关键字参数

  根据形参声明的变量名来传递信息，关键字实参不需要考虑形参顺序

```
def introduction(name, age):
    print("My name is %s, I'm %s years old." % (name, age))

introduction(age=18, name="Tom") # My name is Tom, I'm 18 years old.
```



#### 混合参数(位置参数+关键字参数)

  位置参数在前，关键字参数在后，否则报错

```

def introduction(name, age, sex):
    print("My name is %s, I'm %s years old %s" % (name, age, sex))

introduction("Tom", sex="boy", age=18) # My name is Tom, I'm 18 years old boy
```

 

```
def introduction(name, age, sex):
    print("My name is %s, I'm %s years old %s" % (name, age, sex))

introduction("Tom", sex="boy", 18) # SyntaxError: positional argument follows keyword argument
```

 

### 传参

  给函数传递信息的时候将实际参数交给形式参数的过程



## 命名空间和作用域

### 命名空间

- 内置命名空间：存放python解释器为我们提供的名字，list，tuple，str，int这些都是内置命名空间
- 全局命名空间：py文件中，函数外声明的变量都属于全局命名空间
- 局部命名空间：在函数中声明的变量会放在局部命名空间
- 加载顺序：内置命名空间--->全局命名空间--->局部命名空间(函数被执行的时候)
- 取值顺序：局部命名空间--->全局命名空间--->内置命名空间



### 作用域

- 全局作用域：全局命名空间 + 内置命名空间
- 局部作用域：局部命名空间
- 变量可从局部向全局寻找，但不可从全局向局部寻找



### locals() 和 globals()

- locals()：查看当前作用域中的名字，如果在函数中调用locals()，则查看局部作用域中的名字，如果在函数外调用locals()，则查看全局作用域中的名字
- globals()：查看全局作用域中(内置+全局)中的名字



### global 和 nonlocal

#### global

- 如果函数内部未引入全局变量，则函数可以使用全局变量，但无法修改

```
a = 10

def func():
    b = a + 10
    print(a)
    print(b)

func()  # 10 20
```

- 如果利用global在函数内引入全局变量，则在函数内部可对全局变量进行修改

```
a = 10

def func():
    global a
    a = 20
    print(a)

func()
print(a)  # 20 20
```

- 如果利用global在函数内部引入全局变量，而函数外无此变量，则此局部变量升华为全局变量，即此变量为全局变量

```
def func():
    global a
    a = 20
    print(a)

func()
print(a)  # 20 20 
```



#### nonlocal

- 在局部命名空间，引入上一层名称空间中的名字，如果上一层没有，继续往上一层中寻找，如果整个局部命名空间都无，则报错

```
def func1():
    a = 10
    def func2():
        nonlocal a
        a = 20
        print(a)
    print(a)
    func2()
    print(a)

func1() # 10 20 20
```



## 函数嵌套

- 函数内部可以嵌套函数
- 每一层都会产生独自的名称空间，在寻找变量时，则为向上一层一层寻找。

```
def func1():
    a = 10
    def func2():
        def func3():
            nonlocal a
            a = 20
        print(a)
        func3()
    print(a)
    func2()
    print(a)

func1()  # 10 10 20 
```



##  闭包

  内层函数对外层函数的变量的引用

作用：

- 保护变量不受侵害
- 可以让一个变量常驻内存

```
def outer():
    name = "Tom"
    def inner():
        print(name)
    inner()

outer() # Tom
    
```



###  检测函数是否为闭包

  可以使用__closure__来检测函数是否为闭包

- 如果返回cell，则为闭包
- 如果返回None，则不是闭包

```
def outer():
    name = "Tom"
    def inner():
        print(name)
    inner()
    print(inner.__closure__)

outer()


结果:
Tom
(<cell at 0x000001F751868588: str object at 0x000001F7518F80A0>,)
```



### 变量常驻内存说明

  函数外边调用内部函数：

```
def outer():
    name = "Tom"
    def inner():
        print(name)
    return inner

ret = outer()
ret()  # Tom
```

   如果一个函数执行完毕，则这个函数中的变量以及局部命名空间中的内容都将会被销毁。但由于外界可以通过ret()去访问内部函数。那这个时候内部函数访问的时间和时机就不一定了。如果此时，闭包中变量被销毁了，那么内部函数将不能正常执行。所以，Python规定，如果在内部函数中访问了外层函数的变量，那么这个么变量将不会消亡。
