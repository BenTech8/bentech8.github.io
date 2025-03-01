---
title: Shell编程
date: 2025-02-27 11:12:53
tags:
categories: Shell
---

## Shell编程

### 正则表达式

#### 概述

正则表达式用来在文件中匹配符合条件的字符串，通配符用来匹配符合条件的文件名。其实这种区别只在Shell当中适用，因为用来文件当中搜索字符串的命令如grep、awk、sed等命令可以支持正则表达式，而在系统当中搜索文件的命令如ls、find、cp这些命令不支持正则表达式，所以只能使用Shell自己的通配符来进行匹配了。



#### 基础正则表达式

| 元字符    | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| *         | 前一个字符匹配0次或任意多次                                  |
| .         | 匹配除了换行符外任意一个字符                                 |
| ^         | 匹配行首。例如^hello会匹配以hello开头的行                    |
| $         | 匹配行尾。例如hello$会匹配以hello结尾的行                    |
| []        | 匹配中括号中指定的任意一个字符，只匹配一个字符。例如[aoeiu]匹配任意一个元音字母 |
| [^]       | 匹配除中括号的字符以外的任意一个字符。例如\[^0-9]匹配任意一位非数字字符 |
| \         | 转义符。用于取消特殊符号的转义                               |
| \\{n\\}   | 表示其前面的字符恰好出现n次。例如[0-9]\\{4\\}匹配4位数字     |
| \\{n, \\} | 表示其前面的字符出现不小于n次。例如[0-9]\\{2, \\}表示两位及以上的数字 |
| \\{n,m\\} | 表示其前面的字符至少出现n次，最多出现m次。例如[a-z]\\{6,8\\}匹配6到8位的小写字母。 |

练习文件建立：

```shell
[root@localhost ~]# vim test_rule.txt
Mr. Li Ming said:
he was the most honest man.
123despise him.
But since Mr. Zhang San came,
he never saaaid thos words.
5555nice!
because, actuaaaally,
M. Zhang San is the most honest man
Later, Mr. Li Ming soid his hot body.
```

"a*"前一个字符匹配0次，或任意多次

```shell
[root@localhost ~]# grep "a*" test_rule.txt
Mr. Li Ming said:
he was the most honest man.
123despise him.
But since Mr. Zhang San came,
he never saaaid thos words.
5555nice!
because, actuaaaally,
M. Zhang San is the most honest man
Later, Mr. Li Ming soid his hot body.
```

"aa*"代表这行字符串最少要有一个a

```shell
[root@localhost ~]# grep "aa*" test_rule.txt
Mr. Li Ming said:
he was the most honest man.
But since Mr. Zhang San came,
he never saaaid thos words.
because, actuaaaally,
M. Zhang San is the most honest man
Later, Mr. Li Ming soid his hot body.
```

"."匹配除了换行符外任意一个字符，只能匹配一个字符，这个字符可以是任意字符

```shell
[root@localhost ~]# grep "s..d" test_rule.txt
Mr. Li Ming said:
Later, Mr. Li Ming soid his hot body.

[root@localhost ~]# grep "s.*d" test_rule.txt
Mr. Li Ming said:
he never saaaid thos words.
Later, Mr. Li Ming soid his hot body.
```

"^"代表匹配行首，比如“^M”会匹配以大写“M”开头的行

```shell
[root@localhost ~]# grep "^M" test_rule.txt
Mr. Li Ming said:
M. Zhang San is the most honest man
```

"$"代表匹配行尾，比如“n$”会匹配以小写“n”结尾的行

```
[root@localhost ~]# grep "n$" test_rule.txt
M. Zhang San is the most honest man
```

"^$"则会匹配空白行

``` shell
[root@localhost ~]# grep "^$" test_rule.txt

```

"[]"会匹配中括号中指定任意一个字符，注意只能匹配一个字符。比如[ao]会匹配a或o

```shell
[root@localhost ~]# grep "s[ao]id" test_rule.txt
Mr. Li Ming said:
Later, Mr. Li Ming soid his hot body.
```

"[^]"匹配除中括号的字符以外的任意一个字符

```shell
[root@localhost ~]# grep "^[^a-z]" test_rule.txt
Mr. Li Ming said:
123despise him.
But since Mr. Zhang San came,
5555nice!
M. Zhang San is the most honest man
Later, Mr. Li Ming soid his hot body.
```

"\\"转义符

```shell
[root@localhost ~]# grep "\.$" test_rule.txt
he was the most honest man.
123despise him.
he never saaaid thos words.
Later, Mr. Li Ming soid his hot body.
```

"\\{n\\}"表示其前面的字符恰好出现n次

```
[root@localhost ~]# grep "a\{3\}" test_rule.txt
he never saaaid thos words.
because, actuaaaally,
```

"\\{n,\\}"会匹配前面的字符出现最少n次。比如“^[0-9]\\{3, \\}[a-z]”这个正则就能匹配最少用连续三个数字开头的字符串

```shell
[root@localhost ~]# grep "^[0-9]\{3,\}[a-z]" test_rule.txt
123despise him.
5555nice!
```

"\\{n,m\\}"匹配其前面的字符至少出现n次，最多出现m次

```shell
[root@localhost ~]# grep "sa\{1,3\}i" test_rule.txt
Mr. Li Ming said:
he never saaaid thos words.
```



#### 扩展正则表达式

