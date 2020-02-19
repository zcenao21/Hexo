---
title: RDD转换
date: 2019-11-02 22:43:50
tags:
    - Spark
    - High Performance Spark
    - RDD
categories: 
    - Spark
---



# RDD

RDD(Resilient Distributed Datasets，弹性分布式数据集)，是一个惰性计算、静态类型的分布式数据集合，是Spark实现并发计算的基础数据结构。<!--more-->

RDD包含以下特性（前3个必须有，后两个可选）：

1. partitions()

   返回组成分布式数据集的分区对象数组。

2. itearator(p, parentIters)

   为每个父分区计算分区p的iteartors。

3. dependencies

   返回依赖对象序列。

4. partitioner()---可选

   若RDD有相关元素与分区信息，则返回Scala option type的分区对象。

5. prefferedLocations(p)---可选

   返回数据分区的存储位置信息。



针对RDD的处理有丰富的操作，包括Action算子和Transformation算子。Action操作返回对象非RDD，Transformation操作返回仍然是RDD。Action算子是触发行动的算子，**Action算子数量等于Spark Job的数量**；Transformation算子会对RDD进行变换，根据父RDD和子RDD的依赖关系不同，Transformation又分为宽依赖和窄依赖。宽依赖的示意图如下：

![image.png](https://i.loli.net/2019/11/03/5EpJ6In4DhlVSxj.png)

窄依赖的严格定义：**each partition of the parrent RDD is used by at most one partition of the child RDD（译：每个父RDD的分区最多被一个子RDD分区使用）**。

这是区分宽窄依赖最严格的定义，还有一个并不是非常严格的说法，但是便于理解：

需要进行shuffle的为宽依赖，不需要的为窄依赖。

**Spark Job中的Stage个数就等于宽依赖个数。**



常用的Action算子有reduce,collect,count,first,take,aggregate, fold, foreach, saveAsTextFile, countByKey等; 常用的Transformation算子有map, filter, flatmap, mapPartitions, sample, union, intersection, distinct, groupByKey, countByKey, sortBy, join, coalesce, repartition, repartitionAndSortWithinPatitions, mapValues等。





# Spark Job阶段划分

![](https://i.loli.net/2019/11/02/Iw4YD79qiK2h6kp.jpg)

