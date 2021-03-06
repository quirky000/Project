# 前言
在之前参与的项目开发中曾出于项目优化的角度引入了MongoDB数据库。借这个契机简单的思考了MongoDB和Mysql间的**差异**，并进行记录。

# 情景
当时开发的项目是一个功能类似于钉钉的软件，这是一个微服务项目，其中又一个是**消息微服务**。当时项目使用的数据库为Mysql，但在实际开发过程中，能感觉到数据库的读写速度不太理想。

**解决方法**：引入了**MongoDB**数据库。
## 消息微服务
**功能**：负责推送**通知**，如：用户申请加入企业通知、工作流相关通知。

# 思考
通过引入新的数据库解决了读写速度的问题，在这里也借机记录下使用MongoDB的**理由**，以及简单的记录二者的**区别**。

## 理由
首先**MongoDB**和**Mysql**有一个重要的共同特点，那就是两者都是**免费、开源**的数据库。这个是选择**MongoDB**的一个重要前提。接下来就是对比二者的**区别**，或者说**MongoDB**在什么方面**优于Mysql**。

### 区别
首先我们先搞清楚它们两个的**特点**。
#### **Mysql**
- 一个**关系型**数据库管理系统。
- 支持**跨平台**。
- 支持面向**对象**。
- 支持**内置函数**。
- 支持事务**一致性**。

#### **MongoDB**
- 一个基于**分布式文件**存储的数据库。
- 由**C++**语言编写。
- 为WEB 应用提供**可扩展**的高性能数据存储解决方案。
- 将数据存储为**文档**，数据结构是由**键值对**组成。
- **不支持**事务一致性。

MongoDB和Mysql之间最**主要**的区别是**非关系型数据库**和**关系型数据库**的区别。
#### 非关系型数据库（MongoDB）
###### 特点：
- 存储数据使用的是**键值对**
- 分布式
- 不支持**ACID**特性
- 严格上，非关系型数据库是一种数据结构化存储方法的**集合**，而不是数据库

###### 优势：
- **无需**sql层解析，读写性能高
- 基于键值对，数据**没有**耦合性，易于拓展
- 存储格式可以以key:val、文档、图片等形式

###### 缺点
- 不提供sql支持，学习**成本高**
- **无**事务处理

#### 关系型数据库（Mysql）

###### 特点：
- 采用**数据模型**组织数据
- 事务一致性
- 关系模型是**二维表格**模型，关系型数据库就是二维表及其之间的关联组成的数据组织

###### 优：
- **容易**理解
- 使用**方便**
- 维护**容易**

###### 缺：
- 维护—执行耗费**性能**
- 影响**读写**
- 表结构**固定**
- 不满足**高并发**读写需求
- 不满足海量数据的**高效率**读写

将二者的优缺进行对比，我们可以初步得到**非关系型数据库**（**MongoDB**）的适用范围：
1.	数据量**大**；
2.	读写操作**频繁**
3.	数据价值**低**，**无**事务要求。

现在我们就对在上文已经提到过情景进行分析。

在消息微服务中我们面对的数据主要是**用户申请加入企业通知**、**工作流相关通知**，这些信息最大的特点就在于他们具有**时效性**，以及时常会出现多人**同时访问**（即**高并发**）的问题，并且它们是**无**事务要求。由于时效性的特点，这些数据其实是可以看做一次性数据。它们的**复用率低**，相对的价值就不高。

总结起来就是：
1.	涉及到公司的通知以及用户申请，数据**存量大**；
2.	通知面向的对象为公司内部的用户，面向对象广，容易出现**高并发**情况，对数据的**读写效率**有要求；
3.	大部分数据为公司内部通知和用户申请，这些数据的复用性低，**价值较低**；
4.	通知只**要求查看**，而基本无事务要求。

以上分析的特点完美符合上面总结的**非关系型数据库**（**MongoDB**）的适用范围，所以最后引入**MongoDB**就是一件顺理成章的事情。