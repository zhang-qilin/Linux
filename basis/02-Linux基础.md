# linux基础

## 1、用户登录

`root 用户`

- 一个特殊的管理帐户
- 也被称为超级用户
- root已接近完整的系统控制
- 对系统损害几乎有无限的能力
- 除非必要,不要登录为 root

```shell
[root@ali ~]# id
uid=0(root) gid=0(root) groups=0(root)
```

`普通（ 非特权 ）用户`

- 权限有限
- 造成损害的能力比较有限

```shell
[zsjshao@ali ~]$ id
uid=1001(zsjshao) gid=1001(zsjshao) groups=1001(zsjshao)
```

## 2、终端terminal

终端（Terminal）也称终端设备，是计算机网络中处于网络最外围的设备，主要用于用户信息的输入以及处理结果的输出等。

在早期计算机系统中，由于计算机主机昂贵，因此一个主机（IBM大型计算机）一般会配置多个终端，这些终端本身不具备计算能力，仅仅承担信息输入输出的工作，运算和处理均由主机来完成。

在个人计算机时代，个人计算机可以运行称为终端仿真器的程序来模仿一个终端的工作。随着移动网络的发展，移动终端（如手机、PAD）等得到了广泛的应用。此时，终端不仅能承担输入输出的工作，同时也能进行一定的运算和处理，实现部分系统功能。

![linux_01](./images/02-linux_basic/linux_01.png)

设备终端

- 键盘、鼠标、显示器

物理终端（ /dev/console ）

- 控制台console

串行终端（ /dev/ttyS# ）

- ttyS

