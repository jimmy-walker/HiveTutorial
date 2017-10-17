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