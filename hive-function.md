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