在正则表达式中应该还可以支持一些元字符，比如“+”、“?”、“|”、“()”。其实Linux是支持这些元字符的，只是grep命令默认不支持而已。如果想要支持这些元字符，必须使用egrep命令或grep -E选项，所以我们又把这些元字符称作扩展元字符。

如果查询grep的帮助，对egrep的说明就是和grep -E选项一样的命令，所以我们可以把这两个命令当做别名来对待。

| 扩展元字符 | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| +          | 前一个字符匹配1次或任意多次。如“go+gle”会匹配“gogle”、“google”或“gooogle”等。 |
| ?          | 前一个字符匹配0次或1次。如"colou?r"可以匹配“colour”或“color”。 |
| \|         | 匹配两个或多个分支的选择。如"was\|his"会匹配既包含“was”的行，也匹配包含"his"的行。 |
| ()         | 匹配其整体为一个字符，即模式单元。可以理解为由多个单字符组成的大字符。如“(dog)+”会匹配“dog”、“dogdog”、“dogdogdog”等。因为被()包含的字符会当成一个整体。 |



### 字符截取和替换命令

#### cut

```
[root@localhost ~]# cut [选项] 文件名
选项：
  -f 列号：         提取第几列
  -d 分隔号：       按照指定分隔符分割列
  -c 字符范围：      不依赖分隔符来区分列，而是通过字符范围(行首为0)来进行字段提取。“n-”表示从第n个字符到行尾；"n-m"表示从第n个字符到第m个字符；"-                     m"表示从第1个字符到第m个字符。
```

cut命令的默认分隔符是制表符，也就是“tab”键。

```shell
[root@localhost ~]# vim test.txt
ID      Name    Gender  Mark
1       Liming  M       86
2       Sc      M       90
3       Tg      M       83
```

提取第二列：

```shell
[root@localhost ~]# cut -f 2 test.txt
Name
Liming
Sc
Tg
```

提取多列：

```shell
[root@localhost ~]# cut -f 2,3 test.txt
Name	Gender
Liming	M
Sc	M
Tg	M
```

按照字符进行提取：

```shell
[root@localhost ~]# cut -c 8- test.txt        # "8-"代表提取所有行的第8个字符到结尾
	Gender	Mark
g	M	86
90
83
```

以“:”作为分隔符提取：

```
[root@localhost ~]# cut -d ":" -f 1,3 /etc/passwd
root:0
...
```



#### awk

a）printf格式化输出

```
[root@localhost ~]# printf '输出类型输出格式' 输出内容
输出类型：
  %ns:        输出字符串。n是数字指代输出几个字符
  %ni:        输出整数。n是数字指代输出几个数字
  %m.nf:      输出浮点数。m和n是数字，指代输出的整数位和小数位数。如%8.2f代表共输出8位数，其中2位是小数，6位是整数。
输出格式：
  \a：         输出警告声音
  \b：         输出退格键，也就是Backspace键
  \f：         清除屏幕
  \n:          换行
  \r:          回车，也就是Enter键
  \t:          水平输出退格键，也就是Tab键
  \v:          垂直输出退格键，也就是Tab键
```

```
[root@localhost ~]# vim test.txt
ID      Name    PHP     Linux   MySQL   Average 
1       Liming  82      95      86      87.66
2       Sc      74      96      87      85.66
3       Tg      99      83      93      91.66
```

使用printf输出test.txt文件的内容：

```
[root@localhost ~]# printf '%s' $(cat test.txt)
IDNamePHPLinuxMySQLAverage1Liming82958687.662Sc74968785.663Tg99839391.66[root@localhost ~]#
```

如果不指定输出格式，则会把所有输出内容连在一起输出。其实文本的输出本身就是这样的，cat等文本输出命令之所以可以按照格式漂亮的输出，那是因为cat命令已经设定了输出格式。为了用printf输出合理的格式，可以这样做：

```shell
# 注意在printf命令的单引号中，只能识别格式输出符号，而手工输入的空格是无效的
[root@localhost ~]# printf '%s\t %s\t %s\t %s\t %s\t %s\t \n' $(cat test.txt)
ID	 Name	 PHP	 Linux	 MySQL	 Average	 
1	 Liming	 82	 95	 86	 87.66	 
2	 Sc	 74	 96	 87	 85.66	 
3	 Tg	 99	 83	 93	 91.66
```

如果不想把成绩当成字符串输出，而是按照整形和浮点型输出，则要这样：

```shell
[root@localhost ~]# printf '%s\t %s\t %i\t %i\t %i\t %8.2f\t \n' $(cat test.txt | grep -v Name)
1	 Liming	 82	 95	 86	    87.66	 
2	 Sc	 74	 96	 87	    85.66	 
3	 Tg	 99	 83	 93	    91.66
```



b）awk基本使用

```
[root@localhost ~]# awk '条件1{动作1} 条件2{动作2}...' 文件名
条件(Pattern)：
    一般使用关系表达式作为条件。这些关系表达式非常多。如：
    x > 10    判断变量x是否大于10
    x == y    判断变是x是否等于变量y
    A`B       判断字符串A中是否包含能匹配B表达式的子字符串
    A!`B      判断字符串A中是否不包含能匹配B表达式的子字符串
动作(Action)：
    格式化输出
    流程控制语句
```

```shell 
# 输出第二列和第六列
[root@localhost ~]# awk '{printf $2 "\t" $6 "\n"}' test.txt
Name	Average
Liming	87.66
Sc	85.66
Tg	91.66
```



