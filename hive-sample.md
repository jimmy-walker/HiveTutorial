# Hive抽样

Let’s say you have a Hive table with ten billion rows, but you want to efficiently randomly sample a fixed number- maybe ten thousand. 

使用此方法并非真正抽样。
```hive
select * from my_table
limit 10000;
```

而order by效率太差。会造成集中在一个reducer，使得性能不平衡
```hive
select * from my_table
order by rand()
limit 10000;
```

而sort by只是保证在各个reducer中，并非整体随机。
```hive
select * from my_table
sort by rand()
limit 10000;
```

所以应该在分配时就指定随机分布。
```hive
select * from my_table
distribute by rand()
sort by rand()
limit 10000;
```

最后可以优化的地方是，若知道要取的数量，则限制rand大小，为了保险起见，选择比0.000001更大一点，否则会导致数量不够。

```hive
select * from my_table
where rand() <= 0.0001
distribute by rand()
sort by rand()
limit 10000;
```

# Reference

[RANDOM SAMPLING IN HIVE](http://www.joefkelley.com/736/)