虚拟终端(tty：teletypewriters， /dev/tty#

- tty 可有n个，Ctrl+Alt+F#

图形终端（ /dev/tty7 ） startx, xwindows

- CentOS 6: Ctrl + Alt + F7
- CentOS 7: 在哪个终端启动，即位于哪个虚拟终端

伪终端（ pty：pseudo-tty ， /dev/pts/# ）

- pty, SSH远程连接

查看当前的终端设备：

```shell
[root@ali ~]# tty
/dev/pts/1
```

查看当前登录用户，登录的所有用户

```shell
[root@ali ~]# who am i
root     pts/1        2020-04-14 01:08 (120.85.147.229)

[root@ali ~]# who
root     pts/0        2020-04-14 01:09 (120.85.147.229)

[root@ali ~]# w
 01:09:08 up 8 days, 19:11,  2 users,  load average: 0.00, 0.03, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    120.85.147.229   23:41    5:34   0.14s  0.14s -bash
root     pts/1    120.85.147.229   01:08    1.00s  0.02s  0.00s w
```

## 3、交互式接口

交互式接口：启动终端后，在终端设备附加一个交互式应用程序

GUI：Graphic User Interface

- X protocol, window manager, desktop
- Desktop:
  - GNOME (C, 图形库gtk)，
  - KDE (C++,图形库qt)
  - XFCE (轻量级桌面)

CLI：Command Line Interface

- shell程序

## 4、什么是shell

Linux（or UNIX）Shell也叫作命令行界面，它是Linux/UNIX操作系统下传统的用户和计算机交互界面，用户可直接输入命令来执行各种各样的任务。Linux系统的shell作为操作系统的外壳，为用户提供使用操作系统的接口。它是命令语言、命令解释程序及程序设计语言的统称。

Linux中有多种shell，其中缺省使用的是Bash。 shell还有sh，csh，ksh，tcsh，zsh。默认系统所支持的shell保存在/etc/shells文件中，可以为特殊应用的用户选择不同的shell，比如选择/sbin/nologin可以禁止用户登录

<img src="./images/02-linux_basic/linux_02.png" alt="linux_02" style="zoom:50%;" />

## 5、bash shell

GNU Bourne-Again Shell(bash)是GNU计划中重要的工具软件之一，目前也是 Linux标准的shell，与sh兼容，CentOS默认使用bash

显示当前使用的shell

```shell
[root@ali ~]# echo ${SHELL}
/bin/bash
```

显示当前系统使用的所有shell

```shell
[root@ali ~]# cat /etc/shells 
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

退出当前shell

```shell
[root@ali ~]# bash
[root@ali ~]# exit
exit

或者按组合键Ctrl+D
```

### 5.1、命令提示符

命令提示符：prompt

```shell
[root@ali ~]# 
# 管理员
$ 普通用户
```

显示提示符格式

```shell
[root@ali ~]# echo $PS1
[\u@\h \W]\$
```

修改提示符格式

```shell
PS1="\[\e[1;5;41;33m\][\u@\h \W]\\$\[\e[0m\]"
1-8：设置字体属性，1：高亮，4：下划线，5：闪烁，7：反显，8：消隐
31-37：字体颜色
41-47：背景色

PS1="\[\e[1;32m\][\[\e[0m\]\t \[\e[1;33m\]\u\[\e[36m\]@\h\[\e[1;31m\] \W\[\e[1;32m\]]\[\e[0m\]\\$"
\e 控制符\033      \u 当前用户
\h 主机名简称      \H 主机名
\w 当前工作目录    \W 当前工作目录基名
\t 24小时时间格式  \T 12小时时间格式
\! 命令历史数      \# 开机后命令历史数
```

### 5.2、执行命令

输入命令后回车

- 命令提交给shell程序，shell程序找到键入命令所对应的可执行程序或代码，并由其分析后提交给内核分配资源将其运行起来

在shell中可执行的命令有两类

- 内部命令：由shell自带的，而且通过某命令形式提供
```shell
help 内部命令列表
enable cmd 启用内部命令
enable –n cmd 禁用内部命令
enable –n 查看所有禁用的内部命令
```

- 外部命令：在文件系统路径下有对应的可执行程序文件
  - 查看路径：which -a |--skip-alias ; whereis

```shell
[root@ali ~]# which -a ls
alias ls='ls --color=auto'
	/usr/bin/ls
[root@ali ~]# which --skip-alias ls
/usr/bin/ls

[root@ali ~]# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz /usr/share/man/man1p/ls.1p.gz
```

区别指定的命令是内部或外部命令

- type [-a] COMMAND

```shell
[root@ali ~]# type -a cd
cd is a shell builtin
cd is /usr/bin/cd
```

### 5.3、执行外部命令

#### 5.3.1、Hash缓存表

系统初始hash表为空，当外部命令执行时，默认会从PATH路径下寻找该命令，找到后会将这条命令的路径记录到hash表中，当再次使用该命令时，shell解释器首先会查看hash表，存在将执行之，如果不存在，将会去PATH路径下寻找，利用hash缓存表可大大提高命令的调用速率

hash常见用法

- hash 显示hash缓存

- hash –l 显示hash缓存，可作为输入使用

- hash –p path name 将命令全路径path起别名为name

- hash –t name 打印缓存中name的路径

- hash –d name 清除name缓存

- hash –r 清除缓存

```shell
[root@ali ~]# hash 
hits	command
   4	/usr/bin/who
   3	/usr/bin/man
   1	/usr/bin/id
   2	/usr/bin/w
   1	/usr/bin/su

[root@ali ~]# hash -l
builtin hash -p /usr/bin/who who
builtin hash -p /usr/bin/man man
builtin hash -p /usr/bin/id id
builtin hash -p /usr/bin/w w
builtin hash -p /usr/bin/su su

[root@ali ~]# hash -t w
/usr/bin/w
```

PATH路径

```shell
[root@ali ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/go/bin:/root/bin
从左到右查找

[root@ali ~]# type whereis
whereis is hashed (/usr/bin/whereis)
```

cache is king，redis、memcached

### 5.4、命令别名

对于一些较长的命令，且又经常使用，可以使用别名的方式进行定义，以减少反复较长的输入。使用alias命令可以显示和定义别名，使用unalias取消命令别名。除非将别名的定义写到配置文件中，否则别名只在当前会话中有效。

显示当前shell进程所有可用的命令别名

```shell
[root@ali ~]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='(alias; declare -f) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde -
-show-dot'alias xzegrep='xzegrep --color=auto'
alias xzfgrep='xzfgrep --color=auto'
alias xzgrep='xzgrep --color=auto'
alias zegrep='zegrep --color=auto'
alias zfgrep='zfgrep --color=auto'
alias zgrep='zgrep --color=auto
```

定义别名NAME，其相当于执行命令VALUE

- alias NAME='VALUE'

在命令行中定义的别名，仅对当前shell进程有效，如果想永久有效，要定义在配置文件中

- 仅对当前用户：~/.bashrc
- 对所有用户有效：/etc/bashrc

编辑配置给出的新配置不会立即生效，bash进程重新读取配置文件

- source /path/to/config_file
- . /path/to/config_file

撤消别名：unalias

- unalias [-a] name [name ...]
- -a 取消所有别名

命令生效优先级：alias --- 内部命令 --- hash表 --- $PATH --- 命令找不到

如果别名同原命令同名，如果要执行原命令，可使用

- \ALIASNAME
- “ALIASNAME”
- ‘ALIASNAME’
- command ALIASNAME
- /path/commmand

### 5.5、命令格式

COMMAND [OPTIONS...] [ARGUMENTS...]

- 选项：用于启用或关闭命令的某个或某些功能  
短选项：-c 例如：-l，-h，-lh  
长选项：--word 例如：--all, --human-readable

- 参数：命令的作用对象，比如文件名，用户名等

注意：

- 多个选项以及多参数和命令之间使用空白字符分隔
- 取消和结束命令执行：Ctrl+c，Ctrl+d
- 多个命令可以用;符号分开
- 一个命令可以用\分成多行

### 5.6、日期和时间

Linux的两种时钟

- 系统时钟：由Linux内核通过CPU的工作频率进行计时
- 硬件时钟：主板

相关命令

```shell
date 显示和设置系统时间
  date MMDDHHmmYYYY.ss
  date +%s
  date -d @1509536033
  date -d '-1 day'
  date -d '1 day'
  date -s '-1 day'

hwclock，clock: 显示硬件时钟
  -s, --hctosys 以硬件时钟为准，校正系统时钟
  -w, --systohc 以系统时钟为准，校正硬件时钟

修改时区：timedatectl
[root@ali ~]# timedatectl set-timezone Asia/Shangha
[root@ali ~]# ll /etc/localtime 
lrwxrwxrwx. 1 root root 35 Feb 18 16:02 /etc/localtime -> ../usr/share/zoneinfo/Asia/Shanghai

显示日历：cal –y
cal 9 1752
```

### 5.7、简单命令

#### 5.7.1、关机：halt, poweroff

#### 5.7.2、重启：reboot

```shell
reboot
  -f: 强制，不调用shutdown
  -p: 切断电源
```

#### 5.7.3、关机或重启：shutdown

```shell
shutdown [OPTION]... [TIME] [MESSAGE]
  -r: reboot
  -h: halt
  -c：cancel
  TIME：无指定，默认相当于+1（CentOS7）

  now: 立刻,相当于+0
  +m: 相对时间表示法，几分钟之后；例如 +3
  hh:mm: 绝对时间表示，指明具体时间
```

#### 5.7.4、用户登录信息查看命令：

```shell
whoami: 显示当前登录有效用户
who: 系统当前所有的登录会话
w: 系统当前所有的登录会话及所做的操作
```

#### 5.7.5、nano 文本编辑

```shell
[root@ali ~]# nano /etc/issue

GNU nano 2.9.8      /etc/issue   
\S
Kernel \r on an \m


                             [ Read 3 lines ]
^G Get Help ^O Write Out ^W Where Is ^K Cut Text   ^J Justify  ^C Cur Pos    M-U Undo	M-A Mark Text
^X Exit     ^R Read File ^\ Replace  ^U Uncut Text ^T To Spell ^_ Go To Line M-E Redo	 M-6 Copy Text
```

#### 5.7.6、screen命令：

```shell
- 创建新screen会话
  screen –S [SESSION]

- 显示所有已经打开的screen会话
  screen -ls

- 加入screen会话
  screen –x [SESSION]

- 退出并关闭screen会话
  exit

- 剥离当前screen会话
  Ctrl+a,d

- 恢复某screen会话
  screen -r [SESSION]
```

#### 5.7.7、echo命令

```shell
功能：显示字符
语法：echo [-neE][字符串]
说明：echo会将输入的字符串送往标准输出。输出的字符串间以空白字符隔开, 并在最后加上换行号
选项：
  -E （默认）不支持 \ 解释功能
  -n 不自动换行
  -e 启用 \ 字符的解释功能

显示变量
  echo "$VAR_NAME” 变量会替换，弱引用
  echo '$VAR_NAME’ 变量不会替换，强引用
  注意：``反引号，命令引用

启用命令选项-e，若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出
 \a 发出警告声
 \b 退格键
 \c 最后不加上换行符号
 \e escape，相当于\033
 \n 换行且光标移至行首
 \r 回车，即光标移至行首，但不换行
 \t 插入tab
 \\ 插入\字符
 \0nnn 插入nnn（八进制）所代表的ASCII字符
   echo -e '\033[43;31;5mmagedu\e[0m'
 \xHH插入HH（十六进制）所代表的ASCII数字（man 7 ascii）
```

#### 5.7.8、hexdump命令

```shell
功能：以十六进制、十进制、八进制、二进制或 ascii 显示文件内容
语法：hexdump [选项] <文件>...
选项：
 -b, --one-byte-octal      单字节八进制显示
 -c, --one-byte-char       单字节字符显示
 -C, --canonical           规范化 hex+ASCII 显示
 -n, --length <长度>       只解释规定字节长度的输入
 -s, --skip <偏移>         跳过开头的指定字节偏移

[root@ali ~]# hexdump -C -n 66 -s 446 /dev/vda
000001be  80 00 21 02 83 04 63 14  00 08 00 00 df f7 ff 04  |..!...c.........|
000001ce  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001fe  55 aa                                             |U.|
00000200
```

### 5.8、字符集和编码

`ASCII码：`计算机内部，所有信息最终都是一个二进制值。上个世纪60年代，美国制定了一套字符编码，对英语字符与二进制位之间的关系，做了统一规定。ASCII 码一共规定了128个字符的编码，占用了一个字节的后面7位，最前面的一位统一规定为0

`Unicode：`用于表示世界上所有语言中的所有字符。每一个符号都给予一个独一无二的编码数字，Unicode 是一个很大的集合，现在的规模可以容纳100多万个符号。Unicode 仅仅只是一个字符集，规定了每个字符对应的二进制代码，至于这个二进制代码如何存储则没有规定

`Unicode编码方案：`

- UTF-8：变长，1到4个字节
- UTF-16：变长，2或4个字节
- UTF-32：固定长度，4个字节

![linux_03](./images/02-linux_basic/linux_03.png)

UTF-8 是目前互联网上使用最广泛的一种 Unicode 编码方式，可变长存储。使用 1 - 4 个字节表示一个字符，根据字符的不同变换长度。编码规则如下：

- 对于单个字节的字符，第一位设为 0，后面的 7 位对应这个字符的 Unicode 码。因此，对于英文中的 0 - 127 号字符，与 ASCII 码完全相同。这意味着 ASCII 码的文档可用 UTF-8 编码打开
- 对于需要使用 N 个字节来表示的字符（N > 1），第一个字节的前 N 位都设为 1，第 N + 1 位设为0，剩余的 N - 1 个字节的前两位都设位 10，剩下的二进制位则使用这个字符的 Unicode 码来填充

编码转换和查询：
http://www.chi2ko.com/tool/CJK.htm
https://javawind.net/tools/native2ascii.jsp?action=transform
http://tool.oschina.net/encode

#### 5.8.1、Unicode和UTF-8

| Unicode符号范围 (十六进制) | UTF-8编码方式 （二进制 |
| -------------------------- | ---------------------- |
|0000 0000-0000 007F|0xxxxxxx|
|0000 0080-0000 07FF|110xxxxx 10xxxxxx|
|0000 0800-0000 FFFF|1110xxxx 10xxxxxx 10xxxxxx|
|0001 0000-0010 FFFF|11110xxx 10xxxxxx 10xxxxxx 10xxxxxx|

示例：
“汉”的 Unicode 码 0x6C49（110 110001 001001），需要三个字节存储，格式为： 1110xxxx 10xxxxxx 10xxxxxx，从后向前依次填充对应格式中的 x，多出的 x 用 0 补，得出UTF-8 编码为 11100110 10110001 10001001
“马”的 Unicode 码 0x9A6C（1001 101001 101100），需要三个字节存储，格式为： 1110xxxx 10xxxxxx 10xxxxxx，从后向前依次填充对应格式中的 x，多出的 x 用 0 补，得出UTF-8 编码为11101001 10101001 10101100

##### 5.8.1.1、localectl命令

```shell
功能：系统语言和键盘设置。
用法： localectl [OPTIONS...] COMMAND ...
Commands:
  status                   Show current locale settings
  set-locale LOCALE...     Set system locale
  list-locales             Show known locales

[root@ali ~]# localectl status
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: n/a
[root@ali ~]# localectl set-locale LANG=en_US.UTF-8
[root@ali ~]# cat /etc/locale.conf 
LANG=en_US.UTF-8
```

##### 5.8.1.2、iconv命令

```shell
功能：转换给定文件的编码。
用法： iconv [选项...] [文件...]
输入/输出格式规范：
  -f, --from-code=名称     原始文本编码
  -t, --to-code=名称       输出编码
  -l, --list                 列举所有已知的字符集
  -o, --output=文件        输出文件

[root@ali ~]# iconv -f gb2312 w.txt -o l.txt
```

### 5.9、命令引用、花括号展开

命令引用：$( ) 或 ``

- 把一个命令的输出打印给另一个命令的参数

```shell
echo "This system's name is $(hostname) "
This system's name is server1.example.com
echo "i am `whoami` "
i am root
```

花括号展开：{ }

- 打印重复字符串的简化形式

```shell
echo file{1,3,5} 结果为：file1 file3 file5
rm -f file{1,3,5}
echo {1..10}
echo {a..z}
echo {000..20..2}

[root@ali ~]# echo {000..20..2}
000 002 004 006 008 010 012 014 016 018 020
```



### 5.10、tab键

Tab补全允许用户在提示符下键入足够的内容以使其唯一后快速补全命令或文件名。

`命令补全`

- 内部命令：
- 外部命令：bash根据PATH环境变量定义的路径，自左而右在每个路径搜寻以给定命令名命名的文件，第一次找到的命令即为要执行的命令
- 用户给定的字符串只有一条惟一对应的命令，直接补全
- 否则，再次Tab会给出列表
- 许多命令可以通过Tab补全匹配参数和选项。需安装bash-completion软件包

`路径补全`

- 把用户给出的字符串当做路径开头，并在其指定上级目录下搜索以指定的字符串开头的文件名
- 如果惟一：则直接补全
- 否则：再次Tab给出列表

### 5.11、命令行历史

保存你输入的命令历史。可以用它来重复执行命令，登录shell时，会读取命令历史文件中记录下的命令~/.bash_history，登录进shell后新执行的命令只会记录在缓存中；这些命令会用户退出时“追加”至命令历史文件中

#### 5.11.1、命令历史快捷键

|!:0 |执行前一条命令（去除参数）|
| ---------- | --------------------- |
|Ctrl + p，up（向上） |显示当前历史中的上一条命令，但不执行|
|Ctrl + n，down（向下） |显示当前历史中的下一条命令，但不执行|
|Ctrl + j |执行当前命令|
|ctrl + r |在命令历史中搜索命令，（reverse-i-search）`’：|
|Ctrl + g |从历史搜索模式退出|
|!n |执行history命令输出对应序号n的命令|
|!-n |执行history历史中倒数第n个命令|
|!string |重复前一个以“string”开头的命令|
|!?string |重复前一个包含string的命令|
|!string:p |仅打印命令历史，而不执行|
|!$:p |打印输出 !$ （上一条命令的最后一个参数）的内容|
|!\*:p |打印输出 !\*（上一条命令的所有参数）的内容|
|^string |删除上一条命令中的第一个string|
|^string1^string2 |将上一条命令中的第一个string1替换为string2|
|!:gs/string1/string2 |将上一条命令中所有的string1都替换为 string2|

重复前一个命令，有4种方法

- 重复前一个命令使用上方向键，并回车执行
- 按 !! 并回车执行
- 输入 !-1 并回车执行
- 按 Ctrl+p 并回车执行

要重新调用前一个命令中最后一个参数

- !$ 表示
- Esc, .（点击Esc键后松开，然后点击 . 键）
- Alt+ .（按住Alt键的同时点击 . 键）

#### 5.11.2、调用历史参数
|command !^ |利用上一个命令的第一个参数做cmd的参数|
| ---------- | --------------------- |
|command !$ |利用上一个命令的最后一个参数做cmd的参数|
|command !\* |利用上一个命令的全部参数做cmd的参数|
|command !:n |利用上一个命令的第n个参数做cmd的参数|
|command !n:^ |调用第n条命令的第一个参数|
|command !n:$ |调用第n条命令的最后一个参数|
|command !n:m |调用第n条命令的第m个参数|
|command !n:\* |调用第n条命令的所有参数|
|command !string:^ |从命令历史中搜索以 string 开头的命令，并获取它的第一个参数|
|command !string:$ |从命令历史中搜索以 string 开头的命令,并获取它的最后一个参数|
|command !string:n |从命令历史中搜索以 string 开头的命令，并获取它的第n个参数|
|command !string:\* |从命令历史中搜索以 string 开头的命令，并获取它的所有参数|


#### 5.11.3、命令history

```shell
history [-c] [-d offset] [n]
history -anrw [filename]
history -ps arg [arg...]
  -c: 清空命令历史
  -d offset: 删除历史中指定的第offset个命令
  n: 显示最近的n条历史
  -a: 追加本次会话新执行的命令历史列表至历史文件
  -r: 读历史文件附加到历史列表
  -w: 保存历史列表到指定的历史文件
  -n: 读历史文件中未读过的行到历史列表
  -p: 展开历史参数成多行，但不存在历史列表中
  -s: 展开历史参数成一行，附加在历史列表后

[root@ali ~]# history 3
 1030  echo {000..20..2}
 1031  echo {000..20}
 1032  history 3
```

#### 5.11.4、命令历史相关环境变量

HISTSIZE：命令历史记录的条数  
HISTFILE：指定历史文件，默认为~/.bash_history  
HISTFILESIZE：命令历史文件记录历史的条数  
HISTTIMEFORMAT=“%F %T “ 显示时间  
HISTIGNORE=“str1:str2*:… “ 忽略str1命令，str2开头的历史

控制命令历史的记录方式：

- 环境变量：HISTCONTROL  
  ignoredups：默认，忽略重复的命令，连续且相同为“重复”  
  ignorespace：忽略所有以空白开头的命令  
  ignoreboth：相当于ignoredups, ignorespace的组合  
  erasedups：删除重复命令

export 变量名="值“

存放在 /etc/profile 或 ~/.bash_profile

```shell
[root@ali ~]# echo $HISTCONTROL
ignoredups
```

### 5.12、bash的快捷键

| 快捷键 | 功能 |
| ---------- | --------------------- |
| Ctrl + l | 清屏，相当于clear命令|
| Ctrl + o | 执行当前命令，并重新显示本命令|
| Ctrl + s | 阻止屏幕输出，锁定|
| Ctrl + q | 允许屏幕输出|
| Ctrl + c | 终止命令|
| Ctrl + z | 挂起命令|
| Ctrl + a | 光标移到命令行首，相当于Home|
| Ctrl + e | 光标移到命令行尾，相当于End|
| Ctrl + f | 光标向右移动一个字符|
| Ctrl + b | 光标向左移动一个字符|
| Alt + f | 光标向右移动一个单词尾|
| Alt + b | 光标向左移动一个单词首|
| Ctrl + xx | 光标在命令行首和光标之间移动|
| Ctrl + u | 从光标处删除至命令行首|
| Ctrl + k | 从光标处删除至命令行尾|
| Alt + r | 删除当前整行|
| Ctrl + w | 从光标处向左删除至单词首|
| Alt + d | 从光标处向右删除至单词尾|
| Ctrl + d|  删除光标处的一个字符|
| Ctrl + h|  删除光标前的一个字符|
| Ctrl + y|  将删除的字符粘贴至光标后|
| Alt + c | 从光标处开始向右更改为首字母大写的单词|
| Alt + u | 从光标处开始，将右边一个单词更改为大写|
| Alt + l | 从光标处开始，将右边一个单词更改为小写|
| Ctrl + t | 交换光标处和之前的字符位置|
| Alt + t | 交换光标处和之前的单词位置|
| Alt + N | 提示输入指定字符后，重复显示该字符N次|
`注意：Alt组合快捷键经常和其它软件冲突`

## 6、获得帮助

获取帮助的能力决定了技术的能力！

多层次的帮助

- whatis
- command --help
- man and info
- /usr/share/doc/
- Red Hat documentation
- 其它网站和搜索

### 6.1、whatis

whatis显示命令的简短描述，使用数据库存储检索信息，刚安装后数据库未建立不可立即使用，需使用makewhatis | mandb生成数据库

使用示例：

- whatis cal 或 man –f cal

### 6.2、命令帮助

```shell
内部命令：help COMMAND 或 man bash
外部命令： 
  (1) COMMAND --help 或 COMMAND -h
  (2) 使用手册(manual)
    man COMMAND
  (3) 信息页
    info COMMAND
  (4) 程序自身的帮助文档
    README
    INSTALL
    ChangeLog
  (5) 程序官方文档
    官方站点：Documentation
  (6) 发行版的官方文档
  (7) Google
```

--help或-h 选项

- 大多数命令都有-h或--help的帮助选项，该选项会在终端输出简洁的帮助信息。

```shell
显示用法总结和参数列表
使用的大多数，但并非所有的
示例：
  date --help
  Usage: date [OPTION]... [+FORMAT] or: date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
    [] 表示可选项
    CAPS或 <> 表示变化的数据
    ... 表示一个列表
    x |y| z 的意思是“ x 或 y 或 z “
    -abc的 意思是-a -b –c
    { } 表示分组
```

### 6.3、man命令

man page源自过去的linux程序员手册，该手册篇幅很长，足以打印成多本书册，手册页存放在/usr/share/man，几乎每个命令都有man的“页面”，man页面分组为不同的“章节”，统称为Linux手册

man命令的配置文件：/etc/man.config | man_db.conf

- MANPATH /PATH/TO/SOMEWHERE: 指明man文件搜索位置

man -M /PATH/TO/SOMEWHERE COMMAND: 到指定位置下搜索COMMAND命令的手册页并显示

中文man需安装包man-pages-zh-CN

#### 6.3.1、man 章节

分别包含具体文件类型的信息，现已成为如下所列章节。

| 章节 | 内容类型                             |
| ---- | ------------------------------------ |
| 1    | 用户命令（可执行命令和shell程序）    |
| 2    | 系统调用（从用户空间调用的内核例程） |
| 3    | 库函数（由程序库提供）               |
| 4    | 特殊文件（如设备文件）               |
| 5    | 文件格式（用于许多配置文件和结构）   |
| 6    | 游戏（过去的有趣程序章节）           |
| 7    | 惯例、标准和其他（协议、文件系统）   |
| 8    | 系统管理和特权命令（维护任务）       |
| 9    | linux内核API（内核调用）             |

#### 6.3.2、man 帮助段落说明


| NAME|名称及简要说明|
| ---- | ------------------------------------ |
| SYNOPSIS| 用法格式说明    <br>  \[] 可选内容  <br>  <> 必选内容   <br> a\|b 二选一    <br>{ } 分组  <br>... 同一内容可出现多次 |
| DESCRIPTION| 详细说明b |
| OPTIONS| 选项说明|
| EXAMPLES| 示例|
| FILES| 相关文件|
| AUTHOR| 作者|
| COPYRIGHT| 版本信息|
| REPORTING| BUGS bug信息|
| SEE ALSO| 其它帮助参考|

#### 6.3.3、man 帮助

```shell
- 查看man手册页
  man [章节] keyword

- 列出所有帮助
  man –a keyword

- 搜索man手册
  man -k keyword 列出所有匹配的页面
  使用 whatis 数据库

- 相当于whatis
  man –f keyword

- 打印man帮助文件的路径
  man –w [章节] keyword
```

#### 6.3.4、man导航

| 命令                         | 功能                             |
| ---------------------------- | -------------------------------- |
| space, ^v, ^f, ^F            | 向前（向下）滚动一个屏幕         |
| b, ^b                        | 向后（向上）滚动一个屏幕         |
| PageDown                     | 向前（向下）滚动一个屏幕         |
| PageUp                       | 向后（向上）滚动一个屏幕         |
| RETURN, ^N, e, ^E or j or ^J | 向前（向下）滚动一行             |
| y or ^Y or ^P or k or ^K     | 向后（向上）滚动一行             |
| d, ^d                        | 向前（向下）滚动半个屏幕         |
| u, ^u                        | 向后（向上）滚动半个屏幕         |
| g                            | 转到man page的开头               |
| G                            | 转到man page的末尾               |
| q                            | 退出man，并返回到命令shell提示符 |

#### 6.3.5、man搜索

/KEYWORD:

- 以KEYWORD指定的字符串为关键字，从当前位置向文件尾部搜索；不区分字符大小写；
- n: 下一个
- N：上一个

?KEYWORD:

- 以KEYWORD指定的字符串为关键字，从当前位置向文件首部搜索；不区分字符大小写；
- n: 跟搜索命令同方向，下一个
- N：跟搜索命令反方向，上一个

在执行搜素时，string允许使用正则表达式语法。简单的文本（如passwd）按照预期工作，正则表达式使用元字符（如\$、*、.和^）进行更复杂的模式匹配。因此，搜索包含程序表达式的元字符的字符串（如man$$$）可能会产生意外的结果。

查询/etc/issue用法

## 7、info

man常用于命令参考 ，GNU工具info适合通用文档参考，没有参数,列出所有的页面，info 页面的结构就像一个网站，每一页分为“节点”，链接节点之前 *

- info [ 命令 ]

### 7.1、导航info页

|方向键，PgUp，PgDn |导航|
| ---- | -------------- |
|Tab键 |移动到下一个链接|
|d |显示主题目录|
|Home |显示主题首部|
|Enter进入| 选定链接|
|n/p/u/l |进入下/前/上一层/最后一个链接|
|s |文字 文本搜索|
|q |退出 info|

## 8、通过本地文档获取帮助

System->help（centos6）

Applications -> documentation->help（centos7）

- 提供的官方使用指南和发行注记

/usr/share/doc目录

- 多数安装了的软件包的子目录,包括了这些软件的相关原理说明
- 常见文档：README INSTALL CHANGES
- 不适合其它地方的文档的位置
  配置文件范例
  HTML/PDF/PS 格式的文档
  授权书详情

## 9、通过在线文档获取帮助

第三方应用官方文档

- http://httpd.apache.org
- http://www.nginx.org
- https://mariadb.com/kb/en
- https://dev.mysql.com/doc/
- http://tomcat.apache.org
- http://www.python.org

通过发行版官方的文档光盘或网站可以获得

- 安装指南、部署指南、虚拟化指南等
- 红帽知识库和官方在线文档
  http://kbase.redhat.com
  http://www.redhat.com/docs
  http://access.redhat.com
  https://help.ubuntu.com/lts/serverguide/index.html

## 10、网站和搜索

- http://tldp.org

- http://www.slideshare.net

- http://www.google.com

  - Openstack filetype:pdf

  - rhca site:redhat.com/docs