c）awk的条件

| 条件的类型 | 条件   | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| awk保留字  | BEGIN  | 在awk程序一开始时，尚未读取任何数据之前执行。BEGIN后的动作只在程序开始时执行一次。 |
|            | END    | 在awk程序处理完所有数据，即将结束时执行。END后的动作只在程序结束时执行一次 |
| 关系运算符 | >      | 大于                                                         |
|            | <      | 小于                                                         |
|            | >=     | 大于等于                                                     |
|            | <=     | 小于等于                                                     |
|            | ==     | 等于。用于判断两个值是否相等，如果是给变量赋值，请使用“=”号  |
|            | !=     | 不等于                                                       |
|            | A~B    | 判断字符串A中是否包含能匹配B表达式的子字符串                 |
|            | A!~B   | 判断字符串B中是否不包含能匹配B表达式的子字符串               |
| 正则表达式 | /正则/ | 如果在“//”中可以写入字符，也可以支持正则表达式               |

- BEGIN

  BEGIN是awk的保留字，是一种特殊的条件类型。BEGIN的执行时机是“在awk程序一开始时，尚未读取任何数据之前执行”。一旦BEGIN后的动作执行一次，当awk开始从文件中读入数据，BEGIN的条件就不再成立，所以BEGIN定义的动作只能被执行一次。例如：

  ```shell
  # awk命令只要检测不到完整的单引号不会执行，所以这个命令的换行不用加入"\"
  [root@localhost ~]# awk 'BEGIN{printf "This is a transcript \n"} 
  {printf $2 "\t" $6 "\n"}' test.txt
  This is a transcript 
  Name	Average
  Liming	87.66
  Sc	85.66
  Tg	91.66
  ```

- END

  END也是awk保留字，不过刚好和BEGIN相反。END是在awk程序处理完所有数据，即将结束时执行。END后的动作只在程序结束时执行一次。例如：

  ```shell
  [root@localhost ~]# awk 'END{printf "The END \n"} {printf $2 "\t" $6 "\n"}' test.txt
  Name	Average
  Liming	87.66
  Sc	85.66
  Tg	91.66
  The END
  ```

- 关系运算符

  查看平均成绩大于等于87分的学员是谁：

  ```shell
  [root@localhost ~]# cat test.txt | grep -v Name | awk '$6 >= 87{printf $2 "\n"}'
  Liming
  Tg
  ```

  加入条件之后，只有条件成立动作才会执行，如果条件不满足，则运作不运行。通过这个实验，大家可以发现，虽然awk是列提取命令，但是也要按行来读入的。这个命令的执行过程是这样的：

  1）如果有BEGIN条件，则先执行BEGIN定义的动作

  2）如果没有BEGIN条件，则读入第一行，把第一行的数据依次赋予$0、$1、$2等变量。其中$0代表此行的整体数据，$1代表第一字段，$2代表第二字段。

  3）依据条件类型判断动作是否执行。如果条件符合，则执行动作，否则读入下一行数据。如果没有条件，则每行都执行动作。

  4）读入下一行数据，重复执行以上步骤。

  查看Sc用户的平均成绩：

  ```shell
  [root@localhost ~]# cat test.txt | grep -v Name | awk '$2 ~ "Sc"{printf $6 "\n"}'
  85.66
  ```

- 正则表达式

  如果想让awk识别字符串，必须使用“//”包含，例如：

  ```shell
  # 打印LiMing的成绩
  [root@localhost ~]# awk '/Liming/ {print}' test.txt 
  1	Liming	82	95	86	87.66
  ```

  当使用df命令查看分区使用情况时，如果只想看真正的系统分区的使用状况，而不想查看光盘和临时分区的使用状况，则可以：

  ```
  [root@localhost ~]# df -h | awk '/sda[0-9]/ {printf $1 "\t" $5 "\n"}'
  ```



4）awk内置变量

| awk内置变量 | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| $0          | 代表目前awk所读入的整行数据。我们已知awk是一行一行读入数据的，$0就代表当前读入行的整行数据。 |
| $n          | 代表目前读入行的第n个字段。                                  |
| NF          | 当前行拥有的字段(列)总数。                                   |
| NR          | 当前awk所处理的行，是总数据的第几行。                        |
| FS          | 用户定义分隔符。awk的默认分隔符是任何空格，如果想要使用其他分隔符(如"：")，就需要FS变量定义。 |
| ARGC        | 命令行参数个数。                                             |
| ARGV        | 命令行参数数组。                                             |
| FNR         | 当前文件中的当前记录数(对输入文件起始为1)。                  |
| OFMT        | 数值的输出格式(默认为%.6g)。                                 |
| OFS         | 输出字段的分隔符(默认为空格)。                               |
| ORS         | 输出记录分隔符(默认为换行符)。                               |
| RS          | 输入记录分隔符(默认为换行符)。                               |

查询可以登录的用户的用户名和UID：

```shell
[root@localhost ~]# cat /etc/passwd | grep "/bin/bash" | awk 'BEGIN {FS=":"} {printf $1 "\t" $3 "\n"}'
root	0
ben	1000

[root@localhost ~]# cat /etc/passwd | grep "/bin/bash" | awk 'BEGIN {FS=":"} {printf $1 "\t" $3 "\t 行号:" NR "\t 字段数: " NF "\n"}'
root	0	 行号:1	 字段数: 7
ben	1000	 行号:2	 字段数: 7
```

