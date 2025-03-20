---
title: time模块
date: 2019-03-20 17:32:54
tags:
- Python
categories:
- Python
---

## 时间戳

- 返回从1970年01月01日00点00分00秒到现在一共经过了多少秒。用float表示

```python
import time

print(time.time())  # 1542203668.0212119
```



## 格式化时间

- 根据需求对时间进行任意的格式化

```python
import time

s = time.strftime("%Y-%m-%d %H:%M:%S")
print(s)  # 2018-11-14 22:01:40
```

中文：

```python
import time
# 中文
import locale
locale.setlocale(locale.LC_CTYPE, "chinese")

s = time.strftime("%Y年%m月%d日")
print(s)  # 2018年11月18日
```

 

日期格式化标准：

- %y：两位数的年份表示(00-99)
- %Y：四位数的年份表示(000-999)
- %m：月份(01-12)
- %d：月内中的一天(0-31)
- %H：24小时制小时数(0-23)
- %I：12小时制小时数(01-12)
- %M：分钟数(00-59)
- %a：本地简化星期名称
- %A：本地完整星期名称
- %b：本地简化的月份名称
- %B：本地完整的月份名称
- %c：本地相应的日期表示和时间表示
- %j：年内的一天(001-366)
- %p：本地A.M或P.M的等价符
- %U：一年中的星期数(00-53)星期天为星期的开始
- %w：星期(0-6)，星期天为星期的开始
- %W：一年中的星期数(00-53)星期一为星期的开始
- %x：本地相应的日期表示
- %X：本地相应的时间表示
- %Z：当前时区的名称
- %%：%本身



## 结构化时间

  时间戳不能直接转化成格式化时间，须先转化为结构化时间，再由结构化时间转化为格式化时间

  同理，格式化时间转化为时间戳，须先转化为结构化时间，再由结构化时间转化为时间戳

### localtime

- 东八区时间

```python
import time

t = time.localtime(1888888888)
print(t)  # time.struct_time(tm_year=2029, tm_mon=11, tm_mday=9, tm_hour=11, tm_min=21, tm_sec=28, tm_wday=4, tm_yday=313, tm_isdst=0)

s = time.strftime("%Y-%m-%d %H:%M:%S", t)
print(s)  # 2029-11-09 11:21:28
```

 

### gmtime

- 格林尼治时间

```python
import time

t = time.gmtime(1888888888)
print(t)  # time.struct_time(tm_year=2029, tm_mon=11, tm_mday=9, tm_hour=3, tm_min=21, tm_sec=28, tm_wday=4, tm_yday=313, tm_isdst=0)

s = time.strftime("%Y-%m-%d %H:%M:%S", t)
print(s)  # 2029-11-09 03:21:28
```



## 时间转换

### 时间戳 ------> 格式化时间

- localtime/gmtime + strftime

```python
import time

t = time.localtime(1888888888)
print(t)  # time.struct_time(tm_year=2029, tm_mon=11, tm_mday=9, tm_hour=11, tm_min=21, tm_sec=28, tm_wday=4, tm_yday=313, tm_isdst=0)

s = time.strftime("%Y-%m-%d %H:%M:%S", t)
print(s)  # 2029-11-09 11:21:28
```



### 格式化时间 ------> 时间戳

#### strptime(string, 格式)

  将格式化时间转化为结构化时间

#### mktime() 

  将结构化时间转化为时间戳

```python
import time


s = "2029-11-09 11:21:28"

t = time.strptime(s, "%Y-%m-%d %H:%M:%S")
print(t)  # time.struct_time(tm_year=2029, tm_mon=11, tm_mday=9, tm_hour=11, tm_min=21, tm_sec=28, tm_wday=4, tm_yday=313, tm_isdst=-1)

tt = time.mktime(t)
print(tt)  # 1888888888.0
```



##  时间差

方法一：

```python
import time

begin = "2018-11-14 16:30:00"
end = "2018-11-14 18:00:00"

begin_struct_time = time.strptime(begin, "%Y-%m-%d %H:%M:%S")
end_struct_time = time.strptime(end, "%Y-%m-%d %H:%M:%S")

begin_second = time.mktime(begin_struct_time)
end_second = time.mktime(end_struct_time)

diff_time_sec = end_second - begin_second

diff_hour, diff_min_1 = divmod(diff_time_sec, 3600)
diff_min, diff_sec = divmod(diff_min_1, 60)

print("时间差是 %s小%s分钟%s秒" % (int(diff_hour), int(diff_min), int(diff_sec)))

结果：
时间差是 1小30分钟0秒
```

 方法二：

```python
import time

begin = "2018-11-14 16:30:00"
end = "2018-11-14 18:00:00"

begin_struct_time = time.strptime(begin, "%Y-%m-%d %H:%M:%S")
end_struct_time = time.strptime(end, "%Y-%m-%d %H:%M:%S")

begin_second = time.mktime(begin_struct_time)
end_second = time.mktime(end_struct_time)

diff_time_sec = abs(begin_second - end_second)  # 秒级时间差

# 转化成结构化时间
t = time.gmtime(diff_time_sec)  # 用格林尼治时间，否则会有时间差
# time.struct_time(tm_year=1970, tm_mon=1, tm_mday=1, tm_hour=1, tm_min=30, tm_sec=0, tm_wday=3, tm_yday=1, tm_isdst=0)

print("时间差是%s年 %s月 %s天 %s小时 %s分钟" % (t.tm_year - 1970, t.tm_mon - 1, t.tm_mday - 1, t.tm_hour, t.tm_min))


结果：
时间差是0年 0月 0天 1小时 30分钟
```
