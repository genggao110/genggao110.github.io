---
layout:      post
title:       "Linux基础知识概括"
subtitle:    " \"basic command\""
date:        2019-05-01 12:50:20
author:      "Ming"
catalog: true
header-img:  "img/home-bg-o.jpg"
tags:
    - Linux
    - Command
---

> Look up at the stars, not down at your feet.

#### 1. 权限分配

当我们使用ls -l 命令显示文件或者目录的详细信息具有如下格式：

```
root@ubuntu:/home/xtwy# ls -l
total 48
drwxr-xr-x 2 xtwy xtwy 4096 2015-08-20 23:31 Desktop
drwxr-xr-x 2 xtwy xtwy 4096 2015-08-20 23:31 Documents
drwxr-xr-x 2 xtwy xtwy 4096 2015-08-20 23:31 Downloads
-rw-r--r-- 1 xtwy xtwy  179 2015-08-20 21:53 examples.desktop
-rw-r--r-- 1 root root   30 2015-08-22 17:28 hello1.txt
-rw-r--r-- 1 root root   48 2015-08-22 17:29 hello.txt
drwxr-xr-x 3 root root 4096 2015-08-22 16:51 literature
```

对ls -l 显示的内容进行分解，首先来看前半部分：

![Linux文件权限](https://ws1.sinaimg.cn/large/005CDUpdly1g24hmuw9wtj30ed09e74n.jpg)

首先是文件类型，-表示文本文件，d表示目录，除此之外还有下列几种文件(不常见)：


codeFile Type | Description
---|---
 -  | Standard file
 d  | Stabdard directory
 \|  | Symbolic link(a shortcut to another file)
 s  | Socket(a file designed to send and receive data over a network)
 c  | Character device(a hardware device, ususally found in /dev)
 b  | Block device(a hardware device driver, usually found in /dev)

后面紧跟着的是用户权限、组权限及其他权限，其中r表示读权限，w表示写权限，x表示可执行权限。

再后面的数字表示的是链接数，紧跟着是文件或目录的所属者，所属用户组，文件大小(字节数)，文件最后访问时间，文件名。

**修改文件或目录权限**

```
//1. 增加权限
root@ubuntu:/home/xtwy# chmod a+w hello1.txt //chmod命令，a表示所有，包括用户、组及其他用户都增加写权限，不加a表示作用于当前用户

//2. 减小权限
root@ubuntu:/home/xtwy# chmod a-w hello1.txt //减小权限，用减号表示

//3. 灵活设置权限，采用数字方式。读、写、运行三项权限可以用数字表示，就是r = 4,w =2, x = 1
root@ubuntu:/home/xtwy# chmod 611 hello1.txt //用户具有读写权限，用户组及其他用户具有执行权限，无读写权限

//4.改变用户
root@ubuntu:/home/xtwy# chown xtwy hello1.txt //将root拥有改为xtwy用户拥有

//5.改变用户组
xtwy@ubuntu:~$ chgrp xtwy hello1.txt

```

#### 2. shell终端terminfo命令 tput

tput命令可以更改终端功能，如移动或更改光标，更改文本属性，清除终端屏幕的特定区域等。

##### 光标属性

在shell脚本或命令行中，可以利用tput命令改变光标属性。

```
tput clear    # 清除屏幕
tput sc       # 记录当前光标的位置
tput rc       # 恢复光标到最后保存位置
tput civis    # 光标不可见
tput cnorm    # 光标可见
tput cup x y  # 光标按设定坐标点移动
```

利用上面参数构建一个终端时钟：

```
#!/bin/bash

for ((i=0;i<10;i++))
do
        tput sc; tput civis                     # 记录光标位置,及隐藏光标
        echo -ne $(date +'%Y-%m-%d %H:%M:%S')   # 显示时间
        sleep 1
        tput rc                                 # 恢复光标到记录位置
done

tput el; tput cnorm                             #退出时清理终端,恢复光标显示
```

##### 文本属性

tput可使终端文本加粗、在文本下方添加下划线、更改背景颜色和前景颜色，以及逆转颜色方案等。

```
tput blink    # 文本闪烁
tput bold     # 文本加粗
tput el       # 清除到行尾
tput smso     # 启动突出模式
tput rmso     # 停止突出模式
tput smul     # 下划线模式
tput rmul     # 取消下划线模式
tput sgr0     # 恢复默认终端
tput rev      # 反相终端
tput setb 0   # 设置文本颜色为黑色
tput setf 1   # 设置文本颜色为蓝色
```

颜色代号为：

```
0：黑色
1：蓝色
2：绿色
3：青色
4：红色
5：洋红色
6：黄色
7：白色
```

为之前终端时钟添加变换颜色和闪烁功能：

```
#!/bin/bash

for ((i=0;i<8;i++))
do
        tput sc; tput civis                     # 记录光标位置,及隐藏光标
        tput blink; tput setf $i                # 文本闪烁,更改文本颜色
        echo -ne $(date +'%Y-%m-%d %H:%M:%S')   # 显示时间
        sleep 1
        tput rc                                 # 恢复光标到记录位置
done

tput el; tput cnorm                         #退出时清理终端,恢复光标显示
```

#### 3.Shell脚本的入门

[Shell的入门](https://www.cnblogs.com/jxhd1/p/6274868.html)


##### shell命令行参数解析工具: getopts

在shell脚本中，对于简单的参数，我们使用 $1 $2来处理即可，具体如下：

```
#!/bin/bash

SOFT_DIR=$1
MAVEN_DIR=$2
echo $SOFT_DIR
echo $MAVEN_DIR
-----------------
$ sh test.sh /home/soft /home/soft/maven
/home/soft
/home/soft/maven
```

但是，如果你的脚本参数非常多，那使用上面的这种方式就非常不合适，你无法清除地记得每个位置对应的是什么参数。所有，我们可以使用bash内置的 **getopts** ,下面给出一个简单的案例。

```
#!/bin/bash

usage() {
    echo "Usage:"
    echo "  test.sh [-j JAVA_DIR] [-m MAVEN_DIR]"
    echo "Description:"
    echo "    JAVA_DIR, the path of java."
    echo "    MAVEN_DIR, the path of maven."
    exit -1
}

upload="false"

while getopts 'h:j:m:u' OPT; do
    case $OPT in
        j) JAVA_DIR="$OPTARG";;
        m) MAVEN_DIR="$OPTARG";;
        u) upload="true";;
        h) usage;;
        ?) usage;;
    esac
done

echo $JAVA_DIR
echo $MAVEN_DIR
echo $upload
---------------------------
$ sh test.sh -j /home/soft/java -m /home/soft/maven
/home/soft/java
/home/soft/maven
false

$ sh test.sh -j /home/soft/java -m /home/soft/maven -u
/home/soft/java
/home/soft/maven
true

$ sh test.sh -h   
test.sh: option requires an argument -- h
Usage:
  test.sh [-j JAVA_DIR] [-m MAVEN_DIR]
Description:
    JAVA_DIR, the path of java.
    MAVEN_DIR, the path of maven.
```

**getopts**后面跟的字符串就是参数列表，每个字母代表一个选项，如果字母后面跟一个：就表示这个选项还会有一个值，比如上面例子中对应的 -j /home/soft/java 和 -m /home/soft/maven 。而getopts字符串没有跟随：的字母就是开关型选项，不需要指定值，等同于true/false,只要带上了这个参数就是true。

**getopts**识别出各个选项之后，就可以配合case进行操作。操作中，有两个“常量”，一个是OPTARG,用来获取当前选项的值；另一个就是OPTIND，表示当前选项在参数列表中的位移。case的最后一项是 ？,用来识别非法的选项，进行相应的操作，我们的脚本中输出了帮助信息。

当选项参数识别完成之后，我们就能识别剩余的参数了，我们可以使用shift进行位移，抹去选项参数。

```
#!/bin/bash

usage() {
    echo "Usage:"
    echo "  test.sh [-j JAVA_DIR] [-m MAVEN_DIR]"
    echo "Description:"
    echo "    JAVA_DIR, the path of java."
    echo "    MAVEN_DIR, the path of maven."
    exit -1
}

upload="false"

echo $OPTIND

while getopts 'j:m:u' OPT; do
    case $OPT in
        j) JAVA_DIR="$OPTARG";;
        m) MAVEN_DIR="$OPTARG";;
        u) upload="true";;
        h) usage;;
        ?) usage;;
    esac
done

echo $OPTIND
shift $(($OPTIND - 1))
echo $1

---------------
$ sh test.sh -j /home/soft/java -m /home/soft/maven otherargs
1
5
otherargs

sh test.sh -j /home/soft/java -m /home/soft/maven -u otherargs
1
6
otherargs
```

在上面的脚本中，我们位移的长度等于case循环结束后的OPTIND - 1,OPTIND 的初始值为1，当选项参数处理结束后，其指向剩余参数的第一个。getopts在处理参数时，处理带值的选项参数时，OPTIND加2；处理开关型变量时，OPTIND加1.

#### 4.Linux中grep的用法

grep作为Linux中最为常用的三大文本(awk,sed,grep)之一，了解其具体用法很有必要。grep的常用格式为： grep  [选项]  “模式” [文件]

##### 4.1 常用选项

```
-E : 开启扩展(Extend)的正则表达式
-i : 忽略大小写
-v : 反过来，只打印没匹配的，而匹配的反而不打印
-n : 显示行号
-w : 被匹配的文本只能是单词，而不能是单词中的某一部分
-c : 显示总共有多少行被匹配到了，而不显示被匹配到的内容(-cv同时使用是显示有多少行没有被匹配到)
-o : 只显示被模式匹配到的字符串
--color : 将匹配到的内容以颜色高亮显示
-A n : 显示匹配到的字符串所在的行及其后n行，after
-B n : 显示匹配到的字符串所在的行及其前n行，before
-C n : 显示匹配到的字符串所在的行及其前后n行，context
```

##### 4.2 模式部分

1. 直接输入要匹配的字符串，可以用fgrep(fast grep)代替来提高查找速度。例如匹配hello.c文件中printf的个数：fgrep -c "printf" hello.c
2. 使用基本正则表达式：

**匹配字符**
```
. : 任意一个字符
[abc] : 表示匹配一个字符，这个字符必须是abc中的一个
[a-zA-Z] : 表示匹配一个字符，这个字符必须是a-z或A-Z这52个字母中的一个
[^123] : 匹配一个字符，这个字符是除了1、2、3以外的所有字符

对于一些常用的字符集，Linux系统定义如下:
[A-Za-z] 相当于 [[:alpha:]]
[0-9] 等价于 [[:digit:]]
[A-Za-z0-9] 相当于 [[:alnum:]]
tab,space等空白字符 [[:space:]
[A-Z] 等价于 [[:upper:]]
[a-z] 等价于 [[:lower:]]
标点符号 [[:punct:]]
```

**匹配次数**
```
\{m,n\} : 匹配其前面出现的字符至少m次，至多n次
\? : 匹配其前面出现的内容0次或1次，等价于\{0,1\}
* : 匹配其前面出现的内容任意次，等价于\{0,\},所以“.*”表示任意字符任意次，即无论什么内容全部匹配。
```

**位置锚定**
```
^ : 锚定行首
$ : 锚定行尾。技巧：“^$”用于匹配空白行
\b 或 \< : 锚定单词的词首。例如“\blike”不会匹配alike,但是会匹配liker
\> :锚定单词的词尾。
\B : 与\b作用相反。
```

**分组及引用**
```
\(string\) : 将string作为一个整体方便后面引用
\1 : 引用第1个左括号及其对应的右括号所匹配的内容
\2 : 引用第2个左括号及其对应的右括号所匹配的内容
\n : 引用第n个左括号及其对应的右括号所匹配的内容

$ grep "^\([[alpha]]\).*\1$" /etc/passwd    # 以相同字母为开始并结尾的行
```

3. 扩展的正则表达式(要加-E选项，或者使用egrep)

**匹配字符**： 与正则表达式一样

**匹配次数**： 

```
* : 和基本正则表达式一样
? : 基本正则表达式为\?, 这里不需要\
{m,n} : 对比基本正则表达式，不需要\
+ : 匹配其前面的字符至少一次，相当于{1,}
```

**位置锚定**：与基本正则表达式一样

**分组及引用**

```
(string) : 对比基本正则表达式，不需要\
\1 ： 一样
\n : 一样
```

**或者**
```
a | b : 匹配a或b，注意a是指 | 的左边的整体，b也同理。
```

==注意1==：默认情况下，正则表达式的匹配模式工作在贪婪模式下，也就是说它会尽可能长地去匹配。例如，一行有一字符串abbcb， 如果搜索内容为“a.*b”那么会直接匹配abbcb这个字符串，而不会只匹配ab或acb。

==注意2==： 所有的正则字符，如 [ 、* 、( 等，若要搜索 * ，而不是想把 * 解释为重复先前字符任意次，可以使用 \* 来转义。

#### 5.系统状态检测命令

应了解快速查看Linux系统运行状态的命令，下面给出与网卡网络、系统内核、系统负载、内存使用情况、当前启用终端数量、历史登录记录、命令执行记录以及救援诊断等相关命令的使用方法。

##### 5.1 ifconfig命令

ifconfig命令用于获取网卡配置与网络状态等信息，格式为“ifconfig [网络设备][参数]”。使用ifconfig命令来查看本机当前的网卡配置与网络状态等信息时，其实主要查看的就是网卡名称、inet参数后面的ip地址、ether参数后面的网卡物理地址，以及RX、TX的接收数据包与发送数据包的个数及累计流量。

```shell
[root@linuxprobe Desktop]# ifconfig
eno16777728: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.10 netmask 255.255.255.0 broadcast 192.168.10.255
        inet6 fe80::20c:29ff:fec4:a409 prefixlen 64 scopeid 0x20<link>
        ether 00:0c:29:5f:c2:90  txqueuelen 1000  (Ethernet)
        RX packets 22  bytes 3081 (3.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 2  bytes 140 (140.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2  bytes 140 (140.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

##### 5.2 uname命令

uname命令用于查看系统内核与系统版本等信息，格式为 “uname [-a]”。在使用uname命令时，一般会固定搭配上 -a 参数来完整地查看当前系统的内核名称、主机名、内核发行版本、节点名、系统时间、硬件名称、硬件平台、处理器类型以及操作系统名称等信息。

```shell
[root@linuxprobe Desktop]# uname -a
Linux linuxprobe.com 3.10.0-123.el7.x86_64 #1 SMP Mon May 5 11:16:57 EDT 2014 x86_64 x86_64 x86_64 GNU/Linux

```

##### 5.3 uptime命令

uptime用于查看系统的负载信息，格式为uptime。它可以显示当前系统时间、系统已经运行时间、启用终端数量以及平均负载等信息(top命令也可以获取这些信息)。平均负载值指的是在最近1分钟、5分钟、15分钟的压力情况；负载值越低越好，尽量不要长期超过1，生产环境中不要超过5.

```shell
[root@linuxprobe Desktop]# uptime
 11:42:11 up  1:45,  2 users,  load average: 0.00, 0.01, 0.05
```

##### 5.4 free命令

free用于显示当前系统中内存的使用量信息，格式为"free [-h]"。(-h命令使得以更人性化的方式输出当前内存的实时使用量信息)。

```shell
[root@linuxprobe Desktop]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.9G       1.2G       759M       9.7M       928K       439M
-/+ buffers/cache:       788M       1.2G
Swap:         2.0G         0B       2.0G

```

##### 5.5 who命令

who用于查看当前登入主机的用户终端信息，格式为“who [参数]”。可以快速显示出所有正在登录本机的用户以及他们正在开启的终端信息。

```shell
[root@linuxprobe Desktop]# who
root     :0           2019-06-05 10:35 (:0)
root     pts/0        2019-06-05 11:50 (:0)

```

##### 5.6 last命令

last命令用于查看本机系统的登录记录，格式为“last [参数]”。

```shell
[root@linuxprobe Desktop]# last
root     pts/0        :0               Wed Jun  5 11:50   still logged in   
root     pts/0        :0               Wed Jun  5 10:49 - 11:50  (01:00)    
root     pts/0        :0               Wed Jun  5 10:35 - 10:36  (00:01)    
root     :0           :0               Wed Jun  5 10:35   still logged in   
(unknown :0           :0               Wed Jun  5 10:34 - 10:35  (00:00)    
linuxpro pts/0        :0               Wed Jun  5 10:30 - 10:31  (00:00)    
linuxpro pts/0        :0               Wed Jun  5 10:03 - 10:18  (00:14)    
linuxpro :0           :0               Wed Jun  5 09:58 - 10:34  (00:36)    
(unknown :0           :0               Wed Jun  5 09:57 - 09:58  (00:01)    
reboot   system boot  3.10.0-123.el7.x Wed Jun  5 17:56 - 12:00  (-5:-56)   
linuxpro :0           :0               Tue Jun  4 21:57 - crash  (19:59)    
(unknown :0           :0               Tue Jun  4 21:55 - 21:57  (00:01)    
reboot   system boot  3.10.0-123.el7.x Wed Jun  5 05:55 - 12:00  (06:04)    
reboot   system boot  3.10.0-123.el7.x Wed Jun  5 05:51 - 21:55  (-7:-55)   

wtmp begins Wed Jun  5 05:51:23 2019

```

##### 5.7 history命令

history命令用于显示历史执行过的命令，格式为“history [-c]”。执行history命令能显示出当前用户在本地计算机中执行过的最近1000条命令记录。(可以自定义/etc/profile文件中的HISTSIZE变量值调整数值大小).在使用history命令时，如果使用-c参数会清空所有的命令历史记录。

```shell
[root@linuxprobe Desktop]# history
    1  man man
    2  echo Linux
    3  echo $SHELL
    4  date
    5  wget http://www.linuxprobe.com/docs/LinuxProbe.pdf
    6  ping www.baidu.com
    7  ifconfig 
    8  ping 223.2.38.105
    9  ps
   10  ps -a
   11  ps aux
   12  top
   13  ifconfig
   14  uname -a
   15  uptime
   16  free -h
   17  who
   18  last
   19  history

```

##### 5.8 sosreport命令

sosreport命令用于收集系统配置及架构信息并输出诊断文档，格式为sosreport。当Linux系统出现故障需要联系技术人员时，大多数时候都要先使用这个命令来简单收集系统的运行状态和服务配置信息。

#### 6 Linux下的两个特殊的文件

下面就简要介绍一下Linux系统下两个特殊的文件——/dev/null和/dev/zero。

##### 6.1 /dev/null文件

在类Unix系统中，/dev/null，或称为空设备，是一个特殊的设备文件，它丢弃一切写入其中的数据(但是报告写入操作成功),读取它则会立即得到一个EOF。(行话称之为位桶(bit bucket)或者黑洞(black hole)。空设备通常被用于丢弃不需要的输出流，或作为用于输入流的空文件)

**典型用法1**： 有时候当我们不想看到有任何输出时，例如下面(同时禁止标准输出和标准错误的输出)：

```shell
cat $filename 2> /dev/null > /dev/null

等同于
cat $filename &> /dev/null
```
分析：
- 如何"$filename"不存在，将不会有任何错误信息提示
- 如何"$filename"存在，文件的内容不会打印到标准输出

因此，上述的代码根本不会输出任何信息，当只想测试命令的退出码而不想有任何输出时非常有用。之后，我们可以用 echo $? 来插件上条命令的退出码： 0为命令正常执行，1-255为有出错。

**典型用法2**： 有时候，我们需要删除一些文件的内容而不删除文件本身时，例如当删除cookie并且不再使用cookie时：

```shell
if [ -f ~/.netscape/cookies ]
then
rm -f ~/.netscape/cookies
fi
# 删除后添加软链接
ln -s /dev/null ~/.netscape/cookies 
```

##### 6.2 /dev/zero的日常使用

像/dev/null一样，/dev/zero也是一个伪文件，但它实际上产生连续不断的null的流(二进制的流)。写入它的输出会丢失不见，/dev/zero主要的作用是用来创建一个指定长度用于初始化的空文件，像临时交换文件。

```shell
# 创建一个560MB的数据块
[root@linuxprobe ~]# dd if=/dev/zero of=560_file count=1 bs=560M
1+0 records in
1+0 records out
587202560 bytes (587 MB) copied, 27.1755 s, 21.6 MB/s
```

/dev/zero的另一个应用是为特定的目的而用零去填充一个指定大小的文件，如挂载一个文件系统到环回设备或者安全地删除一个文件。

#### 7.