查看sshd这个伪用户的相关信息：

```shell
[root@localhost ~]# cat /etc/passwd | awk 'BEGIN {FS=":"} $1=="sshd" {printf $1 "\t" $3 "\t 行号：" NR "\t 字段数：" NF "\n"}'
```



5）awk流程控制

统计PHP成绩的总分：

```shell
[root@localhost ~]# awk 'NR==2{php1=$3} NR==3{php2=$3} NR==4{php3=$3;total=php1+php2+php3;print "total php is " total}' test.txt
total php is 255
```

- "NR==2{php1=$3}"：条件是NR==2，动作是php1=$3，指如果输入数据是第二行(第一行是标题行)，就把第二行的第三字段的值赋予变量“php1”。
- “NR==3{php2=$3}”：如果输入数据是第三行，就把第三行的第三字段的值赋予变量“php2”。
- 'NR==4{php3=$3;total=php1+php2+php3;print "total php is " total}'：如果输入数据是第四行，就把第四行的第三字段的值赋予变量“php3”；然后定义变量total的值为‘php1+php2+php3’；然后输出"total php is "关键字，后面加变量total的值。

在awk编程中，因为命令语句非常长，在输入格式时需要注意以下内容：

- 多个条件{动作}可以用空格分割，也可以用回车分割。
- 在一个动作中，如果需要执行多个命令，需要用“;”分割，或用回车分割。
- 在awk中，变量的赋值与调用都不需要加入“$”符。
- 条件中判断两个值是否相同，请使用"=="，以便和变量赋值进行区分。

如果Linux成绩大于90，就是一个好学生：

```shell
[root@localhost ~]# awk '{if (NR>=2){if ($3>90) printf $2 " is a good student!\n"}}' test.txt
Tg is a good student!
```

其实awk中if判断语句，完全可以直接利用awk自带的条件来取代：

```shell
[root@localhost ~]# awk 'NR>=2{test=$3} test>90{printf $2 " is a good student!\n"}' test.txt
Tg is a good student!
```



6）awk函数

awk编程也允许在编程时使用函数，awk函数定义方法：

```
function 函数名 (参数列表) {
函数体
}
```

定义一简单的函数，使用函数来打印test.txt的学员姓名和平均成绩：

```shell
[root@localhost ~]# awk 'function test(a,b) {printf a "\t" b "\n"} {test($2,$6)}' test.txt
Name	Average
Liming	87.66
Sc	85.66
Tg	91.66
```



7）awk中调用脚本

对于小的单行程序来说，将脚本作为命令行自变量传递给awk是非常简单的，而对于多行程序就比较难处理。当程序是多行的时候，使用外部脚本是很适合的。首先在外部文件中写好脚本，然后可以使用awk的-f选项，使其读入脚本并且执行。

例如，我们可以先编写一个awk脚本：

```shell
[root@localhost ~]# vim pass.awk
BEGIN	{FS=":"}
{print $1 "\t" $3}
```

然后可以使用"-f"选项来调用这个脚本：

```shell
[root@localhost ~]# awk -f pass.awk /etc/passwd
root	0
daemon	1
bin	2
sys	3
...
```



#### sed

sed主要是用来将数据进行选取、替换、删除和新增的命令。

```
[root@localhost ~]# sed [选项] '[动作]' 文件名
选项：
  -n:              一般sed命令会把所有数据都输出到屏幕，如果加入此选择，则只会把经过sed命令处理的行输出到屏幕。
  -e:              允许对输入数据应用多条sed命令编辑。
  -f 脚本文件名:    从sed脚本中读入sed操作。和awk命令的-f非常类似。
  -r:              在sed中支持扩展正则表达式。
  -i:              用sed的修改结果直接修改读取数据的文件，而不是由屏幕输出
动作：
  a \n:             追加，在当前行后添加一行或多行。添加多行时，除最后一行外，每行末尾需要用"\"代表数据未完结。
  c \n:             行替换，用c后面的字符串替换原数据行，替换多行时，除最后一行外，每行末尾需用"\"代表数据未完结。
  i \n:             插入，在当前行前插入一行或多行。插入多行时，除最后一行外，每行末尾需要用"\"代表数据未完结。
  d:               删除，删除指定的行。
  p:               打印，输出指定的行。
  s:               字串替换，用一个字符串替换另外一个字符串。格式为"行范围 s/旧字串/新字串/g"（和vim中的替换格式类似）
```

对sed命令要注意，sed所做的修改并不会直接改变文件的内容(如果是用管道符接收的命令的输出，这种情况连文件都没有)，而是把修改结果只显示到屏幕上，除非使用“-i”选项才会直接修改文件。

