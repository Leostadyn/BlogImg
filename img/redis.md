# 概述

*Redis*（Remote Dictionary Server )，即远程字典服务

是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

**功能：**

1. 内存存储、持久化
2. 效率高，可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计数器、计时器



**特点：**

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务



# 安装

## Windows下安装

1. 下载地址：https://github.com/MicrosoftArchive/redis/releases/tag/win-3.2.100
2. 解压
3. 开启redis-server.exe，默认端口6379
4. 开启redis-cli.exe，连接
5. 输入ping，返回pong，则连接成功

![image-20230201062421727](D:/OneDrive/%E6%96%87%E6%A1%A3/%E8%87%AA%E6%88%91%E5%AD%A6%E4%B9%A0/redis/assets/image-20230201062421727.png)

Windows使用Redis确实简单，但是官方推荐还是走Linux

## Linux下安装

1. 下载地址：https://redis.io/download

2. 上传服务器，解压 tar -zxvf redis-6.2.5.tar.gz 

3. 基本环境安装

     ```shell
     yum install gcc-c++
     make
     make install
     ```

4. redis 的默认安装路径：usr/local/bin,自定义配置文件

   ```shell
   mkdir myconf
   cp /opt/redis-6.2.5/redis.conf myconf
   ```

5. 设置后台启动，将配置文件的daemonize 改为yes

6. 通过指定配置文件启动redis

   ```shell
   redis-server  myconf/redis.conf 
   ```

7. 启动客户端测试

   ```shell
   redis-cli -p 6379
   ```
   
   ![image-20210804160815362](redis.assets/image-20210804160815362.png)
   
8. 查看进程是否开启

     ```shell
     ps -ef|grep redis
     ```
9. 关闭 shutdown



# 测试性能

**redis-benchmark**官方压力测试工具

| 序号 | 选项                      | 描述                                       | 默认值    |
| :--- | :------------------------ | :----------------------------------------- | :-------- |
| 1    | **-h**                    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**                    | 指定服务器端口                             | 6379      |
| 3    | **-s**                    | 指定服务器 socket                          |           |
| 4    | **-c**                    | 指定并发连接数                             | 50        |
| 5    | **-n**                    | 指定请求数                                 | 10000     |
| 6    | **-d**                    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**                    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**                    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**                    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**                    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv**                 | 以 CSV 格式输出                            |           |
| 12   | ***-l\*（L 的小写字母）** | 生成循环，永久执行测试                     |           |
| 13   | **-t**                    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | ***-I\*（i 的大写字母）** | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

测试：

```bash
#测试100个并发连接，100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```



# 基础知识

1. redis默认有16个数据库

   默认使用的是第0个

   可以使用select进行切换

   ```bash
   127.0.0.1:6379> select 3	#切换数据库
   OK
   127.0.0.1:6379[3]> DBSIZE	#查看DB大小
   (integer) 0
   ```

   ```bash
   127.0.0.1:6379> keys *	#查看所有数据库的key
   1) "key:__rand_int__"
   2) "myhash"
   3) "mylist"
   4) "counter:__rand_int__"
   5) "name"
   ```

2. 清除当前数据库

   ```bash
   127.0.0.1:6379[1]> flushdb
   OK
   ```

   清除所有数据库

   ```bash
   127.0.0.1:6379[1]> flushall
   OK
   ```

3. redis是由C编写

4. redis是单线程，redis是基于内存操作，CPU不是redis的性能瓶颈，而是内存和网络带宽。

**为什么Redis单线程还这么快？**

误区1：高性能的服务器一定是多线程的

误区2：多线程一定比单线程效率高

核心：redis是将所有数据全部放在内存中，所以使用单线程操作效率最高，对于内存系统来说，如果没有上下文切换，那效率就是最高的。



# 数据类型

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作**数据库、缓存和消息中间件**。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。

## Redis-key

```bash
127.0.0.1:6379> set age 1
OK
127.0.0.1:6379> exists age		#查询key是否存在
(integer) 1
127.0.0.1:6379> move age 1		#将key移动到1号数据库
(integer) 1
127.0.0.1:6379> exists age
(integer) 0
##############################################################################################################
127.0.0.1:6379> set age 1
OK
127.0.0.1:6379> expire age 1	#设置过期时间
(integer) 1
127.0.0.1:6379> ttl age			#查看当前key的剩余时间
(integer) -2
127.0.0.1:6379> get age
(nil)
##############################################################################################################
127.0.0.1:6379> set age 1
OK
127.0.0.1:6379> type age		#查看key的数据类型
string
```

遇到不会的命令可以查询官方网站



## String（字符串）

```bash
127.0.0.1:6379> set key1 v1
OK
127.0.0.1:6379> get key1
"v1"
127.0.0.1:6379> APPEND key1 "v2"	#在string后拼接字符串，如果key1不存在，则新建key
(integer) 4
127.0.0.1:6379> get key1
"v1v2"
127.0.0.1:6379> STRLEN key1			#查询字符串长度
(integer) 4
##############################################################################################################
127.0.0.1:6379> set key2 0
OK
127.0.0.1:6379> incr key2			#自增一次
(integer) 1
127.0.0.1:6379> get key2
"1"
127.0.0.1:6379> incr key1			#对value有要求
(error) ERR value is not an integer or out of range
##############################################################################################################
127.0.0.1:6379> decr key2			#自减一次
(integer) 0
127.0.0.1:6379> get key2
"0"
127.0.0.1:6379> decr key2
(integer) -1
127.0.0.1:6379> get key2
"-1"
##############################################################################################################
127.0.0.1:6379> INCRBY key2 10		#增加10
(integer) 9
127.0.0.1:6379> get key2
"9"
##############################################################################################################
127.0.0.1:6379> DECRBY key2 10		#减少10
(integer) -1
127.0.0.1:6379> get key2
"-1"
##############################################################################################################
127.0.0.1:6379> set key3 "hello"	
OK
127.0.0.1:6379> GETRANGE key3 0 3	#划取字符串
"hell"
127.0.0.1:6379> GETRANGE key3 0 -1
"hello"
##############################################################################################################
127.0.0.1:6379> set key4 abcdefg
OK
127.0.0.1:6379> SETRANGE key4 2 MMM	#替换指定范围的字符串
(integer) 7
127.0.0.1:6379> get key4
"abMMMfg"
##############################################################################################################
127.0.0.1:6379> SETEX key5 30 abc	#set with expire
OK
127.0.0.1:6379> ttl key5
(integer) 6
127.0.0.1:6379> get key5
(nil)
##############################################################################################################
127.0.0.1:6379> set key6 123
OK
127.0.0.1:6379> SETNX key6 456		#set if not exists,在分布式锁中常用，存在则失败
(integer) 0
127.0.0.1:6379> get key6
"123"
##############################################################################################################
127.0.0.1:6379> mset k1 1 k2 2 k3 3	#设置复数key
OK
127.0.0.1:6379> keys *
 1) "key4"
 2) "k2"
 3) "key6"
 4) "age"
 5) "key2"
 6) "key1"
 7) "k1"
 8) "k3"
 9) "key3"
10) "name"
127.0.0.1:6379> mget k1 k2 k3		#获取复数key的value
1) "1"
2) (nil)
127.0.0.1:6379> MSETNX k1 a k4 4	#设置复数key时检查是否存在，失败则不执行所有操作，是个原子操作,msetex同理
(integer) 0
127.0.0.1:6379> mget k1 k4			
1) "1"
2) (nil)
##############################################################################################################
127.0.0.1:6379> set user:1 {name:admin,age:3}	#设置user类对象1
OK
127.0.0.1:6379> get user:1
"{name:admin,age:3}"
##############################################################################################################
127.0.0.1:6379> getset db redis		#存在则执行get后再赋予新值，不存在则执行set
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db redis
"redis"
127.0.0.1:6379> getset db mongodb
"redis"
127.0.0.1:6379> get db
"mongodb"
```

