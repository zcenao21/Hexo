---
title: spark架构
date: 2019-11-02 12:43:16
tags: 
    - Spark
    - High Performance Spark
    - spark架构
categories: 
    - Spark
---

# Spark架构

![](https://i.loli.net/2019/11/02/TLMwH9aV1xStu5Y.png)

<!--more-->

一个sparkApplication对应 Driver程序中一个SparkContext，而SparkContext由一系列Spark job组成。一个Worker Node可以由多个Executer组成，但是Executer不能跨Worker Node。



# spark数据处理系统

 Spark可以在仅有单个JVM的单台机器上运行，但更常与分布式存储系统和集群管理器串联组成如下数据处理系统。分布式存储系统用来存放数据，集群管理器用来协调管理集群spark任务。Spark目前支持4种集群管理器：Standalone集群管理器，Apache Mesos，Hadoop YARN，EC2。

![Spark](https://i.loli.net/2019/11/02/5ijlposeYMDScIU.png)

# spark生态系统

![spark生态系统](https://i.loli.net/2019/11/02/B2JjgU7hczEba1r.png)