- 行数据操作

  【查看】

  查看test.txt的第二行：

  ```shell
  [root@localhost ~]# sed '2p' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1       Liming  82      95      86      87.66
  1       Liming  82      95      86      87.66
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  ```

  "p"命令确实输出了第二行数据，但是sed命令还会把所有数据都输出一次。如果想指定输出某行数据，需要"-n"选项的帮助：

  ```shell
  [root@localhost ~]# sed -n '2p' test.txt
  1       Liming  82      95      86      87.66
  ```

  

  【删除】

  删除第二行到第四行的数据

  ```shell
  [root@localhost ~]# sed '2,4d' test.txt
  ID      Name    PHP     Linux   MySQL   Average
  
  # 文件本身并没有修改
  [root@localhost ~]# cat test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1       Liming  82      95      86      87.66
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  ```

  

  【追加】

  “a“会在指定行后面追加入数据：

  ```shell
  [root@localhost ~]# sed '2a hello' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1       Liming  82      95      86      87.66
  hello
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  ```

  

  【插入】

  "i"会在指定行前面插入数据:

  ```shell
  [root@localhost ~]# sed '2i hello\nworld' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  hello
  world
  1       Liming  82      95      86      87.66
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  
  [root@localhost ~]# sed -n '2i hello\nworld' test.txt
  hello
  world
  ```

  

  【替换】

  ```shell
  [root@localhost ~]# sed '2c No such person' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  No such person
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  
  [root@localhost ~]# cat test.txt | sed '2c No sucn person'
  ID      Name    PHP     Linux   MySQL   Average 
  No sucn person
  2       Sc      74      96      87      85.66
  3       Tg      99      83      93      91.66
  ```

  sed命令默认情况下是不会修改文件内容的，如果确定需要让sed命令直接处理文件的内容，可以使用"-i"选项。

  ```shell
  [root@localhost ~]# sed -i '2c No such person' test.txt
  ```

  

- 字符串替换

  "c"动作是进行整行替换的，如果仅仅想替换行中的部分数据，就要使用“s”动作：

  ```shell
  [root@localhost ~]# sed 's/旧字串/新字串/g' 文件名
  ```

  在第三行中，把74替换成99：

  ```shell
  [root@localhost ~]# sed '3s/74/99/g' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1       Liming  82      95      86      87.66
  2       Sc      99      96      87      85.66
  3       Tg      99      83      93      91.66
  ```

  把第四行注释掉：

  ```shell
  [root@localhost ~]# sed '4s/^/#/g' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1       Liming  82      95      86      87.66
  2       Sc      74      96      87      85.66
  #3       Tg      99      83      93      91.66
  ```

  "-e"选项可以同时执行多个sed动作：

  ```shell
  [root@localhost ~]# sed -e 's/Liming//g;s/Tg//g' test.txt
  ID      Name    PHP     Linux   MySQL   Average 
  1         82      95      86      87.66
  2       Sc      74      96      87      85.66
  3             99      83      93      91.66
  ```

  

### 字符处理命令

#### sort

```
[root@localhost ~]# sort [选项] 文件名
选项：
  -f:        忽略大小写
  -b:        忽略每行前面的空白部分
  -n:        以数值型进行排序，默认使用字符串型排序
  -r:        反向排序
  -u:        删除重复行。就是uniq命令
  -t:        指定分隔符，默认分隔符是制表符
  -k n[,m]:  按照指定的字段范围排序。从第n字段开始，m字段结束(默认到行尾)
```

sort命令默认是用每行开头第一个字符来进行排序的：

```shell
[root@localhost ~]# sort /etc/passwd
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:109:116:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
avahi:x:115:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
...
```

反向排序：

```shell
[root@localhost ~]# sort -r /etc/passwd
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
whoopsie:x:120:125::/nonexistent:/bin/false
uuidd:x:107:114::/run/uuidd:/usr/sbin/nologin
...
```

如果想要指定排序的字段，需要先使用“-t”选项指定分隔符，并使用“-k”选项指定字段号。例如按照UID字段排序/etc/passwd文件：

```shell
[root@localhost ~]# sort -n -t ":" -k 3,3 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

当然“-k”选项可以直接使用“-k 3”，代表从第三字段到行尾排序（第一个字符先排序，如果一致，第二个字符再排序，直到行尾）。



#### uniq

uniq命令是用来取消重复行的命令，其实和"sort -u"选项是一样的。

```
[root@localhost ~]# uniq [选项] 文件名
选项：
  -i:        忽略大小写
```



#### wc

统计命令。

```
[root@localhost ~]# wc [选项] 文件名
选项：
  -l:        只统计行数
  -w:        只统计单词数
  -m:        只统计字符数
```



### 条件判断

#### 按照文件类型进行判断

| 测试选项 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| -b 文件  | 判断该文件是否存在，并且是否为块设备文件（是块设备文件为真） |
| -c 文件  | 判断该文件是否存在，并且是否为字符设备文件（是字符设备文件为真） |
| -d 文件  | 判断该文件是否存在，并且是否为目录文件（是目录为真）         |
| -e 文件  | 判断该文件是否存在（存在为真）                               |
| -f 文件  | 判断该文件是否存在，并且是否为普通文件（是普通文件为真）     |
| -L 文件  | 判断该文件是否存在，并且是否为符号链接文件（是符号链接文件为真） |
| -p 文件  | 判断该文件是否存在，并且是否为管道文件（是管道文件为真）     |
| -s 文件  | 判断该文件是否存在，并且是否为非空（非空为真）               |
| -S 文件  | 判断该文件是否存在，并且是否为套接字文件（是套接字文件为真） |

```shell
[root@localhost ~]# [ -e /root/sh/ ]
[root@localhost ~]# echo $?
0        # 判断结果为0，/root/sh/目录是存在的

[root@localhost ~]# [ -e /root/test/ ]
[root@localhost ~]# echo $?
1        # 判断结果为0，/root/test/目录是不存在的

