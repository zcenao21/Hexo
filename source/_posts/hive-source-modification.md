---
title: hive源码调试入门
date: 2020-02-21 12:45:21
tags:
    - hive
    - 源码
    - 开发
categories: 
    - Hive
---



在windows上折腾安装好了hadoop，hive因为文件路径太长不支持等各种奇奇怪怪的问题始终运行不起来，下了大决心放弃windows系统，unbuntu走起！重装系统还算顺利，几个小时搞定。然后就是ubuntu上运行hadoop,成功; ubuntu上运行hive，成功！下一步hive源码走起。<!--more-->

hive依赖hadoop，看hive启动脚本，最终是调用hadoop启动，而hadoop最终还是执行的java -jar xxx.jar形式，所以决定先从hive java源码入口看。先试一试改动源码打印个信息。

首先在hive的main函数入口增加一行

```
System.out.println("Will's first hive source code modification: test err print info");
```

<img src="https://i.loli.net/2020/02/21/bQOyidqXgsMR3Ca.png" alt="启动" style="zoom:80%;" />

打包，然后替换lib目录下hive-cli-xxx.jar。

运行hive

![选区_001.png](https://i.loli.net/2020/02/21/LZ16gVt4rfdXqyu.png)



好啦，正式开启与hive源码的斗争！

