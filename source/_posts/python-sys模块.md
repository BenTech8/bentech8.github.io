---
title: sys模块
date: 2019-03-20 17:40:03
tags:
- Python
categories:
- Python
---

## sys模块

  所有和python解释器相关的都在sys模块中



## 相关功能

### path

- 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值

```python
mport sys

print(sys.path)


结果：
['E:\\python_个人\\day 023 re模块', 'E:\\python_个人', 'C:\\Python36\\python36.zip', 'C:\\Python36\\DLLs', 'C:\\Python36\\lib', 'C:\\Python36', 'C:\\Python36\\lib\\site-packages', 'E:\\Program Files\\JetBrains\\PyCharm 2018.2.4\\helpers\\pycharm_matplotlib_backend']
```

由返回结果可知，返回的是列表，由此，可通过append方法手动增加搜索路径

```python
import sys

sys.path.append("e:/")
print(sys.path)


# 结果：
['E:\\python_个人\\day 023 re模块', 'E:\\python_个人', 'C:\\Python36\\python36.zip', 'C:\\Python36\\DLLs', 'C:\\Python36\\lib', 'C:\\Python36', 'C:\\Python36\\lib\\site-packages', 'E:\\Program Files\\JetBrains\\PyCharm 2018.2.4\\helpers\\pycharm_matplotlib_backend', 'e:/']
```

 

### argv

- 命令行参数List，第一个元素是程序本身路径

如:在E盘下新建test.py文件，文件里写入import sys/print(sys.argv)内容，在cmd下，运行test.py zhang li，则列表里接收到zhang li参数



###  exit(n)

- 退出程序
- exit(0)：正常退出
- exit(1)：错误退出



### version

- 获取Python解释器的版本信息

```python
import sys

print(sys.version)  # 3.6.6 (v3.6.6:4cf1f54eb7, Jun 27 2018, 03:37:03) [MSC v.1900 64 bit (AMD64)]
```



### platform

- 返回操作系统平台名称

```py
import sys

print(sys.platform)  # win32
```
