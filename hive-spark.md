# Hive用spark执行

- 与hive执行sql一样，只是用spark作为引擎，其中`spark的必调参数`和`spark执行前准备`都没有改动。
- 注意spark sql无法执行多段语句，可用hive执行，而spark sql只执行需要加速的sql。
- 敏感字用XXXXX代替。

```shell
#!/bin/bash
##********************************************************************#
##
## 日期支持运算，通过以下方式：
## ${DATA_DATE offset field formatter}，
## DATE_DATE：*固定值，为当前作业的业务时间
## offet：*必填，当前的日期偏移量，根据field判定偏移字段，取值为数值可正可负
## field：*必填，偏移的字段，取值可以为：day，month，year，minute，hour，week，second
## formatter：选填，计算偏移后的日期格式，如：yyyy-MM-dd HH:mm:ss
## 如：${DATA_DATE -1 day 'yyyy-MM-dd HH:mm'}
##********************************************************************#
vDay=${DATA_DATE}

sql="
insert overwrite table temp.dm_search_keyword_result_d partition(dt='${vDay}',pt='android')
select 
    keyword,
    result_num             ---计算搜索关键字对应的单曲搜索结果数
from 
(
    select 
        keyword,
        result_num,
        row_number() over(partition by keyword order by ren desc) as rank
    from 
    (
        select keyword, result_num, count(distinct uuid) as ren
        from 
        (
            select 
                kw as keyword,
                ivar5 as result_num,
                uuid
            from ddl.dt_list_ard_d
            where dt='${vDay}'
                    and action='search'
                    and b rlike '^搜索页-'
                    and ivar5 is not null
                    and ivar5 <>'null'
                    and ivar5 <>''
                    and ivar5 <>'-1'
                    and ivar5 <>'-2'
                    and kw is not null and kw<>''
            group by kw, ivar5, uuid
            
            union all
            select 
                kw as keyword,
                split(ivar5, ',')[1] as result_num,
                uuid
            from ddl.dt_list_ard_d
            where dt='${vDay}'
                    and action in ('search', 'click', 'trash')
                    and b rlike '^搜索综合页-'
                    and ivar5 is not null
                    and ivar5 <>'null'
                    and ivar5 <>''
                    and ivar5 <>'-1'
                    and ivar5 <>'-2'
                    and kw is not null and kw<>''
            group by kw, split(ivar5, ',')[1], uuid
        )a
        group by keyword, result_num
    )b
) b 
where rank=1
"

###### spark的必调参数 ##########
app_queue=${q};
app_name=test_spark
executor_cores=4
executor_memory=4G
initialExecutors=10
maxExecutors=100
minExecutors=10
driver_memory=2G
partitions=50
autoBroadcastJoinThreshold=10485760
memoryOverhead=2048
broadcastTimeout=2000

###### spark执行前准备（不需要修改） ##########
export JAVA_HOME=/usr/local/jdk1.7.0_51
export SPARK_HOME=/data1/app/spark-2.1.0-bin-hadoop2.7
export PATH=${JAVA_HOME}/bin:${SPARK_HOME}/bin:${PATH}
executeEnv="spark-submit   \
--master yarn \
--deploy-mode cluster  \
--executor-cores ${executor_cores} \
--executor-memory ${executor_memory} \
--driver-memory  ${driver_memory} \
--conf spark.sql.shuffle.partitions=${partitions} \
--conf spark.dynamicAllocation.initialExecutors=${initialExecutors} \
--conf spark.dynamicAllocation.minExecutors=${minExecutors} \
--conf spark.dynamicAllocation.maxExecutors=${maxExecutors} \
--conf spark.yarn.executor.memoryOverhead=${memoryOverhead} \
--conf spark.sql.broadcastTimeout=${broadcastTimeout} \
--conf spark.sql.autoBroadcastJoinThreshold=${autoBroadcastJoinThreshold} \
--conf spark.locality.wait=0 \
--conf spark.shuffle.io.maxRetries=5 \
--conf spark.shuffle.file.buffer=128k \
--conf spark.shuffle.io.retryWait=30s \
--conf spark.executor.heartbeatInterval=60s \
--conf spark.network.timeout=1200s \
--conf spark.files.fetchTimeout=600s \
--conf spark.dynamicAllocation.enabled=true \
--conf spark.shuffle.service.enabled=true \
--conf spark.dynamicAllocation.executorIdleTimeout=30s \
--conf spark.dynamicAllocation.schedulerBacklogTimeout=3s \
--conf spark.dynamicAllocation.cachedExecutorIdleTimeout=30s \
--conf spark.yarn.archive=hdfs://kgdc/data1/sparkJars \
--conf spark.yarn.dist.files=hdfs://kgdc/data1/sparkJars/hive-site.xml \
--conf spark.kryoserializer.buffer.max=256m \
--conf spark.kryoserializer.buffer=64m \
--queue ${app_queue} \
--name  ${app_name} \
--class net.XXXXX.sparkenv.ExecuteSql \
/data1/app/sparkJars/original-sparkenv.jar"
${executeEnv} "${sql}"
if [ $? -ne 0 ]; then exit 1; fi
```



