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



hive自定义函数(udf)和自定义聚合函数(udaf,user defined aggregation function)使用<!--more-->



# hive udf

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