String的使用场景：

* 计数器
* 统计多单位的数量 
* 对象缓存存储



## List（列表）

redis中，list可以用作栈、队列、阻塞队列

所有list的命令都是以l开头的

```bash
127.0.0.1:6379> LPUSH list one			#将一个值或多个值插入列表头部
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1		#划取列表值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> RPUSH list right		#将一个值或多个值插入列表尾部		
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
##############################################################################################################
127.0.0.1:6379> LPOP list				#移除列表头部元素
"three"
127.0.0.1:6379> RPOP list				#移除列表尾部元素
"right"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
##############################################################################################################
127.0.0.1:6379> LINDEX list 1			#通过下标获取list中某一个值
"one"
127.0.0.1:6379> LINDEX list 0
"two"
##############################################################################################################
127.0.0.1:6379> LLEN list				#获取当前列表长度
(integer) 2
##############################################################################################################
127.0.0.1:6379> lrem list 1 one			#移除指定值，可设置移除的个数
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
##############################################################################################################
127.0.0.1:6379> LPUSH list one
(integer) 2
127.0.0.1:6379> RPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> LTRIM list 1 2			#截断元素
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "three"
##############################################################################################################
127.0.0.1:6379> RPOPLPUSH list list1	#移除列表最后一个元素，并移动到新的列表
"three"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
127.0.0.1:6379> LRANGE list1 0 -1
1) "three"
##############################################################################################################
127.0.0.1:6379> LSET list 0 one			#替换指定index的值，不存在会报错
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "one"
##############################################################################################################
127.0.0.1:6379> LINSERT list before one O	#将某个值插入到指定列表元素的前后
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
1) "O"
2) "one"
```

小结：

* 实际上是一个链表
* 如果key不存在，插入会创建一个新的链表
* 如果key存在，新增内容
* 如果移除了所有制，空链表也是代表不存在的
* 两边插入和改动效率高，直接操作中间元素效率会低一点



## Set（集合）

set中的值是不能重复的

```bash
127.0.0.1:6379> SADD myset hello				#添加元素到set
(integer) 1
127.0.0.1:6379> SADD myset world !
(integer) 2
127.0.0.1:6379> SMEMBERS myset					#获取set中的元素
1) "!"
2) "world"
3) "hello"
##############################################################################################################
127.0.0.1:6379> SISMEMBER myset hello			#判断某个元素是否在set中
(integer) 1
127.0.0.1:6379> SISMEMBER myset 123
(integer) 0
##############################################################################################################
127.0.0.1:6379> SCARD myset						#获取set集合中的个数
(integer) 3
##############################################################################################################
127.0.0.1:6379> SREM myset !					#移除set中某一个元素
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
##############################################################################################################
127.0.0.1:6379> SRANDMEMBER myset 1				#随机抽取指定个数的元素
1) "world"
127.0.0.1:6379> SRANDMEMBER myset 1
1) "hello"
##############################################################################################################
127.0.0.1:6379> SPOP myset						#随机删除元素
"world"
127.0.0.1:6379> SMEMBERS myset
1) "hello"
##############################################################################################################
127.0.0.1:6379> SADD myset2 set2
(integer) 1
127.0.0.1:6379> SMOVE myset myset2 hello		#将一个元素移动到另一个集合
(integer) 1
127.0.0.1:6379> SMEMBERS myset
(empty array)
127.0.0.1:6379> SMEMBERS myset2
1) "set2"
2) "hello"
##############################################################################################################
127.0.0.1:6379> SADD key1 a
(integer) 1
127.0.0.1:6379> SADD key1 b
(integer) 1
127.0.0.1:6379> SADD key1 c
(integer) 1
127.0.0.1:6379> SADD key2 c d e
(integer) 3
127.0.0.1:6379> SDIFF key1 key2					#得出两个集合的差集
1) "b"
2) "a"
127.0.0.1:6379> SDIFF key2 key1
1) "e"
2) "d"
##############################################################################################################
127.0.0.1:6379> SINTER key1 key2				#做交集
1) "c"
##############################################################################################################
127.0.0.1:6379> SUNION key1 key2				#做并集
1) "e"
2) "a"
3) "c"
4) "b"
5) "d"
```

主要做交集、并集、差集，用于共同关注、推荐好友等



## Hash（哈希）

Map集合，key-map

