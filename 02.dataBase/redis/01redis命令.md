#### INCR

incr命令将key中存储的数字值增一

如果key不存在，则key的值会被初始化为0，然后再执行INCR操作

命令语法：

```
redis 127.0.0.1:6379> INCR KEY_NAME 
```

实例

```
redis> SET page_view 20
OK
redis> INCR page_view
(integer) 21
redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
"21"
```



#### 发布订阅

发布订阅是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](https://www.runoob.com/wp-content/uploads/2014/11/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端

![img](https://www.runoob.com/wp-content/uploads/2014/11/pubsub2.png)

发布订阅需要开启两个redis-cli客户端

首先打开第一个客户端：

```
	SUBSCRIBE yctestChat	//订阅yctestChat
redis会输出
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "redisChat"
    3) (integer) 1
```

这个时候我们开启第二个客户端

```
	PUBLISH yctestChat "Redis PUBLISH test"
这个时候第一个客户端就会输出
	1) "message"
    2) "runoobChat"
    3) "Redis PUBLISH test"
修改发布信息
	PUBLISH yctestChat "Redis PUBLISH yctest"
这个时候第一个客户端就会输出
    1) "message"
    2) "runoobChat"
    3) "Learn redis by yctest"
```



#### 事务

redis可以一次执行多个命令，并且带有以下三个重要保证：

- 批量操作在发送exec命令之前被放入队列缓存
- 收到exec命令进入事务执行，事务中任意命令执行失败，其余的命令依然被执行
- 在事务执行过程中，其他客户端提交的命令请求不回插入到事务执行命令序列中。

一个事务从开始执行会经历以下三个阶段：

- 开始事务
- 命令队列
- 执行事务

```
redis 127.0.0.1:6379> MULTI		//开始事务
OK
redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED
redis 127.0.0.1:6379> GET book-name
QUEUED
redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED
redis 127.0.0.1:6379> SMEMBERS tag
QUEUED
redis 127.0.0.1:6379> EXEC	//触发事务
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```

单个Redis命令的执行是原子性的，但Redis没有在事务上增加任何维持原子性的机制，所以Redis事务的执行并不是原子性的。

事务可以理解为一个大包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做



#### Redis脚本

redis脚本试用lua解释器来执行脚本，执行脚本的常用命令是EVAL

eval的基本语法为：

```
eval script numkeys key [key ...] arg [arg ...]
```

```
redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second

1) "key1"
2) "key2"
3) "first"
4) "second"
```

#### Redis连接

```
AUTH "password"	//验证密码是否正确
PING //查看服务是否运行，PONG为运行
```

#### Redis查看服务器信息

```
INFO	//查看服务器信息
```

#### Redis GEO

GEO主要用于存储地理位置信息，并对存储的信息进行操作。

geoadd:添加地理位置的坐标，可以将一个或者多个经度、纬度、位置名称添加到指定的key中

命令格式：GEOADD key longitude latitude member [longitude latitude member ...]

```
redis>GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEODIST Sicily Palermo Catania
"166274.1516"
redis> GEORADIUS Sicily 15 37 100 km
1) "Catania"
redis> GEORADIUS Sicily 15 37 200 km
1) "Palermo"
2) "Catania"
```

geopos：获取地理位置的坐标，返回指定名称的位置（经度和纬度），不存在返回nil。
命令格式：GEOPOS key member [member ...]

```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEOPOS Sicily Palermo Catania NonExisting
1) 1) "13.36138933897018433"
   2) "38.11555639549629859"
2) 1) "15.08726745843887329"
   2) "37.50266842333162032"
3) (nil)
```

geodist：计算两个位置之间的距离
命令格式：GEODIST key member1 member2 [m|km|ft|mi]
//最后的是距离单位参数说明	m ：米，默认单位。km ：千米。mi ：英里。ft ：英尺。

```
redis>GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEODIST Sicily Palermo Catania
"166274.1516"
redis> GEODIST Sicily Palermo Catania km
"166.2742"
redis> GEODIST Sicily Palermo Catania mi
"103.3182"
redis> GEODIST Sicily Foo Bar
(nil)
```

georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合，以给定的经纬度为中心，返回键包含的位置元素当中，与中心距离不超过给定最大距离的所有位置元素。

georadiusbymember：根据存储在位置集合里面的某个地点获取指定范围内的地理位置集合

命令格式：GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
参数说明：m ：米，默认单位。
km ：千米。
mi ：英里。
ft ：英尺。
WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
WITHCOORD: 将位置元素的经度和维度也一并返回。
WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。
COUNT 限定返回的记录数。
ASC: 查找结果根据距离从近到远排序。
DESC: 查找结果根据从远到近排序。

```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEORADIUS Sicily 15 37 200 km WITHDIST
1) 1) "Palermo"
   2) "190.4424"
2) 1) "Catania"
   2) "56.4413"
redis> GEORADIUS Sicily 15 37 200 km WITHCOORD
1) 1) "Palermo"
   2) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) 1) "15.08726745843887329"
      2) "37.50266842333162032"
redis> GEORADIUS Sicily 15 37 200 km WITHDIST WITHCOORD
1) 1) "Palermo"
   2) "190.4424"
   3) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) "56.4413"
   3) 1) "15.08726745843887329"
      2) "37.50266842333162032"
```

geohash：返回一个或多个位置对象的geohash值,geohash 用于获取一个或多个位置元素的 geohash 值。

命令格式：GEOHASH key member [member ...]

```
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
redis> GEOHASH Sicily Palermo Catania
1) "sqc8b49rny0"
2) "sqdtr74hyu0"
```



#### Stream

stream是5.0新增的数据结构，主要用于消息队列，redis发布订阅可以实现消息队列，但是他无法实现持久化，比如网络断开，redis宕机，消息就会被丢弃。

简单来说发布订阅只能分发消息，但是无法记录历史消息。Stream提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能够记住每一个客户断的访问位置，还能保证消息不丢失。

Stream结构如下：他有一个消息链表，将所有加入的消息都穿起来，每个消息都有一个唯一的ID和对应的内容：

![img](https://www.runoob.com/wp-content/uploads/2020/09/en-us_image_0167982791.png)

每一个Stream都有一个唯一的名称，他就是redis的key，在我们上搜测试用xadd指令追加消息的时候创建。

上图分析：

- Consumer Group：消息组，使用XGROUP CREATE命令创建，一个消费组有多个消费者（Consumer）
- last_delivered_id：游标，每个消费组会有一个游标last_delivered_id，任意一个消费者读取了消息都会使游标last_delivered_id往前移动
- pending_ids：消费者（Consumer）的状态变量，作用是维护消费者的未确认id。pending_ids记录了当前已经被客户端读取的消息，但是还没有ack（Acknowledge character：确认字符）

消息队列相关命令：

```
XADD：添加消息到末尾
语法格式：XADD key ID field value [field value ...]
实例：XADD mystream * name Sara surname OConnor	//队列名称是mytream；*代表redis自己生成消息的id，可以自己指定，但是要保证递增行；后面的是消息记录


XTRIM：对流进行修剪，限制长度
语法格式：XTRIM key MAXLEN [~] count	//key队列名称，MAXLEN：长度；count数量
实例：127.0.0.1:6379> XADD mystream * field1 A field2 B field3 C field4 D
"1601372434568-0"
127.0.0.1:6379> XTRIM mystream MAXLEN 2
(integer) 0
127.0.0.1:6379> XRANGE mystream - +
1) 1) "1601372434568-0"
   2) 1) "field1"
      2) "A"
      3) "field2"
      4) "B"
      5) "field3"
      6) "C"
      7) "field4"
      8) "D"
      
XDEL：删除消息
语法格式：XDEL key ID [ID ...] 	//key队列名称，ID消息id
实例：> XADD mystream * a 1
1538561698944-0
> XDEL mystream 1538561698944-0

XLEN：获取流包含的元素数量，即消息长度
语法格式：XLEN key
实例：redis> XLEN mystream
(integer) 3

XRANGE：获取消息列表，会自动过滤已经删除的消息
语法格式：XRANGE key start end [COUNT count]	//key队列名，star：开始值，-表示最小值，end结束值，+表示最大值；count数量
实例：redis> XRANGE writers - + COUNT 2	//输出writer队列中的2个消息，每个消息输出所有

XREVRANGE：反向获取列表，ID从大到小
语法格式：XREVRANGE key end start [COUNT count]
实例：XREVRANGE writers + - COUNT 1	//输出writer队列中的一个消息，这个消息输出所有

XREAD：以组赛或非阻塞方式获取消息列表
语法格式：XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]		//count数量，milliseconds可选，阻塞毫秒数，没有设置就是非阻塞模式；key队列名；id消息id
实例：> XREAD COUNT 2 STREAMS mystream writers 0-0 0-0	//从Stream头部读取两条消息
```

消费者组相关命令：

```
XGROUP CREATTE：创建消费者组
语法格式：XGROUP [CREATE key groupname id-or-$] [SETID key groupname id-or-$] [DESTROY key groupname] [DELCONSUMER key groupname consumername]	//key队列名称，groupname组名，￥表示从尾部开始消费，只接受新消息，当前Stream消息会全部忽略。
//XGROUP CREATE mystream consumer-group-name 0-0 从头开始消费
//XGROUP CREATE mystream consumer-group-name %	 从尾部开始消费

XRADGROUP GROUP：读取消费者组中的消息
语法格式：XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]	//group消费组名，cosumer消费者名，count读取数量，milliseconds阻塞毫秒数，key队列名，id消息id

XACK：将消息标记为“已处理”
XGROUP SETID：为消费者组设置新的最后传送消息id
XGROUP DELCONSUMER：删除消费者
XGROUP DESTROY：删除消费者组
XPENDING：显示待处理消息的相关信息
XCLAIM：转移消息的归属权
XINFO：查看流和消费者组的相关信息
XINFO GROUPS：打印消费者组的消息
XINFO STREAM：打印流信息
```























