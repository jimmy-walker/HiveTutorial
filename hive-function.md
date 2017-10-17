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