```bash
127.0.0.1:6379> hset myhash field1 a
(integer) 1
127.0.0.1:6379> hget myhash field1
"a"
127.0.0.1:6379> hmset myhash field1 hello field2 world
OK
127.0.0.1:6379> hmget myhash field1 field2
1) "hello"
2) "world"
127.0.0.1:6379> hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "world"
##############################################################################################################
127.0.0.1:6379> HDEL myhash field1				#删除指定的key
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
##############################################################################################################
127.0.0.1:6379> HLEN myhash						#获取Hash的字段数
(integer) 1
##############################################################################################################
127.0.0.1:6379> HEXISTS myhash field1			#判断某一字段是否存在
(integer) 0
127.0.0.1:6379> HEXISTS myhash field2
(integer) 1
##############################################################################################################
127.0.0.1:6379> HKEYS myhash					#获取所有的字段名
1) "field2"
127.0.0.1:6379> HVALS myhash					#获取所有的值
1) "world"
##############################################################################################################
127.0.0.1:6379> HSET myhash field3 4
(integer) 1
127.0.0.1:6379> HINCRBY myhash field3 3			#使字段值增加
(integer) 7
127.0.0.1:6379> hgetall myhash
1) "field2"
2) "world"
3) "field3"
4) "7"
##############################################################################################################
127.0.0.1:6379> HSETNX myhash field4 !!!		#set if not exist
(integer) 1
127.0.0.1:6379> HSETNX myhash field4 ???
(integer) 0
```

hash主要用来存一些变更的数据，更适合用来存对象，string还是拿来存字符串比较好



## Zset（有序集合）

在set基础上增加了一个值

```bash
127.0.0.1:6379> ZADD myset 1 one
(integer) 1
127.0.0.1:6379> ZADD myset 2 two
(integer) 1
127.0.0.1:6379> ZADD myset 3 three
(integer) 1
127.0.0.1:6379> ZRANGE myset 0 -1
1) "one"
2) "two"
3) "three"
##############################################################################################################
127.0.0.1:6379> ZRANGEBYSCORE myset -inf +inf				#根据score从小到大排序
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> ZREVRANGE myset 0 -1						#逆序
1) "two"
2) "one"
127.0.0.1:6379> ZRANGEBYSCORE myset -inf +inf withscores	#输出带上score
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
##############################################################################################################
127.0.0.1:6379> zrem myset three							#移除某一元素
(integer) 1
127.0.0.1:6379> ZRANGE myset 0 -1
1) "one"
2) "two"
##############################################################################################################
127.0.0.1:6379> ZCARD myset									#获取有序集合中的个数
(integer) 2
##############################################################################################################
127.0.0.1:6379> ZCOUNT myset 1 2							#获取指定区间的成员数量
(integer) 2
```

在set的情景基础上满足做排序的需求，比如工资表排序、排行榜等等



## Geospatial（地理位置）

只有六个命令

```bash
127.0.0.1:6379> GEOADD china:city 116.40 39.90 beijing						#无法添加两极的坐标，值为经度、纬度、名称
(integer) 1
127.0.0.1:6379> GEOADD china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> GEOADD china:city 106.50 29.53 chongqing
(integer) 1
127.0.0.1:6379> GEOADD china:city 114.08 22.52 shenzhen
(integer) 1
127.0.0.1:6379> GEOADD china:city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2
##############################################################################################################
127.0.0.1:6379> GEOPOS china:city beijing									#获取指定城市的经度和纬度
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
##############################################################################################################
127.0.0.1:6379> GEODIST china:city beijing shanghai							#两个位置之间的距离，单位是米
"1067378.7564"
127.0.0.1:6379> GEODIST china:city beijing shanghai km						#单位变为km
"1067.3788"
##############################################################################################################
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km							#在给定坐标自定义半径范围内存在的位置对象
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km
1) "chongqing"
2) "xian"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord				#附带经纬度
1) 1) "chongqing"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) 1) "108.96000176668167114"
      2) "34.25999964418929977"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withdist					#附带距离
1) 1) "chongqing"
   2) "341.9374"
2) 1) "xian"
   2) "483.8340"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km count 1					#指定只显示一个
1) "chongqing"
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1000 km				#中心点改为位置对象
1) "beijing"
2) "xian"
##############################################################################################################
127.0.0.1:6379> GEOHASH china:city beijing chongqing						#将二维的位置对象转换为一维的字符串
1) "wx4fbxxfke0"
2) "wm5xzrybty0"

```

GEO的底层是Zset，所以我们可以用Zset来实现删除位置对象

```bash
127.0.0.1:6379> ZREM china:city beijing
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
5) "shanghai"
```



## Hyperloglog（基数统计）

基数：两个集合里不重复的元素

优点：占用的内存是固定的

```bash
127.0.0.1:6379> PFADD mykey a b c d e f g h i j
(integer) 1
127.0.0.1:6379> PFCOUNT mykey							#统计第一组中的基数数量
(integer) 10
127.0.0.1:6379> PFADD mykey2 i j z x c v b n m
(integer) 1
127.0.0.1:6379> PFCOUNT mykey2							#统计第二组中的基数数量
(integer) 9
127.0.0.1:6379> PFMERGE mykey3 mykey mykey2				#合并第一组和第二组
OK
127.0.0.1:6379> PFCOUNT mykey3							#统计第三组中的基数数量
(integer) 15
```

有0.81%的错误率，如果容许错误，可以使用Hyperloglog，不允许还是set



## Bitmaps（位图）

操作二进制位来进行记录，只有0和1两个状态

```bash
127.0.0.1:6379> SETBIT sign 0 1
(integer) 0
127.0.0.1:6379> SETBIT sign 1 0
(integer) 0
127.0.0.1:6379> SETBIT sign 2 0
(integer) 0
127.0.0.1:6379> SETBIT sign 3 1
(integer) 0
127.0.0.1:6379> SETBIT sign 4 1
(integer) 0
127.0.0.1:6379> SETBIT sign 5 0 
(integer) 0
127.0.0.1:6379> SETBIT sign 6 0
(integer) 0
##############################################################################################################
127.0.0.1:6379> GETBIT sign 3
(integer) 1
127.0.0.1:6379> GETBIT sign 1
(integer) 0
127.0.0.1:6379> GETBIT sign 5
(integer) 0
127.0.0.1:6379> GETBIT sign 4
(integer) 1
##############################################################################################################
127.0.0.1:6379> BITCOUNT sign					#统计
(integer) 3
```



# 事务

本质：一组命令的集合，所有命令会序列化，会在执行的过程中按照顺序执行

