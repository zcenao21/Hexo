---
title: Spark绪论
date: 2019-10-24 16:33:13
tags: 
    - Spark
    - High Performance Spark
    - 大数据工具
categories: 
    - Spark
---


# 前言

- 为什么会有spark

  现在的计算机性能越来越高，单台机器处理M级别的数据简直小菜一碟。但是如果数据量更大，达到G、T甚至P、E级别，这时单台机器的性能可能就有点不够了。也许你会反驳，目前世界上最快的超级计算<!--more--> 机“Summit”浮点运算速度峰值可达20亿亿次/秒，这相当快了吧！确实很快，但是它的使用和维护费用都很高呀，一般人或者说公司根本负担不起。所以聪明的人类就发明了大数据工具。虽然单台机器的计算性能可能并不是很好，但是如果多台机器联合起来一起做计算，那么不就可以实现相对廉价的机器实现超大数据规模的计算吗？所以诞生了大数据工具Hadoop。Hadoop基于HDFS和Map-Reduce奠定了大数据领域的霸主地位。随后各种工具层出不穷。Spark针对Hadoop计算中慢的缺点，即每次计算结果都要读写到磁盘导致速度变慢的设计，作了如下改进：

  1. 计算中间结果保存在内存中。Hadoop每次的计算结果都会放回HDFS，这种设计严重影响了计算速度。spark基于内存的设计则改进了这一点。
  2. 惰性计算。Spark使用有向无环图（DAG）调度机制，遇到action操作算子才进行实际计算，所有就有了优化空间。

  > 可参考对比：https://www.zhihu.com/question/26568496

- Spark是什么

  官方定义：**Apache Spark™** is a unified analytics engine for large-scale data processing。翻译过来就是：用于大规模数据处理的分析引擎。再直白一点就是说：spark是用于快速处理大量数据的工具。

  <img src="https://ericfu.me/images/2018/06/spark-banner.png" width="700" hegiht="113" align=center />



# 和其他工具对比

> 引用自：https://www.boxuegu.com/news/458.html



- **Hadoop框架**

  提起大数据，第一个想起的肯定是Hadoop，因为Hadoop是目前世界上应用最广泛的大数据工具，它凭借极高的容错率和极低的硬件价格，在大数据市场上风生水起。Hadoop还是第一个在开源社区上引发高度关注的批处理框架，他提出的Map和Reduce的计算模式简洁而优雅。迄今为止，Hadoop已经成为了一个广阔的生态圈，实现了大量算法和组件。由于Hadoop的计算任务需要在集群的多个节点上多次读写，因此在速度上会稍显劣势，但是其吞吐量也同样是其他框架所不能匹敌的。



- **Storm框架**
  与Hadoop的批处理模式不同，Storm采用的是流计算框架，由Twitter开源并且托管在GitHub上。与Hadoop类似的是，Storm也提出了两个计算角色，分别为Spout和Bolt。如果说Hadoop是水桶，只能一桶一桶的去井里扛，那么Storm就是水龙头，只要打开就可以源源不断的出水。Storm支持的语言也比较多，Java、Ruby、Python等语言都能很好的支持。由于Storm是流计算框架，因此使用的是内存，延迟上有极大的优势，但是Storm不会持久化数据。

 

- **Samza框架**
  Smaza也是一种流计算框架，但他目前只支持JVM语言，灵活度上略显不足，并且Samza必须和Kafka共同使用。但是响应的，其也继承了Kafka的低延时、分区、避免回压等优势。对于已经有Hadoop+Kafka工作环境的团队来说，Samza是一个不错的选择，并且Samza在多个团队使用的时候能体现良好的性能。

 

- **Spark框架**
  Spark属于前两种框架形式的集合体，是一种混合式的计算框架。它既有自带的实时流处理工具，也可以和Hadoop集成，代替其中的MapReduce，甚至Spark还可以单独拿出来部署集群，但是还得借助HDFS等分布式存储系统。Spark的强大之处在于其运算速度，与Storm类似，Spark也是基于内存的，并且在内存满负载的时候，硬盘也能运算，运算结果表示，Spark的速度大约为Hadoop的一百倍，并且其成本可能比Hadoop更低。但是Spark目前还没有像Hadoop哪有拥有上万级别的集群，因此现阶段的Spark和Hadoop搭配起来使用更加合适。

 

- **Flink框架**
  Flink也是一种混合式的计算框架，但是在设计初始，Fink的侧重点在于处理流式数据，这与Spark的设计初衷恰恰相反，而在市场需求的驱使下，两者都在朝着更多的兼容性发展。Flink目前不是很成熟，更多情况下Flink还是起到一个借鉴的作用。


