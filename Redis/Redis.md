# Redis

------

Redis能干吗

1、内存持久化

2、效率高

3、发布订阅系统

4、地图信息分析

5、计时器、计数器

特性

1、多余的数据类型

2、持久化

3、集群

4、事务

## 1、Redis操作

查看默认安装目录：

redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何

redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲

redis-check-dump：修复有问题的dump.rdb文件

redis-sentinel：Redis集群使用

redis-server：Redis服务器启动命令

redis-cli：客户端，操作入口

### 1.1、Redis键

keys *                 查看当前库所有key  (匹配：keys *1)

exists key           判断某个key是否存在

type key             查看你的key是什么类型

del key               删除指定的key数据

unlink key          根据value选择非阻塞删除

仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。

expire key 10     10秒钟：为给定的key设置过期时间

ttl key                 查看还有多少秒过期，-1表示永不过期，-2表示已过期

select   1             切换数据库

dbsize                  查看当前数据库的key的数量

flushdb                清空当前库

flushall                 通杀全部库

### 1.2、Redis字符串

#### 1.2.1简介

String是Redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value。

String类型是二进制安全的。意味着Redis的string可以包含任何数据。比如jpg图片或者序列化的对象。

String类型是Redis最基本的数据类型，一个Redis中字符串value最多可以是512M

#### 1.2.2**常用命令**

set  <key><value>             添加键值对

get  <key>                             查询对应键值

append <key><value>      将给定的<value> 追加到原值的末尾

strlen <key>                          获得值的长度

setnx <key><value>          只有在 key 不存在时  设置 key 的值



incr <key>

​	将 key 中储存的数字值增1

​	只能对数字值操作，如果为空，新增值为1

decr <key>

​	将 key 中储存的数字值减1

​	只能对数字值操作，如果为空，新增值为-1

incrby / decrby <key><步长>将 key 中储存的数字值增减。自定义步长。

**原子性，有一个失败则都失败**

mset <key1><value1><key2><value2> ..... 

同时设置一个或多个 key-value对 

mget <key1><key2><key3> .....

同时获取一个或多个 value 

msetnx <key1><value1><key2><value2> ..... 

同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。



getrange <key><起始位置><结束位置>

获得值的范围，类似java中的substring，**前包，后包**

setrange <key><起始位置><value>

用 <value> 覆写<key>所储存的字符串值，从<起始位置>开始(**索引从0****开始**)。

 

**setex <key><过期时间>**<value>

设置键值的同时，设置过期时间，单位秒。

getset <key><value>

以新换旧，设置了新值同时获得旧值。

#### 1.2.3  **数据结构**

String的数据结构为简单动态字符串(Simple Dynamic String,缩写SDS)。是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配.

![image-20210622164453408](img\9.png)

如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

### 1.3、**Redis**列表(List)

#### 1.3.1简介

单键多值

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

它的底层实际是个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![image-20210623140030984](img\10.png)

#### 1.3.2常用命令

lpush / rpush <key><value1><value2><value3> ....       从左边/右边插入一个或多个值。

lpop / rpop <key>                                                        从左边/右边吐出一个值。值在键在，值光键亡。

rpoplpush <key1><key2>										 从<key1>列表右边吐出一个值，插到<key2>列表左边。

lrange <key><start><stop>             					  按照索引下标获得元素(从左到右)

eg：lrange mylist 0 -1  												0左边第一个，-1右边第一个，（0-1表示获取所有）

lindex <key><index>												按照索引下标获得元素(从左到右)

llen <key>																	获得列表长度 

linsert <key> before <value><newvalue>		   在<value>的后面插入<newvalue>插入值

lrem <key><n><value>										    从左边删除n个value(从左到右)

lset<key><index><value>									  将列表key下标为index的值替换成value

ltrim <key><n><value>											通过下标的长度截取指定的长度

### 1.4、**Redis**集合(Set)

#### 1.4.1简介

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于set是可以**自动排重**的，当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。

Redis的Set是string类型的无序集合。它底层其实是一个value为null的hash表，所以添加，删除，查找的**复杂度都是**O(1)。

