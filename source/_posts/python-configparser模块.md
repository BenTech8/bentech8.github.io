---
title: configparser模块
date: 2019-03-20 17:51:58
tags:
- Python
categories:
- Python
---

## configparser

  该模块适用格式与windows ini文件类似的配置文件，可以包含一个或多个节（section），每个节可以有多个参数（键=值）



## 创建对象

```python
import configparser

config = configparser.ConfigParser()

config["DEFAULT"] = {
    "sleep": 1000,
    "session-time-out": 30,
    "user-alive": 999999
}

config["TEST-DB"] = {
    "db-ip": "192.168.17.189",
    "port": "3306",
    "u_name": "root",
    "u_pwd": "123456"
}

config["168-DB"] = {
    "db-ip": "152.163.18.168",
    "port": "3306",
    "u_name": "root",
    "u_pwd": "123456"
}

config["173-DB"] = {
    "db_ip": "152.163.18.173",
    "port": "3306",
    "u_name": "root",
    "u_pwd": "123456"
}
```



## 写入到文件

```python
f = open("db.ini", mode="w", encoding="utf-8")
config.write(f)
f.flush()
f.close()
```



## 读取文件信息

### 读取文件

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini")  # 读取文件
```



### 读取章节信息

- DEFAULT章节特殊，它是给每个章节都配备的信息

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件 此时把文件的内容读取到config

print(config.sections())  # ['TEST-DB', '168-DB', '173-DB']
```



### 读取特定章节信息

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

print(config.get("TEST-DB", "DB-IP"))  # 192.168.17.189
print(config["TEST-DB"]["DB-IP"])  # 192.168.17.189
```



### 增删改操作：

  先读取，然后修改，最后写回文件

#### 增

增加一个章节

- add_section()

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

# 添加
config.add_section("189-DB")
config["189-DB"] = {
    "db_ip": "167.76.22.189",
    "port": "3306",
    "u_name": "root",
    "u_pwd": "123456"
}

config.write(open("db.ini", mode="w"))结果：
```

 增加元素

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

config["168-DB"]["code"] = "AFRE"

config.write(open("db.ini", mode="w"))结果：
```



####  删

删除章节

- remove_section(section)

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

config.remove_section("173-DB")

config.write(open("db.ini", mode="w"))结果：
```

 

删除元素

- remove_option(section, 元素)

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

config.remove_option("168-DB", "code")

config.write(open("db.ini", mode="w"))结果：
```



####  改

- set(section, 元素，信息)

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

config.set("168-DB", "db-ip", "10.10.10.168")

config.write(open("db.ini", mode="w"))结果：
```



### 遍历

#### 遍历key

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

for k in config["168-DB"]:
    print(k)

结果：
db-ip
port
u_name
u_pwd
sleep
session-time-out
user-alive
```

 

options()

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

print(config.options("168-DB"))

结果：
['db-ip', 'port', 'u_name', 'u_pwd', 'sleep', 'session-time-out', 'user-alive']
```



### 遍历items

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

for k, v in config["168-DB"].items():
    print(k, v)
    
    
结果：
db-ip 152.163.18.168
port 3306
u_name root
u_pwd 123456
sleep 1000
session-time-out 30
user-alive 999999
```

 

items()

```python
import configparser

config = configparser.ConfigParser()

config.read("db.ini", encoding="utf-8")  # 读取文件

print(config.items("168-DB"))

结果：
[('sleep', '1000'), ('session-time-out', '30'), ('user-alive', '999999'), ('db-ip', '152.163.18.168'), ('port', '3306'), ('u_name', 'root'), ('u_pwd', '123456')
```
