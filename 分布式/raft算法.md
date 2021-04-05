## 一致性
分布式存储系统通常通过维护多个副本来进行容错，提高系统的可用性。要实现此目标，就必须解决分布式系统的核心问题：维护多个副本的一致性。

那什么是一致性：
它是构建容错性的分布式系统基础。在一个具有一致性的集群里面，同一时刻所有的节点对存储在其中的某个值都有相同的结果。

一致性协议通常给予replcation state machines，即所有节点都从同一个state出发，都经过同样的一系列操作系列（log），最后达到同样的state。
## Raft算法
Raft中的角色分为：领导者（leader）、跟从者（follower）和候选人（candidate)。
- leader：所有请求的处理者，本地处理后再同步至多个其他副本；接受客户端请求，并向follower同步请求日志，当日志同步到大多数节点上后告诉follower提交日志。
- follower：请求的被动更新者；接受并持久化leader同步的日志，在leader告之日志可以提交之后，提交日志。
- condidate：leader选举中的临时角色。如果follower副本在一段时间之内没有收到leader的心跳，则判断leader可能已经故障，此时启动选主过程，此时副本会变成candidate状态，直到选举结束。

Raft将一致性分解为多个子问题：leader选举（leader election)、日志同步（log replication）、安全性（safely）、日志压缩（log compaction）、成员变更（membership change）等。