Redis单条命令是原子性的，但是事务是不保证原子性的，只保证一次性、顺序性、排他性！

Redis事务中没有隔离级别的概念

所有命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行！

Redis的事务：

* 开启事务（multi）
* 命令入队（...）
* 执行事务（exec）

正常执行：

```bash
127.0.0.1:6379> multi					#开启事务
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> exec				#执行事务
1) OK
2) OK
3) "v2"
4) OK
```

放弃执行：

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> discard				#取消事务
OK
127.0.0.1:6379> get k4					#队列中的命令都不会执行
(nil)
```

编译型异常：

​		代码有问题，即命令有错，事务中所有的命令都不会被执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> getset k3				#错误命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> set k5 v5
QUEUED
127.0.0.1:6379(TX)> exec					#没执行
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5
(nil)
```

运行时异常：

​		队列中存在语法错误，在执行命令的时候，错误的命令会抛出异常，其余正常执行

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> incr k1				#失败命令
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> get k3
QUEUED
127.0.0.1:6379(TX)> exec
1) (error) ERR value is not an integer or out of range	#失败
2) OK
3) OK
4) "v3"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
"v3"
```

悲观锁：无论做什么都要加锁

乐观锁：更新数据时候比较version



Redis监控测试：

​		正常执行：

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money					#监视money对象
OK
127.0.0.1:6379> multi					
OK
127.0.0.1:6379(TX)> DECRBY money 20
QUEUED
127.0.0.1:6379(TX)> INCRBY out 20
QUEUED
127.0.0.1:6379(TX)> EXEC
1) (integer) 80
2) (integer) 20
```

​		多线程情况下的争用：

```bash
127.0.0.1:6379> watch money			#监视money对象
OK
127.0.0.1:6379> multi					
OK
127.0.0.1:6379(TX)> DECRBY money 20
QUEUED
127.0.0.1:6379(TX)> INCRBY out 20
QUEUED
127.0.0.1:6379(TX)> EXEC			#在执行之前，另一个事务修改money，所以执行失败
(nil)
```

1. 如果发现事务执行失败，就先**解锁**（unwatch）
2. 获取最新的值，再次**监视**（watch key）
3. 在**执行**（exec）时比对监视的 值是否发生了变化，如果没有变化那么可以执行成功，如果变了就执行失败



# Jedis

使用Java操作Redis，Jedis是中间件

1. 导入对应的依赖

   ```xml
   <dependencies>
       <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
       <dependency>
           <groupId>redis.clients</groupId>
           <artifactId>jedis</artifactId>
           <version>3.6.3</version>
       </dependency>
   
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
           <version>1.2.75</version>
       </dependency>
   </dependencies>
   ```

2. 编码测试：

   1. 连接数据库
   2. 操作命令
   3. 断开连接

   ```java
   public class TestPing {
       public static void main(String[] args) {
           Jedis jedis=new Jedis("127.0.0.1",6379);
           System.out.println(jedis.ping());
       }
   }
   ```

   输出：

   ![image-20210806104405595](redis.assets/image-20210806104405595.png)

## 常用API

### String

```java
public class TestString {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        jedis.flushDB();
        System.out.println("===========增加数据===========");
        System.out.println(jedis.set("key1", "value1"));
        System.out.println(jedis.set("key2", "value2"));
        System.out.println(jedis.set("key3", "value3"));
        System.out.println("删除键key2:" + jedis.del("key2"));
        System.out.println("获取键key2:" + jedis.get("key2"));
        System.out.println("修改key1:" + jedis.set("key1", "value1Changed"));
        System.out.println("获取key1的值：" + jedis.get("key1"));
        System.out.println("在key3后面加入值：" + jedis.append("key3", "End"));
        System.out.println("key3的值：" + jedis.get("key3"));
        System.out.println("增加多个键值对：" + jedis.mset("key01", "value01", "key02", "value02", "key03", "value03"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03", "key04"));
        System.out.println("删除多个键值对：" + jedis.del("key01", "key02"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));

        jedis.flushDB();
        System.out.println("===========新增键值对防止覆盖原先值==============");
        System.out.println(jedis.setnx("key1", "value1"));
        System.out.println(jedis.setnx("key2", "value2"));
        System.out.println(jedis.setnx("key2", "value2-new"));
        System.out.println(jedis.get("key1"));
        System.out.println(jedis.get("key2"));

        System.out.println("===========新增键值对并设置有效时间=============");
        System.out.println(jedis.setex("key3", 2, "value3"));
        System.out.println(jedis.get("key3"));
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(jedis.get("key3"));

        System.out.println("===========获取原值，更新为新值==========");
        System.out.println(jedis.getSet("key2", "key2GetSet"));
        System.out.println(jedis.get("key2"));

        System.out.println("获得key2的值的字串：" + jedis.getrange("key2", 2, 4));
    }
}
```

### List

```java
public class TestList {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("===========添加一个list===========");
        jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("collections", "HashSet");
        jedis.lpush("collections", "TreeSet");
        jedis.lpush("collections", "TreeMap");
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));//-1代表倒数第一个元素，-2代表倒数第二个元素,end为-1表示查询全部
        System.out.println("collections区间0-3的元素：" + jedis.lrange("collections", 0, 3));
        System.out.println("===============================");
        // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
        System.out.println("删除指定元素个数：" + jedis.lrem("collections", 2, "HashMap"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("删除下表0-3区间之外的元素：" + jedis.ltrim("collections", 0, 3));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（左端）：" + jedis.lpop("collections"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections添加元素，从列表右端，与lpush相对应：" + jedis.rpush("collections", "EnumMap"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（右端）：" + jedis.rpop("collections"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("修改collections指定下标1的内容：" + jedis.lset("collections", 1, "LinkedArrayList"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("===============================");
        System.out.println("collections的长度：" + jedis.llen("collections"));
        System.out.println("获取collections下标为2的元素：" + jedis.lindex("collections", 2));
        System.out.println("===============================");
        jedis.lpush("sortedList", "3", "6", "2", "0", "7", "4");
        System.out.println("sortedList排序前：" + jedis.lrange("sortedList", 0, -1));
        System.out.println(jedis.sort("sortedList"));
        System.out.println("sortedList排序后：" + jedis.lrange("sortedList", 0, -1));
    }
}
```

### Set

