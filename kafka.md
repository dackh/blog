# 基础架构及术语

![https://pic1.zhimg.com/v2-4692429e9184ed4a93911fa3a1361d28_b.jpg](https://pic1.zhimg.com/v2-4692429e9184ed4a93911fa3a1361d28_b.jpg)

### 主题和分区
topic是发布记录的类别或订阅源名称。 Kafka的topic总是多用户; 也就是说，一个主题可以有零个，一个或多个消费者订阅写入它的数据。

对于每个主题，Kafka群集都维护一个如下所示的分区日志：

![https://segmentfault.com/img/bVbjGIj?w=416&h=267](https://segmentfault.com/img/bVbjGIj?w=416&h=267)

每个分区都是一个有序的，不可变的记录序列，不断附加到结构化的提交日志中。 分区中的记录每个都分配了一个称为offset的顺序ID号，它唯一地标识分区中的每个记录。

Kafka集群持久保存所有已发布的记录 - 无论是否已使用 - 使用可配置的保留期。 例如，如果保留策略设置为两天，则在发布记录后的两天内，它可供使用，之后将被丢弃以释放空间。 Kafka的性能在数据大小方面实际上是恒定的，因此长时间存储数据不是问题。

![https://segmentfault.com/img/bVbjGKP?w=2041&h=1243](https://segmentfault.com/img/bVbjGKP?w=2041&h=1243)

实际上，基于每个消费者保留的唯一元数据是该消费者在日志中的偏移或位置。 这种偏移由消费者控制：通常消费者在读取记录时会线性地提高其偏移量，但事实上，由于该位置由消费者控制，因此它可以按照自己喜欢的任何顺序消费记录。 例如，消费者可以重置为较旧的偏移量来重新处理过去的数据，或者跳到最近的记录并从“现在”开始消费。（offset由comsumer决定，comsumer可以决定offset的数据）。


### 生产者和消费者

**生产者**将数据发布到他们选择的topic。 生产者负责选择分配给topic中哪个partition的记录。 这可以以round-robin方式完成，仅仅是为了balance load，或者可以根据一些语义分区功能（例如基于记录中的某些键）来完成。


**消费者**使用消费者组名称标记自己，并且发布到主题的每个记录被传递到每个订阅消费者组中的一个消费者实例。 消费者实例可以在不同的进程中，也可以在不同的机器。

如果所有使用者实例具有相同的使用者组，则记录将有效地在使用者实例上进行负载平衡。

如果所有消费者实例具有不同的消费者组，则每个记录将广播到所有消费者进程。
![https://segmentfault.com/img/bVbei9M?w=474&h=252](https://segmentfault.com/img/bVbei9M?w=474&h=252)

### broker和集群
一个独立的Kafka服务器被称为broker，broker是集群的组成部分，每个集群都有一个broker充当了集群控制者的角色（自动从集群活跃成员中选举出来），**控制者负责管理工作，包括将分区分配给broker和监控broker。**

# 容错

日志的分区分布在Kafka集群中的服务器上，每个服务器处理数据并请求分区的共享。 每个分区都在可配置数量的服务器上进行复制，以实现容错。（多个服务器保存partition数据，实现容错）

每个分区都有一个服务器充当“领导者”，零个或多个服务器充当“追随者”。 领导者处理分区的所有读取和写入请求，而关注者被动地复制领导者。 如果领导者失败，其中一个粉丝将自动成为新的领导者。 每个服务器都充当其某些分区的领导者和其他服务器的追随者，因此负载在群集中得到很好的平衡。（高性能实现）

# zookeeper在kafka中的作用
### Broker注册
在Zookeeper上会有一个专门用来进行Broker服务器列表记录的节点，节点路径为/broker/ids。

每个Broker服务器在启动的时候都会在Zookeeper上进行注册，节点路径为/broker/ids/[0-N]。

Broker创建的是临时节点，一旦Broker服务器宕机或者下线，那么对应的Broker节点也就会被删除。

### Topic注册
在Kafka中，同一个Topic的多个分为会分不到不同的Broker上，而这些分区信息与Broker的对应关系都是由Zookeeper维护的，节点为`/broker/topics`。Kafka每个Topic都会以`/broker/topic/[topic]`的形式记录在该节点，例如：`/broker/topics/login`。

Broker服务器启动后，会到对应的Topic节点下注册自己的BrokerID，并写入针对该Topic的分区总数。例如：/broker/topics/login/3 -> 2，这个节点表示Broker ID为3的一个Broker服务器对于`login`这个Topic的消息提供了2个分区进行消息存储。同样，这个分区数节点也是临时节点。

### 生产者负载均衡
因为同一个Topic的消息进行分区并将其分布到不同的Broker服务器上，因此生产者发送到Broker需要进行负载均衡。

在Kafka中，客户端使用了基于Zookeeper的负载均衡策略来解决生产者的负载均衡问题。Kafka的生产者会对Zookeeper上的`Broker的新增或减少`、`Topic的新增或减少`和`Broker与Topic关联关系的变化`等事件注册Watcher监听，这样就可以实现动态的负载均衡机制了。

在此模式下，还允许开发人员控制生产者根据一定的规则(例如消费者的消费行为)进行数据分区。

通过Zookeeper的Watcher机制通知能够让生产者动态地获取Broker和Topic的变化情况。

### 消费者负载均衡
因为不同的消费者组消费自己特定的Topic下面的消息，互不干扰，也不需要互相协调。因为消费者的负载均衡可以看作同一个消费者分组内部的消息消费策略。

##### 消费者分区与消费者关系
每个消费者分组，Kafka都会为其分配一个全局唯一的GroupID，并且为每个消费者分配一个ComsumerID。在Kafka中每个分区只能由一个消费者进行消息的消费，因此，Zookeeper上记录下消息分区与消费者之间的对应关系。
每个消费者一旦确定了消息分区的消费权利，那么需要将其ConsumerID写入到对应消息分区的临时节点上，例如`/consumer/[group_id]/owners/[topic]/[broker_id-partition_id]`就是一个消息分区的标识，节点内容就是消费该分区上消息的消费者的ConsumerID

##### Offset记录
在消费者指定消息分区进行消息消费的过程中，需要将Offset记录到Zookeeper上去，以便在该消费者重启或是其他消费者重新接管该消息分区的消息消费后，可以从之前的进度开始继续进行消息的消费。Offset的节点路径为`/consumer/[group_id]/offsets/[topic]`
`/broker_id-patition_id]`，其节点内容就是Offset的值。

##### 消费者注册
每个消费者服务在启动的时候，都会到Zookeeper的指定节点下创建一个属于自己的消费者节点，例如`/consumer/group_id/ids/[consumer_id]`
完成节点创建后，消费者将订阅自己的Topic信息写入该节点。该节点是一个临时节点，当消费者服务器下线后，对应消费者节点就会被删掉。

每个消费者都需要关注所属消费者组中消费者服务器的变化情况，即对`/comsumer/[group_id]/ids`节点注册子节点的Watcher监听，一旦发现消费者新增或减少，就会触发消费者负载均衡。

消费者需要对`/broker/ids[0...N]`中的进行监听的注册，如果发现Broker服务器列表发生变化，那么就根据具体情况决定是否需要进行消费者的负载均衡。

# 集群成员之间的关系
kafka通过zookeeper来维护集群成员之间的关系，每个broker都有一个唯一标识符，这个标识符可以配置里指定，也可以自动生成。在broker启动的时候，通过创建zookeeper的临时节点把自己的ID注册到zookeeper。kafka组件订阅zookeeper的/brokers/ids路径，当有broker加入集群或者退出集群时，这个组件就会获得通知。

在broker停机、出现网络分区或长时间垃圾回收停顿时，broker会从zookeeper上断开连接，此时broker在启动时创建的临时节点会自动从zookeeper移除，监听broker列表的kafka组件会被告知该broker已移除。

在broker关闭之后，它的节点会消息，但是ID会继续存在于其他数据结构中，在之后如果使用相同ID启动一个全新的broker，它会立刻加入集群并且拥有与旧broker相同的分区和主题。

### 控制器
控制器也是broker，同时还负责**分区首领的选举**。集群中第一个启动的broker通过在zookeeper里创建一个临时节点/controller让自己成为控制器。其他节点在该节点上创建watch对象，通过这种方式确保集群里只有一个控制器存在。

 

### 分区复制

kafka中的每个分区都有多个副本，这些副本保存在broker中，副本可以分为：
- 首领副本(leader)：所有生产者消费者请求都会经过这个副本。
- 跟随者副本(follower)：从首领复制消息，保持与首领一致。

leader通过查看每个follower请求的最新offset了解follower进度，**如果follower在10s(通过replica.lag.time.max.ms配值)内没有请求最新消息，那么它被认为不同步**。只有同步的follower才能在当前leader宕机时被选定为新的leader。

每个分区都会有一个首选leader——创建主题时选定的首领就是分区的首选首领，默认情况下，kafka的auto.leader.rebalance.enable被设为true，它会检查首选首领是不是当前首领，如果不是，并且该副本同步，那么就会触发首领选举，让首选首领成为当前首领。



# 幂等性
为了实现Producer的幂等性，kafka引入了Producer ID（即PID)和Sequence Number
### PID
每个producer在初始化的时候都会被分配一个唯一的PID，这个PID对应用透明，完全没有暴露给用户，对于一个给定的PID，sequence number将会从0开始自增，每个topic-partition都会有一个独立的sequence number，producer在发送数据时，将会给msg标识一个sequence number，Server也就是通过这个验证数据是否重复，这里的PID是全局唯一，producer故障后重新启动后会被分配一个新的PID，这也是幂等性无法做到夸会话的一个原因。

### Sequence Number
有PID之后，在PID+topic-partition级别上添加一个sequence numbers信息，就可以实现producer的幂等性。

### 幂等性前后对比
#### 前
![https://upload-images.jianshu.io/upload_images/12038882-90e207f78b291b8e.png](https://upload-images.jianshu.io/upload_images/12038882-90e207f78b291b8e.png)
#### 后
![https://upload-images.jianshu.io/upload_images/12038882-014066219b43e8fc.png](https://upload-images.jianshu.io/upload_images/12038882-014066219b43e8fc.png)


# 持久化
kafka很大程度上依赖文件系统来存储和缓存消息，有一普遍的认识：磁盘很慢。但是其实顺序的磁盘读写比任意内存读写都快。

基于JVM内存有一下缺点：
- 对象的内存开销很大，通常会让存储数据的大小加倍
- 随着堆内数据的增加，GC的速度越来越慢，而且可能导致错误

基于OS的文件系统设计有一下好处：
- 可以通过os的pagecache来有效利用主内存空间，由于数据紧凑，可以cache大量数据，并且没有gc的压力
- 即使服务重启，缓存中的数据也是热的（不需要预热）。而基于进程的缓存，需要程序进行预热，而且会消耗很长的时间。（10G大概需要10分钟）
- 大大简化了代码。因为在缓存和文件系统之间保持一致性的所有逻辑都在OS中。以上建议和设计使得代码实现起来十分简单，不需要尽力想办法去维护内存中的数据，数据会立即写入磁盘。

### 数据持久化
- 发现线性的访问磁盘（即：按顺序的访问磁盘），很多时候比随机的内存访问快得多，而且有利于持久化
- 传统的使用内存做为磁盘的缓存
- Kafka直接将数据写入到日志文件中，以追加的形式写入

### 日志数据持久化
- 写操作：通过将数据追加到文件中实现
- 读操作：读的时候从文件中读就好了

### 优势
- 读操作不会阻塞写操作和其他操作（因为读和写都是追加的形式，都是顺序的，不会乱，所以不会发生阻塞），数据大小不对性能产生影响；
- 没有容量限制（相对于内存来说）的硬盘空间建立消息系统；
- 线性访问磁盘，速度快，可以保存任意一段时间！

# kafka快速的原因

### 顺序写入
磁盘每次写入都会寻址->写入，其中寻址是最耗时的，所以对磁盘来说随机I/O是最慢的，为了提高磁盘的读写速度，kafka就是使用顺序I/O。

[http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1](http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


# 生产者
## 发送消息到kafka
- 同步发送，producer.send().get()  ，producer.send()返回一个Future对象，通过调用Future对象的get()方法等待kafka响应，如果服务返回错误，get()方法会抛出异常，get() 返回一个RecordMetadata对象，可以用它获取消息的偏移量。
- 异步发送，大多数情况下并不需要等待发送消息的响应，不过在遇到消息发送失败的时候，还是需要对错误进行处理。生产者提供了回调支持，producer.send(record,new CallBack{})，其中record是ProductRecord对象，指消息内容，CallBack就是回调接口。

## 序列化
默认未字符串序列化器，kafka还提供了整形和字节数组序列化器，同时也支持自定义序列化序列化器。
常见的序列化器包括json、avro、protobuf等
- [序列化和反序列化 - 美团技术团队](https://tech.meituan.com/2015/02/26/serialization-vs-deserialization.html)

## 分区
生产者在发送的ProductRecord对象中包含了目标主题、键和值，Kafka消息是一个个键值对，ProducerRecord对象可以只包含目标主题和值，键可以设置为默认的null,不过大多数情况下都会用到键。
键主要有两个用途：
- 作为消息的附加信息。
- 决定消息该被写到主题的哪个分区，拥有相同键的消息将被写到同一个分区。

如果键值为null，并且使用默认的分区器，那么记录将被随机地发送到主题的各个分区可用分区上，分区器使用轮询算法。
如果键值不为null，并且使用默认的分区器，那么Kafka会对键进行散列，然后根据散列值把消息映射到特定的分区上。即同一个键总是会被映射到同一个分区。

# 消费者

## 再均衡
在kafka中，当消费者发生崩溃或者有新的消费者加入时，将会触发**再均衡**。

消费者会像叫做_consumer_offset的特殊主题发送消息，消息内包含灭个分区的偏移量。

再均衡之后，消费者可能分配到新的分区，为了能够继续之前的工作，消费者需要读取每个分区最后一次提交的offset，但如果提交的offset小于客户端处理的最后一个offset，那么消息将会被重复处理。

KafkaConsumer API提供了很多种方式来提交偏移量。

### 自动提交
最简单的提交方式，将enable.auto.commit设为true，提交时间为auto.commit.interval.ms设置的值，默认为5s。每过5s，消费者会自动把从poll()方法接收的最大offset提交上去。

这种方式虽然简单，但是并没有避免重复处理消息的问题。(在5s内发生再均衡)

### 提交当前offset
消费者API提供了另一种提交偏移量的方式，开发者可以在必要的时候提交当前的偏移量而不是基于时间间隔。通过调用`commitSync()`或者`commitAsync()`方法进行提交。

- `commitSync()`在broker回应之前一直阻塞，限制了应用程序的吞吐量。
- `commitAsync()`是异步提交的方式，但是commitAsync()无法保证一定成功，commitSync()在成功提交或者遇到无法恢复的错误之前会一直重试，但是commitAsync()不会。之所以不进行重试是因为它收到响应之前可能另一个更大的offset提交成功了。commitAsync同时也支持回调。


# kafka为什么这么快

- [为什么kafka那么快](https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945468&idx=1&sn=b622788361b384e152080b60e5ea69a7#rd&utm_source=tuicool&utm_medium=referral)
- [什么是Zero-Copy？](https://blog.csdn.net/u013256816/article/details/52589524)
- [Linux 中的零拷贝技术](https://zhuanlan.zhihu.com/p/66595734)
- [Kafka副本同步机制理解](https://blog.csdn.net/lizhitao/article/details/51718185)
- [Kafka深度解析](http://www.jasongj.com/2015/01/02/Kafka%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90/)



# 参考
- kafka权威指南 
- 从Pasox到Zookeeper
- [http://kafka.apache.org/intro](http://kafka.apache.org/intro "http://kafka.apache.org/intro")
- [Kafka的内部机制深入(持久化，分布式，通讯协议)](https://www.cnblogs.com/pony1223/p/9807715.html)