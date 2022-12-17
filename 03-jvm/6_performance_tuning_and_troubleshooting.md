# Performance Tuning

## 内存相关

### 堆

#### -Xmx

指定堆的最大大小。

#### -Xms

指定堆的最小大小。

#### -Xmn

指定新生代的大小。

<!--
-Xss

-XX:SurvivorRatio

-XX:MaxTenuringThreshold

-XX:NewSize

-XX:MaxNewSize

-XX:NewRatio
-->

### 方法区

#### -XX:PermSize

设置方法区(永久代)的初始(最小)大小。

#### -XX:MaxPermSize

设置方法区(永久代)的最大大小。

#### -XX:MetaspaceSize

设置方法区(元空间)的初始(最小)大小。

#### -XX:MaxMetaspaceSize

设置方法区(元空间)的最大大小。

## GC相关

### 垃圾收集器

#### -XX:+UseSerialGC

#### -XX:+UseParallelGC

#### -XX:+USeParNewGC

#### -XX:+UseG1GC

### GC日志

#### -XX:+UseGCLogFileRotation

#### -XX:NumberOfGCLogFiles=< number of log files >

#### -XX:GCLogFileSize=< file size >[ unit ]

#### -Xloggc:/path/to/gc.log

# Troubleshooting

## 命令行故障处理工具

### jhat (JVM Heap Analysis Tool)

对jmap生成的堆转储快照进行分析。

### jinfo (Configuration Info for Java)

实时查看和动态修改JVM的各项参数。

### jmap (Memory Map for Java)

生成堆转储快照(i.e. headdump文件)，查看finalize执行队列、堆和方法区的详细信息。

### jps (JVM Process State Tool)

查看正在运行的虚拟机进程。

### jstat (JVM Statistics Monitoring Tool)

查看JVM的各种运行状态数据，例如与类加载、内存空间、垃圾回收等相关的数据。

### jstack (Stack Trace for Java)

生成线程快照(i.e. threaddump、javacore)，即JVM中每条线程正在执行的方法堆栈的集合。

## 可视化故障处理工具

### jconsole (Java Monitoring and Management Console)

### JHSDB

### VisualVM

### Java Mission Control


