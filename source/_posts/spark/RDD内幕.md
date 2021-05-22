---
title: RDD内幕
date: 2021-04-17 21:34:50
tags:
    - Spark
    - RDD
    - RDD内幕
categories: 
    - Spark
---



# RDD分区

RDD内部使用分区表示并行计算的一个计算单元，因此分区和并发计算有关系。RDD的分区源码：<!--more--> 

```
package org.apache.spark

/**
 * An identifier for a partition in an RDD.
 */
trait Partition extends Serializable {
  /**
   * Get the partition's index within its parent RDD
   */
  def index: Int

  // A better default implementation of HashCode
  override def hashCode(): Int = index

  override def equals(other: Any): Boolean = super.equals(other)
}
```

在org.apache.spark.rdd.RDD中，定义了getPartitions方法，可以获取分区。

分区个数如何决定呢？

例如map操作得到的RDD分区由第一个父RDD分区决定

```
override def getPartitions: Array[Partition] = firstParent[T].partitions
```

再比如parallize操作的分区个数由用户指定，默认为集群核心数目

```
/** Distribute a local Scala collection to form an RDD.
   *
   * @note Parallelize acts lazily. If `seq` is a mutable collection and is altered after the call
   * to parallelize and before the first action on the RDD, the resultant RDD will reflect the
   * modified collection. Pass a copy of the argument to avoid this.
   * @note avoid using `parallelize(Seq())` to create an empty `RDD`. Consider `emptyRDD` for an
   * RDD with no partitions, or `parallelize(Seq[T]())` for an RDD of `T` with empty partitions.
   * @param seq Scala collection to distribute
   * @param numSlices number of partitions to divide the collection into
   * @return RDD representing distributed collection
   */
  def parallelize[T: ClassTag](
      seq: Seq[T],
      numSlices: Int = defaultParallelism): RDD[T] = withScope {
    assertNotStopped()
    new ParallelCollectionRDD[T](this, seq, numSlices, Map[Int, Seq[String]]())
  }
```

defaultParallelism的设置，可以看LocalSchedulerBackend源码

```
override def defaultParallelism(): Int =
    scheduler.k.getInt("spark.default.parallelism", totalCores)
```




> 参考
https://ihainan.gitbooks.io/spark-source-code/content/section1/rddPartitions.html