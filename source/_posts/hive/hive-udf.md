---
title: hive udf&udaf
date: 2020-02-21 18:20:21
tags:
    - hive
    - udf
    - udaf
categories: 
    - Hive
---



hive为我们定义了很多函数，大多数情况下是能够满足我们的需求的。但是在有些情况下，很有必要自己定义一些函数，这样使用起来非常方便。这就是自定义函数(udf)和自定义聚合函数(udaf,user defined aggregation function)。udf是每行返回一个结果，而udaf则是聚合的结果，多行产生一个结果。另外还有udtf，是一行产生多个结果。目前我还没有用到过udtf，就先不介绍它了。下面分别用两个示例介绍udf和udaf<!--more-->



# udf

### 功能

查找array中是否包含被查询值



### 步骤

- 测试数据准备

  ```
  zhangsan        beijing,shanghai,tianjin,hangzhou
  lisi    changchu,chengdu,wuhan
  ```

- hive建表与导入

  ```
  Create table users(name string, worklocations array<string> ) row format delimited fields terminated by '\t' collection items terminated by ','; 
  
  load data local inpath '/root/person.txt ' OVERWRITE INTO TABLE users; 
  ```

- udf包生成与导入

  ```
  package com.will;
  
  import org.apache.hadoop.hive.ql.exec.UDF;
  import java.util.ArrayList;
  
  public class FindInArray extends UDF {
      public ArrayList<String> evaluate(String keywords, ArrayList<String> column){
          //参数类型使用arraylist<String>对应hive中的array<string>,而不是String[]
          if(column.contains(keywords)){
              return column;
          }else{
              return null;
          }
      }
      
      public String evaluate(String keywords,ArrayList<String> column,String name){
          //重载evaluate，另一种查询方式，返回name值
          if(column.contains(keywords)){
              return name;
          }else{
              return null;
          }
      }
  }
  ```

  使用mvn 打包

  ```
  mvn clean package
  ```

- 导入hive

  ```
  add jar /home/will/work/projects/hive_udf_test/target/hive_udf_test-1.0-SNAPSHOT.jar;
  create temporary function find_in_array as 'com.will.FindInArray';
  ```

- 使用

  ```
  hive> select find_in_array('beijing',worklocations) from users;
  OK
  ["beijing","shanghai","tianjin","hangzhou"]
  NULL
  Time taken: 0.424 seconds, Fetched: 2 row(s)
  ```

> 参考：https://blog.csdn.net/Nougats/article/details/71158318



# udaf

> 参考： 
>
> https://blog.51cto.com/xiaolanlan/2397771
>
> https://www.cnblogs.com/Rudd/p/5137612.html



### 基础知识

hive的udtf有两种，Simple和Generic。这是由于历史问题，Simple先出现，但是因为反射问题效率较低，所以现在推荐使用Generic的写法。

- Simple。即继承`org.apache.hadoop.hive.ql.exec.UDAF`类，并在派生类中以静态内部类的方式实现`org.apache.hadoop.hive.ql.exec.UDAFEvaluator`接口。在Hive源码包`org.apache.hadoop.hive.contrib.udaf.example`中包含几个示例。可以直接参阅。但是这些接口已经被注解为Deprecated，建议不要使用这种方式开发新的UDAF函数。
- Generic。这是Hive社区推荐的新的写法，以抽象类代替原有的接口。新的抽象类`org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver`替代老的UDAF接口，新的抽象类`org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator`替代老的UDAFEvaluator接口。



hive是立足于hadoop之上，也就是hive基于mapreduce，hive sql最终还是会转化为mapreduce执行。为了实现mapreduce，udaf中用model来表示mapreduce各个阶段。

```
public static enum Mode {
        PARTIAL1,
        PARTIAL2,
        FINAL,
        COMPLETE;

        private Mode() {}
    }
```

- PARTIAL1: 这个是mapreduce的map阶段:从原始数据到部分数据聚合，将会调用**iterate()**和**terminatePartial() **
-  PARTIAL2: 这个是mapreduce的map端的Combiner阶段，负责在map端合并map的数据::从部分数据聚合到部分数据聚合，将会调用**merge()** 和 **terminatePartial()** 
- FINAL: mapreduce的reduce阶段，从部分数据的聚合到完全聚合，将会调用**merge()**和**terminate() **

