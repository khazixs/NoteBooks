## ctrl+c 和 ctrl+z 的区别

ctrl+c ：强制中断程序的执行

 ctrl+z ：将任务中断,但是此任务并没有结束,他仍然在进程中他只是维持挂起的状态,用户可以使用fg/bg操作继续前台或后台的任务,fg命令重新启动前台被中断的任务,bg命令把被中断的任务放在后台执行.

## Linux重要目录

![img](https://upload-images.jianshu.io/upload_images/228680-eb5e50607bf72e52.gif?imageMogr2/auto-orient/strip|imageView2/2/w/675/format/webp)

/etc == Editable Text Configuration 配置文件

## Linux命令参考

![ ](https://upload-images.jianshu.io/upload_images/228680-92ad761516389d11.jpg)

## Linux中的inode

inode是索引节点，用于存储“设备或存储设备分区被格式化为系统文件后”的信息，信息包括：**文件大小，属主，归属的用户组，读写权限等**，能通过inode值最快找到文件。

## 硬链接与软连接

硬链接：与一个文件拥有相同索引节点的文件，硬链接的作用在于允许一个文件拥有多个有效路径

软连接：也叫符号链接，是特殊文件的一种，其中包含的是另一个文件的位置信息。

若对一个文件f1创建一个硬链接f2，一个软连接f3，三个文件都能访问到同一个文件，当删除f1时f2仍旧可以访问该文件，但是f3失效，当再次删除f2时刚刚创建的文件被彻底删除。

## 常见文件命令

### cat

cat -s file 查看内容时去掉多余空白行，在源文件中也去除了。

cat -n file 查看时显示行号。

cat > file 用标准输入覆盖内容。 可以在回车后CTRL+D结束输入。

cat >> file 尾部追加填入。

cat -b file 使用-n参数时，所有空行也会显示行号，若忽略掉空行，改用-b就行。

cat file1 file2 > file 将两个文件合并到一个文件file中。

### 查看文件大小

ls -ll(以字节为单位)

ls -lh(以KB、MB为单位)

du 查询文件或文件夹的磁盘使用空间，使用--max-depth=？参数能够指定深入目录层数 ，-h能够显示单位

df  查看一级文件夹大小，使用比例，档案系统及其挂入点，但对文件无能为力，-T查看分区文件系统。

df -sh * 查看当前目录下各个文件及其目录占用空间大小

### cat、tail、more、less、head的区别

cat和more都是显示文件内容，不同的是cat能够连接和追加输入文件内容，more能够根据窗口自动调整分页显示

less也是分页查看工具

head从头开始查看文件 head -n file 从头查看n行

tail从尾开始查看文件内容

## 进程管理命令

ps，top，kill，killall，bg，fg，fg -n  将作业n带到前台

### PS

最基本的查看进程的命令

**参数：**

- -A ：所有的进程均显示出来，与 -e 具有同样的效用；
- -a ： 不显示与终端有关的所有进程；
- -u ：以用户为主的进程状态 ；
- x ：通常与 a 这个参数一起使用，可列出较完整信息

**输出格式规划：**

- l ：较长、较详细的将该PID 的的信息列出；
- j ：工作的格式 (jobs format)
- -f ：做一个更为完整的输出。

**各相关信息的意义为：**

- F 代表这个程序的旗标 (flag)， 4 代表使用者为 superuser；
- S 代表这个程序的状态 (STAT)；
  - R ：该程序目前正在运作，或者是可被运作；
  - S ：该程序目前正在睡眠当中，但可被某些讯号(signal) 唤醒。
  - D：不可唤醒的睡眠状态，通常这个进程可能在等待I/O的情况（ex>打印）。
  - T ：停止状态(stop)，可能是在任务控制（后台暂停）或跟踪状态；
  - Z ：该程序应该已经终止，但无法被删除到内存外，造成 zombie (疆尸) 程序的状态
- UID 代表执行者身份
- PID 进程的ID号！
- PPID 父进程的ID；
- C CPU使用的资源百分比
- PRI指进程的执行优先权(Priority的简写)，其值越小越早被执行；
- NI 这个进程的nice值，其表示进程可被执行的优先级的修正数值。
- ADDR 这个是内核函数，指出该程序在内存的那个部分。如果是个执行 的程序，一般就是『 - 』
- SZ 使用掉的内存大小；
- WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作；
- TTY 登入者的终端机位置；
- TIME 使用掉的 CPU 时间。
- CMD 所下达的指令名称



### 获得占用cpu最高的10个进程

使用组合命令：

ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head

ps aux|head -1;-1表示从获取的进程详细信息中打印第一行即标题：（USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND）

grep -v PID 表示去掉前一个结果中含有PID字段的行（-v表示不显示匹配到字段的行）

sort 命令的 -r 表示逆序排列，-n表示按照数值大小排列，-k +4表示按照第四列的数据进行排列

head默认是显示前10行

## man

man命令能够查询命令的使用手册

例如：man  touch查看touch的作用和其相关参数的作用

### 关于压缩命令

tar 

参数：

x：提取

c：创建文件

f：指定档案文件名或设备名

k：保存已经存在的文件。例如把某个文件还原，在还原的过程中遇到相同的文件，不会进行覆盖。

v：详细报告tar处理的文件信息。

z：用gzip来压缩/解压缩文件，加上该选项后可以将档案文件进行压缩，但还原时也一定要使用该选项进行解压缩。

## 关于网络命令

### hostname

hostname 没有选项，显示主机名字

hostname –d 显示机器所属域名

hostname –f 显示完整的主机名和域名

hostname –i 显示当前机器的 ip 地址

### ping

ping 将数据包发向用户指定地址。当包被接收，目标机器发送返回数据包。ping 主要有两个作用：

- 用来确认网络连接是畅通的。
- 用来查看连接的速度信息。

## ifconfig

查看用户网络配置。它显示当前网络设备配置。对于需要接收或者发送数据错误查找，这个工具极为好用。

## nslookup

nslookup 这个命令在 有 ip 地址时，可以用这个命令来显示主机名，可以找到给定域名的所有 ip 地址。而你必须连接到互联网才能使用这个命令。

## traceroute

可用来查看数据包在提交到远程系统或者网站时候所经过的路由器的 IP 地址、跳数和响应时间。同样你必须链接到互联网才能使用这个命令。

## netstat

netstat -nap | grep port 将会显示使用该端口的应用程序的进程 id

netstat -a or netstat –all 将会显示包括 TCP 和 UDP 的所有连接

netstat –tcp or netstat –t 将会显示 TCP 连接

netstat –udp or netstat –u 将会显示 UDP 连接

netstat -g 将会显示该主机订阅的所有多播网络。

### wget

wget file 下载文件

wget -c file 断点续传

### 查看端口占用情况

lsof -i : 端口号   该命令需要root权限

![img](https://www.runoob.com/wp-content/uploads/2018/09/lsof.png)

```java
lsof -i:8080：查看8080端口占用
lsof abc.txt：显示开启文件abc.txt的进程
lsof -c abc：显示abc进程现在打开的文件
lsof -c -p 1234：列出进程号为1234的进程所打开的文件
lsof -g gid：显示归属gid的进程情况
lsof +d /usr/local/：显示目录下被进程开启的文件
lsof +D /usr/local/：同上，但是会搜索目录下的目录，时间较长
lsof -d 4：显示使用fd为4的进程
lsof -i -U：显示所有打开的端口和UNIX domain文件
```





```java
netstat -tunlp | grep 端口号
```

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名
- -tunlp 看端口的命令

## 僵尸进程

进程状态中F标识为Z的就是僵尸进程，其COMMAND列后边通常有"**<defunct>**",僵尸进程就是指那些应该已经执行完毕，或是应该终止，但是该进程的父进程却无法完整的将该进程结束掉，而造成该进程一直存在内存当中。



造成僵尸进程的原因可能是系统不稳定，因为程序写的不好，或者是用户操作习惯不良等因素，只能kill -9杀掉



改善的话可以找到该进程的父进程，进行追踪好好分析看看哪里需要改进。

## 最常用的信号

1——SIGHUP——启动被终止的进程，可让该PID重新读取自己的配置文件，类似重新启动

9——SIGKILL——代表强制中断一个进程的执行，如果该进程执行一半，那么尚未完成的部分可能会有半成品产生，类似vim会有.filename.swp保留下来

15——SIGTERM——以正常的方式结束进程来终止该进程，由于是正常的终止，所以后续的操作会将它完成，不过如果该进程已经发生了问题，就是无法使用正常的方法来终止时，输入这个信号也是没有用的。



kill -l 可以查看全部信号

kill命令能够帮助我们将信号传递给某个任务

## PRI与NI

PRI表示内核分权动态维护后的优先级值，越小越优先被执行

NI是给予用户影响优先级的手段，PRI(NEW) = PRI(OLD) + NI，但PRI是动态分权的，即左边的不一定就会严格满足

nice值可调整范围是：-20~19

root 可以随意调整他人或自己的nice值，且范围为-20~19

一般用户只能调整自己进程的nice值，范围是0~19

一般用户仅可以将nice值越调越高



调整nice可以使用top命令，输入r，选择要调整的进程号，再输入nice值

或者renice命令 renice nice pid

## strace

用于跟踪进程的状态改变，和动作

```shell
-c 统计每一系统调用的所执行的时间,次数和出错的次数等. 
-d 输出strace关于标准错误的调试信息. 
-f 跟踪由fork调用所产生的子进程. 
-ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号. 
-F 尝试跟踪vfork调用.在-f时,vfork不被跟踪. 
-h 输出简要的帮助信息. 
-i 输出系统调用的入口指针. 
-q 禁止输出关于脱离的消息. 
-r 打印出相对时间关于,,每一个系统调用. 
-t 在输出中的每一行前加上时间信息. 
-tt 在输出中的每一行前加上时间信息,微秒级. 
-ttt 微秒级输出,以秒了表示时间. 
-T 显示每一调用所耗的时间. 
-v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出. 
-V 输出strace的版本信息. 
-x 以十六进制形式输出非标准字符串 
-xx 所有字符串以十六进制形式输出. 
-a column 
设置返回值的输出位置.默认 为40. 
-e expr 
指定一个表达式,用来控制如何跟踪.格式如下: 
[qualifier=][!]value1[,value2]... 
qualifier只能是 trace,abbrev,verbose,raw,signal,read,write其中之一.value是用来限定的符号或数字.默认的 qualifier是 trace.感叹号是否定符号.例如: 
-eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open表示跟踪除了open以外的其他调用.有两个特殊的符号 all 和 none. 
注意有些shell使用!来执行历史记录里的命令,所以要使用\\. 
-e trace=set 
只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all. 
-e trace=file 
只跟踪有关文件操作的系统调用. 
-e trace=process 
只跟踪有关进程控制的系统调用. 
-e trace=network 
跟踪与网络有关的所有系统调用. 
-e strace=signal 
跟踪所有与系统信号有关的 系统调用 
-e trace=ipc 
跟踪所有与进程通讯有关的系统调用 
-e abbrev=set 
设定 strace输出的系统调用的结果集.-v 等与 abbrev=none.默认为abbrev=all. 
-e raw=set 
将指 定的系统调用的参数以十六进制显示. 
-e signal=set 
指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号. 
-e read=set 
输出从指定文件中读出 的数据.例如: 
-e read=3,5 
-e write=set 
输出写入到指定文件中的数据. 
-o filename 
将strace的输出写入文件filename 
-p pid 
跟踪指定的进程pid. 
-s strsize 
指定输出的字符串的最大长度.默认为32.文件名一直全部输出. 
-u username 
以username 的UID和GID执行被跟踪的命令
```