```shell
#!/bin/bash
##********************************************************************#
##
## 日期支持运算，通过以下方式：
## ${DATA_DATE offset field formatter}，
## DATE_DATE：*固定值，为当前作业的业务时间
## offet：*必填，当前的日期偏移量，根据field判定偏移字段，取值为数值可正可负
## field：*必填，偏移的字段，取值可以为：day，month，year，minute，hour，week，second
## formatter：选填，计算偏移后的日期格式，如：yyyy-MM-dd HH:mm:ss
## 如：${DATA_DATE -1 day 'yyyy-MM-dd HH:mm'}
##********************************************************************#
vDay=${DATA_DATE}
vDay_1d_ago=`date -d "$vDay 1 day ago" +%Y-%m-%d`
vDay_2d_ago=`date -d "$vDay 2 day ago" +%Y-%m-%d`
vDay_3d_ago=`date -d "$vDay 3 day ago" +%Y-%m-%d`

sql_hive="
alter table temp.search_keyword_list_kw_d drop partition (dt='${vDay}');
alter table temp.search_keyword_list_kw_d drop partition (dt='${vDay_3d_ago}');
"
EXE_HIVE "$sql_hive"

sql="
insert overwrite table temp.search_keyword_list_kw_d partition(dt='${vDay}')
select 
    '${vDay}' as etldate,
    t1.kw, 
    t2.result_num as search_result_num, 
    t1.search_play_cnt,
    t1.search_play_cnt_30s,
    round(t3.search_valid_pv/t3.search_pv, 4) as search_valid
from 
(
    select 
        kw,
        sum(search_play_cnt) as search_play_cnt,
        sum(search_play_cnt_30s) as search_play_cnt_30s
    from 
    (
        select 
            'android' as plat,
            lower(kw) as kw,
            sum(play_count) as search_play_cnt,
            sum(case when (status='完整播放' or spttag in ('30-40','40-50','50-60','60-70','大于70')) then play_count else 0 end) as search_play_cnt_30s
        from 
        (
            select 
                regexp_extract(fo, '.*/搜索/([^/]+)', 1) as kw,
                spttag,
                status,
                play_count
            from dsl.restruct_dwm_list_all_play_d
            where dt between '${vDay_2d_ago}' and '${vDay}'
                and pt='android'
                and sty='音频'
                and (fo rlike '搜索/' or fo rlike '^(/)?搜索$') 
                and fo not rlike '本地音乐/'
                and regexp_extract(fo,'/搜索/[^/].*/([^/]+)$',1)='手动输入'
                and fo not rlike '/搜索(/综合)?/单曲/手动输入$'
        ) a
        group by lower(kw)
        
        union all
        select 
            'ios' as plat,
            lower(kw) as kw,
            sum(play_count) as search_play_cnt,
            sum(case when (status='完整播放' or spttag in ('30-40','40-50','50-60','60-70','大于70')) then play_count else 0 end) as search_play_cnt_30s
        from 
        (
            select 
                case when fo rlike '/搜索/综合/[^/]+/综合' or fo rlike'/搜索/单曲/[^/]+/单曲' 
                          or fo rlike '/搜索/歌曲/[^/]+/歌曲' then regexp_extract(fo, '.*/搜索/(综合|单曲|歌曲)/([^/]+)', 2)
                     else regexp_extract(fo, '.*/搜索/([^/]+)', 1) end as kw,
                spttag,
                status,
                play_count
            from dsl.restruct_dwm_list_all_play_d
            where dt between '${vDay_2d_ago}' and '${vDay}'
                and pt='ios'
                and sty='音频'
                and (fo rlike '搜索/' or fo rlike '/搜索$') 
                and fo not rlike '本地音乐/'
                and regexp_extract(fo,'/搜索/[^/].*/([^/]+)$',1)='手动输入'
                and fo not rlike '/搜索/(单曲|歌曲)/手动输入$'
        ) a
        group by lower(kw)
        
        union all
        select 
            'pc' as plat,
            lower(kw) as kw,
            sum(play_count) as search_play_cnt,
            sum(case when (status='完整播放' or spttag in ('30-40','40-50','50-60','60-70','大于70')) then play_count else 0 end) as search_play_cnt_30s
        from 
        (
            select 
                regexp_extract(fo, '/关键字搜索/([^/]+)',1) as kw,
                spttag,
                status,
                play_count
            from dsl.restruct_dwm_list_all_play_d
            where dt between '${vDay_2d_ago}' and '${vDay}'
                and pt='pc'
                and sty='音频'
                and fo rlike '^搜索结果/'
                and regexp_extract(fo,'/关键字搜索/[^/].*/([^/]+)$',1)='手动完全输入'
        ) a
        group by lower(kw)
    ) b
    group by kw
) t1
left join 
(
    select 
        keyword,
        result_num
    from dal.dm_search_keyword_result_d 
    where dt ='${vDay}' 
        and pt ='android'
) t2 on t1.kw=t2.keyword
left join 
(
    select lower(inputstring) as inputstring,
       count(1) as search_pv, sum(case when is_valid=1 then 1 else 0 end) as search_valid_pv
    from dcl.st_valid_search_d
    where dt='${vDay}'
        and pt='android'
        and inputtype in (1,2,4,7,8)
    group by lower(inputstring)
) t3 on t1.kw=t3.inputstring
order by cast(search_play_cnt_30s as int) desc
"

###### spark的必调参数 ##########
app_queue=${q};
app_name=test_spark
executor_cores=4
executor_memory=4G
initialExecutors=10
maxExecutors=100
minExecutors=10
driver_memory=2G
partitions=50
autoBroadcastJoinThreshold=10485760
memoryOverhead=2048
broadcastTimeout=2000

###### spark执行前准备（不需要修改） ##########
export JAVA_HOME=/usr/local/jdk1.7.0_51
export SPARK_HOME=/data1/app/spark-2.1.0-bin-hadoop2.7
export PATH=${JAVA_HOME}/bin:${SPARK_HOME}/bin:${PATH}
executeEnv="spark-submit   \
--master yarn \
--deploy-mode cluster  \
--executor-cores ${executor_cores} \
--executor-memory ${executor_memory} \
--driver-memory  ${driver_memory} \
--conf spark.sql.shuffle.partitions=${partitions} \
--conf spark.dynamicAllocation.initialExecutors=${initialExecutors} \
--conf spark.dynamicAllocation.minExecutors=${minExecutors} \
--conf spark.dynamicAllocation.maxExecutors=${maxExecutors} \
--conf spark.yarn.executor.memoryOverhead=${memoryOverhead} \
--conf spark.sql.broadcastTimeout=${broadcastTimeout} \
--conf spark.sql.autoBroadcastJoinThreshold=${autoBroadcastJoinThreshold} \
--conf spark.locality.wait=0 \
--conf spark.shuffle.io.maxRetries=5 \
--conf spark.shuffle.file.buffer=128k \
--conf spark.shuffle.io.retryWait=30s \
--conf spark.executor.heartbeatInterval=60s \
--conf spark.network.timeout=1200s \
--conf spark.files.fetchTimeout=600s \
--conf spark.dynamicAllocation.enabled=true \
--conf spark.shuffle.service.enabled=true \
--conf spark.dynamicAllocation.executorIdleTimeout=30s \
--conf spark.dynamicAllocation.schedulerBacklogTimeout=3s \
--conf spark.dynamicAllocation.cachedExecutorIdleTimeout=30s \
--conf spark.yarn.archive=hdfs://kgdc/data1/sparkJars \
--conf spark.yarn.dist.files=hdfs://kgdc/data1/sparkJars/hive-site.xml \
--conf spark.kryoserializer.buffer.max=256m \
--conf spark.kryoserializer.buffer=64m \
--queue ${app_queue} \
--name  ${app_name} \
--class net.XXXXX.sparkenv.ExecuteSql \
/data1/app/sparkJars/original-sparkenv.jar"
${executeEnv} "${sql}"
if [ $? -ne 0 ]; then exit 1; fi
```