```Java
public class TestSet {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("============向集合中添加元素（不重复）============");
        System.out.println(jedis.sadd("eleSet", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("删除一个元素e0：" + jedis.srem("eleSet", "e0"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("删除两个元素e7和e6：" + jedis.srem("eleSet", "e7", "e6"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
        System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("eleSet中包含元素的个数：" + jedis.scard("eleSet"));
        System.out.println("e3是否在eleSet中：" + jedis.sismember("eleSet", "e3"));
        System.out.println("e1是否在eleSet中：" + jedis.sismember("eleSet", "e1"));
        System.out.println("e1是否在eleSet中：" + jedis.sismember("eleSet", "e5"));
        System.out.println("=================================");
        System.out.println(jedis.sadd("eleSet1", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet2", "e1", "e2", "e4", "e3", "e0", "e8"));
        System.out.println("将eleSet1中删除e1并存入eleSet3中：" + jedis.smove("eleSet1", "eleSet3", "e1"));//移到集合元素
        System.out.println("将eleSet1中删除e2并存入eleSet3中：" + jedis.smove("eleSet1", "eleSet3", "e2"));
        System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
        System.out.println("eleSet3中的元素：" + jedis.smembers("eleSet3"));
        System.out.println("============集合运算=================");
        System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
        System.out.println("eleSet2中的元素：" + jedis.smembers("eleSet2"));
        System.out.println("eleSet1和eleSet2的交集:" + jedis.sinter("eleSet1", "eleSet2"));
        System.out.println("eleSet1和eleSet2的并集:" + jedis.sunion("eleSet1", "eleSet2"));
        System.out.println("eleSet1和eleSet2的差集:" + jedis.sdiff("eleSet1", "eleSet2"));//eleSet1中有，eleSet2中没有
        jedis.sinterstore("eleSet4", "eleSet1", "eleSet2");//求交集并将交集保存到dstkey的集合
        System.out.println("eleSet4中的元素：" + jedis.smembers("eleSet4"));
    }
}
```

### Hash

```java
public class TestHash {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        Map<String, String> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", "value2");
        map.put("key3", "value3");
        map.put("key4", "value4");
        //添加名称为hash（key）的hash元素
        jedis.hmset("hash", map);
        //向名称为hash的hash中添加key为key5，value为value5元素
        jedis.hset("hash", "key5", "value5");
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));//return Map<String,String>
        System.out.println("散列hash的所有键为：" + jedis.hkeys("hash"));//return Set<String>
        System.out.println("散列hash的所有值为：" + jedis.hvals("hash"));//return List<String>
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 6));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 3));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("删除一个或者多个键值对：" + jedis.hdel("hash", "key2"));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("散列hash中键值对的个数：" + jedis.hlen("hash"));
        System.out.println("判断hash中是否存在key2：" + jedis.hexists("hash", "key2"));
        System.out.println("判断hash中是否存在key3：" + jedis.hexists("hash", "key3"));
        System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3"));
        System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3", "key4"));
    }
}
```

## 事务

```java
public class TestTX {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        jedis.flushDB();

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello", "world");
        jsonObject.put("name", "admin");
        // 开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        // jedis.watch(result)
        try {
            multi.set("user1", result);
            multi.set("user2", result);
            int i = 1 / 0; // 代码抛出异常事务，执行失败！
            multi.exec(); // 执行事务！
        } catch (Exception e) {
            multi.discard(); // 放弃事务
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close(); // 关闭连接
        }

    }
}
```

# SpringBoot整合

SpringBoot操作数据：spring-data、jpa、jdbc、mongodb、redis

SpringData也是和SpringBoot齐名的项目

在SpringBoot在2.x之后，原来的jedis被替换为lettuce

jedis：直连，多个线程操作的话，是不安全的，使用jedis pool 连接池来解决，类似于BIO模式

lettuce：采用netty，实例可以在多个线程中共享，不存在线程不安全的情况！可以减少线程数量，类似于NIO模式

源码分析：

```java
@Bean
@ConditionalOnMissingBean(name = "redisTemplate")
//我们可以自己定义一个redisTemplate来替换这个默认的
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
    throws UnknownHostException {
    //默认的RedisTemplate没有过多的设置，redis对象都是需要序列化的
    //两个泛型都是Object，Object的类型，我们后期使用要强制类型转换<String,Object>
    RedisTemplate<Object, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}

@Bean
@ConditionalOnMissingBean
//由于String是redis中最常用的类型，所以单独提出来了一个bean
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
    throws UnknownHostException {
    StringRedisTemplate template = new StringRedisTemplate();
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```

测试：

1. 导入依赖

2. 配置连接

   ```properties
   #配置redis
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   ```

3. 测试

自定义的RedisTemplate

```java
public class RedisConfig {
    //一个常用的固定模板
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        //配置具体的序列化方式
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);

        //序列化配置
        //json的序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        //string的学历恶化
        StringRedisSerializer stringRedisSerializer=new StringRedisSerializer();

        //key采用String的序列方式
        template.setKeySerializer(stringRedisSerializer);
        //hash的key也采用string的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        //value的序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        //hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        //载入配置
        template.afterPropertiesSet();

        return template;
    }
}
```

封装一些Redis的操作：

```java
// 在我们真实的分发中，或者你们在公司，一般都可以看到一个公司自己封装RedisUtil
@Component
public final class RedisUtil {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // =============================common============================
    /**
     * 指定缓存失效时间
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }


    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }


    // ============================String=============================

    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }
    
    /**
     * 普通缓存放入
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 普通缓存放入并设置时间
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */

    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 递增
     * @param key   键
     * @param delta 要增加几(大于0)
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }


    /**
     * 递减
     * @param key   键
     * @param delta 要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }


    // ================================Map=================================

    /**
     * HashGet
     * @param key  键 不能为null
     * @param item 项 不能为null
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }
    
    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }
    
    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * HashSet 并设置时间
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }


    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }


    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }


    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }


    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     * @param key 键
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 获取set缓存的长度
     *
     * @param key 键
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */

    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    // ===============================list=================================
    
    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取list缓存的长度
     *
     * @param key 键
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将list放入缓存
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */

    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */

    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }
}
```

测试：

