---
title: Hadoop总结
date: 202１-03-２９ 08:01:58
tags: 
    - map 
    - reduce
    - hadoop源码
    - shuffle
categories: 
    - hadoop
---

# 序言

Hadoop框架包含计算和存储两部分，MapReduce实现计算部分，存储则是HDFS。大数据在map reduce思想基石上，形成了一整套Hadoop生态系统。了解map reduce还是相当重要的。<!--more-->



# Hadoop发展史

为什么取名Hadoop，Hadoop其实是一个造出来的词，来源于Doug Cutting儿子毛绒玩具象。我们就看看Hadoop的发展历程。

- 2003-2004 Google公布GFS和MapReduce思想的两篇论文，受此启发，Doug Cutting等人用2年时间实现了DFS和MapReduce机制。
- 2005　Hadoop作为Lucene的子项目Nutch的一部分正式引入Apache基金会。
- 2006.2 被分离.出来，成为一套完整独立的软件，起名为Hadoop
- 2012.5 发布２.0版本，抽象出单独的资源管理模块Yarn进行资源管理。

所以从根源上说，MapReduce思想来源于Google的论文，Doug Cutting等人做了开源的工程实现。




# Hadoop架构
![hadoop-structure](https://raw.githubusercontent.com/zcenao21/photos-blog/main/hadoop/hadoop-structure.png)

### NameNode

NameNode管理文件系统的命名空间，维护文件系统树和整棵树内的所有文件和目录。这些信息是以两个文件形式永久保存在本地磁盘上，分别是命名空间镜像文件(fsimage)和编辑日志文件(edit logs)。

NameNode存储信息列表：

- 文件系统的命名空间，文件名称，文件目录结构，文件的属性(权限/创建时间/副本数)
- 文件对应哪些数据块，数据块存储在哪些datanode节点上

NameNode在启动时，会先将fsimage加载进内存，然后与edit logs合并更新到最新状态。在hadoop1.x版本中，启动之后产生的新的edit logs由Secondary NameNode定期和fsimage合并并更新到NameNode当中。在hadoop2.x之后，增加了NameNode HA（高可用）功能，合并的工作由Secondary NameNode转移到了Standy NameNode上。另外，Secondary NameNode可以在NameNode出现故障时快速切换。



### DataNode

DataNode是实际的数据存储模块。NameNode仅用于存放元数据信息，实际的数据信息都放在DataNode上。DataNode存储数据块和数据块校验和，和客户端进行数据通信，DataNode之间也进行数据传输。

DataNode和NameNode之间通过心跳机制联系。每隔３秒DataNode发送一次心跳，心跳结果返回NameNode的命令如块的复制/块的删除等。

如果十分钟未收到DataNode的心跳，则认为DataNode丢失，它上面的block会拷贝到其他DataNode上。



#　Hadoop配置

### 配置文件及其作用

![img](https://github.com/zcenao21/photos-blog/raw/main/hadoop/file-func.jpg)

**core-site.xml**

| 配置项         | 作用                                    |
| -------------- | --------------------------------------- |
| fs.defaultFS   | 配置namenode的dfs协议的文件系统通信地址 |
| hadoop.tmp.dir | 指定hadoop临时目录                      |

**hdfs-site.xml**

| 作用                  | 配置项                 |
| --------------------- | ---------------------- |
| dfs.namenode.name.dir | namenode数据的存放地点 |
| dfs.datanode.data.dir | datanode数据的存放地点 |
| dfs.replication       | hdfs副本数量设置       |

**yarn-site.xml**

| 作用                                          | 配置项                                            |
| --------------------------------------------- | ------------------------------------------------- |
| yarn.resourcemanager.address                  | ResourceManager 对客户端暴露的地址                |
| yarn.resourcemanager.scheduler.address        | ResourceManager 对ApplicationMaster暴露的访问地址 |
| yarn.resourcemanager.resource-tracker.address | ResourceManager 对NodeManager暴露的地址           |
| yarn.resourcemanager.admin.address            | ResourceManager 对管理员暴露的访问地址            |

**mapred-site.xml**

| 作用                     | 配置项                 |
| ------------------------ | ---------------------- |
| mapreduce.framework.name | 指定mr框架为yarn的方式 |












































































































