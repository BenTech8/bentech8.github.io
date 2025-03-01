---
title: Shell基础
date: 2025-02-25 12:16:28
tags:
categories: Shell
---

## Shell概述

### Shell分类

| Shell类别            | 易学性 | 可移植性 | 编辑性 | 快捷性 |
| -------------------- | ------ | -------- | ------ | ------ |
| Bourne Shell（sh）   | 容易   | 好       | 较差   | 较差   |
| Korn Shell（ksh）    | 较难   | 较好     | 好     | 较好   |
| Bourne Again（Bash） | 难     | 较好     | 好     | 好     |
| POSIX Shell（psh）   | 较难   | 好       | 好     | 较好   |
| C Shell（csh）       | 较难   | 差       | 较好   | 较好   |
| TC Shell（tcsh）     | 难     | 差       | 好     | 好     |

Shell的两种主语法类型有Bourne和C，这两种语法彼此不兼容。Bourne家族主要包括sh、ksh、Bash、psh、zsh；C家族主要包括：csh、tcsh（Bash和zsh在不同程度上支持csh的语法）。

可以通过/etc/shells文件来查询Linux支持的Shell：

```shell
[root@localhost ~]# cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/dash
/usr/bin/sh
```



### Shell脚本的执行方式

#### echo命令

```
[root@localhost ~]# echo [选项] [输出内容]
选项：
  -e：    支持反斜线控制的字符转换（如下表）
  -n：    取消输出后行末的换行符号(就是内容输出后不换行)
```

```
[root@localhost ~]# echo "Mr. Ben is the most honest man!"
Mr. Ben is the most honest man!

[root@localhost ~]# echo -n "Mr. Ben is the most honest man!"
Mr. Ben is the most honest man![root@localhost ~]#
```

在echo命令中如果使用了"-e"选项，则可以支持控制字符。

| 控制字符 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| \\\      | 输出\本身                                                    |
| \a       | 输出警告音                                                   |
| \b       | 退格键，也就是向左删除键                                     |
| \c       | 取消输出行末的换行符。和“-n”选项一致                         |
| \e       | ESCAPE键                                                     |
| \f       | 换页符                                                       |
| \n       | 换行符                                                       |
| \r       | 回车键                                                       |
| \t       | 制表符，也就是Tab键                                          |
| \v       | 垂直制表符                                                   |
| \0nnn    | 按照八进制ASCII码表输出字符。其中0为数字零，nnn是三位八进制数 |
| \xhh     | 按照十六进制ASCII码表输出字符。其中hh是两位十六进制数        |

```
[root@localhost ~]# echo -e "\\ \a"
\
# 这个输出会输出\，同时会在系统音响中输出一声提示音

[root@localhost ~]# echo -e "ab\bc"
ac

[root@localhost ~]# echo -e "a\tb\tc\nd\te\tf"
a    b    c
d    e    f

# 八进制输出
[root@localhost ~]# echo -e "\0141\t\0142\t\0143\n\0144\t\0145\t\0146"
a    b    c
d    e    f

# 十六进制
[root@localhost ~]# echo -e "\x61\t\x62\t\x63\n\x64\t\x65\t\x66"
a    b    c
d    e    f

# 按颜色输出
[root@localhost ~]# echo -e "\e[1;31m    abcd \e[0m"
    abcd
```