## 架构图
![https://pic2.zhimg.com/v2-e747e997d58aee9252b187c8434a1cc1_b.jpg](https://pic2.zhimg.com/v2-e747e997d58aee9252b187c8434a1cc1_b.jpg)
系统中每个节点都有三个组件：
- 状态机：当我们说一致性的时候，实际就是在说要保证这个状态机的一致性，状态机会从log里面取出所有命令，然后执行一遍，得到的结果就是我们对外提供的保证一致性的数据。
- log：保存了所有修改记录。
- 一致性模块：一致性模块算法就是用来保证写入的log的命令一致性，这也是raft算法的核心内容。

## leader选举

![https://pic2.zhimg.com/v2-40d42747bec5c00503e4bd47566beb65_b.jpg](https://pic2.zhimg.com/v2-40d42747bec5c00503e4bd47566beb65_b.jpg)

Raft算法将时间分为一个个的任期（term），每一个term的开始都是leader选举，在成功选举leader之后，leader会在整个term内管理整个集群。如果leader选举失败，该term就会因为没有leader而结束。

Raft使用心跳（heartbeat）出发leader选举，当服务器启动时，初始化为follower，leader向所有follower周期性发送heartbeat。如果follower在选举超市时间内没有收到leader的heartbeat，就会等待一段随机的时间后发起一次leader选举。

follwer将当前term加一然后转换为candidate，它首先自己投票给自己并且给集群中的其他服务器发送RequestVote RPC。

结果有三种情况：
- 赢得多数的选票，成功选举为leader。
- 收到了leader的消息，表示有其他服务器已经强选成为了leader。
- 没有服务器赢得多数的选票，leader选举失败，等待选举时间超时后发起下一次选举。

![https://pic2.zhimg.com/v2-0471619d1b78ba6d57326d97825d9495_b.jpg](https://pic2.zhimg.com/v2-0471619d1b78ba6d57326d97825d9495_b.jpg)


选举出Leader后，Leader通过定期向所有Followers发送心跳信息维持其统治。若Follower一段时间未收到Leader的心跳则认为Leader可能已经挂了，再次发起Leader选举过程。

Raft保证选举出的Leader上一定具有最新的已提交的日志。

## 日志同步
laeder选举后，就开始接收客户端的请求。leader把请求作为日志条目（log entries）加入到它的日志中，然后并行的想其他服务器发起AppendEntry RPC复制这条日志条目。当这条日志被复制到大多数服务器上，leader将这条日志应用到他的状态机并想客户端返回执行结果。

![https://pic3.zhimg.com/v2-7cdaa12c6f34b1e92ef86b99c3bdcf32_b.jpg](https://pic3.zhimg.com/v2-7cdaa12c6f34b1e92ef86b99c3bdcf32_b.jpg)

某些Followers可能没有成功的复制日志，Leader会无限的重试 AppendEntries RPC直到所有的Followers最终存储了所有的日志条目。

日志有序编号（log index）的日志条目组成。每个日志条目包含它被创建时的任期号（term），和用于状态机执行的命令。如果一个日志条目被复制到大多数服务器上，就被认为可以提交（commit）了。

![https://pic3.zhimg.com/v2-ee29a89e4eb63468e142bb6103dbe4de_b.jpg](https://pic3.zhimg.com/v2-ee29a89e4eb63468e142bb6103dbe4de_b.jpg)

Raft日志同步保证如下两点：
- 如果不同日志中的两个条目有着相同的索引和任期号，则它们所储存的命令是相同的。
- 如果不同日志中的两个条目有着相同的索引和任期号，则它们之前的所有条目都是完全一样的。

第一条特性源于leader在一个term内在给定的一个log index最多创建一个日志条目，同时该条目在日志中的位置也从来不会改变。

第二条特性源于AppendEntries的一个简单一致性检查。当发送一个AppendEntries RPC时，leader会把新日志条目紧接着之前的条目的log index和term都包含在里面，如果follower没有在它的日志中找到log index和term都相同的日志，它就会拒绝新的日志条目。

一般情况下，leader和followers的日志保持一致，因此AppendEntries一致性检查通常不会失败。然而，leader奔溃可能会导致日志不一致：旧的leader可能没有完全复制日志中的所有条目。

![https://pic4.zhimg.com/v2-d36c587901391cae50788061f568d24f_b.jpg](https://pic4.zhimg.com/v2-d36c587901391cae50788061f568d24f_b.jpg)

上图阐述了一些Followers可能和新的Leader日志不同的情况。一个Follower可能会丢失掉Leader上的一些条目，也有可能包含一些Leader没有的条目，也有可能两者都会发生。丢失的或者多出来的条目可能会持续多个任期。

Leader通过强制Followers复制它的日志来处理日志的不一致，Followers上的不一致的日志会被Leader的日志覆盖。

Leader为了使Followers的日志同自己的一致，Leader需要找到Followers同它的日志一致的地方，然后覆盖Followers在该位置之后的条目。

Leader为了使Followers的日志同自己的一致，Leader需要找到Followers同它的日志一致的地方，然后覆盖Followers在该位置之后的条目。

## 安全性
Raft增加了如下两条限制来保证安全性：
- 拥有最新的已提交的log entries的follower才有资格成为leader。
 
这个保证是在RequestVote RPC中做的，candidate在发送RequestVote RPC时，要带上自己的最后一条日志的term和log index，其他节点在收到信息时，如果发现自己的日志比请求携带的更新，则拒绝投票。日志比较的原则是，如果本地的最后一条log entry的term更大，则term大的更新，如果term一样大，则log index更大的更新。

- leader只能推进commit index来提交当前term的已经复制到大多数服务器上的日志，旧term日志的提交要等到提交当前term的日志来间接提交（log index小于commit index的日志被间接提交）。

之所以要这样，是因为可能会出现已提交的日志又被覆盖的情况：
![https://pic4.zhimg.com/v2-12a5ebab63781f9ec49e14e331775537_b.jpg](https://pic4.zhimg.com/v2-12a5ebab63781f9ec49e14e331775537_b.jpg)


- 在阶段a，term为2，S1是Leader，且S1写入日志（term, index）为(2, 2)，并且日志被同步写入了S2；

- 在阶段b，S1离线，触发一次新的选主，此时S5被选为新的Leader，此时系统term为3，且写入了日志（term, index）为（3， 2）;

- S5尚未将日志推送到Followers就离线了，进而触发了一次新的选主，而之前离线的S1经过重新上线后被选中变成Leader，此时系统term为4，此时S1会将自己的日志同步到Followers，按照上图就是将日志（2， 2）同步到了S3，而此时由于该日志已经被同步到了多数节点（S1, S2, S3），因此，此时日志（2，2）可以被提交了；
- 在阶段d，S1又下线了，触发一次选主，而S5有可能被选为新的Leader（这是因为S5可以满足作为主的一切条件：
    - 1. term = 5 > 4，
    - 2. 最新的日志为（3，2），比大多数节点（如S2/S3/S4的日志都新），然后S5会将自己的日志更新到Followers，于是S2、S3中已经被提交的日志（2，2）被截断了。
- 增加上述限制后，即使日志（2，2）已经被大多数节点（S1、S2、S3）确认了，但是它不能被提交，因为它是来自之前term（2）的日志，直到S1在当前term（4）产生的日志（4， 4）被大多数Followers确认，S1方可提交日志（4，4）这条日志，当然，根据Raft定义，（4，4）之前的所有日志也会被提交。此时即使S1再下线，重新选主时S5不可能成为Leader，因为它没有包含大多数节点已经拥有的日志（4，4）。

## 日志压缩
在实际的系统中，不能让日志无限增长，否则系统重启时需要花很长时间进行回放，从而影响可用性。Raft采用整个系统进行snapshot来解决。snapshot之前的日志都可以丢弃。

每个副本独立的对自己状态进行snapshot，并且只能对已经提交的日志记录进行snapshot。

snapshot中包含一下内容：
- 日志元数据。最后一条已提交的 log entry的 log index和term。这两个值在snapshot之后的第一条log entry的AppendEntries RPC的完整性检查的时候会被用上。
- 系统当前状态。

当Leader要发给某个日志落后太多的Follower的log entry被丢弃，Leader会将snapshot发给Follower。或者当新加进一台机器时，也会发送snapshot给它。发送snapshot使用InstalledSnapshot RPC。

做snapshot既不要做的太频繁，否则消耗磁盘带宽， 也不要做的太不频繁，否则一旦节点重启需要回放大量日志，影响可用性。推荐当日志达到某个固定的大小做一次snapshot。做一次snapshot可能耗时过长，会影响正常日志同步。可以通过使用copy-on-write技术避免snapshot过程影响正常日志同步。

## 成员变更
成员变更是在集群运行中副本发生变化，如增加\减少副本数，节点替换等。

成员变更也是一个分布式一致性问题，即所有服务器对新成员达成一致。但是成员变更更有特殊性，因为在成员变更的一致性达成的过程中，参与投票的过程会发生变化。

如果将成员变更当成一般的一致性问题，直接向leader发送成员变更请求，leader复制成员变更日志，达成多数派之后提交，各服务器提交成员变更后从旧成员变更（Cold）切换到新成员配置（Cnew）的时刻不同。

因为各个服务器提交成员变更日志的时刻可能不同，造成各个服务器从旧成员配置（Cold）切换到新成员配置（Cnew）的时刻不同。

成员变更不能影响服务的可用性，但是成员变更过程的某一时刻，可能出现在Cold和Cnew中同时存在两个不相交的多数派，进而可能选出两个Leader，形成不同的决议，破坏安全性。

![https://pic3.zhimg.com/v2-c8e4ead21f6f2e9d40361717739519c6_b.jpg](https://pic3.zhimg.com/v2-c8e4ead21f6f2e9d40361717739519c6_b.jpg)

由于成员变更的这一特殊性，成员变更不能当成一般的一致性问题去解决。

为了解决这一问题，Raft提出了两阶段的成员变更方法。集群先从旧成员配置Cold切换到一个过渡成员配置，称为共同一致（joint consensus），共同一致是旧成员配置Cold和新成员配置Cnew的组合`Cold,new`，一旦共同一致`Cold,new`被提交，系统再切换到新成员配置Cnew。

![https://pic3.zhimg.com/v2-6b85a141cd131aa129a4e70d060f37be_b.jpg](https://pic3.zhimg.com/v2-6b85a141cd131aa129a4e70d060f37be_b.jpg)

Raft两阶段成员变更过程如下：
- leader收到成员变更请求从Cold到Cnew。
- leader在本地生成一个新的log entry，其内容是`Cold,new`，代表当前时刻新旧成员配置共存，写入本地日志，同时将该log entry复制至`Cold,new`的所有副本。在此之后更新同步需要保证得到Cold和Cnew两个多数派的确认。
- follower收到`Cold,new`的log entry后更新本地日志，并且此时就以该配置作为自己的成员配置。
- 如果Cold和Cnew中的两个多数派确认了`Cold,new`这条日志，leader就提交这条log entry。
- 接下来leader生成一条新的log entry，其内容是新成员配置Cnew，同样将该log entry写入本地日志，同时复制到follower上。
- follower收到新成员配置Cnew后，将其写入日志，并且从此刻开始，就以该配置作为自己的成员配置，并且如果发现自己不在Cnew这个成员配置中会自动退出。
- leader收到Cnew的多数派确认后，表示成员变更成功，后续的日志只要得到Cnew多数派确认即可。leader给客户端回复成员变更执行成功。

异常分析：
- 如果Leader的`Cold,new`尚未推送到Follower，Leader就挂了，此后选出的新Leader并不包含这条日志，此时新Leader依然使用Cold作为自己的成员配置。
- 如果Leader的`Cold,new`推送到大部分的Follower后就挂了，此后选出的新Leader可能是Cold也可能是Cnew中的某个Follower，此后客户端继续执行一次改变配置的命令即可。
- 如果Leader在推送Cnew配置的过程中挂了，那么同样，新选出来的Leader可能是Cold也可能是Cnew中的某一个，此后客户端继续执行一次改变配置的命令即可。
- 如果大多数的Follower确认了Cnew这个消息后，那么接下来即使Leader挂了，新选出来的Leader肯定位于Cnew中。

两阶段成员变更比较通用且容易理解，但实现比较复杂，同时两阶段的变更协议也会在一定程度上影响变更过程中的服务可用性，因此我们期望增加成员变更的限制，以简化操作流程。

两阶段成员变更，之所以分成两个阶段，是因为对Cold和Cnew的关系没有做任何假设，为了避免Cold和Cnew各自形成不相交的多数派选出两个leader，才引入两阶段方案。

如果增强成员变更的限制，假设Cold和Cnew任意多数派交集不为空，这两个成员配置就无法各自形成多数派，那么成员变更方案就能简化为一阶段。

那么如何限制Cold和Cnew，使之任意的多数派交集不为空，这两个成员配置就无法各自形成多数派，那么成员变更方案就可以简化为一阶段。

可从数学上严格证明，只要每次只允许增加或删除一个成员，Cold与Cnew不可能形成两个不相交的多数派。

一阶段成员变更：
- 成员变更限制每次只能增加或删除一次成员（如果要变更多个成员，连续变更多次）。
- 成员变成由leader发起，Cnew得到多数派确认后，返回客户端成员变更成功。
- 一次成员变更成功前不允许开始下一次成员变更，因此新任leader在开始提供服务前要将自己本地保存的最新成员配置重新投票形成多数派确认。
- leader只要开始同步新成员配置，即可开始使用新的成员配置进行日志同步。









# 参考
- [Raft算法详解](https://zhuanlan.zhihu.com/p/32052223)
- [Raft协议详解](https://zhuanlan.zhihu.com/p/27207160)
- [https://raft.github.io/raft.pdf](https://raft.github.io/raft.pdf)