---
title: Spark使用
date: 2021-04-21 08:32:50
tags:
    - Spark
    - Spark使用
categories: 
    - Spark
---



### Spark运行模式

Spark运行方式分为3种

- 本地运行模式(单机)
- 本地伪集群运行模式(单机模拟集群)
- 集群运行模式

本地模式一般用于初步调试或者学习，本地伪集群模式提供了真实集群运行模式的模拟环境。<!--more-->



### 本地运行模式下的任务提交

比如运行Spark提供的计算PI的例子

```
./spark-submit --class org.apache.spark.examples.SparkPi --master "local[2]" ../examples/target/original-spark-examples_2.12-3.2.0-SNAPSHOT.jar
```

Spark 提供了 Web UI 来对 Spark 应用进行监控。每个 SparkContext 会启动一个 web UI，默认是在 4040 端口。它显示了应用相关的有用的信息，包括：

- 一系列的 scheduler stage 和 task
- RDD 大小和内存占用的概要
- 环境信息
- 正在运行的 executor 的一些信息

但是由于上面的程序执行过程很快，可能来不及打开。或者执行完了怎么在Web上查看呢？我们可以使用spark history server的功能

首先设置conf/spark-defaults.conf(如果没有，拷贝conf/spark-defaults.conf.template)，参数如下：

```
spark.eventLog.enabled           true
spark.eventLog.dir               /home/will/study/projects/spark/tmp/spark-history-server
spark.history.fs.logDirectory    file:/home/will/study/projects/spark/tmp/spark-history-server
```

设置成功后运行history server，具体操作是执行sbin/start-history-server.sh，启动history server。启动后访问http://localhost:18080/即可看到执行历史

![](https://github.com/zcenao21/photos-blog/raw/main/spark/history-server.png)



### client模式

```
./spark-submit --class org.apache.spark.examples.SparkPi --master spark://localhost:7077 ../examples/target/original-spark-examples_2.12-3.2.0-SNAPSHOT.jar
```



### Debug

启动之后如果想看进行单步调试，可以进行如下操作

首先启动时设置远程监听

```
./spark-submit --class org.apache.spark.examples.SparkPi --master spark://localhost:7077 --conf spark.driver.extraJavaOptions="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=7777" ../examples/target/original-spark-examples_2.12-3.2.0-SNAPSHOT.jar 
```

然后在IDEA中设置调试参数

![spark-submit-debug.png](https://github.com/zcenao21/photos-blog/raw/main/spark/spark-submit-debug.png)

-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=7777

再在需要的地方打断点

![debug-breakpoint.png](https://github.com/zcenao21/photos-blog/raw/main/spark/debug-breakpoint.png)



### 任务提交

通过spark-submit提交任务，spark-submit所做的工作是设置一些环境参数，比如检测Java安装目录，设定SPARK_HOME变量等，然后提交运行任务

spark 3.2 版本spark-submit内容

```
#!/usr/bin/env bash

# 若没有设定SPARK_HOME则使用脚本查找设置
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

# disable randomized hash for string in Python 3.3+
export PYTHONHASHSEED=0

exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

设置SPARK_HOME目录后运行脚本spark-class，这个脚本主要用于设置JAVA_HOME(Java环境)及LAUNCH_CLASSPATH(spark运行依赖的jar包)。

最终调用org.apache.spark.deploy.SparkSubmit这个类，提交任务。

在org.apache.spark.deploy.SparkSubmit这个类中，调用doSubmit方法

```
override def main(args: Array[String]): Unit = {
    val submit = new SparkSubmit() {
      self =>
      ... // 省略部分代码
      override def doSubmit(args: Array[String]): Unit = {
        try {
          super.doSubmit(args)
        } catch {
          case e: SparkUserAppException =>
            exitFn(e.exitCode)
        }
      }
    }

    submit.doSubmit(args)
  }
```

在doSubmit方法中,再选择调用submit(appArgs, uninitLog)

```
def doSubmit(args: Array[String]): Unit = {
    // Initialize logging if it hasn't been done yet. Keep track of whether logging needs to
    // be reset before the application starts.
    val uninitLog = initializeLogIfNecessary(true, silent = true)

    val appArgs = parseArguments(args)
    if (appArgs.verbose) {
      logInfo(appArgs.toString)
    }
    appArgs.action match {
      case SparkSubmitAction.SUBMIT => submit(appArgs, uninitLog)
      case SparkSubmitAction.KILL => kill(appArgs)
      case SparkSubmitAction.REQUEST_STATUS => requestStatus(appArgs)
      case SparkSubmitAction.PRINT_VERSION => printVersion()
    }
  }
```

在submit方法中，又选择调用了doRunMain()方法，接着调用runMain(args, uninitLog)方法。接下来

```
val app: SparkApplication = if (classOf[SparkApplication].isAssignableFrom(mainClass)) {
   mainClass.getConstructor().newInstance().asInstanceOf[SparkApplication]
    } else {
      new JavaMainApplication(mainClass)
    }
... // 省略代码
try {
	app.start(childArgs.toArray, sparkConf)
} catch {
	case t: Throwable => throw findCause(t)
}
```

在app.start中，反射调用

```
mainMethod.invoke(null, args)
```

于是调用了我们自己的spark程序，以PI计算程序为例

```
val spark = SparkSession
                .builder
                .appName("Spark Pi")
                .getOrCreate()
```

在getOrCreate()方法中



### Master程序调试

版本：３.2.0-SNAPSHOT

在start-master.sh文件中添加如下参数

```
export SPARK_MASTER_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=10000"
```

然后在运行start-master.sh，运行后在命令行执行`jps`

```
8817 Main
753 Jps
623 -- main class information unavailable
```

再在idea端增加remote配置，端口配置为上述参数中的10000，use module classpath选择spark-core_2.12。之后在org.apache.spark.deploy.master.Master中的main函数上打断点。最后点击Debug开始调试