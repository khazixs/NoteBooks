## 定义

**binlog是记录所有数据库表结构变更（例如CREATE、ALTER TABLE…）以及表数据修改（INSERT、UPDATE、DELETE…）的二进制日志。**

**binlog不会记录SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改，但你可以通过查询通用日志来查看MySQL执行过的所有语句。**

ps：如果update操作没有造成数据变化，也是会记入binlog



binlog包括：

1. 索引文件（文件名后缀为.index）用于记录哪些日志文件正在被使用
2. 日志文件（文件名后缀为.00000*）记录数据库所有的DDL（数据库定义语言）和DML(数据库操纵语言，除了数据查询语句)语句事件。



假设my.cnf中有这么三条配置

log_bin：on 打开binlog日志

log_bin_basename：bin文件路径及名前缀（/var/log/mysql/mysql-bin）

log_bin_index：bin文件index（/var/log/mysql/mysql-bin.index）

那么你会在文件目录/var/log/mysql/下面发现两个文件mysql-bin.000001和mysql-bin.index。

mysql-bin.index就是我们所说的索引文件，打开瞅瞅，内容是下面这样,记录哪些文件是日志文件。

./mysql-bin.000001

那么说到日志文件。在innodb里其实又可以分为两部分，一部分在缓存中，一部分在磁盘上。这里业内有一个词叫做**刷盘**，就是指将缓存中的日志刷到磁盘上。跟**刷盘**有关的参数有两个个:sync_binlog和binlog_cache_size。这两个参数作用如下

binlog_cache_size: 二进制日志缓存部分的大小，默认值32k

sync_binlog=[N]: 表示写缓冲多少次，刷一次盘,默认值为0



## 作用

**恢复**：这里网上有大把的文章指导你，如何利用binlog日志恢复数据库数据。如果你真的觉得自己很有时间，就自己去创建个库，然后删了，再去恢复一下数据，练练手吧。

**复制**: 如图所示（图片不是自己画的，偷懒了）

![img](http://5b0988e595225.cdn.sohucs.com/images/20181115/624b1123a4a44df28734aa8a2115e24b.jpeg)

主库有一个log dump线程，将binlog传给从库

从库有两个线程，一个I/O线程，一个SQL线程，I/O线程读取主库传过来的binlog内容并写入到relay log,SQL线程从relay log里面读取内容，写入从库的数据库。

**审计**：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击。



## binglog常见格式

![img](http://5b0988e595225.cdn.sohucs.com/images/20181115/046a517fe5dd4316a64401bfa52e85f4.jpeg)

## 查看binlog

binlog本身是一类二进制文件。二进制文件更省空间，写入速度更快，是无法直接打开来查看的。

因此mysql提供了命令mysqlbinlog进行查看。

一般的statement格式的二进制文件，用下面命令就可以

mysqlbinlog   mysql-bin.000001

如果是row格式，加上-v或者-vv参数就行，如

mysqlbinlog   -vv   mysql-bin.000001

## 删除binlog

删binlog的方法很多，有三种是常见的

(1) 使用reset master,该命令将会删除所有日志，并让日志文件重新从000001开始。

(2) 使用命令

PURGE{ BINARY| MASTER} LOGS{ TO'log_name'| BEFOREdatetime_expr }

例如

purgemasterlogsto"binlog_name.00000X"

将会清空00000X之前的所有日志文件.

(3) 使用expire_logs_days=N选项指定过了多少天日志自动过期清空。



## binlog常见参数

![img](http://5b0988e595225.cdn.sohucs.com/images/20181115/6badf9e940ce4077b1c03cc9aa85c114.jpeg)