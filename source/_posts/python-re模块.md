---
title: re模块
date: 2019-03-20 18:00:04
tags:
- Python
categories:
- Python
---

## re模块

  re模块是python提供的一套关于处理正则表达式的模块。



## 核心功能

### search

作用：搜索

- 搜索到结果就返回。
- 如果有多个结果，只返回第一个结果，且多次调用，返回的都是第一个结果
- 如果匹配不上就报错

```python
import re

res = re.search(r"o", "hello world")
print(res.group())   # o

s = re.search(r"c", "hello world")
print(s.group())  # AttributeError: 'NoneType' object has no attribute 'group'
```



###  match

作用：从开头匹配

- 如果匹配到了，就返回
- 如果匹配不到，就报错

```python
import re

res = re.match(r"h", "hello world")
print(res.group())   # h

s = re.match(r"c", "hello world")
print(s.group())  # AttributeError: 'NoneType' object has no attribute 'group'
```



### findall

作用：查找所有，返回list

```python
import re

lst = re.findall(r"\d+", "name Tom age 18 phone 2354786")
print(lst)  # ['18', '2354786']
```

- 对于正则表达中的组"()"，findall会优先把匹配结果组里的内容返回

```python
import re

lst = re.findall(r"www\.(baidu|qq)\.com", "www.baidu.com")
print(lst)  # ['baidu']
```

 

- 如果想要返回匹配结果，添加"？："取消权限即可

```python
import re

lst = re.findall(r"www\.(?:baidu|qq)\.com", "www.baidu.com")
print(lst)  # ['www.baidu.com']
```

### finditer

作用：查找所有，返回迭代器

```python
import re

lst = re.finditer(r"\d+", "name Tom age 18 phone 2354786")
for el in lst:
    print(el.group())

结果：
18
2354786
```



## 其他操作

### split

作用：分割，返回list

```python
import re

ret = re.split(r"[abc]", "qwerafjbfcd")  # 先按a分割，再按b分割，然后按c分割 
print(ret)  # ['qwer', 'fj', 'f', 'd']
```

 

- 在匹配部分加上"()"与不加所得出的结果不同。这个在某些需要保留匹配部分的使用过程是非常重要的

```python
import re

ret = re.split("\d+", "eva3egon4yuan")
print(ret)  # ['eva', 'egon', 'yuan']
```

 

```python
import re

ret = re.split("(\d+)", "eva3egon4yuan")
print(ret)  # ['eva', '3', 'egon', '4', 'yuan']
```

 

### sub

作用：替换

```python
import re

ret = re.sub(r"\s", "__", "hello world")
print(ret)  # hello__world
```

 

### subn

作用：替换，返回元组（替换的结果，替换次数）

```python
import re

ret = re.subn(r"\s", "__", "name age gender phone")
print(ret)  # ('name__age__gender__phone', 3)
```

 

### compile

作用：将正则表达式编译成一个正则表达式对象，进行预加载

```python
import re

obj = re.compile(r"\d{3}")

ret = obj.search("abc333eee")
print(ret.group())  # 333
```



## re.S

  正则表达式中，"."表示匹配除"\n"以外的所有字符。对于字符串中有换行，此时正则匹配到的则是多个字符串，而利用re.S，"."可以匹配"\n"，即得到的就是一个整体字符串。



## 对单一页面内容的抓取

```python
from urllib.request import urlopen
import re

# url
url = "url"

# 获取全部内容
content = urlopen(url).read().decode()

# 预加载正则表达
obj = re.compile(r"正则表达")

# 获取特定内容
res = obj.search(content).group("组名")
```



 

## 对同一结构，多页面内容的抓取

```python
from urllib.request import urlopen
import re

# 预加载正则表达式
obj = re.compile(r'<div class="item">.*?<span class="title">(?P<name>.*?)</span>.*?导演: (?P<director>.*?)&nbsp;&nbsp;&nbsp;.*?<span class="rating_num" property="v:average">(?P<score>.*?)</span>.*?<span>(?P<people>.*?)人评价</span>', re.S)


def get_content(url):
    """
    获取内容
    :param url: 网址
    :return: 网页全部内容
    """
    content = urlopen(url).read().decode("utf-8")
    return content


def parse_content(content):
    """
    解析内容
    :param content: 网页全部内容
    :return: 字典形式的所需内容
    """
    pc = obj.finditer(content)
    for el in pc:
        yield {
            "name": el.group("name"),
            "director": el.group("director"),
            "score": el.group("score"),
            "people": el.group("people")
        }


def main():
    """
    获取并解析内容，将所需内容写入文件中
    :return: None
    """
    for i in range(10):
        url = "https://movie.douban.com/top250?start=%s&filter=" % (i*25)
        p = parse_content(get_content(url))
        with open("movie.txt", mode="a", encoding="utf-8") as f:
            for el in p:
                f.write(str(el) + "\n")


if __name__ == "__main__":
    main()
```

