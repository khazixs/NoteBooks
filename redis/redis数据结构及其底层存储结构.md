## Redis数据结构

![redis数据结构](.\img\redis数据结构.png)

## RedisObject

redis中每个键值对对象都有一层封装即redisObject，这个对象作为redis的主要数据结构

```c
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 对象最后一次被访问的时间
    unsigned lru:REDIS_LRU_BITS; /* lru time (relative to server.lruclock) */

    // 引用计数
    int refcount;

    // 指向实际值的指针
    void *ptr;

} robj;
```

其中通过type标识其基本数据结构类型，编码就指定其底层存储结构，最后一次被访问时间服务于redis的键淘汰策略，引用计数指示该对象是否可以被回收。

![《Redis设计与实现第二版》61页](https://img2018.cnblogs.com/blog/1116398/201907/1116398-20190711211450874-951008469.png)

![《Redis设计与实现第二版》62页](https://img2018.cnblogs.com/blog/1116398/201907/1116398-20190711211520206-1535784683.png)

![《Redis设计与实现第二版》63页](https://img2018.cnblogs.com/blog/1116398/201907/1116398-20190711211545977-842176016.png)

### String

#### int

如果一个字符串对象保存的是整数值，并且这个整数值可以用 long 类型标识，那么字符串对象会讲整数值保存在 ptr 属性中，并将 encoding 设置为 int。

#### embstr

当字符串长度小于一定数值时，使用这个编码格式（redis-server 3.2 之前是39字节，之后是44字节）

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530075632624-696981013.png)

embstr与raw都使用redisObject和sds保存数据，区别在于，**embstr的使用只分配一次内存空间（因此redisObject和sds是连续的），而raw需要分配两次内存空间（分别为redisObject和sds分配空间）。**因此与raw相比，embstr的好处在于创建时少分配一次空间，删除时少释放一次空间，以及对象的所有数据连在一起，寻找方便。而embstr的坏处也很明显，如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间，因此redis中的**embstr实现为只读**。

#### raw

如果字符串对象保存的是一个字符串值，并且这个字符串的长度大于 39字节（redis-server 3.2 之前是39字节，之后是44字节），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串值，并将对象的编码设置为 raw。

SDS是redis自定义的存储动态字符串的结构

```c
struct sdshdr {
    int len;
    int free;
    char buf[];
};
```

使用strlen key_name 可以查看字符串长度。

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530075617307-666390009.png)

#### 对于浮点数类型

Redis中对于浮点数类型也是作为字符串保存的，在需要的时候再将其转换成浮点数类型。

#### 编码转换

当 int 编码保存的值不再是整数，或大小超过了long的范围时，自动转化为raw。

### List

可以在头部和尾部插入元素，其底层实际上是个链表结构

例如执行后两种编码的存储如下：

```c
rpush l 1 three 5
```

#### 压缩链表ziplist

ziplist是使用连续的内存存储信息，减少了内存的使用。

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530110113859-1999704925.png)



#### 双端链表linkedlist

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530110135961-1817773967.png)



#### 编码转换

当同时满足下面两个条件时，使用ziplist（压缩列表）编码：

　　1、列表保存元素个数小于512个

　　2、每个元素长度小于64字节

　　不能满足这两个条件的时候使用 linkedlist 编码。

　　上面两个条件可以在redis.conf 配置文件中的 list-max-ziplist-value选项和 list-max-ziplist-entries 选项进行配置。

### Hash

#### ziplist

当使用ziplist，也就是压缩列表作为底层实现时，新增的键值对是保存到压缩列表的表尾。

例如：执行如下命令

```java
hset profile name ``"Tom"
hset profile age 25
hset profile career ``"Programmer"
```

如果使用ziplist，profile 存储如下：

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530122525661-752750675.png)

压缩列表是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，相对于字典数据结构，压缩列表用于元素个数少、元素长度小的场景。其优势在于集中存储，节省空间。

#### hashtable

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530122608271-980707096.png)

hashtable 编码的哈希表对象底层使用**字典数据结构**，哈希对象中的每个键值对都使用一个字典键值对。

#### 编码转换

和上面列表对象使用 ziplist 编码一样，当同时满足下面两个条件时，使用ziplist（压缩列表）编码：

　　1、列表保存元素个数小于512个