```java
@SpringBootTest
class Redis02SpringbootApplicationTests {
    @Autowired
    @Qualifier("redisTemplate")
    private RedisTemplate redisTemplate;

    @Autowired
    private RedisUtil redisUtil;

    @Test
    public void test1() {
        // redisTemplate    操作不同的数据类型
        // opsForValue 操作字符串，类似String
        // opsForList 操作List，类似List
        // opsForset
        // opsForHash
        // opsForZSet
        // opsForGeo
        // opsForHyperLogLog

        // 除了基本的操作，我们的常用的方法都可以直接通过redisTemplate操作，比如事务和基本的CRUD

        // 获取redis的连接对象
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.flushDb();
//        connection.flushAll();

        redisTemplate.opsForValue().set("mykey", "123");
        System.out.println(redisTemplate.opsForValue().get("mykey"));
    }

    @Test
    public void test2() {
        User user = new User("admin", 3);
//        String jsonUser = new ObjectMapper().writeValueAsString(user);
        redisTemplate.opsForValue().set("user", user);
        System.out.println(redisTemplate.opsForValue().get("user"));
    }

    @Test
    public void test3() {
        User user = new User("admin", 3);
        redisUtil.set("user", user);
        System.out.println(redisUtil.get("user"));
    }
}
```

# Redis.conf详解

启动的时候就是根据配置文件来启动的

**单位**

![image-20210806152630403](redis.assets/image-20210806152630403.png)

配置文件unit单位对大小写不敏感

**包含**

![image-20210806152733853](redis.assets/image-20210806152733853.png)

**网络**

```bash
bind 127.0.0.1				#绑定的IP
protected-mode yes			#保护模式
port 6379			    	#端口设置
```

**通用**

```bash
daemonize yes 				#以守护进程的方式运行，默认是no，要改为yes
pidfile /var/run/redis_6379.pid		#如果以后台的方式运行，我们就需要一个pid文件

#日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)			生产环境
# warning (only very important / critical messages are logged)
loglevel notice
logfile	""		#日志的文件位置名
databases	16	#数据库的数量，默认是16个数据库
always-show-logo	yes	#是否显示logo
```

**快照**

持久化

redis是内存数据库，如果没有持久化，那么数据断电即失

```bash
save 3600 1			#如果3600s内，至少有1个key进行了修改，进行持久化操作
save 300 100		#如果300s内，至少有100个key进行了修改，进行持久化操作
save 60 10000		#如果60s内，至少有10000个key进行了修改，进行持久化操作

stop-writes-on-bgsave-error yes	#持久化出错之后是否继续工作
rdbcompression yes			#是否压缩rdb文件，需要消耗一些cpu资源
rdbchecksum yes				#保存rdb文件的时候是否进行错误校验
dir ./						#rdb文件保存目录
```

**Replication复制**

**Security安全**

可以设置Redis的密码

```bash
127.0.0.1:6379> config set requirepass 123456		#设置密码
OK
127.0.0.1:6379> auth 123456							#登录
OK
127.0.0.1:6379> config get requirepass				#查看密码
1) "requirepass"
2) "123456"
```

**客户端限制**

```bash
maxclients 10000		#最大客户端连接数10000台
maxmemory <bytes>		#redis配置最大的内存容量
maxmemory-policy noeviction		#内存到达上限的处理策略
#1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
#2、allkeys-lru ： 删除lru算法的key   
#3、volatile-random：随机删除即将过期key   
#4、allkeys-random：随机删除   
#5、volatile-ttl ： 删除即将过期的   
#6、noeviction ： 永不过期，返回错误
```



**APPEND ONLY模式**

aof配置

```bash
appendonly no		#默认是不开启aof模式，默认使用rdb方式持久化，在大部分所有的情况下，rdb完全够用
appendfilename "appendonly.aof"		#持久化文件的名字

# appendfsync always		#每次修改都会同步
appendfsync everysec		#每秒执行一次，可能会丢失这1s的数据
# appendfsync no			#不执行同步，这时候操作系统自己同步数据，速度最快
```

# 持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。在主从复制中为从机做备份。

## RDB（Redis DataBase）

![image-20210806155636805](redis.assets/image-20210806155636805.png)

在指定的时间间隔内将内存中的数据集快照写入磁盘,t也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建 （ fork )一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。默认使用的就是RDB，一般情况下不需要修改

**rdb保存的文件是dump.rdb**

触发机制：

1. save的规则满足的情况下
2. 执行flushall命令
3. 退出redis

备份会自动生成一个dump.rdb

恢复rdb文件

1. 只需要把rdb文件移动到redis的启动目录，redis启动时会自动检查dump.rdb，恢复数据

2. 查看redis的启动目录

   ```bash
   127.0.0.1:6379> config get dir
   1) "dir"
   2) "/usr/local/bin"		#如果这个目录存在dump.rdb，启动就会恢复数据
   ```

优点：

1. 适合大规模的数据恢复
2. 对数据的完整性要求不高

缺点：

1. 需要一定时间间隔进行操作，如果此时redis意外宕机了，这个最后一次修改的数据就没有了
2. fork进程的时候会占用一定的内存空间

## AOF（Append Only File）

![image-20210806155707797](redis.assets/image-20210806155707797-16282366290761.png)

将我们所有命令都记录到文件中，恢复的时候就把这个文件执行一遍

以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
**AOF保存的是appendonly.aof文件**

如果aof文件大于64M，会fork一个新进程将文件重写，这个大小可以在配置文件中设置

默认是不开启的，开启要在配置文件中将appendonly 改为yes，重启redis就可以生效

如果aof文件有错误，reids是启动不起来的，需要进行修复

修复

```bash
redis-check-aof --fix appendonly.aof
```

优点：

1. 每一次修改都同步，文件完整性更好
2. 每秒同步一次，可能会丢失一秒的数据
3. 从不同步，效率最高

缺点：

1. 相对于数据文件来说，aof远远大于rdb，修复速度也比rdb慢
2. aof运行效率也要比rdb慢，所以默认使用rdb

## 扩展

1. RDB持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2. AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
3. **只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化**
4. 同时开启两种持久化方式
   * 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。|
   * RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢?作者建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。
5. 性能建议
   * 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。
   * 如果Enable AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的lO，二是AOF rewrite 的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
   * 如果不Enable AOF，仅靠Master-Slave Repllcation实现高可用性也可以，能省掉一大笔lO，也减少了rewrite时带来的系统波动。代价是如果MasterlSlave同时断电，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave 中的RDB文件，载入较新的那个，微博就是这种架构。

