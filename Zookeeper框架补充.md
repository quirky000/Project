# 一、监听机制
- **ZooKeeper**支持watch(监听)机制, 即允许对ZooKeeper注册监听, 当监听的对象发生**指定的事件**的时候, ZK就会**返回**一个**通知**。
- 三个**过程**：  
1.**注册**：客户端向ZK服务端注册Watcher。  
2.**触发**：服务端事件发生**触发Watcher**。  
3.**回调**：客户端**回调Watcher**得到触发事件情况。

触发事件种类：**节点创建，节点删除，节点改变，子节点改变**等。
- Watcher是**一次性**的. **被触发**就会失效. 需要**反复监听**就需要**反复注册**。

### 原理
![image](https://img-blog.csdnimg.cn/20210822135531939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzU3NjY4MDQw,size_16,color_FFFFFF,t_70)
1. 出现**main()**线程。
2. 在main线程中**创建Zookeeper客户端**， 这时就会创建两个线程：  
一个**复制网络连接通信**(connect)。  
一个负责**监听**(listener)。
3. 通过**connect线程**将注册的监听事件**发送**给zk, 常见的监听有：
- 监听节点**数据**的变化**get path [watch]**
- 监听节点**状态**的变化**stat path [watch]**
- 监听**子节点增减**的变化**ls path [watch]**
4. 将注册的**监听事件添加**到zk的**注册的监听器列表**中。
5. 监听到有数据或路径**变化**, 就会将这个消息发送给**listener线程**。
6. listener线程**内部**调用**process()方法**（为程序员**自定义**的方法，包含监听到事件后做如何的**通知操作**）。

### 监听器实际运用
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqbY*DDaHH4w5TAWJvdmOH2w70kN1YY.gc0GGbdb1CpHUsauZ0ncFH8Qd1MajzN8J7J0XJ7xzSCJBTHKwwW1lODg!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)
1. 先向zk集群**注册**一个监听器, 监听某一个**节点路径**。
2. **主要服务器**启动, 去zk指定路径下创建一个**临时节点**。
3. 监听器监听servers下面的子节点有没有变化, 一旦**有变化**, 不管**新增(机器上线)**还是**减少(机器下线)**都会马上给对应的人发送通知。

# 二、Zookeeper的应用场景
ZK框架提供的服务包括: **统一命名服务、 统一配置管理、统一集群管理、集群选主、 服务动态上下线、分布式锁等**。 

### 1.统一命名服务
利用了ZK的node节点**全局唯一**的特点实现。

在**分布式**环境下，为了便于识别，需要对应用/服务进行**统一命名**。创建一个节点后, 节点的路径就是**全局唯一**的, 可以作为**全局名称**使用。 
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqUt*AvGQyj61sPOYTwYYzYrKFnNM5vXIYnjSfKZm3gj6CTGjbc7UbgooHa7BI9fbkWZkfP6oZKHrd2zJBUwhO*8!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

### 2.统一配置管理
利用了Zookeeper的**watch机制**。

**需求场景**: **分布式**环境下, 要求**所有节点**的配置信息**一致**, 如**Kafka集群**。对配置文件修改后, 希望能够**快速同步**到各个节点上。

**解决方案**: 把**所有**的配置都放在**一个配置中心**, 然后各个服务**分别**去监听配置中心, 只要能发现里面的内容**出现变化**, 立即**获取变化**的内容, 然后**更新本地配置**。

**具体实现**: 配置管理可交由Zookeeper实现。
- 将**配置信息**写入**Zookeeper**上的一个**Znode**。
- 由各个客户端服务器**监听**这个Znode。
- 若Znode中的数据**被修改**, Zookeeper将**通知**各个客户端服务器。