　　2、每个元素长度小于64字节

　　不能满足这两个条件的时候使用 hashtable 编码。**第一个条件可以通过配置文件中的 set-max-intset-entries 进行修改。**

### Set

set是string类型（整数也会转换成string类型进行存储）的无序集合

#### intset

底层使用整数集合作为底层实现，集合对象包含的所有元素都被保存在整数集合中。

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530131318187-33688989.png)

#### hashtable

　hashtable 编码的集合对象使用 **字典**作为底层实现，**字典的每个键都是一个字符串对象，这里的每个字符串对象就是一个集合中的元素，而字典的值则全部设置为 null。**这里可以类比Java集合中HashSet 集合的实现，HashSet 集合是由 HashMap 来实现的，集合中的元素就是 HashMap 的key，而 HashMap 的值都设为 null。

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530131811531-167113910.png)



#### 编码转换

当集合同时满足以下两个条件时，使用 intset 编码：

　　1、集合对象中所有元素都是整数

　　2、集合对象所有元素数量不超过512

　　不能满足这两个条件的就使用 hashtable 编码。**第二个条件可以通过配置文件的 set-max-intset-entries 进行配置。**

### ZSet

有序集合对象是有序的。与列表使用索引下标作为排序依据不同，有序集合为每个元素设置一个分数（score）作为排序依据。

#### ziplist

ziplist 编码的有序集合对象使用**压缩列表**作为底层实现，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员，第二个节点保存元素的分值。并且压缩列表内的集合元素按分值从小到大的顺序进行排列，小的放置在靠近表头的位置，大的放置在靠近表尾的位置。

例如：

```c
ZADD price 8.5 apple 5.0 banana 6.0 cherry
```

![img](https://images2018.cnblogs.com/blog/1120165/201805/1120165-20180530210919276-1209186625.png)

#### skiplist

skiplist 编码的有序集合对象使用 zet 结构作为底层实现，一个 zset 结构同时包含一个字典和一个跳跃表：

```c
typedef struct zset{
   //跳跃表
   zskiplist *zsl;
   //字典
   dict *dice;
} zset;
```

字典的键保存元素的值，字典的值则保存元素的分值；跳跃表节点的 object 属性保存元素的成员，跳跃表节点的 score 属性保存元素的分值。

这两种数据结构会通过指针来共享相同元素的成员和分值，所以不会产生重复成员和分值，造成内存的浪费。

##### 为什么两种数据结构一起用？

其实有序集合单独使用字典或跳跃表其中一种数据结构都可以实现，但是这里使用两种数据结构组合起来，原因是假如我们单独使用字典，虽然能以 O(1) 的时间复杂度查找成员的分值，但是因为字典是以无序的方式来保存集合元素，所以每次进行范围操作的时候都要进行排序；假如我们单独使用跳跃表来实现，虽然能执行范围操作，但是查找操作有 O(1)的复杂度变为了O(logN)。因此Redis使用了两种数据结构来共同实现有序集合。

#### 编码转换

当有序集合对象同时满足以下两个条件时，对象使用 ziplist 编码：

　　1、保存的元素数量小于128；

　　2、保存的所有元素长度都小于64字节。

　　不能满足上面两个条件的使用 skiplist 编码。以上两个条件也可以通过Redis配置文件zset-max-ziplist-entries 选项和 zset-max-ziplist-value 进行修改。



## 应用场景：

### string

对于string 数据类型，因为string 类型是二进制安全的，可以用来存放图片，视频等内容，另外由于Redis的高性能读写功能，而string类型的value也可以是数字，可以用作计数器（INCR,DECR），比如分布式环境中统计系统的在线人数，秒杀等。

### hash

对于 hash 数据类型，value 存放的是键值对，比如可以做单点登录存放用户信息。

### list

对于 list 数据类型，可以实现简单的消息队列，另外可以利用lrange命令，做基于redis的分页功能

### set

对于 set 数据类型，由于底层是字典实现的，查找元素特别快，另外set 数据类型不允许重复，利用这两个特性我们可以进行全局去重，比如在用户注册模块，判断用户名是否注册；另外就是利用交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有的喜好等功能。

### zset

对于 zset 数据类型，有序的集合，可以做范围查找，排行榜应用，取 TOP N 操作等。