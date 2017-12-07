#Hive脚本编码规范

##1.脚本跑的时候，要注意不能用tab换行，而是用空格键，否则会导致代码解析错误。

##2.在做join时，要注意更换列名，不能使得多个表中有同样的列名，否则会报错
```hive
FAILED: SemanticException Column mid Found in more than One Tables/Subqueries #所以将下面代码中的mid都更改
```

```linux
hive -e"set hive.cli.print.header=true;
set mapreduce.job.queuename=root.baseDepSarchQueue;
select mid,lvt,z,a,b,ehc,fo,sap,svar1,kw,ivar5,svar2,ft,sn,sh,sty,ivar2,reason,tv1,fs,r 
from temp.jlist_keyword_ard_d as torigin inner join 
(
    select Tcnt.cntmid as allmid
    from
    (
        select mid as cntmid, count(*) as cnt from temp.jlist_keyword_ard_d 
        where dt = '2017-07-20'  
        and song = 'xihuannidengziqi'
        group by mid
        having cnt < 300
    )Tcnt inner join 
    (
        select Tkw.kwmid as searchmid
        from
        (
            select mid as kwmid from temp.jlist_keyword_ard_d 
            where dt = '2017-07-20' 
            and song = 'xihuannidengziqi' 
            and kw = '喜欢你邓紫棋'
            group by mid
        )Tkw inner join 
        (
            select mid as amid from temp.jlist_keyword_ard_d 
            where dt = '2017-07-20'  
            and song = 'xihuannidengziqi' 
            and a = '1428'
            group by mid
        )Ta 
        on Tkw.kwmid = Ta.amid
    )Tsearch 
    on Tcnt.cntmid = Tsearch.searchmid
)tall 
on (torigin.mid = tall.allmid and torigin.dt = '2017-07-20' and torigin.song = 'xihuannidengziqi');">xihuannidengziqi-2017-07-20-test.txt
```

##2.在传入字符串到hive脚本时，如果有单引号，需要注意加入双斜杠进行转义，因为双斜杠先会被转义为单斜杠，再与单引号转义为引号

```scala
val input="('don\\'t say goodbye','战神榜')"
val sql_original=s"""select 1 as label, query, dt, sum(search_count) as cnt from (
select inputstring as query, dt, count(is_valid) as search_count, 'ard' as plat 
from ddl.dt_search_ard_d 
where dt >= '$date_start' and dt <= '$date_end' 
and inputtype in ('1','2','3','4','6','7','8') 
and inputstring in $input 
group by inputstring, dt 
union all 
select keyword as query, dt, count(valid) as search_count, 'pc' as plat 
from ddl.dt_search_pc_d 
where dt >= '$date_start' and dt <= '$date_end' 
and inputtype in ('1','2','3','4','5','6','8') 
and keyword in $input 
group by keyword, dt 
union all 
select inputstring as query, dt, count(is_valid) as search_count, 'ios' as plat 
from ddl.dt_search_ios_d 
where dt >= '$date_start' and dt <= '$date_end' 
and inputtype in ('1','2','3','4','6','7','8') 
and inputstring in $input 
group by inputstring, dt
)triple_count 
group by query, dt"""
```