一个算法，随着数据的增加，执行时间的长短，如果是O(1)，数据增加，查找数据的时间不变

#### 1.4.2常用命令

sadd <key><value1><value2> ..... 

将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略

smembers <key>											取出该集合的所有值。

sismember <key><value>							判断集合<key>是否为含有该<value>值，有1，没有0

scard<key>														返回该集合的元素个数。

srem <key><value1><value2> .... 			删除集合中的某个元素。

spop <key>										   			随机从该集合中吐出一个值

srandmember <key><n>								随机从该集合中取出n个值。不会从集合中删除 。

smove <source><destination><value>把集合中一个值从一个集合移动到另一个集合

sinter <key1><key2>										返回两个集合的交集元素。

sunion <key1><key2>									返回两个集合的并集元素。

sdiff <key1><key2>										返回两个集合的**差集**元素(key1中的，不包含key2中的)

### 1.5、**Redis**哈希(Hash)

#### 1.5.1、简介

Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储

#### 1.5.2常用命令

hset <key><field><value>						给<key>集合中的 <field>键赋值<value>

hget <key1><field>									  从<key1>集合<field>取出 value 

hmset <key1><field1><value1><field2><value2>... 批量设置hash的值

hmget<key1><field1><value1><field2><value2>... 批量获取hash的值

hexists<key1><field>								查看哈希表 key 中，给定域 field 是否存在。 

hgetall <key>												  获取全部的数据

hdel<key>														删除hash指定key字段，对应的value值也就消失了

hlen<key>														获取hash表的字段数量

hkeys <key>													列出该hash集合的所有field

hvals <key>													列出该hash集合的所有value

hincrby <key><field><increment>	  为哈希表 key 中的域 field 的值加上增量 1  -1

hsetnx <key><field><value>			将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 .

哈希适合user name age，尤其是用户信息之类的，经常变动的信息

#### 1.5.3数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 1.6、**Redis**有序集合Zset(sorted set)

#### 1.6.1、简介

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（**score**）**,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

#### **1.6.2、**  **常用命令**

**zadd <key><score1><value1><score2><value2>…**

将一个或多个 member 元素及其 score 值加入到有序集 key 当中。

**zrange <key><start><stop> [BYSCORE | BYLEX] [REV] [LIMIT offset count] [WITHSCORES]**

可以执行不同类型的范围查询：按索引（排名）、按分数或按索引顺序。

**zrevrange <key> <start><stop> [withscores] **

返回有序集 key 中，所有 score 值介于 start 和 stop 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 

**zrevrange key <start><stop> [withscores] [limit offset count]**        

同上，改为从大到小排列。 

**zincrby <key><increment><value>   为元素的score加上增量**

**zcard <key>**													获取有序集合中的个数

**zrem <key><value>**									删除该集合下，指定值的元素

**zcount <key><min><max>**					  统计该集合，分数区间内的元素个数 

**zrank <key><value>**                              返回该值在集合中的排名，从0开始。

#### 1.6.3、数据结构

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

## 2、Redis特点

Redis是单线程+多路IO复用技术

多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）



1、非关系型的键值对数据库，可以根据键以O(1)的时间复杂度取出或插入关联值；

2、Redis的数据时存在内存中的；

3、键值对中的见的类型可以是字符串，整形，浮点型等，且键是唯一的；

4、键值对中的值类型可以是String、hash、list、set、sorted set等；

5、Redis内置了复制，磁盘永久化，LUA脚本，事务，SSL，客户端代理等功能；

6、通过Redis哨兵和自动分区提供高可用性。

### 2、Redis的应用场景----缓存

![cliemt  (2)  db  disk  redis  Memory ](.\img\1.png)

1）第一次查询数据时，会去数据库查询；

2）从数据库查询的信息会缓存到Redis中；

3）下一次再查询同一个信息，会先去Redis中查找，直接访问Redis时，访问内存，比访问数据库效率高得多。

4）高并发时，Redis缓存可以减轻数据库的压力。

------

## 3、Redis事务操作