# 发布订阅

Redis 发布订阅(pub/sub)是一种**消息通信模式**︰发送者(pub)发送消息，订阅者(sub)接收消息。

Redis客户端可以订阅任意数量的频道。

订阅/发布消息图︰

消息发布者+频道+消息订阅者

![image-20210806162422813](redis.assets/image-20210806162422813.png)

下图展示了频道channel1，以及订阅这个频道的三个客户端―—client2、client5和client1之间的关系∶

![image-20210806162742262](redis.assets/image-20210806162742262-16282384634592.png)

当有新消息通过PUBLISH 命令发送给频道channe1时;这个消息就会被发送给订阅它的三个客户端︰

![image-20210806162814180](redis.assets/image-20210806162814180-16282384951033.png)

**命令**

这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等。

![image-20210806162949212](redis.assets/image-20210806162949212-16282385913964.png)

**测试**

订阅端：

```bash
127.0.0.1:6379> SUBSCRIBE leo				#订阅频道leo
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "leo"
3) (integer) 1
#等待推送的信息
1) "message"
2) "leo"
3) "hello"
1) "message"
2) "leo"
3) "hello,redis"
```

发送端：

```bash
127.0.0.1:6379> PUBLISH leo hello			#发布消息
(integer) 1
127.0.0.1:6379> PUBLISH leo hello,redis		#发布消息
(integer) 1
```



**原理**

Redis是使用C实现的，通过分析Redis源码里的pubsub.c文件，了解发布和订阅机制的底层实现，籍此加深对Redis的理解。

Redis通过PUBLISH、SUBSCRIBE和PSUBSCRIBE等命令实现发布和订阅功能。

通过SUBSCRIBE 命令订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个频道!，而字典的值则是一个链表，链表中保存了所有订阅这个channel的客户端。SUBSCRIBE 命令的关键，就是将客户端添加到给定channel的订阅链表中。

通过PUBLISH命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的channel字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。

Pub/Sub从字面上理解就是发布( Publish )与订阅(Subscribe )，在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

使用场景：

1. 实时消息系统
2. 实时聊天
3. 订阅、关注

# 主从复制

## 概念

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master/leader)，后者称为从节点(slave/follower);**数据的复制是单向的，只能由主节点到从节点。**Master以写为主，Slave以读为主。

默认情况下，每台Redis服务器都是主节点;且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。



**主从复制的作用主要包括**:

1. 数据冗余∶主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复∶当主节点出现问题时,可以由从节点提供服务，实现快速的故障恢复;实际上是一种服务的冗余。
3. 负载均衡︰在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载;尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. 高可用（集群）基石∶除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。



一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的（宕机，一主二从），原因如下∶

1. 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大;
2. 从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，**单台Redis最大使用内存不应该超过20G**。

电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是"多读少写"。

![image-20210806165331023](redis.assets/image-20210806165331023.png)

主从复制，读写分离！80%的情况下都在进行读操作，为了减缓服务器的压力，架构中常使用一主二从。

## 环境配置

只配置从库，不配置主库