---

- COMPLETE: 如果出现了这个阶段，表示mapreduce只有map，没有reduce，所以map端就直接出结果了，从原始数据直接到完全聚合，将会调用 **iterate()**和**terminate()**

 ![image.png](https://i.loli.net/2020/02/23/DlNqFRHnZ6zLuoA.png)

![image.png](https://i.loli.net/2020/02/23/IKn3Vv57jPScxs8.png)



udaf骨架示例：

```
public class GenericUDAFHistogramNumeric extends AbstractGenericUDAFResolver {
  static final Log LOG = LogFactory.getLog(GenericUDAFHistogramNumeric.class.getName());
 
  @Override
  public GenericUDAFEvaluator getEvaluator(GenericUDAFParameterInfo info) throws SemanticException {
    // 这里主要做类型检查
 
    return new GenericUDAFHistogramNumericEvaluator();
  }
 
  public static class GenericUDAFHistogramNumericEvaluator extends GenericUDAFEvaluator {
         // 确定各个阶段输入输出参数的数据格式ObjectInspectors
         public  ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException{
             return  null;
         }

         // 保存数据聚集结果的类
         public AggregationBuffer getNewAggregationBuffer() throws HiveException {
             return null;
         }

		 // 重置聚集结果
         public void reset(AggregationBuffer aggregationBuffer) throws HiveException {}

         // map阶段，迭代处理输入sql传过来的列数据 
         public void iterate(AggregationBuffer aggregationBuffer, Object[] objects) throws HiveException {}

         // map与combiner结束返回结果，得到部分数据聚集结果
         public Object terminatePartial(AggregationBuffer aggregationBuffer) throws HiveException {
             return null;
         }

         // combiner合并map返回的结果，还有reducer合并mapper或combiner返回的结果。
         public void merge(AggregationBuffer aggregationBuffer, Object o) throws HiveException {}

		 // reducer阶段，输出最终结果 
         public Object terminate(AggregationBuffer aggregationBuffer) throws HiveException {
             return null;
         }
  }
}
```





### 功能

统计字符数



### 代码

```
package com.will;

import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.parse.SemanticException;
import org.apache.hadoop.hive.ql.udf.generic.AbstractGenericUDAFResolver;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDAFEvaluator;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.PrimitiveObjectInspector;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfo;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfoUtils;

public class TotalNumOfLetttersGenericUDAF extends AbstractGenericUDAFResolver {
    @Override
    public GenericUDAFEvaluator getEvaluator(TypeInfo[] parameters) throws SemanticException {

        if (parameters.length != 1) {
            throw new UDFArgumentTypeException(parameters.length - 1,"Exactly one argument is expected.");
        }

        ObjectInspector oi = TypeInfoUtils.getStandardJavaObjectInspectorFromTypeInfo(parameters[0]);

        if (oi.getCategory() != ObjectInspector.Category.PRIMITIVE){
            throw new UDFArgumentTypeException(0,
                    "Argument must be PRIMITIVE, but "
                    + oi.getCategory().name()
                    + " was passed.");
        }

        PrimitiveObjectInspector inputOI = (PrimitiveObjectInspector) oi;
        if (inputOI.getPrimitiveCategory() != PrimitiveObjectInspector.PrimitiveCategory.STRING){
            throw new UDFArgumentTypeException(0, "Argument must be String, but "
                     + inputOI.getPrimitiveCategory().name()
                     + " was passed.");
        }

        return new TotalNumOfLettersEvaluator();
    }

    public static class TotalNumOfLettersEvaluator extends GenericUDAFEvaluator{
        PrimitiveObjectInspector inputOI;
        ObjectInspector outputOI;
        PrimitiveObjectInspector integerOI;

        int total = 0;
        private boolean warned = false;

        @Override
         public  ObjectInspector init(Mode m, ObjectInspector[] parameters) throws HiveException{
            assert (parameters.length == 1);
            super.init(m, parameters);

            //map阶段读取sql列，输入为String基础数据格式
            if (m == Mode.PARTIAL1 || m == Mode.COMPLETE) {
                inputOI = (PrimitiveObjectInspector) parameters[0];
            } else {
                //其余阶段，输入为Integer基础数据格式
                integerOI = (PrimitiveObjectInspector) parameters[0];
            }

            // 指定各个阶段输出数据格式都为Integer类型
            outputOI = ObjectInspectorFactory.getReflectionObjectInspector(Integer.class,
                                        ObjectInspectorFactory.ObjectInspectorOptions.JAVA);

            return outputOI;
        }

        //存储当前字符总数的类
        static class LetterSumAgg implements AggregationBuffer {
            int sum = 0;
            void add(int num){
                sum += num;
            }
        }

         public AggregationBuffer getNewAggregationBuffer() throws HiveException {
             LetterSumAgg result = new LetterSumAgg();
             return result;
         }

         public void reset(AggregationBuffer aggregationBuffer) throws HiveException {
             LetterSumAgg myagg = new LetterSumAgg();
         }

         public void iterate(AggregationBuffer agg, Object[] parameters) throws HiveException {
             assert (parameters.length == 1);
             if (parameters[0] != null) {
                 LetterSumAgg myagg = (LetterSumAgg) agg;
                 Object p1 = ((PrimitiveObjectInspector) inputOI).getPrimitiveJavaObject(parameters[0]);
                 myagg.add(String.valueOf(p1).length());
             }
         }

         public Object terminatePartial(AggregationBuffer agg) throws HiveException {
             LetterSumAgg myagg = (LetterSumAgg) agg;
             total += myagg.sum;
             return total;
         }

         public void merge(AggregationBuffer agg, Object partial) throws HiveException {
             if (partial != null) {
                 LetterSumAgg myagg1 = (LetterSumAgg) agg;
                 Integer partialSum = (Integer) integerOI.getPrimitiveJavaObject(partial);
                 LetterSumAgg myagg2 = new LetterSumAgg();
                 myagg2.add(partialSum);
                 myagg1.add(myagg2.sum);
             }
         }

         public Object terminate(AggregationBuffer agg) throws HiveException {
             LetterSumAgg myagg = (LetterSumAgg) agg;
             total = myagg.sum;
             return myagg.sum;
         }
    }
}

```



### 验证

首先准备数据

```
hive> select * from users;
OK
zhangsan	["beijing","shanghai","tianjin","hangzhou"]
lisi	["changchu","chengdu","wuhan"]
```



然后添加jar包

```
> ADD JAR /home/will/work/projects/hive_udf_test/target/hive_udf_test-1.0-SNAPSHOT.jar;
Added [/home/will/work/projects/hive_udf_test/target/hive_udf_test-1.0-SNAPSHOT.jar] to class path
Added resources: [/home/will/work/projects/hive_udf_test/target/hive_udf_test-1.0-SNAPSHOT.jar]
```



定义函数

```
hive>  CREATE TEMPORARY FUNCTION letters as 'com.will.TotalNumOfLetttersGenericUDAF';
OK
Time taken: 0.049 seconds
```



执行

```
hive> select letters(name) from users;
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2020-02-23 13:25:06,087 Stage-1 map = 0%,  reduce = 0%
2020-02-23 13:25:11,426 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 2.03 sec
2020-02-23 13:25:16,607 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 4.01 sec
MapReduce Total cumulative CPU time: 4 seconds 10 msec
Total MapReduce CPU Time Spent: 4 seconds 10 msec
OK
12
Time taken: 23.819 seconds, Fetched: 1 row(s)
```



# my own udaf

### 要解决的问题

有一个hive表，其中两列分别代表时间戳和事件。目标是得到指定时间范围的所有事件。



### 分析

根据上一部分的介绍，要实现聚合首先要设计如何存储，传输的问题。在这个过程中我仔细研究了hive udaf示例的histogram设计，然后设计了自己的udaf。聚合存储使用hashmap，初步解析结果使用string的list保存。



代码

[github 演示项目](https://github.com/zcenao21/hive-udaf)