**Redis事务本质：一组命令的集合！一个事务中的所有命令都会被序列化，在事务执行的过程中，会按照顺序执行！**

**一次性、顺序性、排他性！执行一系列的命令！**

没有隔离级别的概念！

所有命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行

```bash
multi#开始事务
set k1 v1
set k2 v2
exec#执行事务
```

```bash
multi
set k1 v1
set ke v2
set k4 v4
discard#放弃事务
```

```
编译型异常（代码有问题！命令有错！），事务中所有的命令都不会被执行
```

错误的命令，所有的命令都不会被执行

```
运行时异常（1/0），如果队列中存在语法性错误，那么其他命令是可以正常执行的
```

命令没有错运行时发生错误，其他命令依旧可以执行

### 1、**Redis的事务定义**

**Redis单条命令要么同时成功，要么同时失败，但是事务不保证原子性**

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要作用就是串联多个命令防止别的命令插队。

### 2、Multi、Exec、discard

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。 

![image-20210523102603136](.\img\3.png)

### 3、事务的错误处理

组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

![image-20210523103625660](.\img\4.png)

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。

![image-20210523103700707](.\img\5.png)

### 4、事务冲突的问题

#### 4.1悲观锁

![image-20210523104821761](.\img\6.png)

**悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。**传统的关系型数据库里边就用到了很多这种锁机制**，比如**行锁**，**表锁**等，**读锁**，**写锁**等，都是在做操作之前先上锁。

#### 4.2 **乐观锁**

![image-20210527202044073](.\img\8.png)

**乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。

#### 4.3 **WATCH** **key** **[key ...]**

**正常监视的话没有修改就会成功，使用watch当作redis的乐观锁操作，执行之前另外一个线程修改改值就会导致事务执行失败**

在执行multi之前，**先执行watch key1 [key2],可以监视一个(或多个) key** ，如果在事务**执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。**

#### **4.4**  **unwatch**

失效后需要先解锁，再去拿锁。

取消 WATCH 命令对所有 key 的监视。

如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。

#### **5**  **Redis**事务三特性

-  **单独的隔离操作** 
  - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

- **没有隔离级别的概念** 
  - 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

-  **不保证原子性** 
  - 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 

## 4、SpringBoot结合Redis

源码分析

```java
@Bean
@ConditionalOnMissingBean(name = "redisTemplate")
@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    //默认的RedisTemplate没有过多的设置，redis对象都是需要序列化的
    //两个泛型都是Object，Object类型，我们后使用需要强制转换<String， Object>
   RedisTemplate<Object, Object> template = new RedisTemplate<>();
   template.setConnectionFactory(redisConnectionFactory);
   return template;
}

@Bean
@ConditionalOnMissingBean//由于String是最常用的类型，所以单独提出来了
@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
   StringRedisTemplate template = new StringRedisTemplate();
   template.setConnectionFactory(redisConnectionFactory);
   return template;
}
```

## 5、Redis配置

配置文件

### 1、对大小写不敏感

### 2、包含文件

```bash
################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include .\path\to\local.conf
# include c:\path\to\other.conf

```

### 3、网络

```
bind 127.0.0.1#绑定的ip
protected-mode yes#是否受保护的模式
port 6379#端口设置

```

### 4、通用配置

```bash
daemonize yes#以守护进程的方式运行,默认是no，需要自己做yes，WIN里不让用
#日志级别
# debug (a lot of information, useful for development/testing)#用于测试和开发
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)#部分重要的信息，生产环境使用
# warning (only very important / critical messages are logged)
loglevel notice
logfile ""#log文件的地址


```

### 5、快照

redis是内存数据库，如果没有持久化，那么数据断点即失

持久化，在规定的时间内执行了多少次操作，则会持久化到文件

```bash
#持久化规则，如果900s内有1个key进行了修改，就进行持久化操作
save 900 1
#持久化规则，如果300s内有10个key进行了修改，就进行持久化操作
save 300 10
#持久化规则，如果360s内有10000个key进行了修改，就进行持久化操作
save 60 10000
#可以自己弄

stop-writes-on-bgsave-error yes#持久化出错后是否继续工作

rdbcompression yes#是否压缩rdb文件，需要消耗CPU资源

rdbchecksum yes#保存rdb文件的时候，进行错误的检查校验

dir ./#rdb文件保存的目录
```

