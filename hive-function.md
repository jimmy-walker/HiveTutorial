# Hive常用函数

##1.统计数目
```
count(*)    所有值不全为NULL时，加1操作
count(1)    不管有没有值，只要有这条记录，值就加1
count(col)  col列里面的值为null，值不会加1，这个列里面的值不为NULL，才加1
```
##2.HAVING弥补了WHERE 关键字无法与合计函数一起使用
```hive
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```
##3.Group by而非distinct，避免数据倾斜至一个reduce
```hive
# distinct
Stage-Stage-1: Map: 396  Reduce: 1   Cumulative CPU: 7915.67 sec   HDFS Read: 119072894175 HDFS Write: 10 SUCCESS
# group by
Stage-Stage-1: Map: 396  Reduce: 457   Cumulative CPU: 10056.7 sec   HDFS Read: 119074266583 HDFS Write: 53469 SUCCESS
数据倾斜
```

##4.order by和sort by的区别
Hive中的order by跟传统的sql语言中的order by作用是一样的，会对查询的结果做一次全局排序，所以说，只有hive的sql中制定了order by所有的数据都会到同一个reducer进行处理（不管有多少map，也不管文件有多少的block只会启动一个reducer）。但是对于大量数据这将会消耗很长的时间去执行。
Hive中指定了sort by，那么在每个reducer端都会做排序，也就是说保证了局部有序（每个reducer出来的数据是有序的，但是不能保证所有的数据是有序的，除非只有一个reducer），好处是：执行了局部排序之后可以为接下去的全局排序提高不少的效率（其实就是做一次归并排序就可以做到全局排序了）。

##5.case when条件判断
case when语法：
```hive
CASE  [ expression ] 
  WHEN condition1 THEN result1 
  WHEN condition2 THEN result2 
  ... 
  WHEN conditionn THEN resultn 
  ELSE result 
END
```

###使用场合1：在select中使用
```hive
SELECT
  CASE WHEN fo LIKE '搜索结果/%' THEN split(fo,'/')[2] WHEN fo LIKE '音乐电台/公共电台/综合搜索/%' THEN split(fo,'/')[3] END kw,
```
**<u>注意，在聚合函数后的一个空格加上一个名字表示别名。**</u>

###适用场合2：与聚合函数搭配使用
```hive
SUM(CASE WHEN sex_age.sex='Male' THEN sex_age.age ELSE 0 END)
```

##6.coalesce返回值条件判断
COALESCE(a1, a2, ...)  返回第一个非null的参数值，如果参数全为null，返回null。

##7.hive字符串函数
hive中字符串函数，常常用于where条件和select中。

trim(string A)：去掉字符串A两端的空格。

运算符`A <> B`，意思是NULL if A or B is NULL, TRUE if expression A is NOT equal to expression B, otherwise FALSE.

```hive
where trim(fs) <> '播放错误'
```
split(string str, string pat)：用pat分割字符串str，pat为正则表达式，返回为一个数组
```hive
where split('fo_2','/')[2] like '薛之谦%'
```
upper(string A)或ucase(string A)：返回字符串A的大写形式

##8.类型转换
cast(salary AS BIGINT)，用于select和where中
```hive
cast(st as bigint)
```
<u>此例子中是将string格式转为int，因为数字可能很大，所以设置成bigint。</u>

## 9.指定精度取整函数

round(a, d)，返回四舍五入保留d个小数位，默认为0个小数位，但是会有`.0`的情况。

## 10.绝对值

abs返回绝对值

## 11.分组计算

类似pandas中的分组再计算的效果，hive可用ROW_NUMBER()
ROW_NUMBER() OVER (partition BY COLUMN_A ORDER BY COLUMN_B ASC/DESC) rn
rn 是排序的别名执行时每组的编码从1开始 
partition by：类似hive的建表，分区的意思；COLUMN_A 是分组字段 
order by ：排序，默认是升序，加desc降序；COLUMN_B 是排序字段

```hive
select sh,sn from (select sh,sn,row_number() over(partition by sh order by cnt desc) rn from (select sh,sn,count(1) cnt from pc_part where spt_cnt>=10 group by sh,sn) c0) c1 where rn=1
```
## 12.通配符

hive通配符与sql通配符类似，在where中与like搭配使用

