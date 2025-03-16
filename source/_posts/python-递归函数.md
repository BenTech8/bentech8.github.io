---
title: 递归函数
date: 2019-03-16 17:20:24
tags:
- Python
categories:
- Python
---

## 递归函数

### 定义

- 在函数中调用函数本身，就是递归
- 在python中递归的深度最大为1000，但实际达不到1000

```
def func():
    print("-----func-----")
    func()

func()
```

### 应用

- 可以使用递归来遍历各种树形结构，比如文件夹系统：可以使用递归来遍历该文件夹中的所有文件

```
import os


def func(filepath, n):
    files_list = os.listdir(filepath)  # 获取当前文件夹中的所有文件
    for file in files_list:
        file_d = os.path.join(filepath, file)  # 拼接文件的真实路径
        if os.path.isdir(file_d):  # 递归入口  判断文件是否为文件夹
            print("\t"*n, file)
            func(file_d, n+1)  #
        else:
            print("\t"*n, file)  # 递归出口
```



## 二分查找

- 优点：每次能够除掉一半的数据，查找效率高
- 要求：查找的序列必须是有序序列

### 非递归算法

#### 利用索引

```python
# 让用户输入一个数n. 判断这个n是否出现在lst中
lst = [4, 56, 178, 253, 625, 1475, 2580, 3574, 15963]

left = 0
right = len(lst) - 1

num = int(input("请输入一个数n："))
while left <= right:
    mid = (left + right) // 2
    if lst[mid] > num:
        right = mid - 1
    elif lst[mid] < num:
        left = mid + 1
    else:
        print("这个数在lst中")
        break
else:
    print("这个数不在lst中")
```



### 递归算法

#### 利用索引

```python
# 让用户输入一个数n. 判断这个n是否出现在lst中
lst = [4, 56, 178, 253, 625, 1475, 2580, 3574, 15963]


def binary_search(lst, num, left, right):
    if left > right:
        return False
    mid = (left + right) // 2
    if lst[mid] > num:
        right = mid - 1
        return binary_search(lst, num, left, right)
    elif lst[mid] < num:
        left = mid + 1
        return binary_search(lst, num, left, right)
    else:return True

num = int(input("请输入一个数n："))
ret = binary_search(lst, num, 0, len(lst)-1)
print(ret)
```



#### 切换列表

```python
# 让用户输入一个数n. 判断这个n是否出现在lst中
lst = [4, 56, 178, 253, 625, 1475, 2580, 3574, 15963]


def binary_search(lst, num):
    if len(lst) == 0:
        return False
    mid = (len(lst) - 1) // 2
    if num > lst[mid]:
        return binary_search(lst[mid+1:], num)
    elif num < lst[mid]:
        return binary_search(lst[:mid], num)
    else:
        print("这个数在lst中")
        return True


num = int(input("请输入一个数n："))

ret = binary_search(lst, num)
print(ret)
```