6、

### 7、安全SECURITY 

```bash
requirepass foobared#设置密码
config set requirepass 123456#或者通过配置文件配置


```

### 8、限制CLIENTS

```bash
 maxclients 10000#最大客户端的数量
 maxmemory <bytes>#redis配置最大的内存容量
 maxmemory-policy noeviction#内存达到上限后的处理策略

```

### 9、AOF配置APPEND ONLY MODE 

```bash
appendonly no #默认是不开启aof
appendfilename "appendonly.aof"#持久化文件的名字
# appendfsync always #每次修改都会同步，消耗性能
appendfsync everysec#每秒执行一次sync，可能会丢失这1s的数据
# appendfsync no #不同步，速度最快
```

## 6、持久化之RDB

### RDB(Redis DataBase)

![image-20210625151534187](img\11.png)

在指定的时间间隔内将内存中的数据集快照写入硬盘，行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存中。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB,一般情况下不需要修改这个位置！

**rdb保存的文件是dump.rdb**都是在我们配置文件的快照中去配置的

1、save的规则满足的情况下，会自动触发rdb规则

2、执行flushall命令，也会触发我们的rdb规则

3、退出redis，也会产生一个rdb文件

### 2、如果恢复rdb文件

1、只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检测dump.rdb恢复其中的数据！

2、查看需要存在的位置

3、优缺点

1、优点：

适合大规模数据恢复！

对数据的完整性要求不高！

2、缺点：

需要一定的时间间隔进程操作，如果意外宕机了，最后一次修改的数据就没有了

fork进程会占用一定的内存空间

## 7、AOF

默认不开启只需要开启appendonly改为yes就开启了aof！

重启，redis就可以生效了

![image-20210625160921877](img\13.png)

### 1、aof启动

如果无法启动，可以使用修复工具`redis-check-aof --fix`

优点：

```bash
appendonly no #默认是不开启aof
appendfilename "appendonly.aof"#持久化文件的名字
# appendfsync always #每次修改都会同步，消耗性能
appendfsync everysec#每秒执行一次sync，可能会丢失这1s的数据
# appendfsync no #不同步，速度最快
```

1、每一次修改都同步文件完整性会更好

2、每秒同步一次，可能会丢失一秒的数据

3、从不同步，效率最高

缺点：

1、相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢！

2、Aof的运行效率比rdb慢，所以我们redis默认的配置就是rdb

### 3、重写规则

redis会判断上一个文件的大小，如果aof大于64m，太大了！会fork一个新的进程来将我们的文件重写

## 8、发布订阅

通信 队列 发送者===管道=== 订阅者

### 1、Redis是一种消息通讯模式，

Redis客户端可以订阅任意数量的频道

消息订阅图：

![image-20210625162405284](img\14.png)

### 2、命令

![image-20210625162750420](img\15.png)

```bash
subscribe bobo #订阅一个频道
publish bobo “hello bobo”#发送一个消息
```

使用场景

1、实时消息系统

2、实时聊天！

## 9、主从复制

### 1、主要作用

1、解决数据冗余：主从复制实现类数据的热备份，是持久化之外的一种数据冗余方式

2、故障恢复：当主节点出现问题时，可以由从结点提供服务，实现快速的故障恢复；实际上是一种服务的冗余

3、负载均衡：在主从复制的基础上，配合读写分离（单台不超过20G），可以由主节点提供写服务，由从节点提供读服务，分担服务器负载，可以大幅度提高服务器的并发量

4、高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础



### 概念

![image-20210625172229325](img\16.png)

redis集群最少要三台，一主二从！不可能单机使用redis

2、基本配置

