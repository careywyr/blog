   由于自己当前做的项目过程中间接的利用到了Redis，对其产生了一点兴趣，就买了本Redis入门指南（李子骅编著）看了下，虽然实际上这本书已经买了半年了，==，但最近还是突然痛定思痛决定把这本书好好看看然后写个总结，这里就先写一点基础的知识好了。

 - NoSQL简介
 - Redis相关介绍
 - Redis的安装
 - Redis数据类型

NoSQL简介
-------

说起Redis首先得提一下NoSQL数据库，什么是NoSQL？查询wiki可以知道其定义：

> NoSQL是对不同于传统的关系数据库的数据库管理系统的统称。

 NoSQL的全名叫做Not Only SQL，与我们常用的MySQL，Oracle等数据库不同，它叫做非关系型数据库，那么问题来了，我们为什么需要NoSQL呢？原因可以用四个H表示：
 	
 - High performance -- 高并发读写
 - Huge Storage --海量数据的高效率存储和访问
 - High Scalability && High Availability --高扩展性和高可用性

在我们开发有大数据或高并发的网站时，往往关系型数据库在这个时候的性能会显得捉襟见肘，然而非关系型的数据库在这个时候就能发挥其作用，比如我们本文将会提到的Redis，其数据是存储在内存当中，由于内存的读写速度远快于硬盘，因此其性能当然会具有明显的优势。再者其简单的存储结构使得程序与其之间的交互十分简单，高扩展性和可用性在此也可体现出来。因此其特点十分鲜明，即：

 - 易扩展
 - 灵活的数据模型
 - 大数据量，高性能
 - 高可用

NoSQL数据库的分类大致有四种：

 - 键值（key-value）存储
 - 列存储
 - 文档数据库
 - 图形数据库

我们这里介绍的Redis就是属于其中的键值存储这个分类的数据库。

 
## Redis相关介绍 ##

    
> Redis是一个开源的、高性能的、基于键值对的缓存与操作系统，通过提供多种键值数据类型来适应不同场景下的缓存与存储需求。同时Redis的诸多高层级功能使其可以胜任消息队列、任务队列等不同的角色。

Redis是REmote DIctionary Server（远程字典服务器）的缩写，它以字典结构存储数据，并允许其他应用通过TCP协议读写字典中的内容。目前为止Redis支持的键值数据类型如下：
    
 - 字符串类型
 - 散列类型
 - 列表类型
 - 集合类型
 - 有序集合类型

后文中我们会详细介绍这些数据类型的基本使用方法。

## Redis的安装 ##

本人使用的是centos7系统进行操作的，Redis兼容大部分POSIX系统，包括Linux和OS X等，安装步骤如下：
```shell
    wget http://download.redis.io/redis-stable.tar.gz
    tar xzf redis-stable.tar.gz
    cd redis-stable
    make
```

Redis没有其他外部依赖，所以安装过程很简单，建议在实际运行前使用make test命令来测试Redis是否编译正确，编译后还可以直接执行make install命令来将这些程序复制到/usr/local/bin目录中以便以后执行程序时不用输入完整的路径。
本人之前有在一个Ubuntu系统上安装过一次，执行make的时候报出如下错误：
```shell
[root@node1 redis]# make
    cd src && make all
    make[1]: Entering directory `/usr/local/redis/src'
        CC adlist.o
    在包含自 adlist.c：34 的文件中:
    zmalloc.h:50:31: 错误：jemalloc/jemalloc.h：没有那个文件或目录
    zmalloc.h:55:2: 错误：#error "Newer version of jemalloc required"
    make[1]: *** [adlist.o] 错误 1
    make[1]: Leaving directory `/usr/local/redis/src'
    make: *** [all] 错误 2
```

这个问题可以看一下readme里面的一段话

    Selecting a non-default memory allocator when building Redis is done by setting
    the `MALLOC` environment variable. Redis is compiled and linked against libc
    malloc by default, with the exception of jemalloc being the default on Linux
    systems. This default was picked because jemalloc has proven to have fewer
    fragmentation problems than libc malloc.

    To force compiling against libc malloc, use:

        % make MALLOC=libc

    To compile against jemalloc on Mac OS X systems, use:

        % make MALLOC=jemalloc

