# 基础架构及术语

![https://pic1.zhimg.com/v2-4692429e9184ed4a93911fa3a1361d28_b.jpg](https://pic1.zhimg.com/v2-4692429e9184ed4a93911fa3a1361d28_b.jpg)
## 主题和分区
topic是发布记录的类别或订阅源名称。 Kafka的topic总是多用户; 也就是说，一个主题可以有零个，一个或多个消费者订阅写入它的数据。

对于每个主题，Kafka群集都维护一个如下所示的分区日志：

![https://segmentfault.com/img/bVbjGIj?w=416&h=267](https://segmentfault.com/img/bVbjGIj?w=416&h=267)

每个分区都是一个有序的，不可变的记录序列，不断附加到结构化的提交日志中。 分区中的记录每个都分配了一个称为offset的顺序ID号，它唯一地标识分区中的每个记录。

Kafka集群持久保存所有已发布的记录 - 无论是否已使用 - 使用可配置的保留期。 例如，如果保留策略设置为两天，则在发布记录后的两天内，它可供使用，之后将被丢弃以释放空间。 Kafka的性能在数据大小方面实际上是恒定的，因此长时间存储数据不是问题。

![https://segmentfault.com/img/bVbjGKP?w=2041&h=1243](https://segmentfault.com/img/bVbjGKP?w=2041&h=1243)

实际上，基于每个消费者保留的唯一元数据是该消费者在日志中的偏移或位置。 这种偏移由消费者控制：通常消费者在读取记录时会线性地提高其偏移量，但事实上，由于该位置由消费者控制，因此它可以按照自己喜欢的任何顺序消费记录。 例如，消费者可以重置为较旧的偏移量来重新处理过去的数据，或者跳到最近的记录并从“现在”开始消费。（offset由comsumer决定，comsumer可以决定offset的数据）。


## 生产者和消费者

**生产者**将数据发布到他们选择的topic。 生产者负责选择分配给topic中哪个partition的记录。 这可以以round-robin方式完成，仅仅是为了balance load，或者可以根据一些语义分区功能（例如基于记录中的某些键）来完成。


**消费者**使用消费者组名称标记自己，并且发布到主题的每个记录被传递到每个订阅消费者组中的一个消费者实例。 消费者实例可以在不同的进程中，也可以在不同的机器。

如果所有使用者实例具有相同的使用者组，则记录将有效地在使用者实例上进行负载平衡。

如果所有消费者实例具有不同的消费者组，则每个记录将广播到所有消费者进程。
![https://segmentfault.com/img/bVbei9M?w=474&h=252](https://segmentfault.com/img/bVbei9M?w=474&h=252)

## broker和集群
一个独立的Kafka服务器被称为broker，broker是集群的组成部分，每个集群都有一个broker充当了集群控制者的角色（自动从集群活跃成员中选举出来），**控制者负责管理工作，包括将分区分配给broker和监控broker。**

# 容错

日志的分区分布在Kafka集群中的服务器上，每个服务器处理数据并请求分区的共享。 每个分区都在可配置数量的服务器上进行复制，以实现容错。（多个服务器保存partition数据，实现容错）

每个分区都有一个服务器充当“领导者”，零个或多个服务器充当“追随者”。 领导者处理分区的所有读取和写入请求，而关注者被动地复制领导者。 如果领导者失败，其中一个粉丝将自动成为新的领导者。 每个服务器都充当其某些分区的领导者和其他服务器的追随者，因此负载在群集中得到很好的平衡。（高性能实现）

# 再均衡
在kafka中，当消费者发生崩溃或者有新的消费者加入时，将会触发**再均衡**。

消费者会像叫做_consumer_offset的特殊主题发送消息，消息内包含灭个分区的偏移量。

再均衡之后，消费者可能分配到新的分区，为了能够继续之前的工作，消费者需要读取每个分区最后一次提交的offset，但如果提交的offset小于客户端处理的最后一个offset，那么消息将会被重复处理。

KafkaConsumer API提供了很多种方式来提交偏移量。

## 自动提交
最简单的提交方式，将enable.auto.commit设为true，提交时间为auto.commit.interval.ms设置的值，默认为5s。每过5s，消费者会自动把从poll()方法接收的最大offset提交上去。

这种方式虽然简单，但是并没有避免重复处理消息的问题。(在5s内发生再均衡)

## 提交当前offset
消费者API提供了另一种提交偏移量的方式，开发者可以在必要的时候提交当前的偏移量而不是基于时间间隔。通过调用`commitSync()`或者`commitAsync()`方法进行提交。

- `commitSync()`在broker回应之前一直阻塞，限制了应用程序的吞吐量。
- `commitAsync()`是异步提交的方式，但是commitAsync()无法保证一定成功，commitSync()在成功提交或者遇到无法恢复的错误之前会一直重试，但是commitAsync()不会。之所以不进行重试是因为它收到响应之前可能另一个更大的offset提交成功了。commitAsync同时也支持回调。
# kafka为什么这么快

- [为什么kafka那么快](https://mp.weixin.qq.com/s?__biz=MzIxMjAzMDA1MQ==&mid=2648945468&idx=1&sn=b622788361b384e152080b60e5ea69a7#rd&utm_source=tuicool&utm_medium=referral)
- [什么是Zero-Copy？](https://blog.csdn.net/u013256816/article/details/52589524)
- [Kafka副本同步机制理解](https://blog.csdn.net/lizhitao/article/details/51718185)
- [Kafka深度解析](http://www.jasongj.com/2015/01/02/Kafka%E6%B7%B1%E5%BA%A6%E8%A7%A3%E6%9E%90/)



# 幂等性
### 唯一id去重


## kafka快速的原因

### 顺序写入
磁盘每次写入都会寻址->写入，其中寻址是最耗时的，所以对磁盘来说随机I/O是最慢的，为了提高磁盘的读写速度，kafka就是使用顺序I/O。

[http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1](http://mmbiz.qpic.cn/mmbiz/nfxUjuI2HXjiahgInoFXLfVoghamdPiaBafMYrFo4rFUykuic1Ks0P5DBzxjVfzgYlscCWeicNnE3HrSKxJkCxOcEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)