[root@localhost ~]# [ -d /root/sh ] && echo "yes" || echo "no"
# 第一个判断命令如果正确执行，则打印"yes"，否则打印"no"
```



#### 按照文件权限进行判断

| 测试选项 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| -r 文件  | 判断该文件是否存在，并且是否该文件拥有读权限（有读权限为真） |
| -w 文件  | 判断该文件是否存在，并且是否该文件拥有写权限（有写权限为真） |
| -x 文件  | 判断该文件是否存在，并且是否该文件拥有执行权限（有执行权限为真） |
| -u 文件  | 判断该文件是否存在，并且是否该文件拥有SUID权限（有SUID权限为真） |
| -g 文件  | 判断该文件是否存在，并且是否该文件拥有SGID权限（有SGID权限为真） |
| -k 文件  | 判断该文件是否存在，并且是否该文件拥有SBIT权限（有SBIT权限为真） |

```shell
[root@localhost ~]# ll test.txt
-rw-rw-r-- 1 ben ben 187  2月 28 02:09 test.txt
[root@localhost ~]# [ -w test.txt ] && echo "yes" || echo "no"
yes
```



#### 两个文件之间进行比较

| 测试选项        | 作用                                                         |
| --------------- | ------------------------------------------------------------ |
| 文件1 -nt 文件2 | 判断文件1的修改时间是否比文件2的新（如果新则为真）           |
| 文件1 -ot 文件2 | 判断文件1的修改时间是否比文件2的旧（如果旧则为真）           |
| 文件1 -ef 文件2 | 判断文件1是否和文件2的Inode号一致，可以理解为两个文件是否为同一个文件。这个判断用于判断硬链接是很好的方法 |

```shell
# 创建个硬链接
[root@localhost ~]# ln /tmp/test.txt /tmp/stu.txt
[root@localhost ~]# [ /tmp/test.txt -ef /tmp/stu.txt ] && echo "yes" || echo "no"
yes
```



#### 两个整数之间比较

| 测试选项        | 作用                                       |
| --------------- | ------------------------------------------ |
| 整数1 -eq 整数2 | 判断整数1是否和整数2相等（相等为真）       |
| 整数1 -ne 整数2 | 判断整数1是否和整数2不相等（不相等为真）   |
| 整数1 -gt 整数2 | 判断整数1是否大于整数2（大于为真）         |
| 整数1 -lt 整数2 | 判断整数1是否小于整数2（小于为真）         |
| 整数1 -ge 整数2 | 判断整数1是否大于等于整数2（大于等于为真） |
| 整数1 -le 整数2 | 判断整数1是否小于等于整数2（小于等于为真） |

```shell
[root@localhost ~]# [ 23 -ge 22 ] && echo "yes" || echo "no"
yes
[root@localhost ~]# [ 23 -le 22 ] && echo "yes" || echo "no"
no
```



#### 字符串的判断

| 测试选项           | 作用                                            |
| ------------------ | ----------------------------------------------- |
| -z 字符串          | 判断字符串是否为空（为空返回真）                |
| -n 字符串          | 判断字符串是否为非空（非空返回真）              |
| 字符串1 == 字符串2 | 判断字符串1是否和字符串2相等（相等返回真）      |
| 字符串1 != 字符串2 | 判断字符串1 是否和字符串2不相等（不相等返回真） |

``` shell
# 给name赋值
[root@localhost ~]# name=ben
[root@localhost ~]# [ -z "$name" ] && echo "yes" || echo "no"
no

[root@localhost ~]# aa=11
[root@localhost ~]# bb=22
[root@localhost ~]# [ "$aa" == "$bb" ] && echo "yes" || echo "no"
no
```



#### 多重条件判断

| 测试选项       | 作用                                             |
| -------------- | ------------------------------------------------ |
| 判断1 -a 判断2 | 逻辑与，判断1和判断2都成立，最终的结果为真       |
| 判断1 -o 判断2 | 逻辑或，判断1和判断2有一个成立，最终的结果就为真 |
| ! 判断         | 逻辑非，使原始的判断式取反                       |

```shell
[root@localhost ~]# aa=11
[root@localhost ~]# [ -n "$aa" -a "$aa" -gt 23 ] && echo "yes" || echo "no"
no
[root@localhost ~]# [ ! -n "$aa" ] && echo "yes" || echo "no"
no
```

**注意：**“!”和“-n”之间必须加入空格，否则会报错的。



### 流程控制

#### if条件判断

a）单分支if条件语句

单分支条件语句最为简单，就是只有一个判断条件，如果符合条件则执行某个程序，否则什么事情都不做。

```
if [ 条件判断式 ];then
	程序
fi
```

单分支条件语句需要注意几个点：

- if语句使用fi结尾，和一般语言使用大括号结尾不同。

- [ 条件判断式 ]就是使用test命令判断，所以中括号和条件判断式之间必须有空格。

- then后面跟符合条件之后执行的程序，可以放在[]之后，用“;”分割。也可以换行写入，就不需要“;”了。

  ```
  if [ 条件判断式 ]
  	then
  		程序
  fi
  ```

```shell
#!/bin/bash
# Author: Ben（Email: 17620170099@163.com）

rate=$(df -h | grep "/dev/sda3" | awk '{print $5}' | cut -d "%" -f1)
if [ $rate -ge 80 ];then
	echo "Warning! /dev/sda3 is full!!"
fi
```



b）双分支if条件语句

```
if [ 条件判断式 ]；then
	条件成立时，执行的程序
else
	条件不成立时，执行的另一个程序
