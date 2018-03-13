# Hive常用操作

## 常用操作
1.查看数据库
```hive
show databases;
```
2.选中数据库
```hive
use ddl;
```
3.展示所有数据库中的表
```hive
show tables;
```
4.显示某一个表的结构
```hive
describe dt_list_ard_d;
```
5.删除表
```hive
DROP TABLE IF EXISTS keywords1002;
```
6.指定把某结果输入到某个结果中
```linux
hive -e "set mapred.job.queue.name=root.common;select userid,filehash from common.st_common_clfile where dt='20151026' and db=0 limit 1000000;">shfile.txt
```
7.将数据插入到表中，overwrite会覆盖，into会追加
```hive
INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement;
INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement;
```
```linux
hive -e"
set mapreduce.job.queuename=root.baseDepSarchQueue;
set hive.support.quoted.identifiers=none;
INSERT OVERWRITE TABLE temp.jimmy_song_houlai PARTITION (dt='2017-07-13') select '(dt)?+.+' from ddl.dt_search_ard_d 
where dt='2017-07-13' 
and inputstring='后来 刘若英' 
and is_valid=0 
and inputtype in ('1','2','4')
;">insertsampletable.txt
```

## 查看该表的建立信息

```hive
show create table common.dim_qk_audio_d;
```

## 建立测试表进行测试

1.新建表

```hive
create table temp.jdual (dummy string);
```

2.加载数据

```hive
load data local inpath'/data1/baseDepSarch/tempstatic/queryhot/jdual.txt' overwrite into table temp.jdual;
```

3.测试数据

```hive
select max(dummy) from temp.jdual; #发现max会对字符串，返回b在a前，不管a后面有多少长度，猜测应该是ascii值的比较
```

## 建立正式表
1）先使用`show create table dt_search_ard_d;`然后将其中的符号改成''，去掉最后的属性值。
2）使用命令创建表
```linux
hive -e"
set mapreduce.job.queuename=root.common;
use temp;
CREATE TABLE IF NOT EXISTS jimmy_song_houlai(
  time string COMMENT '服务器时间,格式：yyyy-mm-dd hh:mm:ss', 
  stimestamp string COMMENT '时间戳,记录设备发送消息时刻', 
  imei string COMMENT '手机IMEI号', 
  version string COMMENT '客户端版本', 
  channel string COMMENT '用户渠道号', 
  networktype string COMMENT '联网状态（0:未知,1:4G,2:wifi,3:3G,4:2G)', 
  inputstring string COMMENT '用户输入的关键字', 
  inputtype string COMMENT '输入类型1手动完全输入,2自动输入下拉框,3没输入右键搜索、随机搜索', 
  is_valid string COMMENT '是否有效搜索，是=1，否=0', 
  reason string COMMENT '第一笔有效播放的原因', 
  pagecount string COMMENT '翻页次数', 
  listencount string COMMENT '试听次数', 
  addcount string COMMENT '添加到列表的歌曲数', 
  downcount string COMMENT '下载歌曲数', 
  playmvcount string COMMENT '播放MV次数', 
  collectcount string COMMENT '添加到网络收藏歌曲数', 
  sharecount string COMMENT '分享歌曲次数', 
  filename string COMMENT '第一笔有效的歌曲名称，valid为1时必填,reason=7时记录推荐位名称', 
  filenameindex string COMMENT '第一笔有效记录歌曲所在位置按照网络类型排序is_valid=1且reason不为7时必填，第n位发送数值n且从1开始算', 
  msec string COMMENT '服务器记录时间(只记录毫秒、微秒值)', 
  localresult string COMMENT '搜索出现的本地列表歌曲数', 
  isextend string COMMENT '当本地结果列表出现“查看更多”时必填，填“0”为没有点击，有点击则填展开前所显示的本地结果数', 
  localfilename string COMMENT '第一笔有效的本地歌曲名称,Is_valid=2、3、4时填本地搜索的第一笔有效歌曲名称', 
  localindex string COMMENT '第一笔本地有效记录歌曲所在的位置(按照本地类型排序)', 
  localreason string COMMENT '第一笔本地有效播放的原因（1.点击、2.添加、3.插播、4.设为铃声）is_valid=2、3、4时填1-4，is_valid=0、1时不填', 
  hint_type string COMMENT '0，1，2，3提示类型（0=无提示，1=强纠，2=提示纠错，3=标签提示）', 
  click_no string COMMENT '0，1，2是否点击提示语（0=否，1=是），当hint_type不为0时必填', 
  correc_type string COMMENT '2，4，5，6，7，8，9，11纠错类型（2=全拼，4=简拼，5=混拼，6=英文空格，7=中文纠错，8=英文纠错，9=别名纠错，11=人工强纠），当hint_type为1、2时必填', 
  hint_key string COMMENT '纠错提示关键字，即系统返回的纠错后的关键字，当hint_type为1、2时必填', 
  correc_id string COMMENT '关联同一次纠错或标签提示中是否点击提示语流水的唯一标识号码（不能与其他纠错或标签提示的流水重复）', 
  netresult int COMMENT '记录网络搜索结果条数，无结果及网络不通均记为0，其他情况记录搜索结果数', 
  ivar1 string COMMENT '二级tab名字(全部、现场、DJ、伴奏、铃声、广场舞),无二级tab时，统一发“全部”', 
  ivar2 string COMMENT '搜索结果页是否有二级tab出现，按照从左至右顺序，将名字依次发送，发送值为：全部/现场/DJ/伴奏/铃声/广场舞')
PARTITIONED BY (dt string)
ROW FORMAT DELIMITED 
  FIELDS TERMINATED BY '|' 
  LINES TERMINATED BY '\n' 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://kgdc/user/hive/warehouse/temp.db/jimmy_song_houlai'
;">createsampletable.txt
```

