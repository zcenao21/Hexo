---
title: hadoop在ubuntu16.0.4的源码调试
date: 2020-02-15 20:05:16
tags: 
    - hadoop
    - hadoop调试
    - 源码
categories: 
    - hadoop
---

# 序言



# 问题及解决方法

### 找不到或者无法加载主类xxx.xxx

**解决方法：** 点击菜单栏`Run-->Edit configuration`，在出现的窗口中，勾选`Include dependencies with 'Provided' scope`，点击OK，重新运行



### 打日志问题

添加log4j到项目中，我的内容

```
log4j.rootLogger=info, ServerDailyRollingFile, stdout
log4j.appender.ServerDailyRollingFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.ServerDailyRollingFile.DatePattern='.'yyyy-MM-dd
log4j.appender.ServerDailyRollingFile.File=/tmp/log-hadoop-will/hadoop.log
log4j.appender.ServerDailyRollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.ServerDailyRollingFile.layout.ConversionPattern=%d - %m%n
log4j.appender.ServerDailyRollingFile.Append=true
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss} %p [%c] %m%n
```



### resourceManager启动报错

```
java.lang.IllegalStateException: Queue configuration missing child queue names for root
    at org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler.parseQueue(CapacityScheduler.java:558)
    at org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler.initializeQueues(CapacityScheduler.java:463)
    at org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler.initScheduler(CapacityScheduler.java:295)
    at org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler.serviceInit(CapacityScheduler.java:324)
    at org.apache.hadoop.service.AbstractService.init(AbstractService.java:163)
```

hadoop-yarn-server-resourcemanagerde的子目录java/resouces下新建的capacity-scheduler.xml，内容如下：

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<configuration>
  <property>
    <name>yarn.scheduler.capacity.root.queues</name>
    <value>alpha,beta,default</value>
    <description>
      The queues at the this level (root is the root queue).
    </description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.alpha.capacity</name>
    <value>50</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.beta.capacity</name>
    <value>30</value>
    <description>Default queue target capacity.</description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.capacity</name>
    <value>20</value>
    <description>Default queue target capacity.</description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.alpha.state</name>
    <value>RUNNING</value>
    <description>
      The state of the default queue. State can be one of RUNNING or STOPPED.
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.beta.state</name>
    <value>RUNNING</value>
    <description>
      The state of the default queue. State can be one of RUNNING or STOPPED.
    </description>
  </property>

  <property>
    <name>yarn.scheduler.capacity.root.default.state</name>
    <value>RUNNING</value>
    <description>
      The state of the default queue. State can be one of RUNNING or STOPPED.
    </description>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.acl_submit_applications</name>
    <value>hadoop,yarn,mapred,hdfs</value>
    <description>
      The ACL of who can submit jobs to the root queue.
    </description>
  </property>
</configuration>

```

然后设置idea![resourceManager设置](https://github.com/zcenao21/photos-blog/blob/main/hadoop-debug/resouceManeger%E9%85%8D%E7%BD%AE.png?raw=true)

### not

![idea配置](https://raw.githubusercontent.com/zcenao21/photos-blog/main/hadoop-debug/namenode-webapp-problem.png)