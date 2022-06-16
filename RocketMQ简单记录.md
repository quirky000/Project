## RocketMQ
**RocketMQ**前身为**MetaQ**，最初就是基于**Kafka**改造而来，经过不断的迭代与版本升级。

## 一、RocketMQ的功能
### 异步
假设一个业务场景：

**支付系统**→**生成订单系统**→**通知用户支付成功系统**

在使用**消息队列**前，该业务具体流程为：  
支付**成功**，支付系统调用**生成订单系统**来生成订单，再继续调用通知**用户业务系统**，最后返回给**支付系统**。  
在这一整个过程中支付系统在发送请求到完成业务需要花费大量时间。  
而引入**消息队列**之后：  
**支付系统⇄消息记录→生成订单系统→通知用户支付成功系统**  

业务流程变为了支付系统**只需要**将**任务信息**提交到消息队列，剩下的事情，则由**消息队列**和其他系统合作去完成。这样一来就实现了响应时间的缩短。

### 解耦
假设一个系统结构：  
**A系统→B系统**  

这系统一眼就可以看出问题：**耦合度**太高。一旦B系统出现宕机或者其他问题，就会连带着A系统**一起失效**。  

现在引入**消息队列**：  
**A系统→消息队列→B系统**  

这样一来，即使B系统出现为题，也不会影响到A系统。

### 削峰
**削峰**是为了应付**短时间**大量数据请求涌入数据库的情况，简单的**实例**就是：**天猫双十一**。  

具体实现方法就是订单由**消息队列**接收，然后在进行**挨个处理**，这样虽然会使得用户下单成功的**时间增加**，但能有效降低**服务器压力**，减少服务器崩溃的可能。

## 二、RocketMQ的组成
**Message** : 消息

**Broker**: RocketMQ的核心模块，负责接收并存储消息。

**NameServer**: RocketMQ的注册中心，集群的Topic-Queue的路由配置；Broker的实时配置信息。其它模块通过Nameservr提供的接口获取最新的Topic配置和路由信息。

**Topic** ：用于将消息按主题做划分，Producer将消息发往指定的Topic，Consumer订阅该Topic就可以收到这条消息。Topic跟发送方和消费方都没有强关联关系，发送方可以同时往多个Topic投放消息，消费方也可以订阅多个Topic的消息。在RocketMQ中，Topic是一个上逻辑概念。消息存储不会按Topic分开。

**Tag**： 标签可以被认为是对 Topic 进一步细化。一般在相同业务模块中通过引入标签来标记不同用途的消息，Tag也可以对消息进行快速的过滤，例如后面consumer在subscribe的时候可指定tag，如果不需要过滤，设置为*即可。

**Producer**：消息的生产者

**Consumer**：消息的消费者

## 三、RocketMQ的整体架构
![image](https://img-blog.csdnimg.cn/img_convert/841f5426e752463524c03e1f02294efe.png#pic_center)
**抽象**角度上来说，**RocketMQ**的实现原理为：  
1. 生产者将消息给到队列(Broker)
2. 队列中的消息在逻辑上是以Topic来分类的，进一步还可以通过Tag来过滤
3. 消费者消费消息

![image](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjYxOTE1OS1hODU4ZDM4ZTBiMzhjNDA2LnBuZw?x-oss-process=image/format,png)
由上面的图可以看出Rocket MQ是**分布**式的，Broker作为**存储消息**的核心组件，支持**主从机制**，一个挂了，还会有备用。

不管是**Broker**还是**Name Server** 都可以做**分布式**，从而让整个系统变得**高可用**。  
![image](https://img-blog.csdnimg.cn/d4a65c0bc970498dbe7815d999580e95.png)
由该图可看出：
1. 每个**Broker**与Name Server集群中的**所有节点**建立长连接，定时注册Topic信息到所有Name Server。
2. **Producer**与Name Server集群中的**其中一个节点**（随机选择）建立长连接，**定期**从Name Server取Topic路由信息。
3. **Consumer**与Name Server集群中的**其中一个节点**（随机选择）建立长连接，**定期**从Name Server取Topic路由信息。



###### 声明
这里只是进行了简单的记录以供参考，其中可能会有错误在这里欢迎指正。

未来不排除会继续进行更新。