---
title: hadoop在windows 10下安装步骤
date: 2020-02-15 20:05:16
tags: 
    - hadoop
    - hadoop安装
    - windows 10
categories: 
    - hadoop
---

# 序言

首先劝大家在有条件情况下能用mac就用mac，再不行用linux系统，在windows上运行hadoop不是一个好主意！真的好麻烦。。。

<!--more-->



# 准备文件

- 在官网上下载hadoop的压缩包
- 然后有个github项目专门做windows下配置的包，具体链接需要自己搜一下

我用的是hadoop 2.10.0，然后配置文件和hadoop包都在里面，需要自己下载

```
链接：https://pan.baidu.com/s/18ZVB89xOUq43gJ7cqlZUGA 
提取码：wj3v 
```



# 安装步骤

- 安装好java环境，这是基础，网上一堆教程
- 解压缩hadoop压缩包，然后解压下载的另一个配置文件，直接拷贝覆盖即可

这里卡了好久，因为文件路径太长经常解压后部分文件无法解压成功，这个可以参考链接：[点这里]( https://knowledge.autodesk.com/zh-hans/search-result/caas/sfdcarticles/sfdcarticles/CHS/The-Windows-10-default-path-length-limitation-MAX-PATH-is-256-characters.html)，最好放在盘的第一层，我就放在C:\下面

- 配置hadoop环境变量

  我的电脑->属性->高级系统设置->环境变量->系统变量

  新建HADOOP_HOME, 我的配置：C:\hadoop-2.10.0\bin

  ![配置](https://wx4.sinaimg.cn/mw690/b146837bly1gbxf3b0kv0j20s9071dfu.jpg)

​     在PATH变量中添加：%HADOOP_HOME%

- 编辑 hadoop安装目录下 etc/hadoop/hadoop-env.cmd, 替换JAVA_HOME路径，"D:\program files\Java\jdk1.8.0_171"为JAVA安装路径。

  set JAVA_HOME="D:\program files\Java\jdk1.8.0_171"

  然后编辑在hadoop根目录下创建data目录，目录中再创建两个空文件夹datanode和namenode，之后编辑 etc/hadoop/hdfs-site.xml，替换路径/hadoop-2.10.0/为你的hadoop安装根目录

  ```
  <configuration>
   <property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>
      <property>
          <name>dfs.namenode.name.dir</name>
          <value>/hadoop-2.10.0/data/namenode</value>
      </property>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>/hadoop-2.10.0/data/datanode</value>
      </property>
  </configuration>
  ```

- 格式化namenode

  ```
  在任意目录执行 hdfs namenode -format
  ```

- 到安装根目录下的sbin目录，执行

  ```
  start-all.cmd
  ```

  ![success](https://wx2.sinaimg.cn/mw690/b146837bly1gbxfmuf908j20z50li7gm.jpg)

  验证是否成功：

  ```
  jps
  ```

  会有以下进程在运行：

  NodeManager
  DataNode
  ResourceManager
  NameNode



# 问题及解决方法

```
java.lang.NoClassDefFoundError: org/apache/hadoop/yarn/server/timelineservice/collector/TimelineCollectorManager
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
        at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
        at java.net.URLClassLoader.defineClass(URLClassLoader.java:467)
        at java.net.URLClassLoader.access$100(URLClassLoader.java:73)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:368)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:362)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:361)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        at java.lang.Class.getDeclaredMethods0(Native Method)
        at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
        at java.lang.Class.getDeclaredMethods(Class.java:1975)
        at com.google.inject.spi.InjectionPoint.getInjectionPoints(InjectionPoint.java:688)
        at com.google.inject.spi.InjectionPoint.forInstanceMethodsAndFields(InjectionPoint.java:380)
        at com.google.inject.spi.InjectionPoint.forInstanceMethodsAndFields(InjectionPoint.java:399)
        at com.google.inject.internal.BindingBuilder.toInstance(BindingBuilder.java:84)
        at org.apache.hadoop.yarn.server.resourcemanager.webapp.RMWebApp.setup(RMWebApp.java:56)
        at org.apache.hadoop.yarn.webapp.WebApp.configureServlets(WebApp.java:160)
        at com.google.inject.servlet.ServletModule.configure(ServletModule.java:55)
        at com.google.inject.AbstractModule.configure(AbstractModule.java:62)
        at com.google.inject.spi.Elements$RecordingBinder.install(Elements.java:340)
        at com.google.inject.spi.Elements.getElements(Elements.java:110)
        at com.google.inject.internal.InjectorShell$Builder.build(InjectorShell.java:138)
        at com.google.inject.internal.InternalInjectorCreator.build(InternalInjectorCreator.java:104)
        at com.google.inject.Guice.createInjector(Guice.java:96)
        at com.google.inject.Guice.createInjector(Guice.java:73)
        at com.google.inject.Guice.createInjector(Guice.java:62)
        at org.apache.hadoop.yarn.webapp.WebApps$Builder.build(WebApps.java:356)
        at org.apache.hadoop.yarn.webapp.WebApps$Builder.start(WebApps.java:401)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.startWepApp(ResourceManager.java:1137)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.serviceStart(ResourceManager.java:1245)
        at org.apache.hadoop.service.AbstractService.start(AbstractService.java:194)
        at org.apache.hadoop.yarn.server.resourcemanager.ResourceManager.main(ResourceManager.java:1446)
Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.yarn.server.timelineservice.collector.TimelineCollectorManager
        at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 36 more
```

**解决方法： **share\hadoop\yarn\timelineservice 下 hadoop-yarn-server-timelineservice-3.0.3.jar copy 到share\hadoop\yarn目录下 