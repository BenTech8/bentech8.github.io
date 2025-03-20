---
title: 日志
date: 2019-03-20 17:02:40
tags:
- Python
categories:
- Python
---

## basicConfig

- 导入logging模块
- 简单配置一下logging
- 出现异常的时候（except），向日志里写错误信息

```python
import logging
import traceback


logging.basicConfig(filename="x1.log",
                    format="%(asctime)s - %(name)s - %(levelname)s - %(module)s: %(message)s",
                    datefmt="%Y-%m-%d %H:%M:%S", level=30)
try:
    print(1/0)
except Exception:
    logging.error(traceback.format_exc())
    print("出现错误")
```

- filename：文件名
- format：数据的格式化输出，最终在日志文件中的样子。时间-名称-级别-模块：错误信息
- datefmt：时间的格式
- level:错误的级别权重，当错误的级别权重大于等于leval的时候才会写入文件

CRITICAL = 50

FATAL = CRITICAL

ERROR = 40

WARNING = 30

WARN = WARNING

INFO = 20

DEBUG = 10

LOG = 0



## FileHandler

- 可实现日志分开记录

```python
import logging
import traceback

file_handler = logging.FileHandler("x2.log", "a", encoding='UTF-8')
file_handler.setFormatter(logging.Formatter(fmt="%(asctime)s - %(name)s - %(levelname)s - %(module)s: %(message)s"))

logger1 = logging.Logger("系统A", level=30)
logger1.addHandler(file_handler)

logger1.error("出现错误")


file_handler = logging.FileHandler("x3.log", "a", encoding='UTF-8')
file_handler.setFormatter(logging.Formatter(fmt="%(asctime)s - %(name)s - %(levelname)s - %(module)s: %(message)s"))

logger2 = logging.Logger("系统B", level=30)
logger2.addHandler(file_handler)

try:
    print(1/0)
except Exception:
    logger2.error(traceback.format_exc())
    print("出错了")
```
