# **数据结构与对象**
| 数据类型 | 可以存储的值 | 操作|
|:-------:| :---------: | :--: |
| String | 字符串、整数或浮点型 | 对整个字符串或者字符串的<br>对整数和浮点数执行自增或自减操作 |
| List | 列表 | 从两端压入或者弹出元素<br>对单个或者多个元素进行修剪、只保留一个范围内的元素  |
|Set|无序集合|添加、获取、移除单个元素<br>检查一个元素是否存在于集合中<br>计算交集、并集、差集<br>从集合里面随机获取元素|
|HASH|包含键值对的无序散列表|添加、获取、移除单个键值对<br>获取所有键值对<br>检查某个键是否存在|
|ZSET|有序集合|添加、获取、删除元素<br>根据分值范围或者成员来获取元素、计算一个键的排名|
### **String**
#### **字符串结构**

```
struct sdshdr{
    //记录buf数组中已使用字节的数量
    int len;
    //记录buf数组中未使用的数量
    int free;
    //字节数组，用于保存字符串
    char buf[];
}
```
#### **常见操作**

```
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> del hello
(integer) 1
127.0.0.1:6379> get hello
(nil)
127.0.0.1:6379>
```
#### **应用场景**
String是最常用的一种数据类型，普通的key/value存储都可以归为此类，value其实不仅是String，也可以是数字：比如想知道什么时候封锁一个IP地址（访问超过几次）。INCRBY命令让这些变得很容易，通过原子递增保持计数。

### **LIST**
#### **数据结构**

```
typedef struct listNode{
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    struct value;
}
```
#### **常见操作**

```
> lpush list-key item
(integer) 1
> lpush list-key item2
(integer) 2
> rpush list-key item3
(integer) 3
> rpush list-key item
(integer) 4
> lrange list-key 0 -1
1) "item2"
2) "item"
3) "item3"
4) "item"
> lindex list-key 2
"item3"
> lpop list-key
"item2"
> lrange list-key 0 -1
1) "item"
2) "item3"
3) "item"
```
#### **应用场景**
Redis list的应用场景非常多，也是Redis最重要的数据结构之一。
我们可以轻松地实现最新消息排行等功能。
Lists的另一个应用就是消息队列，可以利用Lists的PUSH操作，将任务存在Lists中，然后工作线程再用POP操作将任务取出进行执行。

### **HASH**
#### **数据结构**
dictht是一个散列表结构，使用拉链法保存哈希冲突的dictEntry。
```
typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引值
    unsigned long sizemask;
    //该哈希表已有节点的数量
    unsigned long used;
}

typedef struct dictEntry{
    //键
    void *key;
    //值
    union{
        void *val;
        uint64_tu64;
        int64_ts64;
    }
    struct dictEntry *next;
}
```
Redis的字典dict中包含两个哈希表dictht，这是为了方便进行rehash操作。在扩容时，将其中一个dictht上的键值对rehash到另一个dictht上面，完成之后释放空间并交换两个dictht的角色。

```
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```
rehash操作并不是一次性完成、而是采用渐进式方式，目的是为了避免一次性执行过多的rehash操作给服务器带来负担。

渐进式rehash通过记录dict的rehashidx完成，它从0开始，然后没执行一次rehash例如在一次 rehash 中，要把 dict[0] rehash 到 dict[1]，这一次会把 dict[0] 上 table[rehashidx] 的键值对 rehash 到 dict[1] 上，dict[0] 的 table[rehashidx] 指向 null，并令 rehashidx++。

在 rehash 期间，每次对字典执行添加、删除、查找或者更新操作时，都会执行一次渐进式 rehash。

采用渐进式rehash会导致字典中的数据分散在两个dictht中，因此对字典的操作也会在两个哈希表上进行。
例如查找时，先从ht[0]查找，没有再查找ht[1]，添加时直接添加到ht[1]中。


#### **常见操作**

```
> hset hash-key sub-key1 value1
(integer) 1
> hset hash-key sub-key2 value2
(integer) 1
> hset hash-key sub-key1 value1
(integer) 0
> hgetall hash-key
1) "sub-key1"
2) "value1"
3) "sub-key2"
4) "value2"
> hdel hash-key sub-key2
(integer) 1
> hdel hash-key sub-key2
(integer) 0
> hget hash-key sub-key1
"value1"
> hgetall hash-key
1) "sub-key1"
2) "value1"
```

### **SET**
#### **常见操作**

