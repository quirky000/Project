# 一、监听机制
- **ZooKeeper**中还支持一种watch(监听)机制, 它允许对ZooKeeper注册监听, 当监听的对象发生**指定的事件**的时候, ZooKeeper就会**返回**一个**通知**。
- 三个**过程**：  
1.客户端向ZK服务端**注册Watcher**  
2.服务端事件发生**触发Watcher**  
3.客户端**回调Watcher**得到触发事件情况  
触发事件种类：**节点创建，节点删除，节点改变，子节点改变**等。
- Watcher是**一次性**的. 一旦**被触发**将会失效. 如果需要反复进行监听就需要反复进行注册。

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
6. listener线程**内部**调用了**process()方法**.此方法是程序员**自定义**的方法, 里面可以**写明**监听到事件后做如何的**通知操作**。

### 监听器实际运用
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqRsO0GbyIrTnfvPutZtzuoCmS6pHH*6j1riq3VBZvJjCToIDbb4ZzAUASBaY2mn8UAmw5fp0W8vHFro7gQQcZCI!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)
1. 先想zk集群**注册**一个监听器, 监听某一个**节点路径**。
2. **主要服务器**启动, 就去zk上指定路径下创建一个**临时节点**。
3. 监听器监听servers下面的子节点有没有变化, 一旦**有变化**, 不管新增(机器上线)还是减少(机器下线)都会马上给对应的人发送通知。

# 二、Zookeeper的应用场景
ZK框架提供的服务包括: **统一命名服务、 统一配置管理、统一集群管理、集群选主、 服务动态上下线、分布式锁等**。 

### 1.统一命名服务
利用了ZK的node节点**全局唯一**的这个特点实现。

在**分布式**环境下，经常需要对应用/服务进行**统一命名**，便于识别。例如：IP不容易记住，而域名容易记住。创建一个节点后, 节点的路径就是**全局唯一**的, 可以作为**全局名称**使用。 
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqUt*AvGQyj61sPOYTwYYzYrKFnNM5vXIYnjSfKZm3gj6CTGjbc7UbgooHa7BI9fbkWZkfP6oZKHrd2zJBUwhO*8!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

### 2.统一配置管理
利用了Zookeeper的**watch机制**。

**需求**: **分布式**环境下, 要求**所有节点**的配置信息是**一致**的, 比如**Kafka集群**. 对配置文件修改后, 希望能够**快速同步**到各个节点上。

**方案**: 可以把**所有**的配置都放在**一个配置中心**, 然后各个服务**分别**去监听配置中心, 一旦发现里面的内容发生变化, 立即获取变化的内容, 然后更新本地配置即可。

**实现**: 配置管理可交由Zookeeper实现。
- 可将**配置信息**写入**Zookeeper**上的一个**Znode**。
- 各个客户端服务器**监听**这个Znode。
- 一旦Znode中的数据**被修改**, Zookeeper将**通知**各个客户端服务器。

![image](http://a1.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqcCuNKeVN4yvoW.*oBietgi.NGK77gIAwdu5EosSFap6vvraEuA8s24BjBpJT6Vu7rh93.bKpyrXovVTjen3yhA!/b&ek=1&kp=1&pt=0&bo=*wTyAgAAAAADFzk!&tl=1&vuin=515667400&tm=1655902800&dis_t=1655903339&dis_k=2387623fea301c5ef19168cf95b4ad30&sce=60-1-1&rf=viewer_4)

### 3.统一集群管理
利用了Zookeeper的**watch机制**。

**需求**: **分布式**环境中, **实时**掌握**每个节点的状态**是必要的, 可以根据节点实时状态做出一些调整。

**方案**: Zookeeper可以实现实时**监控节点状态变化**。
- 可将节点信息写入Zookeeper上的一个Znode
- 监听这个Znode可获取它的实时状态变化
![image](http://m.qpic.cn/psc?/V51l0rcS20UiU61WKyG44Mc0pk23AOpL/bqQfVz5yrrGYSXMvKr.cqeoHC0c20AHxHJ8SS5*SAtGTdTBoO4fIRLLkTyfoGdYz*66C.Jjq.758m50Don1KwYgqi8ZgETnpmNG0G2PTTCs!/b&bo=*wTyAgAAAAADByk!&rf=viewer_4)

### 4.集群选主

### 5.分布式锁

# 三、
