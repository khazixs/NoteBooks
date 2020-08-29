## 调整堆内存

-XX:InitialHeapSize=数值byte   === -Xms初始堆内存

-XX:MaxHeapSize=数值byte	 === -Xmx最大堆内存

## jvm系统参数的默认值（可以看默认垃圾回收器）

```shell
java -XX:+PrintFlagsInitial  查看出厂默认值
java -XX:+PrintFlagsFinal  查看修改更新 (= 没有修改过 := 人为修改过)
java -XX:+PrintFlagsFinal -XX:MetaspaceSize=512m  查看系统参数，并修改元空间大小 
java -XX:+PrintCommandLineFlags -version  打印命令行参数(可以看默认垃圾回收器)
```

这里我在windows环境下执行，使用的是ParallelGC

## jinfo

用来查看正在运行的 java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息。

```shell
no option   输出全部的参数和系统属性

-flag  name  输出对应名称的参数

-flag [+|-]name  开启或者关闭对应名称的参数

-flag name=value  设定对应名称的参数

-flags  输出全部的参数

-sysprops  输出系统属性
```

