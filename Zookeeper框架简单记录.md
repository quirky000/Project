# Zookeeper
**Zookeeper**是一个**分布式协调服**务的开源框架, 主要用来解决分布式集群中应用系统的**一致性**问题, 例如怎样避免同时操作同一数据造成**脏读**的问题。
#### 本质
一个分布式的**小文件存储系统**，提供基于类似于文件系统的**目录树**方式的数据存储, 可以对树中的**节点**进行有效管理。 从而用来维护和监控你存储的数据的**状态变化**。 通过监控这些数据状态的变化，从而可以达到基于数据的**集群管理**。

## 数据模型
- 本质上是一个**分布式**的**小文件存储系统**。
- 表现为一个**分层的文件系统目录树结构**, 既能存储数据, 而且还能像目录一样有子节点。 每个节点可以存最多**1M**左右的数据。
- 每个节点称做一个**Znode**, 每个Znode都可以通过其路径**唯一**标识。
- 客户端还能给节点添加**watch**, 也就是**监听器**, 可以监听节点的变化。

#### 节点结构
![image](http://aidata100.com/bigdata/zphtm/bd0122/4dd9ba9b-1f0c-4bb3-b6dc-a6b9abd7520a.001.png)  
上图**每个节点**称为一个**Znode**，Znode的组成为三部分：
1. **stat**：此为**状态信息**, 描述该 Znode 的版本, 权限等信息。
2. **data**：与该 Znode 关联的**数据**。
3. **children**：该 Znode 下的**子节点**。

#### 节点类型
**两大类**：
- **永久节点(Persistent)**: 客户端和服务器端**断开连接**后，创建的节点**不会消失**, 只有在客户端执行删除操作的时候, 他们才能被删除。 
- **临时节点(Ephemeral)**: 客户端和服务器端断开连接后，创建的节点会**被删除**。 Znode 还有一个**序列化**的特性, 这个序列号对于此节点的**父节点**来说是**唯一**的, 这样便会记录每个**子节点**创建的**先后顺序**。 它的格式为“**%10d**”(10 位数字, 没有数值的数位用 0 补充, 例如“0000000001”)  

**四小类**: 
1. **永久节点(Persistent)**
2. **永久_序列化节点(Persistent_Sequential)**
3. **临时节点(Ephemeral)**
4. **临时_序列化节点(Ephemeral_Sequential)**

###### 声明
这里只是进行了简单的记录以供参考，其中可能会有错误在这里欢迎指正。

未来不排除会继续进行更新。