```bash
127.0.0.1:6379> info replication			#查看当前库的信息
# Replication
role:master								#角色
connected_slaves:0						#从机为0
master_failover_state:no-failover
master_replid:c10da258fd6e60a638c13545227e485e8a1f05d7
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制三个配置文件，修改对应信息

1. port
2. pidfile
3. logfile
4. dbfilename

修改完毕之后启动三个server

![image-20210806171438131](redis.assets/image-20210806171438131.png)

## 一主二从

**默认情况下，每台redis服务器都是主节点**，一般情况下只用配置从机就行了

**如何配置从机**：

```bash
#从机
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379		
OK
127.0.0.1:6381> INFO Replication
# Replication
role:slave				#当前角色是从机
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:2
master_sync_in_progress:0
slave_repl_offset:70
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:6bd717c5d883b5915ab72ef2ce9553e171645863
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:70
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:29
repl_backlog_histlen:42
#########################################################################################################################
#主机
127.0.0.1:6379> INFO Replication
# Replication
role:master
connected_slaves:2			#从机的信息
slave0:ip=127.0.0.1,port=6380,state=online,offset=168,lag=1
slave1:ip=127.0.0.1,port=6381,state=online,offset=168,lag=1
master_failover_state:no-failover
master_replid:6bd717c5d883b5915ab72ef2ce9553e171645863
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:168
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:168
```

真是的主从配置不是用命令，而是在配置文件中设置，这样的话就是永久的

在配置文件中，找到replicaof，为其配置主机的ip和port，如果主机有密码，可以再masterauth中配置

**注意点**：

1. 主机可以写，从机只能读，主机中的数据，会被从机自动保存
2. 主机断开，从机依旧可以连接到主机，主机恢复后，从机依旧可以获取主机的新数据
3. 如果使用命令行配置的主从，从机一旦重启，就会变回主机

**复制原理**：

Slave启动成功连接到master后会发送一个sync同步命令

Master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，**master将传送整个数据文件到slave，并完成一次完全同步**。

1. 全量复制︰而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

2. 增量复制:Master继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步（全量复制）将被自动执行，我们的数据一定可以在从机中看到



## 哨兵模式

自动选取主机

### 概述

主从切换技术的方法是︰当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel (哨兵）架构来解决这个问题。

谋朝篡位的自动版，能够后台监控主机是否故障，如果故障了根据投票数**自动将从库转换为主库**。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**。

![image-20210806192053788](redis.assets/image-20210806192053788-16282488556002.png)

这里的哨兵有两个作用

* 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
* 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

![image-20210806191957100](redis.assets/image-20210806191957100-16282487985881.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

### **测试**

1. 配置哨兵配置文件sentinel.conf

   ```bash
   #sentinel monitor 被监控的名称 host port 1
   sentinel monitor myredis 127.0.0.1 6379 1
   ```

   后面的数字1，代表主机挂了，slave投票让谁接替成为主机，票数最多的，就会成为主机

2. 启动哨兵

   ```bash
   [root@meowy bin]# redis-sentinel myconf/sentinel.conf 
   7675:X 06 Aug 2021 19:30:26.477 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   7675:X 06 Aug 2021 19:30:26.477 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=7675, just started
   7675:X 06 Aug 2021 19:30:26.477 # Configuration loaded
   7675:X 06 Aug 2021 19:30:26.478 * monotonic clock: POSIX clock_gettime
                   _._                                                  
              _.-``__ ''-._                                             
         _.-``    `.  `_.  ''-._           Redis 6.2.5 (00000000/0) 64 bit
     .-`` .-```.  ```\/    _.,_ ''-._                                  
    (    '      ,       .-`  | `,    )     Running in sentinel mode
    |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
    |    `-._   `._    /     _.-'    |     PID: 7675
     `-._    `-._  `-./  _.-'    _.-'                                   
    |`-._`-._    `-.__.-'    _.-'_.-'|                                  
    |    `-._`-._        _.-'_.-'    |           https://redis.io       
     `-._    `-._`-.__.-'_.-'    _.-'                                   
    |`-._`-._    `-.__.-'    _.-'_.-'|                                  
    |    `-._`-._        _.-'_.-'    |                                  
     `-._    `-._`-.__.-'_.-'    _.-'                                   
         `-._    `-.__.-'    _.-'                                       
             `-._        _.-'                                           
                 `-.__.-'                                               
   
   7675:X 06 Aug 2021 19:30:26.478 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
   7675:X 06 Aug 2021 19:30:26.490 # Sentinel ID is c23daf267867b8ac9dc00f8b2bcace685b82b867
   7675:X 06 Aug 2021 19:30:26.490 # +monitor master myredis 127.0.0.1 6379 quorum 1
   7675:X 06 Aug 2021 19:30:26.491 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:30:26.495 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
   ```

   如果Master节点断开，会投票一个新的slave为master

   ```bash
   7675:X 06 Aug 2021 19:32:11.745 # +sdown master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.745 # +odown master myredis 127.0.0.1 6379 #quorum 1/1
   7675:X 06 Aug 2021 19:32:11.745 # +new-epoch 1
   7675:X 06 Aug 2021 19:32:11.745 # +try-failover master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.751 # +vote-for-leader c23daf267867b8ac9dc00f8b2bcace685b82b867 1
   7675:X 06 Aug 2021 19:32:11.751 # +elected-leader master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.751 # +failover-state-select-slave master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.841 # +selected-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.842 * +failover-state-send-slaveof-noone slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:11.897 * +failover-state-wait-promotion slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:12.747 # +promoted-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:12.747 # +failover-state-reconf-slaves master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:12.843 * +slave-reconf-sent slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:13.757 * +slave-reconf-inprog slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:13.757 * +slave-reconf-done slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:13.823 # +failover-end master myredis 127.0.0.1 6379
   7675:X 06 Aug 2021 19:32:13.823 # +switch-master myredis 127.0.0.1 6379 127.0.0.1 6381
   7675:X 06 Aug 2021 19:32:13.824 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6381
   7675:X 06 Aug 2021 19:32:13.824 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6381
   7675:X 06 Aug 2021 19:32:43.832 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6381
   ```

   当之前的主机又回来了，只能给新的主机当从机

   ```bash
   7675:X 06 Aug 2021 19:35:05.933 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6381
   ```

### 优缺点

优点：

1. 哨兵集群，基于主从复制模式，所有的主从配置优点都包含
2. 主从可以进行切换，故障可以转移，系统的可用性就会更好
3. 哨兵模式就是主从模式的升级，从手动到自动，更加健壮

缺点：

1. Redis不好在线扩容，集群容量到达上限，在线扩容十分麻烦
2. 哨兵模式配置其实很麻烦，选择很多 

### **哨兵模式的全部配置**

```bash
#Example sentinel.conf

#哨兵sentinel实例运行的端口默认26379
port 26379

#哨兵sentinel的工作日录
dir /tmp

# 哨兵sentine1监控的redis主节点的 ip port
# master-name可以自己命名的主节点名字只能由字母A-z、数字O-9 、这三个字符".-_"组成。
# quorum配置多少个sentinel哨兵统一认为master主节点失联那么这时客观上认为主节点失联了
# sentinel monitor <master-name> <ip><redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

# 当在Redis实例中开启了requirepass foobared 授权密码这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel连接主从的密码注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passwOrd

# 指定多少亳秒之后 主节点没有应答哨兵sentinel此时哨兵主观上认为主节点下线默认30秒
# sentinel down-after-mi77iseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000

# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
# 故障转移的超时时间failover-timeout可以用在以下这些方面:
# 1．同一个sentinel对同一个master两次failover之间的间隔时间。
# 2．当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
# 3. 当想要取消一个正在进行的failover所需要的时间。
# 4. 当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按paralle1-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION

# 配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
# 对于脚木的运行结果有以下规则:
# 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
# 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
# 如果脚木在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
# 一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。

# 通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等)，将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
# 通知脚本
# shell编程
#sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh

# 客户端重新配置主节点参数脚木
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip><from-port> <to-ip><to-port>#目前<state>总是"failover"，
# <role>是"leader"或者"observer"中的一个，
# 参数 from-ip，from-port，to-ip，to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用,不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh #一般都是由运维来配
```

# 缓存穿透和雪崩



Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。
另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也都有比较流行的解决方案。

## 缓存穿透（查不到）

### 概念

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

### 解决方案

#### 布隆过滤器

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力;

![image-20210806195636984](redis.assets/image-20210806195636984.png)

#### 缓存空对象

当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；

![image-20210806195747557](redis.assets/image-20210806195747557.png)

但是这种方法会存在两个问题:

1. 如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键;
2. 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

## 缓存击穿（查太多）

### 概述

这里需要注意和缓存穿透的区别，缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导使数据库瞬间压力过大。

### 解决方案

#### 设置热点数据永不过期

从缓存层面来看，没有设置过期时间，所以不会出现热点 key过期后产生的问题。

#### 加互斥锁

分布式锁∶使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。

## 缓存雪崩

### 概述

缓存雪崩，是指在某一个时间段，缓存集中过期失效。Redis宕机。
产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。

![image-20210806200336876](redis.assets/image-20210806200336876.png)

其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

### 解决方案

#### redis高可用

这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。

#### 限流降级

这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

#### 数据预热

数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

