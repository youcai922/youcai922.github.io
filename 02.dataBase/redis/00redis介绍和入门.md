# Redis介绍

完全开源的，遵守BSD协议，高性能的key-value数据库

#### redis相比较其他key-value具有以下特点：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份

#### Redis的优势：

- 性能极高-Redis读速度是110000次/s，写的速度是81000次/s
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
- 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。



# redis入门

**五种常用结构：字符串(String)、哈希(Hash)、列表(list)、集合(sets)和有序集合(sorted sets)**

#### String（字符串）

```
SET name "油菜"	   //设置name值
GET name			//获取name值
```

String类型的一个键最多存储512MB

#### Hash（哈希）

Redis hash是一个键值队集合，其实是一个String类型的field和value映射表，hash特别适合用于存储对象

```
DEL name	//删除name
HMSET person name "油菜" weight "100kg"	//设置person
HGET person name	//获取person的name值
```

每个Hash可以存储 2^32 -1键值对

#### List列表

列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部或者尾部

```
DEL person	//删除原有的person
lpush person person1	//添加person1到person列表的左边
lpush person person2	//添加person1到person列表的左边
lpush person person3	//添加person1到person列表的左边
lrange person 0 10		//输出person的0到10个元素
```

每个list可以存储 2^32 -1元素

#### Set集合

Set是String类型的无序且不重复的集合，内部通过hash表实现，增加、删除、查找的复杂度都是O（1）。

sadd命令：添加一个String元素到key对应的set集合中，成功返回1，如果元素已经在集合中则返回0

```
sadd key member
```

```
DEL person	//删除person
sadd person person1	//添加person1到person的set集合中
sadd person person2	//添加person2到person的set集合中
sadd person person3	//添加person3到person的set集合中
sadd person person3	//添加person3到person的set集合中
smembers person		//输出person中的元素
输出的结果包含person1,person2,person3，尽管person3被插入了两次，但是只会存在一个
```

#### ZSet集合（有序的set）

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

- zset的成员是唯一的，但是分数却可以重复

```
zadd key score member //添加元素到集合，元素在集合中存在则更新对应score
```

```
DEL person	//删除person
zadd person 0 person1		//添加person1到zset中
zadd person 0 person2		//添加person2到zset中
zadd person 0 person3		//添加person3到zset中
zadd person 0 person3		//添加person3到zset中
ZRANGEBYSCORE person 0 1000		//输出person，和set一样，person3即使插入2次，也只会存储一份
```







#### Redis HyperLogLog

HyperLogLog用来做基数统计的算法。

优点：

- 在输入元素的数量或者体积非常非常大的时候，计算基数所需的空间总是固定的、并且很小的。