%	替代 0 个或多个字符
_	替代一个字符
[charlist]	字符列中的任何单一字符
[^charlist]或[!charlist]	不在字符列中的任何单一字符

## 13.正则表达式解析函数

语法: regexp_extract(string subject, string pattern, int index)

返回值: string

说明：将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符。
```linux
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 1) fromiteblog;
the
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 2) fromiteblog;
bar
hive> select regexp_extract('foothebar', 'foo(.*?)(bar)', 0) fromiteblog;
foothebar
```
注意，在有些情况下要使用转义字符，下面的等号要用双竖线转义，这是java正则表达式的规则。

```hive
select data_field,
     regexp_extract(data_field,'.*?bgStart\\=([^&]+)',1) as aaa,
     regexp_extract(data_field,'.*?contentLoaded_headStart\\=([^&]+)',1) as bbb,
     regexp_extract(data_field,'.*?AppLoad2Req\\=([^&]+)',1) as ccc
     from pt_nginx_loginlog_st
     where pt = '2012-03-26'limit 2;
```
## 14.RLIKE和regexp，功能一样

like不是正则，而是通配符。这个通配符可以看一下SQL的标准，例如%代表任意多个字符。
rlike是正则，正则的写法与java一样。'\'需要使用'\\',例如'\w'需要使用'\\w'

## 15.OVER(PARTITION BY)函数
分析函数用于计算基于组的某种聚合值，它和聚合函数的不同之处是：对于每个组返回多行，而聚合函数对于每个组只返回一行。

示列:根据day_id日期和mac_id机器码进行聚合分组求每一天的该机器的销量和即sum_num，hive sql语句:

```hive
select day_id,mac_id,mac_color,day_num,sum(day_num)over(partition by day_id,mac_id order by day_id) sum_num from test_temp_mac_id;
```

注:day_id,mac_id,mac_color,day_num为查询原有数据,sum_num为计算结果

| day_id   | mac_id | mac_color | day_num | sum_num |
| -------- | ------ | --------- | ------- | ------- |
| 20171011 | 1292   | 金色        | 1       | 89      |
| 20171011 | 1292   | 金色        | 14      | 89      |
| 20171011 | 1292   | 金色        | 2       | 89      |
| 20171011 | 1292   | 金色        | 11      | 89      |
| 20171011 | 1292   | 黑色        | 2       | 89      |
| 20171011 | 1292   | 粉金        | 58      | 89      |
| 20171011 | 1292   | 金色        | 1       | 89      |
| 20171011 | 2013   | 金色        | 10      | 22      |
| 20171011 | 2013   | 金色        | 9       | 22      |
| 20171011 | 2013   | 金色        | 2       | 22      |
| 20171011 | 2013   | 金色        | 1       | 22      |
| 20171012 | 1292   | 金色        | 5       | 18      |
| 20171012 | 1292   | 金色        | 7       | 18      |
| 20171012 | 1292   | 金色        | 5       | 18      |
| 20171012 | 1292   | 粉金        | 1       | 18      |
| 20171012 | 2013   | 粉金        | 1       | 7       |
| 20171012 | 2013   | 金色        | 6       | 7       |
| 20171013 | 1292   | 黑色        | 1       | 1       |
| 20171013 | 2013   | 粉金        | 2       | 2       |
| 20171011 | 12460  | 茶花金       | 1       | 1       |

如果用group by实现一中根据day_id日期和mac_id机器码进行聚合分组求每一天的该机器的销量和即sum_num,

则hive sql语句为:

```hive
select day_id,mac_id,sum(day_num) sum_num from test_temp_mac_id group by day_id,mac_id order by day_id;
```

注:我们可以观察到group by可以实现同样的分组聚合功能，但sql语句不能写与分组聚合无关的字段，否则会报错，即group by 与over(partition by ......)主要区别为，带上group by的hive sql语句只能显示与分组聚合相关的字段，而带上over(partition by ......)的hive sql语句能显示所有字段.。

| day_id   | mac_id | sum_num |
| -------- | ------ | ------- |
| 20171011 | 124609 | 1       |
| 20171011 | 20130  | 22      |
| 20171011 | 12922  | 89      |
| 20171012 | 12922  | 18      |
| 20171012 | 20130  | 7       |
| 20171013 | 12922  | 1       |
| 20171013 | 20130  | 2       |