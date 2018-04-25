# 具体命令

## 查看所有hadoop任务

```linux
yarn application -list | grep 'root.baseDepSarchQueue'
```

```linux
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/data1/app/hadoop-2.7.1/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/data1/app/hbase-0.98.9-hadoop2/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
18/04/25 18:05:45 INFO impl.TimelineClientImpl: Timeline service address: http://10.1.81.220:8188/ws/v1/timeline/
18/04/25 18:05:45 INFO client.AHSProxy: Connecting to Application History server at /10.1.81.220:10200
application_1522636232843_310997                 jimmy_spark                   SPARK    baseDepSarch    root.baseDepSarchQueue             RUNNING               UNDEFINED               10%             http://10.1.81.220:4046
```

## 杀死具体的hadoop任务

根据上面找到的应用id进行查杀

```linux
yarn application -kill application_1522636232843_310997
```

# Reference

-[官网](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/YarnCommands.html)