![image](http://a1.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqcCuNKeVN4yvoW.*oBietgi.NGK77gIAwdu5EosSFap6vvraEuA8s24BjBpJT6Vu7rh93.bKpyrXovVTjen3yhA!/b&ek=1&kp=1&pt=0&bo=*wTyAgAAAAADFzk!&tl=1&vuin=515667400&tm=1655902800&dis_t=1655903339&dis_k=2387623fea301c5ef19168cf95b4ad30&sce=60-1-1&rf=viewer_4)

### 3.统一集群管理
利用了Zookeeper的**watch机制**。

**需求场景**: **分布式**环境中, 需要**实时**掌握**每个节点的状态**, 能够根据节点实时状态做出一些调整。

**解决方案**: Zookeeper可以实现实时**监控节点状态变化**。
- 将**节点信息**写入Zookeeper上的一个**Znode**。
- **监听**这个Znode可获取它的**实时状态变化**。
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqeoHC0c20AHxHJ8SS5*SAtGTdTBoO4fIRLLkTyfoGdYz*66C.Jjq.758m50Don1KwYgqi8ZgETnpmNG0G2PTTCs!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

### 4.集群选主
利用的是zookeeper的**临时节点**。

**需求场景**: 在集群中需要区分主从节点, 主节点负责**数据写入**, 从节点负责**数据读取**。 这样一来就需要解决以下几个问题：如何**确定**一个节点是**主节点**；当一个主节点**宕机**的时候, 如何从其他从节点怎么**再来选出一个主节点**。

**具体实现**：使用Zookeeper的**临时节点**实现这一需求。上面描述的这个过程可以称为集群选主的过程, 首先**所有**的节点都认为是**从节点**, 都有机会称为主节点, 然后开始选主。

- 所有参与选主的主机都去ZK上创建**同一个**临时节点，最终能创建成功的**只有一个**客户端请求。
- **成功**创建节点的客户端所在的机器就成为了Master，**没有成功**创建该节点的客户端，成为从节点。
- 所有的从节点都会在主节点上注册一个子节点变更的Watcher，用于**监控**当前**主节点**是否**存活**。如果发现当前的主节点挂了，那么其他客户端将会**重新选主**。 

![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqd.yVPhLXX4Dha0.w4QRWBXrWYHPDvcoG.D0oVFII3*U*mYmYfKZ*n9QgVJ8Wjpq5FfUet7J**HW8h7L27jpT74!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

### 5.分布式锁

利用的是Zookeeper的**临时有序节点**。

**需求场景**: 在**分布式**系统中, 容易出现**多台主机操作同一资源**的情况, **如**：两台主机同时往一个文件中追加写入文本,这种情况下，会出现一个写入操作被另一个写入操作**覆盖**。

**解决方案**: 加入一把锁, 只有当一个主机**获取到了这把锁**, 才会**执行写入**, 此时另一台主机等待; 当写入操作执行完毕，另一台主机才会获得锁，然后写入。这把锁就称为**分布式锁**。 分布式锁的**作用**是：控制分布式系统之间**同步访问共享资源**的一种方式。

**具体实现**: 使用Zookeeper的**临时有序节点**。
1. **所有**需要执行操作的主机都去ZK上创建一个**临时有序节点**。
2. 获取这些节点，然后进行**从小到大**的排序。
3. 判断自己创建的节点是不是**最小**。  
**是**, 则自身就**获取锁**;   
**不是**, 则对最小的节点**注册一个监听**。
4. 当自身**获取锁**, 则**执行**相应的操作。 **执行完毕**后**连接断开节点消失**，此时**释放锁**。
5. 如果自己**没有**获取到锁, 继续等待。 持续监听节点的变化。当节点消失，**锁被释放后**, 再**重新执行抢夺锁**的操作。
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqRpvUD4YP6bGPij42gGnDibR3oLW9D2BGCdr79lneJ6JDgf3J4*smMp.LBm02De5w3JuJhzSRB8M7wE6WiZ4Ix8!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

# 三、Zookeeper集群
### 1. ZK集群  

为保证Zookeeper的**高可用**, 需要部署Zookeeper的集群。Zookeeper 有三种运行模式: **单机模式, 集群模式和伪集群模式**。
- **单机模式**: 使用**一台主机不是一个Zookeeper**来对外提供服务, 有单点故障问题, 仅**适合于开发、测试环境**。
- **集群模式**: 使用**多台服务器**, 每台上部署一个Zookeeper一起对外提供服务, 适合于**生产环境**。
- **伪集群模式**: 在**服务器不够多**的情况下, 也可以考虑在**一台服务器**上部署**多个Zookeeper**来对外提供服务。

### 2. 数据一致性处理
保证分布式系统的数据一致性的方法。
- **集群角色**  
1. **Leader**: 负责**投票的发起和决议**, **更新系统状态**, 是**事务请求**(写请求) 的**唯一处理者**, 一个ZooKeeper**同一时刻**只会有一个Leader. 对于create创建/setData修改/delete删除等有写操作的请求, 则需要**统一转发**给leader 处理, leader 需要**决定编号和执行操作**, 这个过程称为一个**事务**。  
2. **Follower**: 接收**客户端请求**, 参与**选主投票**。 处理**客户端非事务(读操作)请求**，转发**事务请求**(写请求)给Leader。  
3. **Observer**: 针对**访问量比较大**的zookeeper 集群, 为了**增加并发的读请求**. 还可**新增观察者角色**。  
     **作用**: 可以接受客户端请求, 把请求转发给leader, **不参与投票, 只同步leader的状态**。

- **Zookeeper的特性**  
1. **Zookeeper**: 一个领导者(Leader), 多个跟随者(Follower)组成的集群。
2. 集群中只要有**半数**以上节点**存活**, Zookeeper集群就能正常服务。
3. **全局数据一致**: 每个Server保存一份相同的数据副本, Client无论连接到哪个Server, 数据都是一致的。
4. **更新请求顺序性**: 从**同一个**客户端发起的事务请求,最终会严格按照顺序被应用到zookeeper中。
5. **数据更新原子性**: 一次数据更新要么成功, 要么失败。
6. **实时性**：在一定时间范围内，Client能读到最新数据。

- **ZAB协议**  
1. Zookeeper采用**ZAB(Zookeeper Atomic Broadcast)协议**来保证**分布式数据一致性**。
2. ZAB并不是一种通用的分布式一致性算法,而是一种**专为Zookeeper设计的崩溃可恢复的原子消息广播算法**。ZAB协议包括两种基本模式: **崩溃恢复模式**和 **消息广播模式**。  
**2-1.** **消息广播模式**：用来进行**事务请求**的处理。  
**2-2.** **崩溃恢复模式**：用来在集群**启动**过程,或者Leader服务器**崩溃退出**后进行新的Leader**服务器的选举**以及**数据同步**。  
- **ZK集群写数据流程**  
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqQAjAKUOaBeT5oCet9GVYKnoBPrVQ.a7pHF3dDO3yjmrh8ARYtojg3GnRFIZ.uW.axG5NpGWSnK8JAtHVkC9UqM!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)
1. **Client**向Zookeeper的**Server1**上写数据, 发送一个写请求。
2. 如果Server1不是**Leader**, 那么Server1会把接受的请求进一步转发给Leader, 因为每个Zookeeper的Server里面有一个是Leader. 这个Leader会将写请求**广播**给各个Server, 比如Server1和Server2, 各个Server会将该写请求加入**待写队列**, 并向Leader发送成功信息(**ack反馈机制**)。
3. 当Leader收到**半数**以上Server的**成功信息**, 说明该写操作**可以执行**。 Leader会向**各个**Server发送**事务提交信息**, 各个Server收到信息后会落实队列里面的写请求, 此时写成功。
4. Server1会进一步**通知**Client数据写成功了, 这是就认为整个写操纵成功。

### 3. ZK集群选举机制
- **Zookeeper服务器的四个状态**：
1. **looking**: **寻找leader状态**, 当服务器处于该状态时, 它会认为当前集群中没有leader, 因此需要进入leader选举状态。
2. **leading**: **领导者状态**, 表明当前服务器角色是leader。
3. **following**: **跟随者状态**, 表明当前服务器角色是follower。
4. **observing**：**观察者状态**, 表明当前服务器角色是observer。
- **半数机制**: 集群中**半数**以上机器存活, 集群可用, 所以Zookeeper适合安装**奇数台**服务器. 集群启动时, 如果当前机器票数超过了总票数一半则为Leader, Leader产生后, 投过票的机器就不能再投票了。
- Zookeeper虽然在**配置文件**中**没有指定主从节点**. 但是, Zookeeper工作时, 是有一个节点Leader, 其他则为Follower, Leader是通过内部的选举机制**临时**产生的. 配置文件中会指定每台ZK的**myid**, 而且**不能重复**, 通常用1,2,3…区分每台ZK的myid。

##### 3-1.集群启动器的选举机制
在集群**初始化**阶段, 当有一台服务器server1启动时, 其单独无法进行和完成leader选举, 当第二台服务器server2启动时, 此时两台机器可以相互通信, 每台机器都试图找到leader, 于是进入leader选举过程。

**实例**：
1. **服务器1**启动, 服务器1状态保持为looking。
2. **服务器2**启动, 发起一次选举. 服务器1投票给比自己ID号大的服务器2。 服务器2投票给自己。  
**投票结果**: 服务器1票数**0票**, 服务器2票数**2票**, 没有半数以上结果, **选举无法完成**, 服务器 1, 2 状态保持looking。

3. **服务器3**启动, 发起一次选举. 此时服务器1和2都会更改选票为服务器3, 服务器3投票给自己。  
**投票结果**: 服务器1为**0票**, 服务器2为**0票**, 服务器3为**3票**. 此时服务器3的票数已经超过半数，**服务器3当选 Leader**. 服务器 1，2 更改状态为follower，服务器 3 更改状态为leader。

4. **服务器4**启动, 发起一次选举. 此时服务器 1，2，3 已经**不是looking状态**, **不会更改选票信息**, 服务器4投票给自己。
投票结果：服务器3为**3票**，服务器4为**1票**。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为following。

5. **服务器5**启动，同4一样当小弟。

##### 3-2.服务器运行时期的Leader选举
在**zk运行期间**, leader与非leader服务器各司其职, 即便有**非leader**服务器**宕机**或者**新加入**, 此时也**不会影响leader**。 但如果**leader**服务器**宕机**了, 那么**整个集群暂停对外服务**, 进入新一轮leader选 举, 其过程和启动时期的Leader选举过程基本一致。

**实例**：
假设存在服务器**server1、server2、server3**，server2为**leader**，server2**宕机**，则再次进行选举。
1. **变更转态**, server1和server3变更为**looking**状态。
2. **开始投票**, 每台服务器投票给**比自己myid大**的机器, 没有比自己大的就投给自己。
3. 这样server3有**2票**, server1有**1票**, server3的票数超过了集群一半, 当选leader, server1变更状态follower。