按颜色输出这个命令，“\e[1”是标准格式，代表颜色输出开始，"\e[0m"代表颜色输出结束，31m定义字体颜色是红色。echo能够识别的颜色：

- 30m：黑色
- 31m：红色
- 32m：绿色
- 33m：黄色
- 34m：蓝色
- 35m：洋红
- 36m：青色
- 37m：白色

```
[root@localhost ~]# echo -e "\e[1;42m abcd \e[0m"
```

这条命令会给abcd加一个绿色的背景。echo可以使用的背景颜色如下：

- 40m：黑色
- 41m：红色
- 42m：绿色
- 43m：黄色
- 44m：蓝色
- 45m：洋红
- 46m：青色
- 47m：白色



#### Shell脚本的执行

```shell
#!/bin/bash
# The first program
# Author: Ben（E-mail: 326525276@qq.com）

echo -e "Mr. Ben is the most honest man! "
```

在Linux中脚本的执行主要有两种方法：

- 赋予执行权限，直接运行（推荐）

  这种方法是最常用的Shell脚本运行方法，也最为直接简单。运行时可以使用绝对路径，也可以使用相对路径。

  ```shell
  # 赋予执行权限
  [root@localhost sh]# chmod +x hello.sh
  # 绝对路径执行
  [root@localhost sh]# /root/sh/hello.sh
  Mr. Ben is the most honest man!
  # 相对路径执行
  [root@localhost sh]# ./hello.sh
  Mr. Ben is the most honest man!
  ```

- 通过Bash调用执行脚本

  ```shell
  [root@localhost sh]# bash hello.sh
  Mr. Ben is the most honest man!
  ```



## Bash的基本功能

### 历史命令

```
[root@localhost ~]# history [选项] [历史命令保存文件]
选项：
  -c：    清空历史命令
  -w：    把缓存中的历史命令写入历史命令保存文件。如果不手工指定历史命令保存文件，则放入默认历史命令保存文件~/.bash_history中
```

history命令查看的历史命令和~/.bash_history文件中保存的历史命令是不同的。那是因为当前登录操作的命令并没有直接写入~/.bash_history文件，而是保存在缓存当中的。需要等当前用户注销之后，缓存中的命令才会写入~/.bash_history文件。如果需要把内存中的命令直接写入~/.bash_history文件，而不等用户注销时再写入，就需要使用"-w"选项。

```
# 把缓存中的历史命令直接写入~/.bash_history
[root@localhost ~]# history -w

# 清空历史命令
[root@localhost ~]# history -c
```

### 历史命令的调用

如果想要使用原先的历史命令有这样几种方法：

- 使用上、下箭头调用以前的历史命令
- 使用“!n”重复执行第n条历史命令
- 使用"!!"重复执行上一条命令
- 使用"!字串"重复执行最后一条以该字串开头的命令
- 使用"!$"重复上一条命令的最后一个参数

```shell
# 调用最后一条以cat开头的命令
[root@localhost ~]# !cat
```

### 输入输出重定向

bash的标准输入输出

| 设备   | 设备文件名  | 文件描述符 | 类型         |
| ------ | ----------- | ---------- | ------------ |
| 键盘   | /dev/stdin  | 0          | 标准输入     |
| 显示器 | /dev/stdout | 1          | 标准输出     |
| 显示器 | /dev/stderr | 2          | 标准错误输出 |

输出重定向

- 标准输出重定向：
  - 命令 > 文件：以覆盖的方式，把命令的正确输出输出到指定的文件或设备当中。
  - 命令 >> 文件：以追加的方式，把命令的正确输出输出到指定的文件或设备当中。

- 标准错误输出重定向：
  - 错误命令 2>文件：以覆盖的方式，把命令的错误输出输出到指定的文件或设备当中。
  - 错误命令 2>>文件：以追加的方式，把命令的错误输出输出到指定的文件或设备当中。

- 正确输出和错误输出同时保存：
  - 命令 > 文件 2>&1：以覆盖的方式，把正确输出和错误输出都保存在同一个文件当中。
  - 命令 >> 文件 2>&1：以追加的方式，把正确输出和错误输出都保存在同一个文件当中。
  - 命令 &>文件：以覆盖的方式，把正确输出和错误输出都保存在同一个文件当中。
  - 命令 &>>文件：以追加的方式，把正确输出和错误输出都保存在同一个文件当中。
  - 命令>>文件1 2>>文件2：把正确的输出追加到文件1中，把错误的输出追加到文件2中。

输入重定向

```
[root@localhost ~]# wc [选项] [文件名]
选项：
  -c    统计字节数
  -w    统计单词数
  -l    统计行数
```



### 多命令执行顺序

| 多命令执行符 | 格式             | 作用                                                         |
| ------------ | ---------------- | ------------------------------------------------------------ |
| ；           | 命令1 ; 命令2    | 多个命令顺序执行，命令之间没有任何逻辑联系                   |
| &&           | 命令1 && 命令2   | 1）当命令1正确执行($?=0)，则命令2才会执行。2）当命令1执行不正确，则命令2不会执行。 |
| \|\|         | 命令1 \|\| 命令2 | 1）当命令1执行不正确，则命令2才会执行。2）当命令1正确执行，则命令2不会执行。 |

```
[root@localhost ~]# ls && echo yes || echo no
yes
```



### 管道符

a）行提取命令grep

```
[root@localhost ~]# grep [选项] "搜索内容" 文件名
选项：
  -A 数字：      列出符合条件的行，并列出后续的n行
  -B 数字：      列出符合条件的行，并列出前面的n行
  -c：          统计找到的符合条件的字符串的次数
  -i：          忽略大小写
  -n：          输出行号
  -v：          反向查找
  --color=auto  搜索出的关键字用颜色显示
```

```shell
# 查找用户信息文件/etc/passwd中，有多少可以登录的用户
[root@localhost ~]# grep "/bin/bash" /etc/passwd

# 查找包含有“root”的行，并列出后续的3行
[root@localhost ~]# grep -A 3 "root" /etc/passwd

# 查找可以登录的用户，并显示行号
[root@localhost ~]# grep -n "/bin/bash" /etc/passwd

# 查找不含有“/bin/bash”的行，其实就是列出所有的伪用户
[root@localhost ~]# grep -v "/bin/bash" /etc/passwd
```



b）find和grep的区别

find命令是在系统当中搜索符合条件的文件名，如果需要模糊查询，使用通配符进行匹配，搜索时文件名是完全匹配。

```shell
[root@localhost ~]# touch abc
[root@localhost ~]# touch abcd
[root@localhost ~]# find . -name "abc"
./abc
# 搜索文件名是abc的文件，只会找到abc文件，而不会找到文件abcd。虽然abcd文件名中包含abc，但是find是完全匹配，只能和要搜索的数据完全一样，才能找到
```

**注意：**find命令是可以通过-regex选项识别正则表达式规则的，也就是说find命令可以按照正则表达式规则匹配，而正则表达式是模糊匹配。

grep命令是在文件当中搜索符合条件的字符串，如果需要模糊查询，使用正则表达式进行匹配，搜索时字符串是包含匹配。

```shell
[root@localhost ~]# echo abc > test
[root@localhost ~]# echo abcd >> test
[root@localhost ~]# grep "abc" test
abc
abcd
```



c）管道符

```shell
[root@localhost ~]# ll -a /etc/ | more
# 查询下本地所有网络连接，提取包含ESTABLISHED(已建立连接)的行，就可以知道服务器上有多个已经成功连接的网络连接
[root@localhost ~]# netstat -an | grep "ESTABLISHED"
[root@localhost ~]# netstat -an | grep "ESTABLISHED" | wc -l
```



### 通配符

| 通配符 | 作用                                                         |
| ------ | ------------------------------------------------------------ |
| ？     | 匹配一个任意字符                                             |
| *      | 匹配0个或任意多个字符，也就是可以匹配任何内容                |
| []     | 匹配中括号任意一个字符。例如[abc]代表一定匹配一个字符，a或b或c |
| [-]    | 匹配中括号中任意一个字符，-代表一个范围。例如[a-z]代表匹配一个小写字母 |
| [^]    | 逻辑非，表示匹配不是中括号内的一个字符。例如[^0-9\]代表匹配一个不是数字的字符 |

```shell
[root@localhost ~]# touch abc
[root@localhost ~]# touch abcd
[root@localhost ~]# touch 012
[root@localhost ~]# touch 0abc
[root@localhost ~]# ls *
012 0abc abc abcd
[root@localhost ~]# ls ?abc
0abc
[root@localhost ~]# ls [0-9]*
012 0abc
[root@localhost ~]# ls [^0-9]*
abc abcd
```



### Bash中其他特殊符号

| 符号 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| ‘’   | 单引号。在单引号中所有的特殊符号，如“$”和“~”都没有特殊含义   |
| “”   | 双引号。在双引号中特殊符号都没有特殊含义，但是“$”、“~”和“\”是例外，拥有"调用变量的值"、“引用命令”和“转义符”的特殊含义。 |
| ``   | 反引号。反引号括起来的内容是系统命令，在Bash中会先执行它。和$()作用一样，不过推荐使用$()，因为反引号不易读。 |
| $()  | 和反引号作用一样，用来引用系统命令。                         |
| ()   | 用于一串命令执行时，()中的命令会在子Shell中运行              |
| {}   | 用于一串命令执行时，{}中的命令会在当前Shell中执行。也可以用于变量变形与替换 |
| []   | 用于变量的测试                                               |
| #    | 在Shell脚本中，#开头的行代表注释                             |
| $    | 用于调用变量的值，                                           |
| \    | 转义符，跟在\之后的特殊符号将失去特殊含义，变为普通字符。    |

a）单引号和双引号

```shell
# 定义变量的值
[root@localhost ~]# name=ben

# 如果输出时使用单引号，则$name原封不动输出
[root@localhost ~]# echo '$name'

# 如果输出时使用双引号，则会输出变量name的值
[root@localhost ~]# echo "$name"
ben

# 反引号括起来的命令会正常执行
[root@localhost ~]# echo `date`
2025年 02月 25日 星期二 17:16:57 CST

# 如果反引号命令被单引号括起来，则这个命令不会执行
[root@localhost ~]# echo '`date`'
`date`

# 如果反引号被双引号括起来，则这个命令会执行
[root@localhost ~]# echo "`date`"
2025年 02月 25日 星期二 17:19:24 CST
```

b）反引号

```shell
# 如果命令不用反引号包含，命令不会执行，而是直接输出
[root@localhost ~]# echo ls
ls

# 只有用反引号包括命令，这个命令才会执行
[root@localhost go]# echo `ls`
bin pkg src

# $()作用和反引号一样
[root@localhost ~]# echo $(date)
2025年 02月 25日 星期二 17:28:03 CST
```

c）小括号、中括号和大括号

父Shell和子Shell。在Bashk中，是可以调用新的Shell的：

```shell
[root@localhost ~]# bash
[root@localhost ~]#
```

通过pstree命令查看下进程数：

```
[root@localhost ~]# pstree
systemd————ModemManager————3*[{ModemManager}]
         |——systemd————(sd-pam)
                     |——gnome-terminal————bash————bash————pstree
```

如果是用于一串命令的执行，那么小括号和大括号的主要区别在于：

- ()执行一串命令时，需要重新开一个子shell进行执行
- {}执行一串命令时，是在当前shell执行
- ()最后一个子命令可以不用分号
- {}最后一个子命令要用分号
- {}的第一个命令和左括号之间必须要有一个空格
- ()里的各命令不必和括号有空格
- ()和{}中括号里面的某个命令的重定向只影响该命令，但括号外的重定向则影响到括号里的所有命令

- ()和{}都是把一串的命令放在括号里面，并且命令之间用:号隔开

```shell
# 父shell中定义变量name
[root@localhost ~]# name=ben

# 子shell中给name赋值
[root@localhost ~]# (name=liming;echo $name)
liming

# 父shell中name值还是ben
[root@localhost ~]# echo $name

# {}是在当前shell执行，会修改name的值
[root@localhost ~]# { name=liming;echo $name;}
liming
[root@localhost ~]# echo $name
liming
```



## Bash的变量和运算符

### 变量

#### 变量定义

在定义变量时，有一些规则需要遵守：

- 变量名称可以由字母、数字和下划线组成，但是不能以数字开头。
- 在Bash中，变量的默认类型是字符串型，如果要进行数值运算，则必须指定变量类型为数值型。
- 变量用等号连接值，等号左右两侧不能有空格。
- 变量的值如果有空格，需要使用单引号或双引号包括。其中双引号括起来的内容“$”、“\”和反引号都拥有特殊含义，而单引号括起来的内容都是普通字符。
- 在变量的值中，可以使用“\”转义符。
- 如果需要增加变量的值，那么可以进行变量值的叠加。不过变量需要用双引号包含“$变量名”或用${变量名}包含变量名。

```shell
[root@localhost ~]# test=123
[root@localhost ~]# test="$test"456
[root@localhost ~]# echo $test
123456
[root@localhost ~]# test=${test}789
[root@localhost ~]# echo $test
123456789
```

变量值的叠加可以使用两种格式：“$变量名”或${变量名}。

- 如果是把命令的结果作为变量值赋予变量，则需要使用反引号或$()包含命令

```shell
[root@localhost ~]# test=$(date)
[root@localhost ~]# echo $test
2025年 02月 25日 星期二 18:22:43 CST
```

- 环境变量名建议大写，便于区分。



#### 变量的分类

- 用户自定义变量

  这种变量是最常见的变量，由用户自由定义变量名和变量的值。

- 环境变量

  这种变量中主要保存的是和系统操作环境相关的数据，比如当前登录用户，用户的家目录，命令的提示符等。环境变量的变量名可以自由定义，但是一般对系统起作用的环境变量的变量名是系统预先设定好的。

- 位置参数变量

  这种变量主要是用来向脚本当中传递参数或数据的，变量名不能自定义，变量作用是固定的。

- 预定义变量

  是Bash中已经定义好的变量，变量名不能自定义，变量作用也是固定的。



#### 用户自定义变量

定义变量

```shell
# 定义变量name
[root@localhost ~]# name="Ben"
```



调用变量

```shell
# 输出变量name的值
[root@localhost ~]# ehco $name
Ben
```



查看变量

```
[root@localhost ~]# set [选项]
选项：
  -u：    如果设定此选项，调用未声明变量时会报错(默认无任何提示)
  -x：    如果设定此选项，在命令执行之前，会把命令先输出一次
```

```shell
# 查看所有变量
[root@localhost ~]# set
...

[root@localhost ~]# set -u
[root@localhost ~]# echo $file
bash: file: 未绑定的变量
[root@localhost ~]# set -x
[root@localhost ~]# ls
+ ls --color=auto
_config.landscape.yml  _config.yml  db.json  deploy.sh  node_modules  package.json
```



删除变量

```shell
[root@localhost ~]# unset name
```



#### 环境变量

设置环境变量

```shell
[root@localhost ~]# export age="18"
```



查看环境变量

```shell
[root@localhost ~]# env | grep age
```

set命令可以查看所有变量，而env命令只能查看环境变量。



删除环境变量

```shell
[root@localhost ~]# unset age
```



系统默认环境变量

```shell
[root@localhost ~]# env
SHELL=/bin/bash                                                # 当前的shell
SESSION_MANAGER=local/ben-NBLK-WAX9X:@/tmp/.ICE-unix/4024,unix/ben-NBLK-WAX9X:/tmp/.ICE-unix/4024
QT_ACCESSIBILITY=1
COLORTERM=truecolor
XDG_CONFIG_DIRS=/etc/xdg/xdg-ubuntu:/etc/xdg
XDG_SESSION_PATH=/org/freedesktop/DisplayManager/Session0
XDG_MENU_PREFIX=gnome-
GNOME_DESKTOP_SESSION_ID=this-is-deprecated
GTK_IM_MODULE=fcitx
LANGUAGE=zh_CN:zh:en_US:en
LC_ADDRESS=zh_CN.UTF-8
GNOME_SHELL_SESSION_MODE=ubuntu
LC_NAME=zh_CN.UTF-8
SSH_AUTH_SOCK=/run/user/1000/keyring/ssh
MEMORY_PRESSURE_WRITE=c29tZSAyMDAwMDAgMjAwMDAwMAA=
HOMEBREW_PREFIX=/home/linuxbrew/.linuxbrew
XMODIFIERS=@im=fcitx
DESKTOP_SESSION=ubuntu
LC_MONETARY=zh_CN.UTF-8
GTK_MODULES=gail:atk-bridge
PWD=/home/ben/work/个人/githubPage
LOGNAME=ben
XDG_SESSION_DESKTOP=ubuntu
XDG_SESSION_TYPE=x11
MANPATH=/home/linuxbrew/.linuxbrew/share/man:
GPG_AGENT_INFO=/run/user/1000/gnupg/S.gpg-agent:0:1
SYSTEMD_EXEC_PID=4060
XAUTHORITY=/home/ben/.Xauthority
XDG_GREETER_DATA_DIR=/var/lib/lightdm-data/ben
GDM_LANG=zh_CN
HOME=/home/ben                                                    # 当前登录用户的家目录
IM_CONFIG_PHASE=1
LC_PAPER=zh_CN.UTF-8
LANG=zh_CN.UTF-8                                                  # 语系
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.avif=01;35:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.crdownload=00;90:*.dpkg-dist=00;90:*.dpkg-new=00;90:*.dpkg-old=00;90:*.dpkg-tmp=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:*.swp=00;90:*.tmp=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:          # 定义颜色显示
XDG_CURRENT_DESKTOP=ubuntu:GNOME
MEMORY_PRESSURE_WATCH=/sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/session.slice/org.gnome.Shell@x11.service/memory.pressure
VTE_VERSION=7600
XDG_SEAT_PATH=/org/freedesktop/DisplayManager/Seat0
GNOME_TERMINAL_SCREEN=/org/gnome/Terminal/screen/e46b9f29_027f_412a_aa8b_f9404dd3bc96
QTWEBENGINE_DICTIONARIES_PATH=/usr/share/hunspell-bdic/
CLUTTER_IM_MODULE=fcitx
INFOPATH=/home/linuxbrew/.linuxbrew/share/info:
SDL_IM_MODULE=fcitx
LESSCLOSE=/usr/bin/lesspipe %s %s
XDG_SESSION_CLASS=user
TERM=xterm-256color                                                     # 终端环境
LC_IDENTIFICATION=zh_CN.UTF-8
LESSOPEN=| /usr/bin/lesspipe %s
USER=ben                                                                # 当前登录的用户
HOMEBREW_CELLAR=/home/linuxbrew/.linuxbrew/Cellar
GNOME_TERMINAL_SERVICE=:1.10145
DISPLAY=:0
SHLVL=1
GSM_SKIP_SSH_AGENT_WORKAROUND=true
LC_TELEPHONE=zh_CN.UTF-8
QT_IM_MODULE=fcitx
HOMEBREW_REPOSITORY=/home/linuxbrew/.linuxbrew/Homebrew
LC_MEASUREMENT=zh_CN.UTF-8
XDG_RUNTIME_DIR=/run/user/1000
DEBUGINFOD_URLS=https://debuginfod.ubuntu.com 
LC_TIME=zh_CN.UTF-8
GTK3_MODULES=xapp-gtk3-module
XDG_DATA_DIRS=/usr/share/ubuntu:/usr/share/gnome:/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:/home/ben/.local/bin:/home/ben/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin    # 系统查找命令的路径
GDMSESSION=ubuntu
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
LC_NUMERIC=zh_CN.UTF-8
_=/usr/bin/env
OLDPWD=/home/ben/work/个人/githubPage/themes/next
```



#### Bash操作接口相关的变量

Bash操作接口相关的变量对我们的Bash操作终端起到了重要的作用。这些变量只能用set命令查看。

```shell
[root@localhost ~]# set
BASH=/usr/bin/bash                       # Bash的位置
BASH_VERSINFO=([0]="5" [1]="2" [2]="21" [3]="1" [4]="release" [5]="x86_64-pc-linux-gnu")    # Bash的版本
BASH_VERSION='5.2.21(1)-release'         # bash的版本
HISTFILE=/home/ben/.bash_history         # 历史命令保存文件
HISTFILESIZE=2000                        # 在文件当中记录的历史命令最大条数
HISTSIZE=1000                            # 在缓存中记录的历史命令最大条数
LANG=zh_CN.UTF-8                         # 语系环境
MACHTYPE=x86_64-pc-linux-gnu             # 软件类型是i386兼容类型
MAILCHECK=60                             # 每60s去扫描新邮件
PPID=557258                              # 父shell的PID
PS1='\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '                                              # 命令提示符
PS2='> '                                 # 如果命令一行没有输入完成，第二行命令的提示符
UID=1000                                 # 当前用户的UID
```



#### PATH变量

系统查找命令的路径

```shell
# 查询PATH环境变量的值
[root@localhost ~]# echo $PATH
/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:/home/ben/.local/bin:/home/ben/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
```

PATH变量的值是用“：”分割的路径，这些路径就是系统查找命令的路径。也就是说当我们输入了一个程序名，如果没有写入路径，系统就会到PATH变量定义的路径中去寻找是否可以执行的程序，如果找到则执行，否则会报“命令没有发现”的错误。

我们把自己的脚本拷贝到PATH变量定义的路径中，自己写的脚本也可以不输入路径而直接运行。

```shell
# 拷贝hello.sh到/bin目录
[root@localhost ~]# cp /root/sh/hello.sh /bin/
# 运行hello.sh
[root@localhost ~]# hello.sh
Mr. Ben is the most honest man.
```

我们也可以修改PATH变量的值，而不把程序脚本复制到/bin/目录中。

```shell
# 在变量PATH的后面，加入/root/sh目录
[root@localhost ~]# PATH="$PATH":/root/sh
# 查询PATH的值
[root@localhost ~]# echo $PATH
/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:/home/ben/.local/bin:/home/ben/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin:/root/sh
```

这样定义的PATH变量只是临时生效，一旦重启或注销就会消失，如果想要永久生效，需要写入环境变量配置文件。



#### PS1变量

命令提示符设置。

PS1是用来定义命令行的提示符的，可以安装我们自己的需求来定义自己喜欢的提示符。PS1可以支持以下这些选项：

- \d：显示日期，格式为"星期 月 日"
- \H：显示完整的主机名，如默认主机名"localhost.localdomain"
- \h：显示简写主机名。如默认主机名"localhost"
- \t：显示24小时制时间，格式为"HH:MM:SS"
- \T：显示12小时制时间，格式为"HH:MM:SS"
- \A：显示24小时制时间，格式为“HH:MM”
- \@：显示12小时制时间，格式为"HH:MM am/pm"
- \u：显示当前用户名
- \v：显示Bash的版本信息
- \w：显示当前所在目录的完整名称
- \W：显示当前所在目录的最后一个目录
- \\#：执行的第几个命令
- \\$：提示符。如果是root用户会显示提示符为“#”， 如果是普通用户会显示提示符为“$”

```shell
[root@localhost ~]# echo $PS1
\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$
```

在PS1变量中，如果是可以解释的符号，如"\u"、“\h”等，则显示这个符号的作用，如果是不能解释的符号，如"@"或“空格”，则原符号输出。

```
# 修改提示符
[root@localhost ~]# PS1='\[\u@\t \w\]\$'
```

PS1变量的值要用单引号包含，否则设置不生效。PS1变量可以自由定制，不过说实话一个提示符已经使用习惯了，还是用默认的吧。



#### LANG语系变量

LANG变量定义了Linux系统的主语系环境。这个变量的默认值是：

```
[root@localhost ~]# echo $LANG
zh_CN.UTF-8
```

这是因为Linux安装时，选择的是中文安装，所以默认的主语系变量是“zh_CN.UTF-8”。

那么Linux到底支持多少语系呢？可以使用以下命令查询：

```shell
[root@localhost ~]# locale -a | more
C
C.utf8
en_AG
en_AG.utf8
en_AU.utf8
en_BW.utf8
en_CA.utf8
en_DK.utf8
en_GB.utf8
en_HK.utf8
en_IE.utf8
en_IL
en_IL.utf8
en_IN
en_IN.utf8
en_NG
en_NG.utf8
en_NZ.utf8
en_PH.utf8
en_SG.utf8
en_US.utf8
en_ZA.utf8
en_ZM
en_ZM.utf8
en_ZW.utf8
POSIX
zh_CN.utf8
zh_SG.utf8
```

当前系统是什么语系呢？

```shell
[root@localhost ~]# locale
LANG=zh_CN.UTF-8
LANGUAGE=zh_CN:zh:en_US:en
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC=zh_CN.UTF-8
LC_TIME=zh_CN.UTF-8
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY=zh_CN.UTF-8
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER=zh_CN.UTF-8
LC_NAME=zh_CN.UTF-8
LC_ADDRESS=zh_CN.UTF-8
LC_TELEPHONE=zh_CN.UTF-8
LC_MEASUREMENT=zh_CN.UTF-8
LC_IDENTIFICATION=zh_CN.UTF-8
LC_ALL=
```

系统默认语系查看

```shell
[root@localhost ~]# cat /etc/sysconfig/i18n
LANG="zh_CN.UTF-8"
```

默认语系是下次重启之后系统所使用的语系，而当前系统语系是当前系统使用的语系。如果系统重启，会从默认语系配置文件/etc/sysconfig/i18n中读出语系，然后赋予变量LANG让这个语系生效。

关于Linux支持中文的问题。是不是只要定义了语系为中文语系，如zh_CN.UTF-8就可以正确显示中文呢？这个要分情况，如果我们在图形界面中，或者是使用远程连接工具(如secureCRT)，只要正确设置了语系，那么是可以正确显示中文的。当然远程工具也要配置正确的语系环境。如果是纯字符界面(本地终端tty1-tty6)是不能显示中文的，因为Linux的纯字符界面是不能显示中文这么复杂的编码的。如果非要在纯字符界面显示中文，那么只能安装中文插件，如zhcon等。



#### 位置参数变量

| 位置参数变量 | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| $n           | n为数字，$0代表命令本身，$1-9代表第一到第九个参数，十以上的参数需要用大括号包含，如${10} |
| $*           | 这个变量代表命令行中所有的参数，$*把所有的参数看成一个整本   |
| $@           | 这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待   |
| $#           | 这个变量代表命令行中所有参数的个数                           |

```shell
#!/bin/bash
# Author：Ben（E-mail：17620170099@163.com）

# 给num1变量赋值是第一个参数
num1=$1
# 给num2变量赋值是第二个参数
num2=$2
# 求和
sum=$(( $num1 + $num2 ))
echo $sum
```

```shell
#!/bin/bash
# Author：Ben（E-mail：17620170099@163.com）

# 打印所有位置参数个数
echo "A total of $# parameters"
# 打印所有位置参数值
for i in "$*"
    do
        echo "The parameters is: $i"
    done
x=1
for y in "$@"
    do
        echo "The parameter$x is: $y"
        x=$(( $x + 1 ))
    done
```



#### 预定义变量

| 预定义变量 | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| $?         | 最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0(具体是哪个数，由命令自己来决定)，则证明上一个命令执行不正确了。 |
| $$         | 当前进程的进程号(PID)                                        |
| $!         | 后台运行的最后一个进程的进程号(PID)                          |

```shell
ben@ben-NBLK-WAX9X:~/test$ls
12  nginx-test  nginx-test.zip
ben@ben-NBLK-WAX9X:~/test$echo $?
0
ben@ben-NBLK-WAX9X:~/test$ls install.log
ls: 无法访问 'install.log': 没有那个文件或目录
ben@ben-NBLK-WAX9X:~/test$echo $?
2
```

```shell
#!/bin/bash
# Author: Ben（E-mail: 17620170099@163.com）

# 输出当前进程的PID，这个PID就是这个脚本执行时，生成的进程的PID
echo "The current process is $$"

# 在/root目录下查找hello.sh文件，&是把命令放入后台执行
find /root -name hello.sh &
echo "The last on Daemon process is $!"
```



#### 接收键盘输入

```
[root@localhost ~]# read [选项] [变量名]
选项：
  -p "提示信息":    在等待read输入时，输出提示信息
  -t 秒数:          read命令会一直等待用户输入，使用此选项可以指定等待时间
  -n 字符数:        read命令只接受指定的字符数，就会执行
  -s:              隐藏输入的数据，适用于机密信息的输入
变量名：变量名可以自定义，如果不指定变量名，会把输入保存入默认变量REPLY。如果只提供了一个变量名，则整个输入行赋予该变量。如果提供了一个以上的变量名，则输入行分为若干字，一个接一个地赋予各个变量，而命令行上的最后一个变量取得剩余的所有字
```

```shell
#!/bin/bash
# Author: Ben（E-mail: 17620170099@163.com）

read -t 30 -p "Please input your name: " name
echo "Name is $name"

read -s -t 30 -p "Please enter your age: " age
# 调整输出格式
echo -e "\n"
echo "Age is $age"

read -n 1 -t 30 -p "Please select your gender[M/F]: " gender
echo -e "\n"
echo "Sex is $gender"
```



### 运算符

#### 数值运算

如果需要进行数值运算，可以采用以下三种方法中的任意一种：

- 使用declare声明变量类型

  既然所有变量的默认类型是字符串型，那么只要把变量声明为整数型就可以了。使用declare命令就可以实现声明变量的类型。

  ```shell
  [root@localhost ~]# declare [+/-][选项] 变量名
  选项：
    -：      给变量设定类型属性
    +：      取消变量的类型属性
    -a：     将变量声明为数组型
    -i：     将变量声明为整数型(integer)
    -r：     将变量声明为只读变量。注意，一旦设置为只读变量，既不能修改变量的值，也不能删除变量，甚至不能通过+r取消只读属性
    -x：     将变量声明为环境变量
    -p：     显示指定变量的被声明的类型
  ```

  ```shell
  ben@ben-NBLK-WAX9X:~$aa=11
  ben@ben-NBLK-WAX9X:~$bb=22
  ben@ben-NBLK-WAX9X:~$declare -i cc=$aa+$bb
  ben@ben-NBLK-WAX9X:~$echo $cc
  33
  ```

  

  【数组变量类型】

  ```shell
  ben@ben-NBLK-WAX9X:~$name[0]="zhang san"
  ben@ben-NBLK-WAX9X:~$name[1]="li si"
  ben@ben-NBLK-WAX9X:~$name[2]="tong gang"
  ben@ben-NBLK-WAX9X:~$
  ben@ben-NBLK-WAX9X:~$echo ${name}
  zhang san
  ben@ben-NBLK-WAX9X:~$echo ${name[1]}
  li si
  ben@ben-NBLK-WAX9X:~$echo ${name[*]}
  zhang san li si tong gang
  ```

  数组的下标是从0开始的，在调用数组值时，需要使用${数组[下标]}的方式来读取。不过上面并没有把name变量声明为数组型，其实只要在定义变量时采用了“变量名[下标]”的格式，这个变量就会被系统认为是数组型了，不用强制声明。

  环境变量

  其实也可以使用declare命令把变量声明为环境变量，和export命令的作用是一样的：

  ```shell
  [root@localhost ~]# declare -x test=123
  ```

  

  【只读属性】

  一旦给变量设定了只读属性，那么这个变量既不能修改变量的值，也不能删除变量，甚至不能使用“+r”选项取消只读属性。

  ```shell
  ben@ben-NBLK-WAX9X:~$declare -r test
  
  # 变量值不能修改
  ben@ben-NBLK-WAX9X:~$test=456
  bash: test: 只读变量
  
  # 变量只读属性不能取消
  ben@ben-NBLK-WAX9X:~$declare +r test
  bash: declare: test: 只读变量
  
  # 变量不能删除
  ben@ben-NBLK-WAX9X:~$unset test
  bash: unset: test: 无法取消设定：只读variable
  ```

  不过这个变量只是命令行声明的，所以只要重新登录或重启，这个变量就会消失。

  

  【查询变量属性和取消变量属性】

  变量属性的查询使用“-p”选项，变量属性的取消使用“+”选项。

  ```shell
  # 查询cc变量类型
  ben@ben-NBLK-WAX9X:~$declare -p cc
  declare -i cc="33"    # int
  
  # 查询name变量类型
  ben@ben-NBLK-WAX9X:~$declare -p name
  declare -a name=([0]="zhang san" [1]="li si" [2]="tong gang")    # 数组
  
  # 查询test变量类型
  ben@ben-NBLK-WAX9X:~$declare -p test
  declare -rx test         # 只读+环境变量
  
  # 取消test变量环境变量属性
  ben@ben-NBLK-WAX9X:~$declare +x test
  
  # 查询test变量类型
  ben@ben-NBLK-WAX9X:~$declare -p test
  declare -r test             # 只读
  ```

- 使用expr或let数值运算工具

  ```shell
  ben@ben-NBLK-WAX9X:~$aa=11
  ben@ben-NBLK-WAX9X:~$bb=22
  ben@ben-NBLK-WAX9X:~$dd=$(expr $aa + $bb)
  ben@ben-NBLK-WAX9X:~$echo $dd
  33
  ```

  使用expr命令进行运算时，要注意“+”号左右两侧必须有空格，否则运算不执行。

  至于let命令，和expr命令基本类似，都是Linux中的运算命令。

  ```shell
  ben@ben-NBLK-WAX9X:~$aa=11
  ben@ben-NBLK-WAX9X:~$bb=22
  ben@ben-NBLK-WAX9X:~$let ee=$aa+$bb
  ben@ben-NBLK-WAX9X:~$echo $ee
  33
  ben@ben-NBLK-WAX9X:~$n=20
  ben@ben-NBLK-WAX9X:~$let n+=1
  ben@ben-NBLK-WAX9X:~$echo $n
  21
  ```

  let命令对格式要求比expr命令宽松，所以推荐使用let命令进行数值运算。

- 使用“$((运算式))”或“$[运算式]”方式运算

  ```shell
  ben@ben-NBLK-WAX9X:~$aa=11
  ben@ben-NBLK-WAX9X:~$bb=22
  ben@ben-NBLK-WAX9X:~$ff=$(( $aa+$bb ))
  ben@ben-NBLK-WAX9X:~$echo $ff
  33
  ben@ben-NBLK-WAX9X:~$gg=$[ $aa+$bb ]
  ben@ben-NBLK-WAX9X:~$echo $gg
  33
  ```

这三种数值运算方式，可以按照自己的习惯来进行使用，不过推荐使用“$((运算式))”的方式。



#### 常用运算符

| 优先级 | 运算符                                      | 说明                           |
| ------ | ------------------------------------------- | ------------------------------ |
| 13     | -，+                                        | 单目负、单目正                 |
| 12     | !，~                                        | 逻辑非、按位取反或补码         |
| 11     | *，/，%                                     | 乘、除、取模                   |
| 10     | +，-                                        | 加，减                         |
| 9      | <<，>>                                      | 按位左移，按位右移             |
| 8      | <=，>=，<，>                                | 小于等于、大于等于、小于、大于 |
| 7      | ==，!=                                      | 等于、不等于                   |
| 6      | &                                           | 按位与                         |
| 5      | ^                                           | 按位异或                       |
| 4      | \|                                          | 按位或                         |
| 3      | &&                                          | 逻辑与                         |
| 2      | \|\|                                        | 逻辑或                         |
| 1      | =，+=，-=，*=、=，%=，&=，^=，\|=，<<=，>>= | 赋值、运算且赋值               |

运算符优先级表明在每个表达式或子表达式中哪一个运算对象首先被求值，数值越大优先级越高，具有较高优先级级别的运算符先于较低级别的运算符进行求值运算。

加减乘除

```shell
ben@ben-NBLK-WAX9X:~$aa=$(( (11+3)*3/2 ))
ben@ben-NBLK-WAX9X:~$echo $aa
21
```

取模运算

```shell
ben@ben-NBLK-WAX9X:~$bb=$(( 14%3 ))
ben@ben-NBLK-WAX9X:~$echo $bb
2
```

逻辑与

```shell
ben@ben-NBLK-WAX9X:~$cc=$(( 1 & 0 ))
ben@ben-NBLK-WAX9X:~$echo $cc
0
```



### 变量的测试与内容置换

| 变量置换方式 | 变量y没有设置                | 变量y为空值            | 变量y设置值   |
| ------------ | ---------------------------- | ---------------------- | ------------- |
| x=${y-新值}  | x=新值                       | x为空                  | x=$y          |
| x=${y:-新值} | x=新值                       | x=新值                 | x=$y          |
| x=${y+新值}  | x为空                        | x=新值                 | x=新值        |
| x=${y:+新值} | x为空                        | x为空                  | x=新值        |
| x=${y=新值}  | x=新值，y=新值               | x为空，y值不变         | x=$y，y值不变 |
| x=${y:=新值} | x=新值，y=新值               | x=新值，y=新值         | x=$y，y值不变 |
| x=${y?新值}  | 新值输出到标准错误输出(屏幕) | x为空                  | x=$y          |
| x=${y:?新值} | 新值输出到标准错误输出       | 新值输出到标准错误输出 | x=$y          |

如果大括号内没有":"，则变量y为空，还是没有设置，处理方法是不同的；如果大括号内有“:”，则变量y不论是为空，还是没有设置，处理方法是一样的。

如果大括号内是“-”或“+”，则在改变变量x值的时候，变量y是不改变的；如果大括号内是“=”，则在改变变量x的同时，变量y的值也会改变。

如果大括号内是“?”，则当变量y不存在或为空时，会把“新值”当成报错输出到屏幕上。

变量y没有设置

```shell
# 变量y没有设置
ben@ben-NBLK-WAX9X:~$unset y
ben@ben-NBLK-WAX9X:~$x=${y-new}
# 变量x等于新值
ben@ben-NBLK-WAX9X:~$echo $x
new
# 变量y不存在
ben@ben-NBLK-WAX9X:~$echo $y

ben@ben-NBLK-WAX9X:~$
```

变量y为空

```shell
# 变量y为空
ben@ben-NBLK-WAX9X:~$y=""
ben@ben-NBLK-WAX9X:~$x=${y-new}
# 则变量x为空
ben@ben-NBLK-WAX9X:~$echo $x

ben@ben-NBLK-WAX9X:~$echo $y

ben@ben-NBLK-WAX9X:~$
```

变量y有值

```shell
ben@ben-NBLK-WAX9X:~$y=old
ben@ben-NBLK-WAX9X:~$x=${y-new}
ben@ben-NBLK-WAX9X:~$echo $x
old
ben@ben-NBLK-WAX9X:~$echo $y
old
ben@ben-NBLK-WAX9X:~$
```

## 环境变量配置文件

###  source命令

```
[root@localhost ~]# source 配置文件
或
[root@localhost ~]# . 配置文件
```



### 环境变量配置文件

#### 登录时生效的环境变量配置文件

在Linux系统登录时主要生效的环境变量配置文件有以下五个：

- /etc/profile
- /etc/profile.d/*.sh
- ~/.bash_profile
- ~/.bashrc
- /etc/bashrc

环境变量配置文件调用过程

- 在用户登录过程先调用/etc/profile文件

  在这个环境变量配置文件中会定义这些默认环境变量：

  - USER变量：根据登录的用户，给这个变量赋值(就是让USER变量的值是当前用户)。
  - LOGNAME变量：根据USER变量的值，给这个变量赋值。
  - MAIL变量：根据登录的用户，定义用户的邮箱为/var/spool/mail/用户名。
  - PATH变量：根据登录用户的UID是否为0，判断PATH变量是否包含/sbin、/usr/sbin和、/usr/local/sbin这三个系统命令目录。
  - HOSTNAME变量：更改主机名，给这个变量赋值。
  - HISTSIZE变量：定义历史命令的保存条数。
  - umask：定义umask默认权限。注意/etc/profile文件中的umask权限是在“有用户登录过程(也就是输入了用户名和密码)”时才会生效。
  - 调用/etc/profile.d/*.sh文件，也就是调用/etc/profile.d/目录下所有以.sh结尾的文件。

- 由/etc/profile文件调用/etc/profile.d/*.sh文件

  这个目录中所有以.sh结尾的文件都会被/etc/profile文件调用，这里最常用的就是lang.sh文件，而这个文件又会调用/etc/sysconfig/i18n文件。

- 由/etc/profile文件调用~/.bash_profile文件

  ~/.bash_profile文件主要实现两个功能：

  - 调用了~/.bashrc文件。
  - 在PATH变量后面加入了“:$HOME/bin”这个目录。也就是说，如果我们在自己的家目录中建立bin目录，然后把自己的脚本放入“~/bin”目录，就可以直接执行脚本，而不用通过目录执行了。

- 由~/.bash_profile文件调用~/.bashrc文件

  在~/.bashrc文件中主要实现了：

  - 定义默认别名。
  - 调用/etc/bashrc

- 由~/.bashrc调用了/etc/bashrc文件

  在/etc/bashrc文件中主要定义了这些内容：

  - PS1变量：也就是用户的提示符，如果想要永久修改提示符，就要在这个文件中修改。
  - umask：定义umask默认权限。这个文件中定的umask是针对"没登录过程(也就是不需要输入用户名和密码时，比如从一个终端切换到另一个终端，或进入子shell)"时生效的。如果是“有用户登录过程”，则是/etc/profile文件中的umask生效。
  - PATH变量：会给PATH变量追加值，当然也是在“没有登录过程”时才生效。
  - 调用/etc/profile.d/\*.sh文件，这也是在"没有用户登录过程"时才调用。在“有用户登录过程”时，/etc/profile.d/\*.sh文件已经被/etc/profile文件调用过了。

这样五个环境变量配置文件会被依次调用，如果是我们自己定义的环境变量，如果你的修改是打算对所有用户生效的，那么可以放入/etc/profile环境变量配置文件；如果你的修改只是给自己使用的，那么可以放入~/.bash_profile或~/.bashrc这两个配置文件中的任一个。

如果我们误删除了这些环境变量，比如删除了/etc/bashrc文件，或删除了~/.bashrc文件，那么这些文件中配置就会失效(~/.bashrc_profile文件会调用/etc/profile文件)。那提示符就会变成：

```shell
-bash-4.1#
```



#### 注销时生效的环境变量配置文件

在用户退出登录时，只会调用一个环境变量配置文件，就是~/.bash_logout。这个文件默认没有写入任何内容，可是如果我们希望在退出登录时执行一些操作，比如清除历史命令，备份某些数据，就可以把命令写入这个文件。



#### 其他配置文件

还有一些环境变量配置文件，最常见的就是~/bash_history文件，也就是历史命令保存文件。



### Shell登录信息

#### /etc/issue

我们在登录tty1-6这六个本地终端时，会有几行的欢迎界面。这些欢迎信息是保存在/etc/issue文件中。

```
[root@localhost ~]# cat /etc/issue
CentOS release 7.9 (Final)
Kernel \r on an \m
```

可以支持的转义符可以通过man agetty命令查询。下表中列出常见的转义符作用：

| 转义符 | 作用                             |
| ------ | -------------------------------- |
| \d     | 显示当前系统日期                 |
| \s     | 显示操作系统名称                 |
| \l     | 显示登录的终端号，这个比较常用。 |
| \m     | 显示硬件体系结构，如i386、i686等 |
| \n     | 显示主机名                       |
| \o     | 显示域名                         |
| \r     | 显示内核版本                     |
| \t     | 显示当前系统时间                 |
| \u     | 显示当前登录用户的序列号         |



#### /etc/issue.net

/etc/issue是在本地终端登录时显示欢迎信息的。如果是远程登录(如ssh远程登录，或telnet远程登录)需要显示欢迎信息，则需要配置/etc/issue.net这个文件了。使用这个文件时需要注意两个点：

- 在/etc/issue文件中支持的转义符，在/etc/issue.net文件中不能使用。
- ssh远程登录是否显示/etc/issue.net的欢迎信息，是由ssh的配置文件决定的。

如果需要ssh远程登录可以查看/etc/issue.net的欢迎信息，那么首先需要修改ssh的配置文件/etc/ssh/sshd_config：

```shell
[root@localhost ~]# cat /etc/ssh/sshd_config
...
# no default banner path
#Banner none
Banner /etc/issue.net
...
```

这样在ssh远程登录时，也可以显示欢迎信息，只是不再可以识别“\d”和“\l”等信息了。



#### /etc/motd

/etc/motd文件也是显示欢迎信息的，这个文件和/etc/issue及/etc/issue.net文件的区别是：/etc/issue及/etc/issue.net是在用户登录之前显示欢迎信息，而/etc/motd是在用户输入用户名和密码，正确登录之后显示欢迎信息。在/etc/motd文件中的欢迎信息，不论是本地登录，还是远程登录都可以显示。



### 定义Bash快捷键

```
[root@localhost ~]# stty 关键字 快捷键
```

```shell
# 定义ctrl+p快捷键为强制终止，"^"字符只要手工输入即可
[root@localhost ~]# stty intr ^p

# 查询所有快捷键
[root@localhost ~]# stty -a
intr = ^P; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; discard = ^O;
min = 1; time = 0;
```