```bash
C:\Users\admin>redis-cli #查看当前库的信息
127.0.0.1:6379> info replication
# Replication
role:master #角色
connected_slaves:0 #有多少个从机
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

### 3、修改配置文件

复制三个配置文件修改对应的信息

#### 1、端口

#### 2、pid名字

#### 3、log文件名字

#### 4、dump.rdp名字

### 一主二从（工作不用）

**没配置的话每一台都是主节点**

一般情况下只配置从机就好了，认老大！一主二从

```bash
slaveof 127.0.0.1 6379#连接到老大
```

主机可以写，从机不能写只能读。

主机断开连接从机依旧连接到主机，主机回来了就正常的读到数据

如果是使用命令行，来配置的主从，这个时候如果重启了，就会变回主机，只要变为从机，立马就会从主机中取得值

### 4、复制原理

Slave启动成功连接到master后会发送一个sync同步命令

Master接到命令，启动后台的存盘程序，同时手机所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。

全量复制：而slave服务在接受到数据库文件数据后，将其存盘并加载到内存中

增量复制：Master继续将新的所有手机到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完成同步（全量复制）将被自动执行

**层层链路模式（工作不用）**

![image-20210625212440871](img\17.png)

由于中间结点为从节点无法进行写入

### 5、宕机后自动配置主机

如果没有老大结点，就会谋朝篡位,如果这个时候老大回来了就会自动让位

```bash
slaveof on one #我自己来当老大
```

### 6、哨兵模式

如果故障了根据投票数自动将从库转换为主库

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，就会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。

![image-20210625213616212](img\18.png)

**哨兵有两个作用**

- 通过发送命令，让Redis服务器返回监控器运行状态1，包括主服务器和从服务器
- 当哨兵检测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各哨兵之间还会进行监控，这样就形成了多哨兵模式

假设主服务器宕机，哨兵1先检测到，系统不会马上进行failover过程，仅仅哨兵1主观认为服务器不可用，这个现象为主观下线，当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵会进行一次投票，投票的结果由一个哨兵发起，进行failover【故障转移操作】。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程为客观下线。

#### 1、配置哨兵配置文件sentinel.conf

```bash
sentinel monitor myredis 127.0.0.1 6379 1#监控主机并且由其进行投票
```

后面是数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的成为主机！

#### 2、启动哨兵

如果Master节点断开后如果回来了，只能去当从机。

#### 3、优点

1、哨兵集群，基于主从复制模式，所有的主从配置优先，它都有

2、主从可以切换，故障可以转移，系统可用性更好

3、哨兵模式就是主从模式的升级，手动到自动健壮性更好

#### 4、缺点

1、麻烦

## 10、缓存穿透和雪崩

**一、缓存处理流程**

前台请求，后台先从缓存中取数据，取到直接返回结果，取不到时从数据库中取，数据库取到更新缓存，并返回结果，数据库也没取到，那直接返回空结果。

![img](img\21.png)

### 1、缓存穿透

如果缓存中没有数据就会到数据库中查询。

缓存穿透很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这回给持久层数据库造成很大压力，这就相当于出现了缓存穿透。

#### 1、解决方案

##### 布隆过滤器

进行哈希化，如果不符合就丢弃

![image-20210626213139400](img\19.png)

##### 缓存空对象

![image-20210626213112521](img\20.png)

缓存不命中就会将返回的空对象也缓存起来，同时会设置一个过期时间，之后再访问这个数据会从缓存中获取，保护了后端数据源。

### 2、缓存击穿

 缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力

 **解决方案：**

1. 设置热点数据永远不过期。
2. 加互斥锁
3. ![img](\img\22.png)

说明：

          1）缓存中有数据，直接走上述代码13行后就返回结果了
    
         2）缓存中没有数据，第1个进入的线程，获取锁并从数据库去取数据，没释放锁之前，其他并行进入的线程会等待100ms，再重新去缓存取数据。这样就防止都去数据库重复取数据，重复往缓存中更新数据情况出现。
    
          3）当然这是简化处理，理论上如果能根据key值加锁就更好了，就是线程A从数据库取key1的数据并不妨碍线程B取key2的数据，上面代码明显做不到这点。
### 3、缓存雪崩

 缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，    缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。

 **解决方案**：

1. 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
2. 如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
3. 设置热点数据永远不过期。
