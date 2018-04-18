# 命令格式
hadoop fs：使用面最广，可以操作任何文件系统。
hadoop dfs与hdfs dfs：只能操作HDFS文件系统相关（包括与Local FS间的操作），前者已经Deprecated，一般使用后者。

# 具体命令
## shell命令（这个方法是最基本的）
```linux
hadoop fs  -ls	<path>	表示对hdfs下一级目录的查看 #常用
hadoop fs -lsr	<path>	表示对hdfs目录的递归查看
hadoop fs -mkdir	<path>	创建目录 #常用
hadoop fs  -put	<src>	<des>	从linux上传文件到hdfs #常用
hadoop fs  -get	<src>	<des>	从hdfs下载文件到linux
hadoop fs  -text	<path>	查看文件内容
hadoop fs  -rm	<path>	表示删除文件
hadoop fs  -rmr	<path>	表示递归删除文件
```
## 先查看该账号下的所有内容

输入`hadoop fs -ls`

```linux
drwxrwxrwx   - baseDepSarch baseDepSarch          0 2018-04-18 08:00 .Trash
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2017-09-27 15:49 .hiveJars
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2018-04-18 09:32 .sparkStaging
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2018-03-07 12:05 Jhanlp
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2018-03-07 19:29 Jhanlp2
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2016-12-27 19:56 correct
drwxrwxrwx   - baseDepSarch baseDepSarch          0 2016-06-03 17:17 mr-tmp
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2018-04-14 22:18 newcars.csv
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2018-04-14 22:13 output.csv
drwxr-xr-x   - baseDepSarch baseDepSarch          0 2016-12-30 13:00 temp
```

因为使用spark读取txt文件时，也是这么读取的：

```linux
scala> val test = spark.read.textFile("1.txt")
org.apache.spark.sql.AnalysisException: Path does not exist: hdfs://kgdc/user/baseDepSarch/1.txt;
```

## 再新建目录，再上传文件

```
hadoop fs -mkdir Jquery
hadoop fs -put 1.txt Jquery/
```

然后就能用spark直接读取了，默认保存为dataset格式

```scala
val test = spark.read.textFile("Jquery/1.txt")
```

## 最后可以保存文件

```scala
val combinefunc = udf{(a:String, b:String, c:String) => 
  if (c != null)
    { a + "|" + b + "|" + c }
  else
    { a + "|" + b + "|" + "0" }
}
val df_save = df_num.withColumn("save", combinefunc($"term", $"result".cast("String"), $"resultnum")).select($"save")
df_save.write.format("csv").option("header", "false").save(s"Jquery/$date_end$-suggest.txt")
hadoop fs  -get	Jquery/2018-04-17-suggest.txt suggest
```

本来可以直接在上面save那部分保存到硬盘，但是有error，以后再解决。

```linux
cat suggest/part* > 2018-04-17-suggest-spark.csv
```