fi
```

```shell
#!/bin/bash
# Author: Ben（Email: 17620170099@163.com）

# 同步系统时间
ntpdate asia.pool.ntp.org &>/dev/null
# 把当前系统时间按照"年月日"格式赋予变量date
date=$(date +%y%m%d)
# 统计mysql数据库的大小，并把大小赋予size变量
size=$(du -sh /var/lib/mysql)

# 判断备份目录是否存在，是否为目录
if [ -d  /tmp/dbak ];then
	# 把当前日期写入临时文件
	echo "Date: $date!" > /tmp/dbbak/dbinfo.txt
	# 把数据库大小写入临时文件
	echo "Date size: $size" >> /tmp/dbbak/dbinfo.txt
	# 进入备份目录
	cd /tmp/dbbak
	# 打包压缩数据库与临时文件，把所有输出丢入垃圾箱
	tar -zcf mysql-lib-$date.tar.gz /var/lib/mysql dbinfo.txt &>/dev/null
	# 删除临时文件
	rm -rf /tmp/dbbak/dbinfo.txt
else
	# 如果判断为假，则建立备份目录
	mkdir /tmp/dbbak
	# 把当前日期写入临时文件
	echo "Date: $date!" > /tmp/dbbak/dbinfo.txt
	# 把数据库大小写入临时文件
	echo "Date size: $size" >> /tmp/dbbak/dbinfo.txt
	# 进入备份目录
	cd /tmp/dbbak
	# 打包压缩数据库与临时文件，把所有输出丢入垃圾箱
	tar -zcf mysql-lib-$date.tar.gz /var/lib/mysql dbinfo.txt &>/dev/null
	# 删除临时文件
	rm -rf /tmp/dbbak/dbinfo.txt
fi
```

在工作中，服务器上的服务经常会宕机。如果我们对服务器监控不好，就会造成服务器中服务宕机了，而管理员却不知道的情况，这时我们可以写一个脚本来监听本机的服务，如果服务停止或宕机了，可以自动重启这些服务。

```shell
#!/bin/bash
# Author: Ben（Email: 17620170099@163.com）

# 使用nmap命令扫描服务器，并截取apache服务的状态，赋予变量port
port=$(nmap -sT 192.168.4.210 | grep tcp | grep http | awk '{print $2}')
if [ "$port" == "open" ];then
	# 则证明apache正常启动，在正常日志中写入一句话即可
	echo "$(date) httpd is ok!" >> /tmp/autostart-acc.log
else
	# 否则证明apache没有启动，自动启动apache
	/etc/rc.d/init.d/httpd start &>/dev/null
	# 并在错误日志中记录自动启动apache的时间
	echo "$(date) restart httpd!!" >> /tmp/autostart-err.log
fi
```



c）多分支if条件语句

```
if [ 条件判断式1 ];then
	当条件判断式1成立时，执行程序1
elif [ 条件判断式2 ];then
	当条件判断式2成立时，执行程序2
else
	当所有条件都不成立时，最后执行此程序
fi
```

判断用户输入的是什么文件：

```shell
#！/bin/bash
# Author: Ben（Email:17620170099@163.com）
# 接收键盘的输入，并赋予变量file
read -p "Please input a filename: " file

# 判断file变量是否为空
if [ -z "$file" ];then
	echo "Error,please input a filename"
	# 退出程序，并返回1
	exit 1
# 判断file的值是否存在
elif [ ! -e "$file" ];then
	echo "Your input is not a file!"
	# 退出程序，并返回2
	exit 2
# 判断file的值是否为普通文件
elif [ -f "$file" ];then
	echo "$file is a regulare file!"
# 判断file的值是否为目录文件
elif [ -d "$file" ];then
	echo "$file is a directory!"
else
	echo "$file is an other file!"
fi
```



#### 多分支case条件语句

case语句和if...elif...else语句一样都是多分支条件语句，不过和if多分支条件语句不同的是，case语句只能判断一种条件关系，而if语句可以判断多种条件关系。

```
case $变量名 in
	"值1")
		如果变量的值等于值1，则执行程序1
		;;
	"值2")
		如果变量的值等于值2，则执行程序2
		;;
	...
	*)
		如果变量的值都不是以上的值，则执行此程序
		;;
esac	
```

这个语句需要注意以下内容：

- case语句，会取出变量中的值，然后与语句体中的值逐一比较。如果数值符合，则执行对应的程序，如果数值不符，则依次比较下一个值。如果所有的值都不符合，则执行“*)”（“\*”代表所有其他值）中的程序。
- case语句以“case”开头，以“esac”结尾。

- 每个分支程序之后要通过“;;”双分号结尾，代表该程序段结束。

```shell
#!/bin/bash
# Author: Ben（E-mail: 17620170099@163.com）

read -p "Please choose yes/no: " -t 30 cho
case $cho in 
	"yes")
		echo "Your choose is yes!"
		;;
	"no")
		echo "Your choose is no!"
		;;
	*)
		echo "Your choose is error!"
		;;
esac
```



#### for循环

for循环是固定循环，也就是在循环时已经知道需要进行几次循环，有时也把for循环称为计数循环。

语法一：

```
for 变量 in 值1 值2 值3...
	do
		程序
	done
```

这种语法中for循环的次数，取决于in后面值的个数（空格分隔），有几个值就循环几次，并且每次循环都把值赋予变量。

打印时间：

```shell
#!/bin/bash
# Author: Ben（E-mail: 17620170099@163.com）