意思是关于分配器allocator， 如果有MALLOC 这个 环境变量， 会有用这个环境变量的 去建立Redis，而且libc 并不是默认的分配器， 默认的是 jemalloc, 因为 jemalloc 被证明相对于libc有更少的 fragmentation problems。如果你又没有jemalloc而只有libc就会make 出错。所以加这么一个参数，即执行：

```shell
make MALLOC=libc
```

 再者后来执行make test的时候也很有可能会报一个错误：
```shell
make[1]: *** [test] 错误 1
make[1]: Leaving directory `/usr/local/src/redis-stable/src'
make: *** [test] 错误 2
```
这个意思是说没安装tcl，去官网 http://www.linuxfromscratch.org/blfs/view/cvs/general/tcl.html 上按照说明安装即可。（其实我也不知道这玩意干嘛的，照着官网把命令复制出来执行以下就好了==）

好了，至此，Redis就成功安装了，接下来就是启动它了，直接启动的话运行redis-server即可，贼简单：
```shell
$ redis-server
    3850:C 08 Sep 00:39:53.036 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
    3850:C 08 Sep 00:39:53.036 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=3850, just started
    ......
    3850:M 08 Sep 00:39:53.084 * DB loaded from disk: 0.033 seconds
    3850:M 08 Sep 00:39:53.084 * Ready to accept connections
```
Redis服务器默认端口号为6379，当然你也可以通过--port参数自定义端口号：
```shell
$redis-server --port 6380
```

我们可以通过PING命令来测试连接是否正常,连接正常会返回PONG：

```shell
$redis-cli PING
    PONG
```

停止Redis的命令是：

```shell
 $redis-cli SHUTDOWN
