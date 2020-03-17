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
hadoop fs  -du [-s] <path> 表示查看使用大小，可以用来查看hive表是否存在和大小
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

## 查看hive表大小和是否存在

`hadoop fs -du -s 地址`

```linux
hadoop fs -du hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_click_position
```

```linux
28082570496  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-15
28347522905  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-16
28451418570  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-17
28974193022  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-18
28697595226  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-19
28361792579  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-20
28323134402  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-21
28030791576  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-22
27374779009  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-23
25678904223  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-24
25426168507  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-25
24759479102  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-26
25922573013  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-27
26865686350  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-28
28861301651  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-29
29243689250  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-30
30334792834  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-01-31
31840135743  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-01
32616016550  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-02
33944398808  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-03
33728012772  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-04
33315320040  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-05
32677355507  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-06
32283386308  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-07
32621850031  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-08
31910388926  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-09
29985558216  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-10
30869948302  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-11
31117253720  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-12
31484468005  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-13
33376429733  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-14
32550139760  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-15
31841061840  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-16
29736600054  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-17
29890651808  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-18
29505517592  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-19
29979088852  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-20
30181753818  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-21
31011655782  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-22
31145727971  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-23
29544695196  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-24
29232691413  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-25
24531071831  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-26
29832747720  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-27
29836908395  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-28
30747143618  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-02-29
30095905648  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-01
28953557221  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-02
28757941242  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-03
29037758801  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-04
28273193881  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-05
29417974917  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-06
30296806472  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-07
30854349071  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-08
30004787218  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-09
29082025352  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-10
28803772230  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-11
29502586901  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-12
29718959473  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-13
30260397699  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-14
30122442429  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-15
29034914061  hdfs://XXXX/user/hive/warehouse/temp.db/jomei_search_cm_9156_click_edit/cdt=2020-03-16
```