```
> sadd set-key item
(integer) 1
> sadd set-key item2
(integer) 1
> sadd set-key item3
(integer) 1
> sadd set-key item
(integer) 0
> smembers set-key
1) "item2"
2) "item"
3) "item3"
> sismember set-key item4
(integer) 0
> sismember set-key item
(integer) 1
> srem set-key item
(integer) 1
> srem set-key item
(integer) 0
> smembers set-key
1) "item2"
2) "item3"
```
#### **应用场景**
Redis为集合提供了求交集、并集、差集等操作，故可以用来求共同好友等操作。

### **ZSET**
#### **数据结构**
```
    typedef struct zskiplistNode{
        //后退指针
        struct zskiplistNode *backward;
        //分值
        double score;
        //成员对象
        robj *obj;
        //层
        struct zskiplistLever{
            //前进指针
            struct zskiplistNode *forward;
            //跨度
            unsigned int span;
        }lever[];
    }
    
    typedef struct zskiplist{
        //表头节点跟表尾结点
        struct zskiplistNode *header, *tail;
        //表中节点的数量
        unsigned long length;
        //表中层数最大的节点的层数
        int lever;
    }
```
跳跃表，基于多指针有序链实现，可以看作多个有序链表。

![clipboard.png](https://segmentfault.com/img/bVbfNnh)

在查找时，先从上层指针开始查找，找到对应的区间之后再到下一层查找。下图演示了查找22的过程。

![clipboard.png](https://segmentfault.com/img/bVbfNns)

与红黑树等平衡树相比，跳跃表具有以下优点：
- 插入速度非常快速，因为不需要进行旋转等操作来维持平衡性。
- 更容易实现。
- 支持无锁操作。

#### **常见操作**

```
> zadd zset-key 728 member1
(integer) 1
> zadd zset-key 982 member0
(integer) 1
> zadd zset-key 982 member0
(integer) 0
> zrange zset-key 0 -1
1) "member1"
2) "member0"
> zrange zset-key 0 -1 withscores
1) "member1"
2) "728"
3) "member0"
4) "982"
> zrangebyscore zset-key 0 800 withscores
1) "member1"
2) "728"
> zrem zset-key member1
(integer) 1
> zrem zset-key member1
(integer) 0
> zrange zset-key 0 -1 withscores
1) "member0"
2) "982"
```
#### **应用场景**
以某个条件为权重，比如按顶的次数排序
ZREVRANGE命令可以用来按照得分来获取前100名的用户，ZRANK可以用来获取用户排名，非常直接而且操作容易。
Redis sorted set的使用场景与set类似，区别是set不是自动有序的，而sorted set可以通过用户额外提供一个优先级(score)的参数来为成员排序，并且是插入有序的，即自动排序。

# **键的过期时间**
Redis可以为每个键设置过期时间，当键过期时，会自动删除该键。

通过EXPIRE命令或者PEXPIRE命令，客户端可以以秒或者秒精度为数据库中的某个键设置生存时间（Time To Live,TTL），在经过执行的秒数或者毫秒书之后，服务器就会自动删除生存时间为0的键。

对于散列表这种容器，只能为整个键设置过期时间（整个散列键），而不能为键里面的单个元素设置过期时间。

# **持久化**
Redis是内存数据库，为了避免数据意外丢失，需要将内存中的数据持久化到磁盘中。
### **RDB持久化**
在指定的时间间隔内将内存中的数据集快照写入磁盘，恢复时是将快照文件直接读到内存里。

Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替代上次持久化完成的文件。整个过程中，主进程不进行任何IO操作，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据的完整性不是特别敏感，那么RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化的数据可能丢失。

##### **特点**
- 适合大规模的数据恢复
- 对数据的完整性和一致性要求不高
- fork的时候，内存被克隆了一份，大致2倍的膨胀需要考虑
- 将某个时间点的所有数据都存磁盘上（压缩后的二进制文件）。
- 可以将快照复制到其他服务器从而创建具有相同数据的服务器版本。
- 如果系统发生故障，将会丢失最后一次创建快照之后的数据。
- 如果数据量很大，保存的快照时间较长。

### **AOF持久化**
以日志的形式来记录每个写操作，将redis执行过的所有写指令记录下来（读操作不记录），只需追加文件不可以改写文件，redis启动之初会读取该文件重构数据，换言之，redis重启的话就是根据日志文件的内容从前到后执行一次以完成数据的恢复工作。

将命令添加到AOF文件的末尾。

使用AOF持久化需要设置同步选项，从而确保命令什么时候会同步到磁盘文件上。这是因为对文件写入并不会立马将内容同步到磁盘上，而是先存储到缓冲区，案后由操作系统决定什么时候同步到磁盘。有一喜爱同步选项：
|选项	|同步频率|
|-----|------|
|always	|每个写命令都同步|
|everysec	|每秒同步一次|
|no	|让操作系统来决定何时同步|
- always 选项会严重减低服务器的性能。
- everysec 选项比较合适，可以保证系统崩溃时只丢失一秒左右的数据，并且redis每秒执行一次同步对服务器性能几乎没什么影响。
- no 选项并不能给服务器性能带来多大的提升，而且也会增加系统崩溃时数据丢失的数量。

##### **特点**
- 相同数据集的数据而言AOF文件远大于RDB文件，恢复速度慢于RDB.
- AOF运行效率要慢于RDB，每秒同步策略效率较好，不同步效率和RDB相同。

# **事务**
Redis事务指一次执行多个命令，本质是一组命令的集合，一个事物中的所有命令都会被序列化，按顺序地串行执行而不是被其他命令插入，不许加塞。一个队列中，一次性、顺序性、排他性的执行一系列命令。

### **常见命令**
- MULTI：标记一个事务的开始。
- EXEC：执行所有事务块内的命令。
- DISCARD：取消事务，放弃执行事务块中的所有命令。
- WATCH key [key...]：监视一个或多个key，如果在事务执行之前这个key被其他命令所改动，那么事务将被打断。
- UNWATCH：取消WATCH命令对所有key的监视。

Redis事务执行的一些情况：
- 当开启事务之后，队列中出现命令执行错误时，Redis将放弃整个事务。（入列时）
- 执行事务出现错误时，此时执行失败的将会回滚，其他命令依旧执行。

### **特性**
- 单独的隔离操作：事务中的所有命令都会被序列化、按顺序地执行。事务在执行过程中，不会被其他客户端发送的命令请求中断。
- 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的执行，因为事务提交前任何指令都不会被实际执行。
- 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。

### 问题
- 一段代码要执行多个redis命令，不加锁的情况下如何保持一致性？

# **主从复制**
Redis的复制也就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，master已写为主，slaver已读为主。

### **实现**
- 配置规则：配从不配主
- 从库配置：slaveof 主库ip 端口，slaveof 192.168.123.100 6379。
- 每次slaver与master断开之后都得重新连接，除非写进配置文件redis.conf
- 使用info replication查看当前server的s-m关系。

### **特点**
- 一旦s-m replicaiton形成，s将共享m的所有数据，连接之后从头开始复制所有数据。
- s只有读权限。
- s-m数据存在一定延迟。
- m宕机之后，从机将原地待命并保留m所有的数据（m最后一次的数据保存可能会丢失），主机重新连接之后与从机重新建立s-m replication关系，并且主机的新添记录仍能同步到从机。
- 可通过slaveof no one命令使当前数据库停止与其他数据的同步，转成主数据库。

# **Sentinel**
Sentinel是Redis高可用的解决方案，由一个或多个Sentinel实例组成的Sentinel系统可以监视多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进行下线时自动将属下的某个从服务器升级为新的主服务器，然后由新的主服务器代替已下线的主服务器继续处理命令请求。

Redis提供的Sentinel(哨兵)机制，通过Sentinel模式启动Redis后，自动监控master/slaver的运行状态，基本原理：心跳机制和投票裁决。

- 监控：Sentinel会不断的检查主服务器和从服务器是否正常。
- 提醒：当被监控的某个Redis服务器出现问题之后，Sentinel可以通过API向管理者或者其他的应用程序发送通知。
- 自动故障迁移：当一个主服务器不能正常工作时，Sentinel会开始一次自动故障操作，它会将失效的主服务器的其他一台从服务器升级为新的主服务器，并让失效的主服务器的其他从服务器改为复制新的主服务器。当客户端试图连接失效的主服务器时，集群也会向客户端返回新服务器的地址，使得集群可以使用新主服务器代替失效服务器。

### **Sentinel主从原理**
##### **服务器与Sentinel系统**
![clipboard.png](https://segmentfault.com/img/bVbgmYQ)

##### **主服务器下线**
![clipboard.png](https://segmentfault.com/img/bVbgmYR)

##### **故障转移**
![clipboard.png](https://segmentfault.com/img/bVbgmYS)

##### **原来的M重新进入s-m replicaiton将自动降为s**
![clipboard.png](https://segmentfault.com/img/bVbgmYT)

# **参考**
- Redis设计与实现
- [Redis 讲解系列之 Redis的持久化][1]
- [Redis 讲解系列之 Redis的事务][2]
- [Redis 讲解系列之 Redis的复制(一)][3]
- [Redis 讲解系列之 Redis的复制(二)][4]
- [Redis事务](https://www.cnblogs.com/jason-xiang/p/5364252.html)

  [1]: https://blog.csdn.net/u012437781/article/details/78215590
  [2]: https://blog.csdn.net/u012437781/article/details/78216882
  [3]: https://blog.csdn.net/u012437781/article/details/78225246
  [4]: https://blog.csdn.net/u012437781/article/details/78227238