```
Redis收到SHUTDOWN命令后，会先断开所有客户端链接，然后根据配置执行持久化，最后完成退出。
我们这里举例几个Redis常见的基础命令：
获得符合规则的键名列表：KEYS pattern
判断一个键是否存在，存在返回1，否则返回0： EXISTS key
删除键,返回删除的键的个数： DEL key [key ...]
获得键值的数据类型： TYPE key

Redis基本数据类型
-----------

字符串类型
-----
字符串类型是Redis中最基本的数据类型。一个字符串类型键允许存储的数据的最大容量是512MB。
命令：
    赋值与取值：SET key value   GET key
	递增数字： INCR key  ------- 若key不存在，则赋值0再加1，decr同理
	增加指定的整数： INCRBY key increment
	减少指定的整数： DECR key (decrement)
	增加指定浮点数： INCRBYFLOAT key increment
	向尾部追加值： APPEND key value   -------key不存在会创建 返回字符串长度
	获取字符串长度： STRLEN key
	同时获得/设置多个键值： MGET key [key…]  MSET key value [key value …]
	位操作： 
		GETBIT key offset ---获得一个字符串类型键指定位置的二进制位的值（0或1）
		SET BIT key offset value --设置字符串类型键指定位置的二进制位的值，返回值是该位置的旧值。
		BITCOUNT key [start]  [end] ---获得字符串类型键中值是1的二进制位的个数
		BITOP operation destkey key [key …] ---对多个字符串类型键进行位运算，并将结果存储在destkey参数指定的键中。

散列类型
----
散列（hash）的键值也是一种字典结构，其存储了字段和字段值的映射，但字段值只支持字符串，不支持其他数据类型。一个散列类型键可以包含最多2^32-1个字段。
散列类型适合存储对象：使用对象类别和ID构成键名，使用字段表示对象的属性，而字段值则存储属性值。
命令：
	赋值取值：
		HSET key field value
		HGET key field
		HMSET key field value [field value …]
		HGETALL key
	判断字段是否存在
		HEXISTS key field
	当字段不存在时赋值
		HSETNX key field value
	增加数字
		HINCREBY key field increment
	删除字段
		HDEL key field [field …]
	只获取字段名或字段值
		HKEYS key
		HVALS key
	获得字段数量
HLEN key

列表类型
----
列表类型可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素，或者获得列表的某一个片段。
列表类型内部是使用双向链表实现的（通过索引访问元素比较慢），所以向列表两端添加元素的时间复杂度为O(1)，获取越接近两端的元素就越快。
一个列表类型键可以包含至多2∧32-1个字段。
命令：
向列表两端增加元素 
	LPUSH key value [value…]   ---- > 左边增加，返回值表示增加元素后列表的长度     lpushx key value ------key必须存在
	RPUSH key value [value…]         ---- > 右边增加，返回值表示增加元素后列表的长度
	rpushx key value -----key必须存在
从列表两端弹出元素
	LPOP key
	RPOP key
获取列表中元素的个数
	LLEN key
获得列表片段(包含两端)
	LRANGE key start stop 
删除列表中指定的值 返回实际删除的元素个数
	LREM key count value  count为负则从后往前删，若为0则删除所有对应value
获得/设置指定索引的元素值
	LINDEX key index
	LSET key index value
只保留列表指定片段
	LTRIM key start end
向列表中插入元素
	LINSERT 可以BEFORE|AFTER pivot value
将元素从一个列表转到另一个列表
	REOPLPUSH source destination

集合类型
----
集中中的每个元素都是不同的，且没有顺序，一个集合类型（set）键可以存储2∧32-1个字符串。
集中类型的常用操作是向集合中加入或删除元素、判断某个元素是否存在等。集合类型在Redis内部是用值为空的散列表（hash table）实现的，所以这些操作的时间复杂度是O(1)。最方便的是多个集合类型键之间可以进行并集、交集和差集运算。
命令：  
增加/删除元素
	SADD key member [member …]
	SREM key member [member …]
获得集合中的所有元素
	SMEMBERS key
判断元素是否在集合中
	SISMEMBER key member
集合间运算
	SDIFF key [key …] 多个集合执行差集运算 A-B
	SINTER key [key …] 交集运算 A∩B
	SUNION key [key …] 并集运算 A∪B
获得集合中元素个数
	SCARD key
进行集合运算并将结果存储
	SDIFFSTORE destination key [key …]
	SINTERSTORE destination key [key …]
	SUNIONSTORE destination key [key …]
随机获得集合中的元素
	SRANDMEMBER key [count] count>0 count个不重复元素 count<0 |count|个有可能相同的元素
从集合中弹出一个元素
SPOP key

有序集合类型
------
有序集合（sorted set）在集合类型的基础上为集合中的每个元素都关联了一个分数。
有序集合类型和列表类型相似点：
	• 二者都是有序的；
	• 二者都可以获得某一范围的元素；
区别：
	• 列表类型是通过链表实现的，获取靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度会较慢，所以它更加适合实现如“新鲜事”或“日志”这样很少访问中间元素的应用；
	• 有序集合类型是使用散列表和跳跃表实现的，所以即使读取位于中间的数据速度也是很快，时间复杂度O(log(N))；
	• 列表中不能简单的调整某个元素的位置，但是有序集合可以（更改这个元素的分数）；
有序集合要比列表类型更耗费内存；
命令：
增加元素
	ZADD key score member [score member …] (也可用于修改)
获得元素的分数
	ZSCORE key member
获得排名在某个范围的元素列表
	ZRANGE key start stop [WITHSCORES] --按照元素分数从小到大的顺序返回从start到stop之间的所有元素。负数代表从后向前；加上WITHSCORES返回据格式编程元素1，分数1，元素2，分数2.。。；时间复杂度O(log n+m)(n为有序集合的基数，m为返回的元素个数)
	ZREVRANGE key start stop [WITHSCORES] 从大到小
获得指定分数范围的元素
	ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
增加某个元素的分数
	ZINCREBY key increment member
获得集合中元素的数量
	ZCARD key
获得指定分数范围内的元素个数
	ZOUNT key min max
删除一个或多个元素
	ZREM key member [member…]
按照排名范围删除元素
	ZREMRANGEBYRANK key start stop
按照分数范围删除元素
	ZREMRANGEBYSCORE key min max
获得元素的排名
	ZRANK key member 从小到大
	ZREVRANK key member 从大到小
计算有序集合的交集
	ZINTERSTORE destination numkeys key [key…] [WEIGHT weight [weight …]] [AGGREGATE SUM|MIN|MAX]
	计算多个有序集合的交集并将结果存储在destination键中（同样以有序集合类型存储）

以上，为Redis入门的基础知识，希望能有所帮助。