for time in morning noon afternoon evening
	do
		echo "This time is $time"
	done
```

批量解压缩脚本：

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

# 进入压缩目录
cd /lamp
# 把所有tar.gz结尾的文件的文件覆盖到ls.log临时文件中
ls *.tar.gz > ls.log
for i in $(cat ls.log)
	do
		# 解压缩
		tar -zxf $i &>/dev/null
	done
# 删除临时文件ls.log
rm -rf /lamp/ls.log
```



语法二：

```
for (( 初始值;循环控制条件;变量变化 ))
	do
		程序
	done
```

这种语法需要注意：

- 初始值：在循环开始时，需要给某个变量赋予初始值，如i=1;
- 循环控制条件：用于指定变量循环的次数，如i<=100，则只要i的值小于等于100，循环就会继续;
- 变量变化：每次循环之后，变量该如何变化，如i=i+1。代表每次循环之后，变量i的值都加1。

从1加到100：

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

s=0
for (( i=1;i<=100;i=i+1 ))
	do
		s=$(( $s+$i ))
	done
echo "The sum of 1+2+...+100 is : $s"
```

批量添加指定数量的用户：

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

# 让用户输入用户名，把输入保存入变量name
read -p "Please input user name: " -t 30 name
# 让用户输入添加用户的数量，把输入保存入变量num
read -p "Please input the number of users: " -t num
# 让用户输入初始密码，把输入保存入变量pass
read -p "Please input the password of users: " -t pass

# 判断三个变量不为空
if [ ! -z "$name" -a ! -z "$num" -a ! -z "#pass" ];then
	# 判断变量num的值是否为数字
	y=$(echo $num | sed 's/[0-9]//g')
	if [ -z "$y" ];then
		for (( i=1;i<=$num;i=i+1 ))
			do
				# 添加用户
				/usr/sbin/useradd $name$i &>/dev/null
				# 修改用户密码
				echo $pass | /usr/bin/passwd --stdin $name$i &>/dev/null
			done
	fi
fi
```

批量删除用户：

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

user=$(cat /etc/passwd | grep "/bin/bash" | grep -v "root" | cut -d ":" -f 1)
for i in $user
	do
		userdel -r $i
	done
```



#### while循环

```
while [ 条件判断式 ]
	do
		程序
	done
```

对while循环来讲，只要条件判断式成立，循环就会一直继续，直到条件判断式不成立，循环才会停止。

从1加到100：

```shell
#!/bin/bash
# Author:Ben（E-mail:17620170099@163.com）

i=1
s=0
while [ $i -le 100 ]
	do
		s=$(( $s+$i ))
		i=$(( $i+ 1 ))
	done
echo "The sum is: $s"
```



#### until循环

until循环和while循环相反，until循环时只要条件判断式不成立则进行循环，并执行循环程序。一旦循环条件成立，则终止循环。

```shell
until [ 条件判断式 ]
	do
		程序
	done
```

从1加到100：

```shell
#!/bin/bash
# Author:Ben（E-mail:17620170099@163.com）

i=1
s=0
until [ $i -gt 100 ]
	do
		s=$(( $s+$i ))
		i=$(( $i+1 ))
	done
echo "The sum is: $s"
```



#### 函数

```
function 函数名（）{
	程序
}
```

```shell
#!/bin/bash
# Author:Ben（E-mail:17620170099@163.com）

function sum () {
	s=0
	for (( i=0;i<=$1;i=i+1 ))
		do
			s=$(( $i+$s ))
		done
	echo "The sum of 1+2+3+...+$l is: $s"
}

read -p "Please input a number: " -t 30 num
y=$(echo $num | sed 's/[0-9]//g')

if [ -z $y ];then
	sum $num
else
	echo "Error!! Please input a number!"
fi
```



#### 特殊流程控制语名

a）exit语句

系统是有exit命令的，用于退出当前用户的登录状态。可是在shell脚本中，exit语句是用来退出当前脚本的。也就是说，在Shell脚本中，只要碰到了exit语句，后续的程序就不再执行，而是直接退出脚本。

```
exit [返回值]
```

如果exit命令之后定义了返回值，那么这个脚本执行之后的返回值就是我们自己定义的返回值。可以通过$?这个变量来查看返回值。如果exit之后没有定义返回值，脚本执行之后的返回值是执行exit语句之前，最后执行的一条命令的返回值。

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

read -p "Please input a number: " -t 30 num

y=$(echo $num | sed "s/[0-9]//g")

# 判断变量num的值如果不为空，则输出报错信息，并且退出脚本，退出返回值为18
[ -n "$y" ] && echo "Error! Please input a number!" && exit 18

echo "The number is: $num"
```



b）break语句

当程序执行到break语句时，会结束整个当前循环。

```shell
#!/bin/bash
# Author:Ben（E-mail: 17620170099@163.com）

for (( i=1;i<=10;i=i+1 ))
	do
		if [ $i -eq 4 ];then
			break
		fi
		echo $i
	done
```



c）continue语句

continue也是结束流程控制的语句。continue语句只会结束单次当前循环。

```shell
#!/bin/bash
# Author:Ben（E-mail:17620170099@163.com）

for (( i=1;i<=10;i=i+1 ))
	do
		if [ $i -eq 4 ];then
			continue
		fi
		echo $i
	done
```

