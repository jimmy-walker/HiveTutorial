#Hive中join操作

##原理如下图
![](picture/join.png)

##注意事项
###1.join就是innerjoin的缩写

###2.从join中select哪一个表的keywords字段的问题，如果自己写程序会发现其实inner join返回两个名字都叫keywords。但是实际上暗含"."，因此select中必须加入'.'，所以若两者完全相等，其实选哪个都一样。

###3.join中的表若有where条件，就放在on里面。
```hive
hive -e"
set mapreduce.job.queuename=root.baseDepSarchQueue;
INSERT INTO TABLE temp.jsearch_keyword_d PARTITION (dt='2017-07-20',song='renzhendexue') select time,timestamp,imei,version,channel,networktype,inputstring,inputtype,is_valid,reason,pagecount,listencount,addcount,downcount,playmvcount,collectcount,sharecount,filename,filenameindex,msec,localresult,isextend,localfilename,localindex,localreason,hint_type,click_no,correc_type,hint_key,correc_id,netresult,ivar1,ivar2 
from ddl.dt_search_ios_d as a
join (select distinct(imei) as mei from ddl.dt_search_ios_d where dt='2017-07-20' and inputstring='认真的雪' and is_valid=0 and inputtype in ('1','2','4'))b 
on (a.imei = b.mei and a.dt='2017-07-20')
;"
```

##常用代码
```hive
SELECT <select_list>
FROM TableA A
INNER JOIN TableB B
ON A.Key = B.Key

SELECT <select_list>
FROM TableA A
LEFT JOIN TableB B
ON A.Key = B.Key

SELECT <select_list>
FROM TableA A
LEFT JOIN TableB B
ON A.Key = B.Key
WHERE B.Key IS NULL

SELECT <select_list>
FROM TableA A
RIGHT JOIN TableB B
ON A.Key = B.Key

SELECT <select_list>
FROM TableA A
RIGHT JOIN TableB B
ON A.Key = B.Key
WHERE A.Key IS NULL

SELECT <select_list>
FROM TableA A
FULL OUTER JOIN TableB B
ON A.Key = B.Key

SELECT <select_list>
FROM TableA A
FULL OUTER JOIN TableB B
ON A.Key = B.Key
WHERE A.Key IS NULL
OR B.Key IS NULL
```

##References
[join原理](https://www.webdezign.co.uk/wp-content/uploads/2015/01/SQL-joins.pdf)