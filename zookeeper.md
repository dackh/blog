# **ZooKeeper**
ZooKeeper是一种为分布式应用所设计的高可用、高性能且一致的开源协调服务，它提供了一项基本服务：**分布式锁服务**。 由于ZooKeeper的开源特性，后来开发者在分布式锁的基础上，摸索出了其他的使用方法：配置维护、组服务、分布式消息队列、分布式通知/协调等。

> ZooKeeper性能上的特点决定了它能够在大型的、分布式的系统当中应用。从可靠性来讲，它并不会因为一个节点的错误而崩溃。除此之外，它严格的序列访问控制意味着复杂的控制原语可以应用在客户端上。Zookeeper在一致性、可用性、容错性的保证，也是zookeeper的成功之处。它获得的一切成功都与它采用的协议-Zab协议密不可分。

Zookeeper所提供的服务主要通过：数据结构+原语+Watcher机制。

# **数据结构**
### **数据模型Znode**
Zookeeper拥有一个层次的命名空间，这个和标准的文件系统非常相似。

![clipboard.png](https://segmentfault.com/img/bVbgO8D)

Zookeeper树中的每个节点都被称为——Znode。和文件系统的目录树一样，Zookeeper树种的每个节点可以拥有子节点。但也有不同之处。
#### **引用方式**
通过路径引用，但是路径必须绝对的，因此它们必须由/字符开头。而且路径必须是唯一的。
#### **Znode结构**
Znode兼文件和目录两种特点，即像文件一样维护着数据、元信息、ACL、时间戳等数据结构，又像目录一样可以作为路径标识的一部分吗。每个Znode由3部分组成：
- stat：此为状态信息，描述该Znode的版本，权限等信息。
- data：与该Znode关联的数据。
- children：该Znode下的子节点。
Zookeeper虽然可以关联一些数据，但并没有被设计成常规的数据库或者大数据存储，相反用来管理调度数据，比如分布式的配置文件信息、状态信息、汇集位置等。 
#### **数据访问**
Zookeeper中每个节点存储的数据要被原子操作。另外每个节点都拥有自己的ACL（访问控制列表），这个列表规定了用户的权限，即限定了特定用户对目标节点可以执行的操作。
#### **节点类型**
- 临时节点：生命周期依赖于创建它们的会话。不允许拥有子节点。
- 永久节点：该节点的生命周期不依赖于会话，并且只有在客户端显示执行删除操作的时候，他们才能被删除。
#### **顺序节点**
当创建Znode的时候，用户可以请求在Zookeeper的路径结尾中添加一个**递增的计数**。这个计数对于此节点的父节点来说是唯一的。当计数大于2^32-1时，计数器将溢出。
#### **观察**
客户端可以在节点设置watch，我们称为监视器。当节点状态发生改变时（Znode的增、删、改）将会触发watch所对应的操作。当watch被触发时，Zookeeper将会向客户端发送且仅发送一条通知，因为watch只能被触发一次，这样可以减少网络流量。

### **Zookeeper中的时间**
Zookeeper有多种记录时间的形式，其中包含以下几个主要属性：
#### **Zxid**
致使Zookeeper节点状态改变的每一个操作都将使节点接收到一个Znode格式的时间戳，并且这个时间戳全局有序。也就是说，每个节点的改变都会产生一个唯一的Zxid。如果Zxid1的值小于Zxid2的值，那么Zxid1所对应的事件发生在Zxid2所对应的事件之前。实际上，Zookeeper的每个节点维护着三个Zxid的值。分别为：cXzid、mXzid、pXzid。
- cXzid：是节点的创建时间所对应的Zxid格式时间戳。
- mXzid：是节点的修改时间所对应的Zxid格式时间戳。
实际上Zxid是一个64位的数字。它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch。低32位是一个递增计数。
#### **版本号**
对节点的每一个操作都将致使这个节点的版本号增加。每个节点维护着三本版本号，他们分别为：
- version：节点数据版本号
- cversion：子节点版本号
- aversion：节点所拥有的ACL版本号
#### **节点属性**

![clipboard.png](https://segmentfault.com/img/bVbgPme)

#### **服务中的操作**
在Zookeeper中有9个基本操作，如下图所示：

![clipboard.png](https://segmentfault.com/img/bVbiTTv)
- 更新Zookeeper操作是有限制的，delete或setData必须明确要更新的Znode的版本号，我们可以调用exists找到，如果版本号不匹配，更新将会失败。
- 更新Zookeeper操作是非阻塞的，因客户端如果丢失了一个更新（由于另一个进程在同时更新这个Znode），它可以在不阻塞其他进程执行的情况下，选择重新尝试或进行其他操作。
- 尽管Zookeeper可以被看做是一个文件系统，但是处于便利，摒弃了一些文件系统的操作原语。因为文件非常的小并且使整体读写的、所以不需要打开、关闭或是寻地的操作。

### **Watch触发器**
#### **watch概述**
Zookeeper可以为所有的读操作设置watch，这些读操作包括：exists()、getChildren()和getData()。watch事件是一次性的触发器，当watch的对象状态发生改变时，将会触发此对象watch所对应的事件。watch事件将被异步地发送给客户端，并且Zookeeper为watch机制提供了**有序的一致性保护**。理论上，客户端接收watch事件的时间快于其看到watch对象状态变化的时间。
#### **watch类型**
Zookeeper所管理的watch可以分为两类：
- 数据watch(data watches)：getData和exists负责设置数据watch
- 孩子watch(child watches)：getChildren负责设置孩子watch
可以通过操作**返回的数据**来设置不同的watch：
- getdata和exists：返回关于节点的数据信息
- getChildren：返回孩子列表
因此
- 一个成功的setData操作将触发Znode的数据watch
- 一个成功的create操作将会触发Znode的数据watch以及孩子watch
- 一个成功的delete操作将会触发数据watch以及孩子watch
#### **watch注册与触发**

![clipboard.png](https://segmentfault.com/img/bVbiUeT)

- exists操作上的watch，在被监视的Znode创建、删除或数据更新时被触发。
- getData操作上的watch，在背监视的Znode删除或数据更新时被触发。在被创建时不能被触发，因为只有一个Znode存在，getData操作时才会成功。
- getChildren操作上的watch，在被监视的Znode的子节点创建或删除，或是这个Znode自身被删除时被触发。可以通过查看watch事件类型来区分是Znode，还是他的子节点被删除：NodeDelete表示Znode被删除，NodeDeletedChanged表示子节点被删除。


# **Leader选举**
Leader选举时保证数据一致性的的关键所在。当Zookeeper集群中的一台服务器出现一下情况之一时，需要进入Leader选举。
- 服务器初始化启动。
- 服务器运行时无法和Leader保持连接。
### **服务器启动时期的Leader选举**
若进行Leader选举，则至少需要两台机器，这里选举3台机器组成的服务器集群为例。在集群初始化阶段，当有一台服务器Server1启动时，其单独无法进行和完成Leader选举，当第二台机器Server2启动时，此时两台机器可以相互通信，每台机器都试图找到Leader，于是进入Leader选举过程。
- **每台Server发出一个投票**，由于初始阶段，Server1和Server都会将自己作为Leader服务器来进行投票，每次投票会包含所推举服务器的myid和Zxid，使用`(myid,Zxid)`来表示，此时Server1的投票为(1,0)，Server2的投票为(2,0),然后各自将这个投票发给集群中其他机器。
- **接受来自其他各个服务器的投票**，集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票，是否来自LOOKING状态的服务器。
- **处理投票**，针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下：
    - 优先检查Zxid，Zxid较大的服务器优先作为Leader。
    - 如果Zxid相同，那么久比较myid。myid较大的服务器作为Leader服务器。
- **统计投票**，每次投票后，服务器都会统计投票信息，判断是否有过半机器接受到相同的投票信息，对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2,0)的投票信息，此时便认为选出了Leader。
- **改变服务器状态**。一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么久变更为FOLLOWING，如果是Leader，就变更为LEADING。 

### **服务器运行期间的Leader选举**
在Zookeeper运行期间，Leader与非Leader服务器各司其职，即便当有非Leader服务器宕机或新加入，此时也不会影响Leader，但是一旦Leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮Leader选举，其过程和启动期间选举过程基本一致。
假设正在运行的有Server1、Server2、Server3三台服务器，当前Leader是Server2，若某一时刻Leader挂了，此时便开始Leader选举。
- **变更状态**，Leader挂后，余下非Observer服务器都会将自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。
- **每个Server会发出一个投票**，在运行期间，每个服务器的Zxid可能不同，此时假定Server1的Zxid为123，Server3的Zxid为122，在第一轮投票中，Server1和Server2都会投自己，产生投票(1,123)，(3,122)，然后各自将投票发送给集群中所有机器。
- **接收来自各个服务器的投票**，与启动时过程相同。
- **处理投票**，此时Server1将成为Leader。
- **统计投票**，与启动过程相同。
- **改变服务器状态**，与启动过程相同。

# **Zookeeper的CP特性**
- **不能保证每次服务请求的可用性**。任何时刻对Zookeeper的访问请求能得到的一致的数据，同时系统对网络分割具备容错性，但是它不能保证每次服务请求的可用性。
- **进行leader选举的时候集群式不可用的**，在使用Zookeeper获取服务列表时，当master节点因为网络故障与其他节点失去联系时，剩余节点会重新进行leader选举。问题在于，选举的时间太长，30~120s，且选举期间整个Zookeeper集群都是不可用的，这就导致在选举期间服务瘫痪，虽然服务能够最终恢复，但是漫长的选举时间是不能容忍的。所以说，Zookeeper不能保证服务可用性。  

# **参考**
[Zookeeper学习第一期][1]
[【分布式】Zookeeper的Leader选举][2]
[Zookeeper的CP特性][3]


  [1]: https://www.cnblogs.com/sunddenly/p/4033574.html
  [2]: http://www.cnblogs.com/leesf456/p/6107600.html
  [3]: https://www.jianshu.com/p/4445842be566